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
