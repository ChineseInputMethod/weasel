### 4.3.4 [register](https://github.com/ChineseInputMethod/weasel/blob/master/doc/4.3%20WeaselSetup/4.3.4%20register/imesetup.cpp.md)

在`imesetup.cpp`文件中完成输入法的部署。
调试时，需要手动删除注册表`\HKEY_CURRENT_USER\Software\Rime`键和系统目录里的`weasel.dll`文件，以模拟第一次安装输入法场景。

#### 4.3.4.1 install()

`install`函数调用`install_ime_file`函数，拷贝输入法文件和注册输入法。

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
```

注册输入法后，将输入法的程序目录和服务程序名，写入到注册表`\HKEY_LOCAL_MACHINE\SOFTWARE\[WOW6432Node]\Rime\Weasel`键中。

```CPP
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

#### 4.3.4.2 install_ime_file()

`install_ime_file`函数调用`copy_file`函数，将输入法复制到系统目录。

```CPP
int install_ime_file(std::wstring& srcPath, const std::wstring& ext, bool hant, bool silent, ime_register_func func)
{
	WCHAR path[MAX_PATH];
	GetModuleFileNameW(GetModuleHandle(NULL), path, _countof(path));

	std::wstring srcFileName = (hant ? L"weaselt" : L"weasel");
	srcFileName += ext;
	WCHAR drive[_MAX_DRIVE];
	WCHAR dir[_MAX_DIR];
	_wsplitpath_s(path, drive, _countof(drive), dir, _countof(dir), NULL, 0, NULL, 0);
	srcPath = std::wstring(drive) + dir + srcFileName;

	GetSystemDirectoryW(path, _countof(path));
	std::wstring destPath = std::wstring(path) + L"\\weasel" + ext;

	int retval = 0;
	// 复制 .dll/.ime 到系统目录
	if (!copy_file(srcPath, destPath))
	{
		MSG_NOT_SILENT_ID_CAP(silent, destPath.c_str(), IDS_STR_INSTALL_FAILED, MB_ICONERROR | MB_OK);
		return 1;
	}
```

然后根据传入的函数指针，调用`func`指向的输入法注册函数。

```CPP
	retval += func(destPath, true, false, hant, silent);
```

如果系统是64位的，则复制64位版到系统目录，注册64位版输入法。

```CPP
	if (is_wow64())
	{
		ireplace_last(srcPath, ext, L"x64" + ext);
		PVOID OldValue = NULL;
		// PW64DW64FR fnWow64DisableWow64FsRedirection = (PW64DW64FR)GetProcAddress(GetModuleHandle(_T("kernel32.dll")), "Wow64DisableWow64FsRedirection");
		// PW64RW64FR fnWow64RevertWow64FsRedirection = (PW64RW64FR)GetProcAddress(GetModuleHandle(_T("kernel32.dll")), "Wow64RevertWow64FsRedirection");
		if (Wow64DisableWow64FsRedirection(&OldValue) == FALSE)
		{
			MSG_NOT_SILENT_BY_IDS(silent, IDS_STR_ERRCANCELFSREDIRECT, IDS_STR_INSTALL_FAILED, MB_ICONERROR | MB_OK);
			return 1;
		}
		if (!copy_file(srcPath, destPath))
		{
			MSG_NOT_SILENT_ID_CAP(silent, destPath.c_str(), IDS_STR_INSTALL_FAILED, MB_ICONERROR | MB_OK);
			return 1;
		}
		retval += func(destPath, true, true, hant, silent);
		if (Wow64RevertWow64FsRedirection(OldValue) == FALSE)
		{
			MSG_NOT_SILENT_BY_IDS(silent, IDS_STR_ERRRECOVERFSREDIRECT, IDS_STR_INSTALL_FAILED, MB_ICONERROR | MB_OK);
			return 1;
		}
	}
	return retval;
}
```