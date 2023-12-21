### 4.2.4 [regex](https://github.com/ChineseInputMethod/weasel/blob/master/doc/4.2%20customization/4.2.4%20regex/custom.md)

本小节讲解正则表达式，制作一个双拼输入法。

本小节相关文件，放在了[附录 A](https://github.com/ChineseInputMethod/weasel/tree/master/doc/appendix/hello)中。

#### 4.2.4.1 正则表达式

正则表达式用于根据某种规则从源字符串中，匹配出目标字符串。

最简单的匹配是完全匹配。
例如正则表达式a会在字符串"axabyabcz"中匹配出三个结果。
ab会在字符串"axabyabcz"中匹配出两个结果。
abc在字符串"axabyabcz"中只有一个匹配结果。

中括号[]表示可选匹配。
例如[ab]，表示匹配a或b，在字符串"axabyabcz"中，有五个匹配结果。

可选匹配和可选匹配可以组合使用。
例如[ab]\[xyz\]，表示匹配ax、ay、az、bx、by、bz这样的结果。

可选匹配一般和完全匹配组合使用。
例如[ab]x，表示匹配ax或bx。

中括号[]还用于表示范围集合。当在[]中用连字符-连接两个字符时，表示两个字符之间的任意字符。
例如[a-z]，表示从字母a到字母z的任意字母。

\`[a-z]表示匹配：\`a、\`b、\`c……这样的组合。

星号*表示重复前一个匹配零次或多次。
例如a*表示：(空)、a、aa、aaa、aaaa……这样的字符串。
[ab]*表示：(空)、a、b、aa、ab、bb、ba、aaa、aba……这样的字符串。

插入符^表示从字符串左边界开始匹配。
例如^abc在字符串"axabyabcz"中没有匹配结果。

美元符号$表示匹配到字符串的右边界。
例如abc$在字符串"axabyabcz"中没有匹配结果。

\`[a-z]*$表示：以\`a、\`ab、\`xyz……这样结尾的字符串。

```
recognizer:
  import_preset: default
  patterns:
    reverse_lookup: "`[a-z]*$"
```

在上面的代码中，标识器组件`recognizer`，将从反引号\`开始到字符串结尾的编码串标记为编码反查段`reverse_lookup`。
例如输入：\`ab，反查翻译器`reverse_lookup`将会显示仓颉编码为ab的汉字。

#### 4.2.4.2 