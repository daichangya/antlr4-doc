# Grammar Structure

语法本质上是一个语法声明,后面是规则列表,但具有以下一般形式:

```
/** Optional javadoc style comment */
grammar Name; ①
options {...}
import ... ;
 	
tokens {...}
channels {...} //lexer only
@actionName {...}
 	 
rule1 //parser and lexer rules, possibly intermingled
...
ruleN
```

包含语法`X`的文件名必须称为`X.g4`。您可以按任何顺序指定选项,导入,令牌规范和操作。选项,导入和令牌规范中最多可以有一个。所有这些元素都是可选的,除了标头â'和至少一个规则。规则采用基本形式:

```
ruleName : alternative1 | ... | alternativeN ;
```

解析器规则名称必须以小写字母开头,而词法分析器规则必须以大写字母开头。

在`grammar`标头上没有前缀定义的语法是组合语法,可以同时包含词汇规则和解析器规则。要创建仅允许解析器规则的解析器语法,请使用以下标头。

```
parser grammar Name;
...
```

而且,自然的,纯词法语法看起来像这样:

```
lexer grammar Name;
...
```

只有词法分析器语法可以包含`mode`规范。

只有词法分析器语法可以包含自定义渠道规范

```
channels {
  WHITESPACE_CHANNEL,
  COMMENTS_CHANNEL
}
```

然后可以将这些通道像词法分析器规则中的枚举一样使用:

```
WS : [ \r\t\n]+ -> channel(WHITESPACE_CHANNEL) ;
```

15.5节,[Lexer规则](http://pragprog.com/book/tpantlr2/the-definitive-antlr-4-reference)和15.3节,[解析器规则](http://pragprog.com/book/tpantlr2/the-definitive-antlr-4-reference)包含有关规则语法的详细信息。第15.8节“选项”描述了语法选项,而第15.4节“动作和属性”提供了有关语法级动作的信息。

## Grammar Imports

语法`imports`使您可以将语法分解为逻辑块和可重用块,如在[导入语法](http://pragprog.com/book/tpantlr2/the-definitive-antlr-4-reference)中所见。ANTLR对待导入的语法非常类似于面向对象的编程语言对待超类。语法从导入的语法继承所有规则,标记规范和命名操作。“主语法”中的规则 覆盖导入语法的规则以实现继承。

将`import`视为更智能的include语句(其中不包括已定义的规则)。所有导入的结果是一个单一的组合语法;ANTLR代码生成器看到了完整的语法,却不知道有导入的语法。

要处理主语法,ANTLR工具会将所有导入的语法加载到从属语法对象中。然后,它将规则,标记类型和命名操作从导入的语法合并到主语法中。在下图中,右侧的语法说明了语法`MyELang`导入语法`ELang`的效果。

<img src = images/combined.png宽度= 400>

`MyELang`继承规则`stat`,`WS`和`ID`,但覆盖规则`expr`并添加`INT`。这是一个示例构建和测试运行,显示`MyELang`可以识别整数表达式,而原始`ELang`不能识别整数表达式。第三个错误的输入语句触发一条错误消息,该错误消息还表明解析器正在寻找`MyELang`的expr而不是`ELang`的表达式。

```
$ antlr4 MyELang.g4
$ javac MyELang*.java
$ grun MyELang stat
=> 	34;
=> 	a;
=> 	;
=> 	EOF
<= 	line 3:0 extraneous input ';' expecting {INT, ID}
```

如果主语法或任何导入的语法中存在模式,则导入过程将导入这些模式并在不覆盖它们的情况下合并其规则。万一所有模式都变成空,
规则已被该模式之外的规则覆盖,该模式将被丢弃。

如果有任何`tokens`规范,则主要语法将合并标记集。如果有任何`channel`规范,则主要语法将合并通道集。任何命名的动作(例如`@members`)都将被合并。通常,应避免在导入的语法中使用命名动作和规则中的动作,因为这样做会限制重用。ANTLR还忽略导入语法中的任何选项。

导入的语法也可以导入其他语法。ANTLR以深度优先的方式学习所有导入的语法。如果两个或多个导入的语法定义了规则`r`,则ANTLR会选择它找到的`r`的第一个版本。在下图中,ANTLR按照以下顺序检查语法:`Nested`,`G1`,`G3`,`G2`。

<img src = images/nested.png宽度= 350>

`Nested`包含来自`G3`的`r`规则,因为它在`G2`中的`r`之前看到该版本。

并非每种语法都可以导入其他每种语法:

*词法分析器语法可以导入词法分析器,包括包含模式的词法分析器。
*解析器可以导入解析器。
*组合语法可以导入没有模式的解析器或词法分析器。

ANTLR在主词法语法中将导入的规则添加到规则列表的末尾。这意味着主语法中的词法分析器规则优先于导入的规则。例如,如果主语法定义了规则`IF : ’if’ ;`,而导入语法定义了规则`ID : [a-z]+ ;`(也可以识别`if`),则导入的`ID`将不会隐藏主语法。语法的`IF`令牌定义。

## Tokens Section

`tokens`部分的目的是定义没有相关词汇规则的语法所需要的标记类型。基本语法为:

```
tokens { Token1, ..., TokenN }
```

在大多数情况下,令牌部分用于定义语法中的操作所需的令牌类型,如第10.3节“识别其关键字不固定的语言”(http://pragprog.com/book/tpantlr2/权威antlr-4-参考):

```
//explicitly define keyword token types to avoid implicit definition warnings
tokens { BEGIN, END, IF, THEN, WHILE }
 
@lexer::members { //keywords map used in lexer to assign token types
Map<String,Integer> keywords = new HashMap<String,Integer>() {{
	put("begin", KeywordsParser.BEGIN);
	put("end", KeywordsParser.END);
	...
}};
}
```

`tokens`部分实际上只是定义了一组标记,以添加到整个集合中。

```
$ cat Tok.g4
grammar Tok;
tokens { A, B, C }
a : X ;
$ antlr4 Tok.g4
warning(125): Tok.g4:3:4: implicit definition of token X in parser
$ cat Tok.tokens
A=1
B=2
C=3
X=4
```

## Actions at the Grammar Level

当前,在语法规则之外仅使用了两个已定义的命名操作(用于Java目标):`header`和`members`。前者将代码注入到识别器类定义之前的生成的识别器类文件中,后者将代码作为字段和方法注入到识别器类定义中。

对于组合语法,ANTLR将动作同时注入解析器和词法分析器。要将操作限制为生成的解析器或词法分析器,请使用`@parser::name`或`@lexer::name`。

这是一个示例,其中语法为生成的代码指定了一个包:

```
grammar Count;
 
@header {
package foo;
}
 
@members {
int count = 0;
}
 
list
@after {System.out.println(count+" ints");}
: INT {count++;} (',' INT {count++;} )*
;
 
INT : [0-9]+ ;
WS : [ \r\t\n]+ -> skip ;
```

然后,语法本身应位于目录`foo`中,以便ANTLR在同一`foo`目录中生成代码(至少在不使用`-o` ANTLR工具选项时):

```
$ cd foo
$ antlr4 Count.g4 # generates code in the current directory (foo)
$ ls
Count.g4		CountLexer.java	CountParser.java
Count.tokens	CountLexer.tokens
CountBaseListener.java CountListener.java
$ javac *.java
$ cd ..
$ grun foo.Count list
=> 	9, 10, 11
=> 	EOF
<= 	3 ints
```

Java编译器希望软件包`foo`中的类位于目录`foo`中。