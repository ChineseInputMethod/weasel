### 4.3.4 [register](https://github.com/ChineseInputMethod/weasel/blob/master/doc/4.3%20WeaselSetup/4.3.4%20register/imesetup.cpp.md)

在`imesetup.cpp`文件中完成输入法的部署。
调试时，需要手动删除注册表`\HKEY_CURRENT_USER\Software\Rime`键和`weasel.dll`文件，以模拟第一次安装输入法场景。

#### 4.3.4.1 install

`install`函数调用`install_ime_file`函数，拷贝输入法文件注册输入法。
注册输入法后，将输入法的程序目录和服务程序名，写入到注册表`\HKEY_LOCAL_MACHINE\SOFTWARE\[WOW6432Node]\Rime\Weasel`键中。

```CPP
int install(bool hant, bool silent, bool old_ime_support)
{
	std::wstring ime_src_path;
	int retval = 0;
	if (old_ime_support)
	{
		retval += install_ime_file(ime_src_path, L".ime", hant, silent, &register_ime);
	}
	retval += install_ime_file(ime_src_path, L".dll", hant, silent, &register_text_service);

	// 写注册表
	HKEY hKey;
	LSTATUS ret = RegCreateKeyEx(HKEY_LOCAL_MACHINE, WEASEL_REG_KEY,
		0, NULL, 0, KEY_ALL_ACCESS, 0, &hKey, NULL);
	if (FAILED(HRESULT_FROM_WIN32(ret)))
	{
		MSG_NOT_SILENT_ID_CAP(silent, WEASEL_REG_KEY, IDS_STR_INSTALL_FAILED, MB_ICONERROR | MB_OK);
		return 1;
	}

	WCHAR drive[_MAX_DRIVE];
	WCHAR dir[_MAX_DIR];
	_wsplitpath_s(ime_src_path.c_str(), drive, _countof(drive), dir, _countof(dir), NULL, 0, NULL, 0);
	std::wstring rootDir = std::wstring(drive) + dir;
	rootDir.pop_back();
	ret = RegSetValueEx(hKey, L"WeaselRoot", 0, REG_SZ,
		(const BYTE*)rootDir.c_str(),
		(rootDir.length() + 1) * sizeof(WCHAR));
	if (FAILED(HRESULT_FROM_WIN32(ret)))
	{
		MSG_NOT_SILENT_BY_IDS(silent, IDS_STR_ERRWRITEWEASELROOT, IDS_STR_INSTALL_FAILED, MB_ICONERROR | MB_OK);
		return 1;
	}

	const std::wstring executable = L"WeaselServer.exe";
	ret = RegSetValueEx(hKey, L"ServerExecutable", 0, REG_SZ,
		(const BYTE*)executable.c_str(),
		(executable.length() + 1) * sizeof(WCHAR));
	if (FAILED(HRESULT_FROM_WIN32(ret)))
	{
		MSG_NOT_SILENT_BY_IDS(silent, IDS_STR_ERRREGIMEWRITESVREXE, IDS_STR_INSTALL_FAILED, MB_ICONERROR | MB_OK);
		return 1;
	}

	RegCloseKey(hKey);

	if (retval)
		return 1;

	MSG_NOT_SILENT_BY_IDS(silent, IDS_STR_INSTALL_SUCCESS_INFO, IDS_STR_INSTALL_SUCCESS_CAP, MB_ICONINFORMATION | MB_OK);
	return 0;
}
```