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
