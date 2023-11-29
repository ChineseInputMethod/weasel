### 4.2.2 [YAML](https://github.com/ChineseInputMethod/weasel/blob/master/doc/4.2%20customization/4.2.2%20YAML/schema.md)

本小节讲解YAML基本语法，完善“大家好”这个输入法方案。

为了节约篇幅，方便阅读，每个专题只展示了相关部分的代码，完整的代码放在了本小节最后。

#### 4.2.2.1 注释

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

#### 4.2.2.2 缩进

`yaml`文件，使用缩进表示层级关系，缩进不允许使用tab，只允许空格。缩进的空格数不重要，只要相同层级的元素左对齐即可。

```
schema:
  name: 大家好         # 將在〔方案選單〕中顯示
  schema_id: hello    # 注意此ID與文件名裏 .schema.yaml 之前的部分相同
  version: "5"        # 這是文字類型而非整數或小數，如 "1.2.3"
```

在上面的代码中，`name`、`schema_id`、`version`为同一层级，均在`schema`层级下。
`schema`被`rime`开发者称为描述档，在该层级下，描述了当前`schema.yml`文件的基本信息。

初学者常见的一个错误是，复制粘贴其他`schema.yml`文件代码，没有注意缩进对齐，导致层级错误。