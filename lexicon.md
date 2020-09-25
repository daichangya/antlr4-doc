# Grammar Lexicon

大多数程序员都熟悉ANTLR的词典,因为它遵循C及其派生词的语法,并带有一些语法描述扩展。

## Comments

有单行,多行和Javadoc样式的注释:

```
/** This grammar is an example illustrating the three kinds
 * of comments.
 */
grammar T;
/* a multi-line
  comment
*/

/** This rule matches a declarator for my language */
decl : ID ; //match a variable name
```

Javadoc注释在解析器中是隐藏的,此刻将被忽略。它们仅用于语法和任何规则的开头。

## Identifiers

令牌名称始终以大写字母开头,而Java的`Character.isUpperCase`方法所定义的词法分析器规则也是如此。解析器规则名称始终以小写字母开头(那些`Character.isUpperCase`失败的字母)。初始字符后可以跟大写和小写字母,数字和下划线。以下是一些示例名称:

```
ID, LPAREN, RIGHT_CURLY //token names/rules
expr, simpleDeclarator, d2, header_file //rule names
```

像Java一样,ANTLR在ANTLR名称中接受Unicode字符:

<img src = images/nonascii.png宽度= 100>

为了支持Unicode解析器和词法分析器规则名称,ANTLR使用以下规则:

```
ID : a=NameStartChar NameChar*
     {  
     if ( Character.isUpperCase(getText().charAt(0)) ) setType(TOKEN_REF);
     else setType(RULE_REF);
     }  
   ;
```

规则`NameChar`标识有效的标识符字符:

```
fragment
NameChar
   : NameStartChar
   | '0'..'9'
   | '_'
   | '\u00B7'
   | '\u0300'..'\u036F'
   | '\u203F'..'\u2040'
   ;
fragment
NameStartChar
   : 'A'..'Z' | 'a'..'z'
   | '\u00C0'..'\u00D6'
   | '\u00D8'..'\u00F6'
   | '\u00F8'..'\u02FF'
   | '\u0370'..'\u037D'
   | '\u037F'..'\u1FFF'
   | '\u200C'..'\u200D'
   | '\u2070'..'\u218F'
   | '\u2C00'..'\u2FEF'
   | '\u3001'..'\uD7FF'
   | '\uF900'..'\uFDCF'
   | '\uFDF0'..'\uFFFD'
   ;
```

规则`NameStartChar`是可以启动标识符(规则,标记或标签名称)的字符列表:
这些或多或少对应于Java的Character类中的`isJavaIdentifierPart`和`isJavaIdentifierStart`。如果语法文件不是UTF-8格式,请确保在ANTLR工具上使用`-encoding`选项,以便ANTLR正确读取字符。

## Literals

ANTLR不能像大多数语言一样区分字符和字符串文字。所有长度为一个或多个字符的文字字符串都用单引号引起来,例如`’;’`,`’if’`,`’>=’`和`’\’`(指的是包含单引号字符)。文字绝不包含正则表达式。

文字可以包含格式为`’\uXXXX’`(对于`’U+FFFF’`以下的Unicode代码点)或`’\u{XXXXXX}’`(对于所有Unicode代码点)的Unicode转义序列,其中`’XXXX’`是十六进制Unicode代码点值。

例如,`’\u00E8’`是带有重音的法语字母:`’è’`,而`’\u{1F4A9}’`是著名的表情符号:`’💩’`。

ANTLR还理解通常的特殊转义序列:`’\n’`(换行符),`’\r’`(回车符),`’\t’`(制表符),`’\b’`(退格键)和`’\f’`(换页)。您可以直接在文字中使用Unicode代码点,也可以使用Unicode转义序列:

```
grammar Foreign;
a : '外' ;
```

ANTLR生成的识别器假定包含所有Unicode字符的字符词汇表。运行时库假定的输入文件编码取决于目标语言。对于Java目标,运行时库假定文件位于UTF-8中。使用`CharStreams`中的工厂方法,可以指定其他编码。

## Actions

动作是用目标语言编写的代码块。您可以在语法中的多个位置使用动作,但是语法始终相同:用花括号括起来的任意文本。如果字符串或注释中的结束卷曲字符为`"}"`或`/*}*/`,则无需转义。如果小冰壶是平衡的,那么您也不需要转义}:`{...}`。否则,请使用反斜杠转出多余的小卷毛:`\{`或`\}`。操作文本应符合语言选项所指定的目标语言。

嵌入式代码可以出现在:`@header`和`@members`命名动作,解析器和词法分析器规则,异常捕获规范,解析器规则的属性部分(返回值,参数和局部变量)以及一些规则元素选项(当前谓词)。

ANTLR在内部动作中所做的唯一解释与语法属性有关。请参阅[令牌属性](http://pragprog.com/book/tpantlr2/the-definitive-antlr-4-reference)和第10章[属性和操作](http://pragprog.com/book/tpantlr2/权威antlr-4-参考)。嵌入在词法分析器规则中的动作无需任何解释或转换为生成的词法分析器。

## Keywords

这是ANTLR语法中保留字的列表:

```
import, fragment, lexer, parser, grammar, returns,
locals, throws, catch, finally, mode, options, tokens
```
  
另外,尽管它不是关键字,但请勿将单词`rule`用作规则名称。此外,请勿将目标语言的任何关键字用作标记,标签或规则名称。例如,规则`if`将产生一个名为`if`的函数。那显然不会编译。