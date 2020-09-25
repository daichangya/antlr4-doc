# Options

您可以在语法和规则元素级别指定许多选项。(当前没有规则选项。)这些更改了ANTLR从语法生成代码的方式。通用语法为:

```
options { name1=value1; ... nameN=valueN; } //ANTLR not target language syntax
```

其中值可以是标识符,合格标识符(例如abc),字符串,大括号`{...}`中的多行字符串和整数。

## Grammar Options

所有语法都可以使用以下选项。在组合语法中,除语言之外的所有选项仅与生成的解析器有关。可以使用选项语法(如上所述)在语法文件内设置选项,也可以使用`-D`选项在命令行上调用ANTLR时设置选项。(请参见第15.9节[ANTLR工具命令行选项](tool-options.md)。)请注意,`-D`会覆盖语法中的选项。

* `superClass`。设置生成的解析器或词法分析器的超类。对于组合语法,它设置了解析器的超类。
```
$ cat Hi.g4
grammar Hi;
a : 'hi' ;
$ antlr4 -DsuperClass=XX Hi.g4
$ grep 'public class' HiParser.java
public class HiParser extends XX {
$ grep 'public class' HiLexer.java
public class HiLexer extends Lexer {
```
* `language`如果ANTLR能够以指定的语言生成代码。否则,您将看到如下错误消息:
```
$ antlr4 -Dlanguage=C MyGrammar.g4
error(31):  ANTLR cannot generate C code as of version 4.0
```
* `tokenVocab` ANTLR在文件中遇到令牌时将令牌类型编号分配给令牌。要使用不同的令牌类型值,例如使用单独的词法分析器,请使用此选项使ANTLR提取<fileextension> tokens </fileextension>文件。ANTLR从每个语法生成一个<fileextension>令牌</fileextension>文件。
```
$ cat SomeLexer.g4
lexer grammar SomeLexer;
ID : [a-z]+ ;
$ cat R.g4
parser grammar R;
options {tokenVocab=SomeLexer;}
tokens {A,B,C} //normally, these would be token types 1, 2, 3
a : ID ;
$ antlr4 SomeLexer.g4
$ cat SomeLexer.tokens 
ID=1
$ antlr4 R.g4
$ cat R.tokens
A=2
B=3
C=4
ID=1
```
* `TokenLabelType` ANTLR在生成引用令牌的变量时通常使用<class> Token </class>类型。如果已将<class> TokenFactory </class>传递给解析器和词法分析器,以便它们创建自定义标记,则应将此选项设置为您的特定类型。这样可以确保上下文对象知道您的字段类型和方法返回值。
```
$ cat T2.g4
grammar T2;
options {TokenLabelType=MyToken;}
a : x=ID ;
$ antlr4 T2.g4
$ grep MyToken T2Parser.java
    public MyToken x;
```
* `contextSuperClass`。指定解析树内部节点的超类。默认值为`ParserRuleContext`。最终应至少源自`RuleContext`。
为了方便起见,Java目标可以使用`contextSuperClass=org.antlr.v4.runtime.RuleContextWithAltNum`。它为`altNumber`添加备用字段,该备用字段与关联的规则节点匹配。

## Rule Options

当前没有有效的规则级选项,但该工具仍支持以下语法以备将来使用:

```
rulename
options {...}
 	: ...
 	;
```

## Rule Element Options

令牌选项的格式为`T<name=value>`,如我们在5.4节[处理优先级,左递归和关联性]中所见(http://pragprog.com/book/tpantlr2/the-definitive-antlr-4-reference )。唯一的令牌选项是`assoc`,它接受值`left`和`right`。这是一个带有左递归表达式规则的示例语法,该规则在`^`指数运算符标记上指定了标记选项:

```
grammar ExprLR;
 	 
expr : expr '^'<assoc=right> expr
 	| expr '*' expr //match subexpressions joined with '*' operator
 	| expr '+' expr //match subexpressions joined with '+' operator
 	| INT //matches simple integer atom
 	;
 	 
INT : '0'..'9'+ ;
WS : [ \n]+ -> skip ;
```

语义谓词也接受[捕获失败的语义谓词](http://pragprog.com/book/tpantlr2/the-definitive-antlr-4-reference)的选项。唯一有效的选项是`fail`选项,该选项采用双引号引起来的字符串文字或求值字符串的操作。操作的字符串文字或字符串结果应该是在谓词失败时发出的消息。

```
ints[int max]
 	locals [int i=1]
 	: INT ( ',' {$i++;} {$i<=$max}?<fail={"exceeded max "+$max}> INT )*
 	;
```

谓词失败时,该操作可以执行函数并计算字符串:`{...}?<fail={doSomethingAndReturnAString()}>`