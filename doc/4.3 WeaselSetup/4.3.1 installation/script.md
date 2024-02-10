### 4.3.1 [NSIS](https://github.com/ChineseInputMethod/weasel/blob/master/doc/4.3%20WeaselSetup/4.3.1%20installation/script.md)

在`weasel/build.bat`文件最后，会调用NSIS将编译后的文件，打包成输入法安装程序。

```bash
if %build_installer% == 1 (
  "%ProgramFiles(x86)%"\NSIS\Bin\makensis.exe ^
  /DWEASEL_VERSION=%WEASEL_VERSION% ^
  /DWEASEL_BUILD=%WEASEL_BUILD% ^
  output\install.nsi
  if errorlevel 1 goto error
)
```

编译好的程序文件在`weasel/output`文件夹中。

输入法的数据文件在`weasel/output/data`文件夹中。

打包的输入法安装程序在`weasel/output/archives`文件夹中。

#### 4.3.1.1 Nullsoft Scriptable Install System

`install.nsi`在`weasel/output`文件夹中。详细说明请参考相关教程，这里只对关键内容进行简要说明。

脚本开始进行了一些初始设置。

```bash
; weasel installation script
!include FileFunc.nsh
!include LogicLib.nsh
!include MUI2.nsh
!include x64.nsh

Unicode true

;--------------------------------
; General

!ifndef WEASEL_VERSION
!define WEASEL_VERSION 1.0.0
!endif

!ifndef WEASEL_BUILD
!define WEASEL_BUILD 0
!endif

!define WEASEL_ROOT $INSTDIR\weasel-${WEASEL_VERSION}
!define REG_UNINST_KEY "Software\Microsoft\Windows\CurrentVersion\Uninstall\Weasel"

; The name of the installer
Name "小狼毫 ${WEASEL_VERSION}"

; The file to write
OutFile "archives\weasel-${WEASEL_VERSION}.${WEASEL_BUILD}-installer.exe"

VIProductVersion "${WEASEL_VERSION}.${WEASEL_BUILD}"
VIAddVersionKey /LANG=2052 "ProductName" "小狼毫"
VIAddVersionKey /LANG=2052 "Comments" "Powered by RIME | 中州韻輸入法引擎"
VIAddVersionKey /LANG=2052 "CompanyName" "式恕堂"
VIAddVersionKey /LANG=2052 "LegalCopyright" "Copyleft RIME Developers"
VIAddVersionKey /LANG=2052 "FileDescription" "小狼毫輸入法"
VIAddVersionKey /LANG=2052 "FileVersion" "${WEASEL_VERSION}"

!define MUI_ICON ..\resource\weasel.ico
SetCompressor /SOLID lzma

; The default installation directory
InstallDir $PROGRAMFILES\Rime

; Registry key to check for directory (so if you install again, it will
; overwrite the old one automatically)
InstallDirRegKey HKLM "Software\Rime\Weasel" "InstallDir"

; Request application privileges for Windows Vista
RequestExecutionLevel admin
```

然后设置了四个安装页面和三个卸载页面。

```bash
; Pages

!insertmacro MUI_PAGE_LICENSE "LICENSE.txt"
!insertmacro MUI_PAGE_DIRECTORY
!insertmacro MUI_PAGE_INSTFILES
!insertmacro MUI_PAGE_FINISH

!insertmacro MUI_UNPAGE_CONFIRM
!insertmacro MUI_UNPAGE_INSTFILES
!insertmacro MUI_UNPAGE_FINISH
```

接着设置了多语言开始菜单。

```bash
; Languages

!insertmacro MUI_LANGUAGE "TradChinese"
LangString DISPLAYNAME ${LANG_TRADCHINESE} "小狼毫輸入法"
LangString LNKFORMANUAL ${LANG_TRADCHINESE} "【小狼毫】說明書"
LangString LNKFORSETTING ${LANG_TRADCHINESE} "【小狼毫】輸入法設定"
LangString LNKFORDICT ${LANG_TRADCHINESE} "【小狼毫】用戶詞典管理"
LangString LNKFORSYNC ${LANG_TRADCHINESE} "【小狼毫】用戶資料同步"
LangString LNKFORDEPLOY ${LANG_TRADCHINESE} "【小狼毫】重新部署"
LangString LNKFORSERVER ${LANG_TRADCHINESE} "小狼毫算法服務"
LangString LNKFORUSERFOLDER ${LANG_TRADCHINESE} "【小狼毫】用戶文件夾"
LangString LNKFORAPPFOLDER ${LANG_TRADCHINESE} "【小狼毫】程序文件夾"
LangString LNKFORUPDATER ${LANG_TRADCHINESE} "【小狼毫】檢查新版本"
LangString LNKFORSETUP ${LANG_TRADCHINESE} "【小狼毫】安裝選項"
LangString LNKFORUNINSTALL ${LANG_TRADCHINESE} "卸載小狼毫"

!insertmacro MUI_LANGUAGE "SimpChinese"
LangString DISPLAYNAME ${LANG_SIMPCHINESE} "小狼毫输入法"
LangString LNKFORMANUAL ${LANG_SIMPCHINESE} "【小狼毫】说明书"
LangString LNKFORSETTING ${LANG_SIMPCHINESE} "【小狼毫】输入法设定"
LangString LNKFORDICT ${LANG_SIMPCHINESE} "【小狼毫】用户词典管理"
LangString LNKFORSYNC ${LANG_SIMPCHINESE} "【小狼毫】用户资料同步"
LangString LNKFORDEPLOY ${LANG_SIMPCHINESE} "【小狼毫】重新部署"
LangString LNKFORSERVER ${LANG_SIMPCHINESE} "小狼毫算法服务"
LangString LNKFORUSERFOLDER ${LANG_SIMPCHINESE} "【小狼毫】用户文件夹"
LangString LNKFORAPPFOLDER ${LANG_SIMPCHINESE} "【小狼毫】程序文件夹"
LangString LNKFORUPDATER ${LANG_SIMPCHINESE} "【小狼毫】检查新版本"
LangString LNKFORSETUP ${LANG_SIMPCHINESE} "【小狼毫】安装选项"
LangString LNKFORUNINSTALL ${LANG_SIMPCHINESE} "卸载小狼毫"

!insertmacro MUI_LANGUAGE "English"
LangString DISPLAYNAME ${LANG_ENGLISH} "Weasel"
LangString LNKFORMANUAL ${LANG_ENGLISH} "[Weasel] Manual"
LangString LNKFORSETTING ${LANG_ENGLISH} "[Weasel] Settings"
LangString LNKFORDICT ${LANG_ENGLISH} "[Weasel] Dictionary Manager"
LangString LNKFORSYNC ${LANG_ENGLISH} "[Weasel] Sync User Profile"
LangString LNKFORDEPLOY ${LANG_ENGLISH} "[Weasel] Deploy"
LangString LNKFORSERVER ${LANG_ENGLISH} "Weasel Server"
LangString LNKFORUSERFOLDER ${LANG_ENGLISH} "[Weasel] User Folder"
LangString LNKFORAPPFOLDER ${LANG_ENGLISH} "[Weasel] App Folder"
LangString LNKFORUPDATER ${LANG_ENGLISH} "[Weasel] Check for Updates"
LangString LNKFORSETUP ${LANG_ENGLISH} "[Weasel] Installation Preference"
LangString LNKFORUNINSTALL ${LANG_ENGLISH} "Uninstall Weasel"
```

#### 4.3.1.2 .onInit函数

在安装页面显示之前，会先调用.onInit函数。
在.onInit函数中，通过读取注册表项检查是否已安装过输入法。

```bash
Function .onInit
  ReadRegStr $R0 HKLM \
  "Software\Microsoft\Windows\CurrentVersion\Uninstall\Weasel" \
  "UninstallString"
  StrCmp $R0 "" done
```

如果已安装过输入法，弹出窗口，提示用户先卸载已有版本输入法。

```bash
  StrCpy $0 "Upgrade"
  IfSilent uninst 0
  MessageBox MB_OKCANCEL|MB_ICONINFORMATION \
  "安裝前，我打盤先卸載舊版本的小狼毫。$\n$\n按下「確定」移除舊版本，按下「取消」放棄本次安裝。" \
  IDOK uninst
  Abort
```

备份已有版本安装的输入方案。

```bash
uninst:
  ; Backup data directory from previous installation, user files may exist
  ReadRegStr $R1 HKLM SOFTWARE\Rime\Weasel "WeaselRoot"
  StrCmp $R1 "" call_uninstaller
  IfFileExists $R1\data\*.* 0 call_uninstaller
  CreateDirectory $TEMP\weasel-backup
  CopyFiles $R1\data\*.* $TEMP\weasel-backup
```

静默运行`uninstall.exe`。

```bash
call_uninstaller:
  ExecWait '$R0 /S'
  Sleep 800

done:
FunctionEnd
```