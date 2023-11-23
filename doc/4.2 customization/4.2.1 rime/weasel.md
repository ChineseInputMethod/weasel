### 4.2.1 [中州韵输入法引擎](https://github.com/ChineseInputMethod/weasel/blob/master/doc/4.2%20customization/4.2.1%20rime/weasel.md)

中州韵输入法引擎是跨平台的输入法引擎。小狼毫输入法是使用中州韵引擎开发的Windows系统输入法。

官方文档在`https://rime.im/docs/`。
我参考其中`https://github.com/rime/home/wiki/RimeWithSchemata#%E7%B6%9C%E5%90%88%E6%BC%94%E7%B7%B4`部分，编写了本小节。

在上一节编译成功后，会在/weasel/output/archives文件夹，生成weasel-x.x.x.x-installer.exe小狼毫安装程序。
运行安装程序，安装小狼毫输入法。

4.2.1.1 定制小狼毫

新建`hello.schema.yaml`文件，粘贴以下内容：

```
# Rime schema
# encoding: utf-8
#
# 最簡單的 Rime 輸入方案
#

schema:
  schema_id: hello    # 注意此ID與文件名裏 .schema.yaml 之前的部分相同
  name: 大家好        # 將在〔方案選單〕中顯示
  version: "1"        # 這是文字類型而非整數或小數，如 "1.2.3"
```

>注意编码格式为utf-8，缩进为两个半角空格。

在小狼毫图标上右键，打开输入法菜单，打开用户文件夹，将`hello.schema.yaml`文件，保存到用户文件夹。

![APPDATA](APPDATA.png)

点击输入法菜单->输入法设定，打开设置窗口，勾选刚刚保存的“大家好”方案。

![scheme](scheme.png)

然后点击输入法菜单->重新部署，就可以测试刚刚定制好的方案了。

>每次修改方案，都要重新部署。