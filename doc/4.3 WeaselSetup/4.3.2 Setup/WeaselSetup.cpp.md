### 4.3.2 [Setup](https://github.com/ChineseInputMethod/weasel/blob/master/doc/4.3%20WeaselSetup/4.3.2%20Setup/WeaselSetup.cpp.md)

在`install.nsi`脚本中，以`WeaselSetup.exe /i`方式调用输入法安装程序。参数`/i`为默认安装方式。

#### 4.3.2.1 _tWinMain

在_tWinMain函数中，以命令行参数`lpstrCmdLine`为参数调用Run函数。

```CPP
int WINAPI _tWinMain(HINSTANCE hInstance, HINSTANCE /*hPrevInstance*/, LPTSTR lpstrCmdLine, int /*nCmdShow*/)
{
	HRESULT hRes = ::CoInitialize(NULL);
	ATLASSERT(SUCCEEDED(hRes));

	AtlInitCommonControls(ICC_BAR_CLASSES);	// add flags to support other controls

	hRes = _Module.Init(NULL, hInstance);
	ATLASSERT(SUCCEEDED(hRes));

	LCID lcid = GetUserDefaultLCID();
	if (lcid == 2052 || lcid == 3072 || lcid == 4100) {
		LANGID langId = SetThreadUILanguage(MAKELANGID(LANG_CHINESE, SUBLANG_CHINESE_SIMPLIFIED));
		SetThreadLocale(langId);
	}
	else {
		LANGID langId = SetThreadUILanguage(MAKELANGID(LANG_CHINESE, SUBLANG_CHINESE_TRADITIONAL));
		SetThreadLocale(langId);
	}

	int nRet = Run(lpstrCmdLine);

	_Module.Term();
	::CoUninitialize();

	return nRet;
}
```

>没有等于3072的LCID，作者（在后文中以作者指称源码的编写人）可能想要标识的是马来西亚或香港的LCID。

#### 4.3.2.2 Run

当参数为`/u`时，静默调用卸载函数。当参数为`/s`时，静默安装简体版。当参数为`/t`时，静默安装繁体版。当参数为`/i`时，调用CustomInstall函数。

```CPP
static int Run(LPTSTR lpCmdLine)
{
	constexpr bool silent = true;
	constexpr bool old_ime_support = false;
	bool uninstalling = !wcscmp(L"/u", lpCmdLine);
	if (uninstalling)
		return uninstall(silent);

	bool hans = !wcscmp(L"/s", lpCmdLine);
	if (hans)
		return install(false, silent, old_ime_support);
	bool hant = !wcscmp(L"/t", lpCmdLine);
	if (hant)
		return install(true, silent, old_ime_support);
	bool installing = !wcscmp(L"/i", lpCmdLine);
	return CustomInstall(installing);
}
```

#### 4.3.2.3 CustomInstall

CustomInstall函数通过打开注册表键`RegOpenKey(HKEY_CURRENT_USER, KEY, &hKey)`，来判断是否曾经安装过输入法（因为卸载输入法时，该键没有被删除）。
如果曾经安装过输入法，那么将会执行`silent = true;`语句，设置为静默安装输入法。

```CPP
static int CustomInstall(bool installing)
{
	bool hant = false;
	bool silent = false;
	bool old_ime_support = false;
	std::wstring user_dir;

	const WCHAR KEY[] = L"Software\\Rime\\Weasel";
	HKEY hKey;
	LSTATUS ret = RegOpenKey(HKEY_CURRENT_USER, KEY, &hKey);
	if (ret == ERROR_SUCCESS)
	{
		WCHAR value[MAX_PATH];
		DWORD len = sizeof(value);
		DWORD type = 0;
		DWORD data = 0;
		ret = RegQueryValueEx(hKey, L"RimeUserDir", NULL, &type, (LPBYTE)value, &len);
		if (ret == ERROR_SUCCESS && type == REG_SZ)
		{
			user_dir = value;
		}
		len = sizeof(data);
		ret = RegQueryValueEx(hKey, L"Hant", NULL, &type, (LPBYTE)&data, &len);
		if (ret == ERROR_SUCCESS && type == REG_DWORD)
		{
			hant = (data != 0);
			if (installing)
				silent = true;
		}
		RegCloseKey(hKey);
	}
```

CustomInstall函数通过获取系统目录`weasel.dll`文件属性的方法，判断当前是否安装有输入法。

```CPP
	bool _has_installed = has_installed();
```

如果是第一次安装输入法，将会弹出安装选项对话框。如果是从开始菜单或系统托盘菜单运行`WeaselSetup.exe`，也会弹出安装选项对话框。

```CPP
	if (!silent)
	{
		InstallOptionsDialog dlg;
		dlg.installed = _has_installed;
		dlg.hant = hant;
		dlg.user_dir = user_dir;
		if (IDOK != dlg.DoModal()) {
			if (!installing)
				return 1;  // aborted by user
		}
		else {
			hant = dlg.hant;
			user_dir = dlg.user_dir;
			old_ime_support = dlg.old_ime_support;
			_has_installed = dlg.installed;
		}
	}
```

因为从安装程序运行`WeaselSetup.exe`，会先卸载已安装的输入法，所以会执行install函数。
而从开始菜单或系统托盘菜单运行`WeaselSetup.exe`，不会执行install函数。

```CPP
	if(!_has_installed)
		if (0 != install(hant, silent, old_ime_support))
			return 1;
```

将从安装选项对话框中获取的`用户目录`和`繁体版本`数据，写入注册表。

```CPP
	ret = RegCreateKeyEx(HKEY_CURRENT_USER, KEY,
		0, NULL, 0, KEY_ALL_ACCESS, 0, &hKey, NULL);
	if (FAILED(HRESULT_FROM_WIN32(ret)))
	{
		MSG_ID_CAP(KEY, IDS_STR_INSTALL_FAILED, MB_ICONERROR | MB_OK);
		return 1;
	}

	ret = RegSetValueEx(hKey, L"RimeUserDir", 0, REG_SZ,
		(const BYTE*)user_dir.c_str(),
		(user_dir.length() + 1) * sizeof(WCHAR));
	if (FAILED(HRESULT_FROM_WIN32(ret)))
	{
		MSG_BY_IDS(IDS_STR_ERR_WRITE_USER_DIR, IDS_STR_INSTALL_FAILED, MB_ICONERROR | MB_OK);
		return 1;
	}

	DWORD data = hant ? 1 : 0;
	ret = RegSetValueEx(hKey, L"Hant", 0, REG_DWORD, (const BYTE*)&data, sizeof(DWORD));
	if (FAILED(HRESULT_FROM_WIN32(ret)))
	{
		MSG_BY_IDS(IDS_STR_ERR_WRITE_HANT, IDS_STR_INSTALL_FAILED, MB_ICONERROR | MB_OK);
		return 1;
	}
```

如果是从开始菜单或系统托盘菜单运行的`WeaselSetup.exe`，重启`WeaselServer.exe`和`WeaselDeployer.exe`。

```CPP
	if (_has_installed)
	{
		std::wstring dir(install_dir());
		std::thread th([dir]() {
			ShellExecuteW(NULL, NULL, (dir + L"\\WeaselServer.exe").c_str(), L"/q", NULL, SW_SHOWNORMAL);
			Sleep(500);
			ShellExecuteW(NULL, NULL, (dir + L"\\WeaselServer.exe").c_str(), L"", NULL, SW_SHOWNORMAL);
			Sleep(500);
			ShellExecuteW(NULL, NULL, (dir + L"\\WeaselDeployer.exe").c_str(), L"/deploy", NULL, SW_SHOWNORMAL);
			});
		th.detach();
		MSG_BY_IDS(IDS_STR_MODIFY_SUCCESS_INFO, IDS_STR_MODIFY_SUCCESS_CAP, MB_ICONINFORMATION | MB_OK);
	}

	return 0;
}
```