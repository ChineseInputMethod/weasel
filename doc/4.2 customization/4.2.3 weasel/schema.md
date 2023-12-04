### 4.2.3 [weasel](https://github.com/ChineseInputMethod/weasel/blob/master/doc/4.2%20customization/4.2.3%20weasel/schema.md)

本小节讲解schema和dict文件，制作一个全拼输入法。

本小节相关文件，放在了[附录A](https://github.com/ChineseInputMethod/weasel/tree/master/doc/appendix/hello)中。

#### 4.2.3.1 schema

`*.schema.yml`文件的`schema`节点，是方案的概述内容。其中`schema_id`是区别于其他方案的唯一标识符。
`version`用于升级。

```
schema:
  name: 大家好              # 方案名称
  schema_id: hello         # 方案标识符
  author:
    - 制作人：一条不会游泳的鱼 # 作者
  dependencies:
    - bopomofo             # 依赖项
  description: 
    一个全拼输入法           # 描述信息
  version: ”6“             # 版本号
```

>版本号要用单引号或双引号，因为rime使用字符串处理版本号。

#### 4.2.3.2 switches
