### 4.2.3 [weasel](https://github.com/ChineseInputMethod/weasel/blob/master/doc/4.2%20customization/4.2.3%20weasel/schema.md)

本小节讲解schema和dict文件，制作一个全拼输入法。

本小节相关文件，放在了[附录 A](https://github.com/ChineseInputMethod/weasel/tree/master/doc/appendix/hello)中。

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
`processors`节点是其中的按键处理部分，当用户按下按键后，通过`processors`节点的配置，rime进行按键处理。

```
engine:                    # 輸入引擎設定，即掛接組件的「處方」
  processors:              # 一、這批組件處理各類按鍵消息
    - ascii_composer       # ※ 處理西文模式及中西文切換
    - recognizer           # ※ 與 matcher 搭配，處理符合特定規則的輸入碼，如網址、反查等
    - key_binder           # ※ 在特定條件下將按鍵綁定到其他按鍵，如重定義逗號、句號爲候選翻頁鍵
      import_preset: default #格式一
    - speller              # ※ 拼寫處理器，接受字符按鍵，編輯輸入碼
    - punctuator           # ※ 句讀處理器，將單個字符按鍵直接映射爲文字符號
    - selector             # ※ 選字處理器，處理數字選字鍵、上、下候選定位、換頁鍵
    - navigator            # ※ 處理輸入欄內的光標移動鍵
    - express_editor       # ※ 編輯器，處理空格、回車上屏、回退鍵等
```

上面的代码通过`key_binder`按键绑定组件，实现了逗号键和句号键翻页功能。
为了避免过多的节点镶套，上面的代码更多的写成下面的形式。

```
engine:                    # 輸入引擎設定，即掛接組件的「處方」
  processors:              # 一、這批組件處理各類按鍵消息
    - ascii_composer       # ※ 處理西文模式及中西文切換
    - recognizer           # ※ 與 matcher 搭配，處理符合特定規則的輸入碼，如網址、反查等
    - key_binder           # ※ 在特定條件下將按鍵綁定到其他按鍵，如重定義逗號、句號爲候選翻頁鍵
    - speller              # ※ 拼寫處理器，接受字符按鍵，編輯輸入碼
    - punctuator           # ※ 句讀處理器，將單個字符按鍵直接映射爲文字符號
    - selector             # ※ 選字處理器，處理數字選字鍵、上、下候選定位、換頁鍵
    - navigator            # ※ 處理輸入欄內的光標移動鍵
    - express_editor       # ※ 編輯器，處理空格、回車上屏、回退鍵等

key_binder:                #格式二
  import_preset: default
```

两种格式在功能上是完全等价的，之所以建议写成后一种格式，是因为在实际应用中详细配置往往非常复杂，并非样例这样只有一条语句。

#### 4.2.3.4 segmentors

当用户按下若干按键后，由`segmentors`节点进行编码串的切分。
例如用户输入`hello@rime`，编码串会被切分成三部分：`hello`、`@`、`rime`，既`字母段`、`符号段`、`字母段`，标识为`abc`、`punct`、`abc`。

```
engine:                    # 輸入引擎設定，即掛接組件的「處方」
  segmentors:              # 二、這批組件識別不同內容類型，將輸入碼分段
    - ascii_segmentor      # ※ 標識西文段落
    - matcher              # ※ 標識符合特定規則的段落，如網址、反查等
    - abc_segmentor        # ※ 標識常規的文字段落
    - punct_segmentor      # ※ 標識句讀段落
    - fallback_segmentor   # ※ 標識其他未標識段落
```

以下代码为`abc`标签附加了`reverse_lookup`标签，即字母段也被切分成`reverse_lookup`编码反查段。

```
abc_segmentor:
  extra_tags:
    - reverse_lookup  #编码反查
```

然后添加`reverse_lookup_translator`组件。

```
engine:
  translators:
    - reverse_lookup_translator #反查翻譯器，用另一種編碼方案查碼
```

最后配置`reverse_lookup`组件。

```
reverse_lookup:            #配置编码查询组件
  dictionary: stroke       #笔画输入法码表
  enable_completion: true  #提前顯示尚未輸入完整碼的字
  prefix: "`"              #前缀
  suffix: "'"              #分隔符
  tips: 〔筆畫〕
  preedit_format:
    - xlit/hspnz/一丨丿丶乙/
  comment_format:
    - xform/([nl])v/$1ü/
```

完成以上设置后，就可以按下`·`键，使用`hspnz`进行笔画输入，也可以直接进行笔画输入。

>如果以前未加载过“五笔画”方案，需在输入法设定中选定“五笔画”方案，以生成所需文件。

#### 4.2.3.5 translators

在`processors`节点对编码串进行切分后，由`translators`节点将切分段转换成对应的输出字符。
例如`punct`切分段的编码会被`punct_translator`组件翻译成标点符号或其他字符。
`abc`切分段的编码会被`table_translator`或`script_translator`组件翻译成汉字。

```
engine:                    # 輸入引擎設定，即掛接組件的「處方」
  translators:             # 三、這批組件翻譯特定類型的編碼段爲一組候選文字
    - echo_translator      # ※ 沒有其他候選字時，回顯輸入碼
    - punct_translator     # ※ 轉換標點符號
    - script_translator    # ※ 腳本翻譯器，用於拼音等基於音節表的輸入方案
    - reverse_lookup_translator  # ※ 反查翻譯器，用另一種編碼方案查碼
```

`table_translator`组件是大多数形码输入法的码表翻译器，`script_translator`组件是拼音输入法的翻译器。

在rime引擎中，一个输入法只有一个主翻译器，可以有多个副翻译器。
在`table_translator`或`script_translator`后+`@`+`翻译器名`表示副翻译器。

通常一个翻译器对应一种编码方案，多个翻译器可以实现多种方案混合输入。

例如，在前面我们通过`segmentors`节点实现的编码查询，现在删除下面这段代码。

```
abc_segmentor:
  extra_tags:
    - reverse_lookup  #编码反查
```

不再使用`reverse_lookup_translator`反查翻译器实现编码查询，删除`translators`节点中的`reverse_lookup_translator`反查翻译器。

```
engine:
  translators:
    - reverse_lookup_translator #反查翻譯器，用另一種編碼方案查碼
```

改用添加副翻译器的形式，实现混合笔画输入，添加下面代码。

```
engine:
  translators:
    - table_translator@reverse_lookup # ※  碼表翻譯器
```

rime引擎会同时在两个翻译器中查找编码所对应的汉字，从而实现混合输入。

>使用反查翻译器和词典翻译器实现混合输入的区别是：反查翻译器会显示主翻译器的编码，也就是编码查询功能。

#### 4.2.3.6 filters

当完成从编码到汉字的转换后，rime引擎还完成一些收尾工作。

```
engine:                    # 輸入引擎設定，即掛接組件的「處方」
  filters:                 # 四、這批組件過濾翻譯的結果
    - simplifier           # ※ 繁簡轉換
    - uniquifier           # ※ 過濾重複的候選字，有可能來自繁簡轉換
```

以上代码实现了简繁转换功能。

#### 4.2.3.7 dict