# Lexer Rules

词法分析器语法由词法分析器规则组成,可以选择将其分解为多种模式。词法模式使我们可以将单个词法分析器语法拆分为多个子词法分析器。词法分析器只能返回当前模式下与规则匹配的标记。

Lexer规则指定标记定义,并且或多或少遵循解析器规则的语法,除了lexer规则不能具有参数,返回值或局部变量。Lexer规则名称必须以大写字母开头,以将其与解析器规则名称区分开:

```
/** Optional document comment */
TokenName : alternative1 | ... | alternativeN ;
```

您也可以定义不是令牌的规则,而是有助于识别令牌的规则。这些片段规则不会导致令牌对解析器可见:

```
fragment
HelperTokenRule : alternative1 | ... | alternativeN ;
```

例如,`DIGIT`是一个非常常见的片段规则:

```
INT : DIGIT+ ; //references the DIGIT helper rule
fragment DIGIT : [0-9] ; //not a token by itself
```

## Lexical Modes

模式允许您按上下文(例如XML标记的内部和外部)对词汇规则进行分组。就像有多个子目录,一个用于上下文。该词法分析器只能通过在当前模式下输入规则来返回匹配的令牌。词法分析器以所谓的默认模式开始。除非您指定模式命令,否则所有规则均被视为在默认模式内。组合语法中不允许使用模式,仅词法分析器语法中不允许使用模式。(请参阅[标记XML]中的语法`XMLLexer`(http://pragprog.com/book/tpantlr2/the-definitive-antlr-4-reference)。)

```
rules in default mode
...
mode MODE1;
rules in MODE1
...
mode MODEN;
rules in MODEN
...
```

## Lexer Rule Elements

Lexer规则允许解析器规则不使用两种构造:..范围运算符和方括号[characters]中的字符集表示法。不要将字符集与解析器规则的参数混淆。[characters]仅表示词法分析器中的字符集。以下是所有词法分析器规则元素的摘要:

<表格>
<tr>
<th>语法</th> <th>说明</th>
</tr>
<tr>
<td> T </td> <td>
在当前输入位置匹配令牌T。令牌始终以大写字母开头。</td>
</tr>

<tr>
<td>â€™literalâ€™</td> <td>
匹配该字符或字符序列。例如,“ while”或“â€=”。</td>
</tr>

<tr>
<td> [字符集] </td> <td>
<p>匹配字符集中指定的字符之一。将<tt> xy </tt>解释为介于<tt> x </tt>和<tt> y </tt>之间的一组字符。以下转义字符被解释为单个特殊字符:<tt> \ n </tt>,<tt> \ r </tt>,<tt> \ b </tt>,<tt> \ t </tt> ,<tt> \ f </tt>,<tt> \ uXXXX </tt>和<tt> \ u {XXXXXX} </tt>。要获取<tt>] </tt>,<tt> \ </tt>或<tt>-</tt>,您必须使用<tt> \ </tt>对其进行转义。</p>

<p>您还可以在<tt> \ p {PropertyName} </tt>或<tt> \ p {EnumProperty = Value} <中包含所有与Unicode属性匹配的字符(一般类别,布尔值或枚举的内容,包括脚本和块)</tt>。(您可以使用<tt> \ P {PropertyName} </tt>或<tt> \ P {EnumProperty = Value} </tt>反转测试。)</p>

<p>有关有效Unicode属性名称的列表,请参见<a href="http://unicode.org/reports/tr44/#Properties"> Unicode标准附件#44 </a>。(ANTLR还支持<a href="http://unicode.org/reports/tr44/#General_Category_Values">短和长的Unicode通用类别名称和值</a>,例如<tt> \ p {Lu} </tt >,<tt> \ p {Z} </tt>,<tt> \ p {Symbol} </tt>,<tt> \ p {Blk = Latin_1_Sup} </tt>和<tt> \ p { Block = Latin_1_Supplement} </tt>。)</p>

<p>作为<tt> \ p {Block = Latin_1_Supplement} </tt>的快捷方式,您可以使用<a href ="http://www.unicode.org/Public/UCD/latest/ucd /Blocks.txt">Unicode块名称</a>,其前缀为<tt> In </tt>,且空格更改为<tt> _ </tt>。例如:<tt> \ p {InLatin_1_Supplement} </tt>,<tt> \ p {InYijing_Hexagram_Symbols} </tt>和<tt> \ p {InAncient_Greek_Numbers} </tt>。</p>。

<p>支持一些其他属性:</p>
<ul>
<li> <tt> \ p {Extended_Pictographic} </tt>(请参见<a href="http://unicode.org/reports/tr35/"> UTS#35 </a>)</li>
<li> <tt> \ p {EmojiPresentation = EmojiDefault} </tt>(默认情况下,代码点具有彩色表情符号样式的演示文稿,但也可以以文本样式显示)</li>
<li> <tt> \ p {EmojiPresentation = TextDefault} </tt>(默认情况下,代码点具有黑白文本样式的表示,但也可以以emoji样式显示)。</li>
<li> <tt> \ p {EmojiPresentation = Text} </tt>(仅具有黑白文本样式且缺少彩色表情符号样式表示的代码点)</li>
</ul>

<p>属性名称不区分大小写</b>,并且<tt> _ </tt>和<tt>-</tt>的对待方式相同</p>

<p>以下是一些示例:</p>

<pre>
WS:[\ n \ u000D]->跳过;//与[\ n \ r]相同

UNICODE_WS:[\ p {White_Space}]->跳过;//匹配所有Unicode空格

ID:[a-zA-Z] [a-zA-Z0-9] *;//匹配通常的标识符规范

UNICODE_ID:[\ p {Alpha} \ p {General_Category = Other_Letter}] [\ p {Alnum} \ p {General_Category = Other_Letter}] *;//匹配完整的Unicode字母ID

表情符号:[\ u {1F4A9} \ u {1F926}];//注意Unicode代码点> U + FFFF

DASHBRACK:[\-\]] +;//匹配-或]一次或多次
</pre>
</td>
</tr>

<tr>
<td>“ x” ..“ y” </td> <td>
匹配范围x和y之间的任何单个字符(包括两端)。例如,“ a” ..“ z”。“ a” ..“ z”与[az]相同。</td>
</tr>

<tr>
<td> T </td> <td>
调用词法分析器规则T; 通常允许递归,但不允许左递归。T可以是常规标记或片段规则。
  
<pre>
ID:字母(LETTER|'0'..'9')*;
  
分段
字母:[a-zA-Z \ u0080- \ u00FF_];
</pre>
</td>
</tr>

<tr>
<td>。</td> <td>
点是与任何单个字符匹配的单字符通配符。例:
<pre>
退出 : '\\' 。; //匹配任何转义的\ x字符
</pre>
</td>
</tr>

<tr>
<td> {Â«actionÂ»} </td> <td>
从4.2开始,Lexer动作可以出现在任何位置,而不仅仅是最外层选项的结尾。词法分析器根据规则中动作的位置在适当的输入位置执行动作。要对具有多个替代选项的角色执行单个操作,可以将alt括在括号中,然后将该操作放在后面:
  
<pre>
END :('endif'|'end'){System.out.println(“找到终点”);};
</pre>

<p>该操作符合目标语言的语法。ANTLR将动作的内容逐字复制到生成的代码中;解析器操作中没有像$ xy这样的表达式的翻译。</p>
<p>
仅执行最外部令牌规则内的操作。换句话说,如果STRING调用ESC_CHAR并且ESC_CHAR有一个动作,则当词法分析器在STRING中开始匹配时,将不执行该动作。</p> </td>
</tr>

<tr>
<td> {Â«pÂ»}?</td> <td>
评估语义谓词“ p”。如果在运行时«p»的值为假,则周围的规则将变为“不可见”。(不可行)。表达式“ p”符合目标语言的语法。尽管语义谓词可以出现在词法分析器规则内的任何位置,但将其置于规则末尾是最有效的。一个警告是语义谓词必须在词法分析器动作之前。请参阅Lexer规则中的谓词。</td>
</tr>

<tr>
<td>〜x </td> <td>
匹配x所描述的集合以外的任何单个字符。集合x可以是单个字符文字,范围或子规则集,例如〜(“ x” x|“ y” |“ z”)或〜[xyz]。这是一个使用〜匹配除使用〜[\ r \ n] *的字符以外的任何字符的规则:
<pre>  
评论:'#'〜[\ r \ n] *'\ r'?'\ n'->跳过;
</pre>
</td>
</tr>
</table>

与解析器规则一样,词法分析器规则允许在括号和EBNF运算符中包含子规则:`?`,`*`,`+`。`COMMENT`规则说明了`*`和`?`运算符。`+`的常见用法是`[0-9]+`匹配整数。Lexer子规则还可以在这些EBNF运算符上使用非贪婪的`?`后缀。

## Recursive Lexer Rules

与大多数词法语法工具不同,ANTLR词法器规则可以是递归的。当您要匹配嵌套标记(如嵌套操作块)时,这非常方便:`{...{...}...}`。

```
lexer grammar Recur;
 
ACTION : '{' ( ACTION | ~[{}] )* '}' ;
 
WS : [ \r\t\n]+ -> skip ;
```

## Redundant String Literals

请注意,不要在多个词法分析器规则的右侧指定相同的字符串文字。这样的文字是不明确的,可以匹配多种令牌类型。ANTLR使此文字对解析器不可用。跨模式的规则也是如此。例如,以下词法分析器语法定义了两个具有相同字符序列的标记:

```
lexer grammar L;
AND : '&' ;
mode STR;
MASK : '&' ;
```

解析器语法无法引用文字“&”,但可以引用标记的名称:

```
parser grammar P;
options { tokenVocab=L; }
a : '&' //results in a tool error: no such token
    AND //no problem
    MASK //no problem
  ;
```

这是一个构建和测试序列:

```bash
$ antlr4 L.g4 # yields L.tokens file needed by tokenVocab option in P.g4
$ antlr4 P.g4
error(126): P.g4:3:4: cannot create implicit token for string literal '&' in non-combined grammar
```

## Lexer Rule Actions

ANTLR词法分析器在匹配词法规则后创建Token对象。每个令牌请求均从`Lexer.nextToken`开始,一旦识别出令牌,该请求便调用`emit`。`emit`从词法分析器的当前状态收集信息以构建令牌。它访问字段`_type`,`_text`,`_channel`,`_tokenStartCharIndex`,`_tokenStartLine`和`_tokenStartCharPositionInLine`。您可以使用各种设置方法(例如`setType`)来设置这些状态。例如,如果`enumIsKeyword`为false,则以下规则将`enum`转换为标识符。

```
ENUM : 'enum' {if (!enumIsKeyword) setType(Identifier);} ;
```

ANTLR在词法分析器操作中没有特殊的`$x`属性转换(与v3不同)。

词汇规则最多只能有一个动作,而不管该规则有多少种选择。

## Lexer Commands

为了避免将语法与特定的目标语言联系在一起,ANTLR支持词法分析器命令。与任意嵌入式操作不同,这些命令遵循特定的语法,并且仅限于一些常用命令。Lexer命令出现在lexer规则定义的最外层选项的末尾。像任意操作一样,每个令牌规则只能有一个。词法分析器命令由`->`运算符组成,后跟一个或多个可以选择采用参数的命令名称:

```
TokenName : «alternative» -> command-name
TokenName : «alternative» -> command-name («identifier or integer»)
```

一个替代方案可以有多个命令,以逗号分隔。这是有效的命令名称:

*跳过
* 更多
* popMode
*模式(x)
* pushMode(x)
*类型(x)
*频道(x)

有关用法,请参见本书的源代码,此处显示了一些示例:

###跳过

一个“跳过”命令告诉词法分析器获取另一个标记并丢弃当前文本。

```
ID : [a-zA-Z]+ ; //match identifiers
INT : [0-9]+ ; //match integers
NEWLINE:'\r'? '\n' ; //return newlines to parser (is end-statement signal)
WS : [ \t]+ -> skip ; //toss out whitespace
```

### mode(),pushMode(),popMode等

模式命令更改模式堆栈,从而更改词法分析器的模式。'more'命令强制词法分析器获取另一个令牌,但不丢弃当前文本。令牌类型将是匹配的“最终”规则的令牌类型(即没有更多或跳过命令的令牌)。

```
//Default "mode": Everything OUTSIDE of a tag
COMMENT : '<!--' .*? '-->' ;
CDATA   : '<![CDATA[' .*? ']]>' ;OPEN : '<' -> pushMode(INSIDE) ;
 ...
XMLDeclOpen : '<?xml' S -> pushMode(INSIDE) ;
SPECIAL_OPEN: '<?' Name -> more, pushMode(PROC_INSTR) ;
//----------------- Everything INSIDE of a tag ---------------------
mode INSIDE;
CLOSE        : '>' -> popMode ;
SPECIAL_CLOSE: '?>' -> popMode ; //close <?xml...?>
SLASH_CLOSE  : '/>' -> popMode ;
```

还要签出:

```
lexer grammar Strings;
LQUOTE : '"' -> more, mode(STR) ;
WS : [ \r\t\n]+ -> skip ;
mode STR;
STRING : '"' -> mode(DEFAULT_MODE) ; //token we want parser to see
TEXT : . -> more ; //collect more text for string
```

弹出模式堆栈的底层将导致异常。使用`mode`切换模式会更改当前堆栈顶部。一个以上的`more`与一个相同,并且位置无关紧要。

### type()

```
lexer grammar SetType;
tokens { STRING }
DOUBLE : '"' .*? '"'   -> type(STRING) ;
SINGLE : '\'' .*? '\'' -> type(STRING) ;
WS     : [ \r\t\n]+    -> skip ;
```

对于多个“ type()”命令,仅最右边的一个起作用。

### channel()

```
BLOCK_COMMENT
	: '/*' .*? '*/' -> channel(HIDDEN)
	;
LINE_COMMENT
	: '//' ~[\r\n]* -> channel(HIDDEN)
	;
... 
//----------
//Whitespace
//
//Characters and character constructs that are of no import
//to the parser and are used to make the grammar easier to read
//for humans.
//
WS : [ \t\r\n\f]+ -> channel(HIDDEN) ;
```

从4.5开始,您还可以使用lexer规则上方的以下结构来定义通道名称(如枚举):

```
channels { WSCHANNEL, MYHIDDEN }
```