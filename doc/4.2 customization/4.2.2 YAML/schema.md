### 4.2.2 [YAML](https://github.com/ChineseInputMethod/weasel/blob/master/doc/4.2%20customization/4.2.2%20YAML/schema.md)

本小节讲解YAML基本语法，完善“大家好”这个输入法方案。

为了节约篇幅，方便阅读，每个专题只展示了相关部分的代码，完整的代码放在了本小节最后。

4.2.2.1 注释

`yaml`文件，使用#号注释内容。从#号处开始至本行结束，均为注释内容。
`yaml`只支持单行注释。

```
# Rime schema
# encoding: utf-8
#
# 最簡單的 Rime 輸入方案
#
```

上面代码提醒使用者，rime引擎使用utf-8格式的文件。