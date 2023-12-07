### 4.2.3 [weasel](https://github.com/ChineseInputMethod/weasel/blob/master/doc/4.2%20customization/4.2.3%20weasel/schema.md)

本小节讲解schema和dict文件，制作一个全拼输入法。

本小节相关文件，放在了[附录A](https://github.com/ChineseInputMethod/weasel/tree/master/doc/appendix/hello)中。

#### 4.2.3.1 schema

`*.schema.yml`文件的`schema`节点，是方案的概述内容。其中`schema_id`是区别于其他方案的唯一标识符。
`version`用于升级。

```
schema:
  name: 大家好			# 方案名称
  schema_id: hello		# 方案标识符
  author:
    - 制作人：一条不会游泳的鱼	# 作者
  dependencies:
    - bopomofo			# 依赖项
  description: 
    一个全拼输入法		# 描述信息
  version: ”6“			# 版本号
```

>版本号要用单引号或双引号，因为rime使用字符串处理版本号。

#### 4.2.3.2 switches

`switches`节点，定义了一些开关设置，实现了状态条功能。其中`reset: 0`，表示默认选定第一个选项。`reset: 1`表示默认选定第二个选项。

```
switches:
  - name: ascii_mode        #中英文轉換開關
    reset: 0
    states: ["中文", "西文"]
  - name: full_shape        #全角符號／半角符號開關
    states: ["半角", "全角"]
  - name: extended_charset  #字符集開關
    states: ["通用", "增廣"]
  - name: simplification    #轉化字開關
    reset: 1
    states: ["漢字", "汉字"]
  - name: ascii_punct       #中西文標點轉換開關
    states: ["句讀", "符號"]
```

>未显式设置`reset`开关，下次打开输入法，会使用上次设置的状态值。

#### 4.2.3.3 processors

`engine`节点是rime引擎的核心部分，包含了四个二级节点。
`processors`节点是其中的按键处理部分，当用户按下编码键后，通过`processors`节点的配置，rime进行按键处理。

```
engine:
  processors:
    - ascii_composer #處理西文模式
    - recognizer     #與matcher搭配
    - key_binder     #將按鍵綁定到其他按鍵
      import_preset: default #格式一
    - speller        #拼寫處理器
    - punctuator     #句讀處理器
    - selector       #選字處理器
    - navigator      #處理輸入欄內的光標移動
    - express_editor #處理空格、回車上屏
```

上面的代码通过`key_binder`按键绑定组件，实现了逗号键和句号键翻页功能。
为了避免过多的节点镶套，上面的代码更多的写成下面的形式。

```
engine:
  processors:
    - ascii_composer #處理西文模式
    - recognizer     #與matcher搭配
    - key_binder     #將按鍵綁定到其他按鍵
    - speller        #拼寫處理器
    - punctuator     #句讀處理器
    - selector       #選字處理器
    - navigator      #處理輸入欄內的光標移動
    - express_editor #處理空格、回車上屏

key_binder:          #`key_binder`组件的详细配置
  import_preset: default
```

两种格式在功能上是完全等价的，之所以建议写成后一种格式，是因为在实际应用中详细配置往往非常复杂，并非样例这样只有一条语句。
