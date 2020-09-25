# Wildcard Operator and Nongreedy Subrules

诸如`(...)?`,`(...)*`和`(...)+`这样的EBNF子目录是贪婪的,它们消耗尽可能多的输入,但是有时并不是必需的。像`.*`这样的构造会消耗到词法分析器(有时甚至是解析器)中的输入结束为止。我们希望该循环是非贪婪的,因此我们需要使用不同的语法:`.*?`从正则表达式表示法借来的。我们可以通过添加另一个`?`后缀使具有`?`,`*`或`+`后缀非贪婪的任何子规则。这样的非贪婪子规则在解析器和词法分析器中都被允许,但是在词法分析器中使用频率更高。

## Nongreedy Lexer Subrules

这是非常常见的C样式注释词法分析器规则,该规则将消耗所有字符,直到看到结尾的`*/`:

```
COMMENT : '/*' .*? '*/' -> skip ; //.*? matches anything until the first */
```

这是另一个示例,该示例匹配允许将\“作为转义引号字符的字符串:

```
grammar Nongreedy;
s : STRING+ ;
STRING : '"' ( '\\"' | . )*? '"' ; //match "foo", "\"", "x\"\"y", ...
WS : [ \r\t\n]+ -> skip ;
```

```bash
$ antlr4 Nongreedy.g4
$ javac Nongreedy*.java
$ grun Nongreedy s -tokens
=> 	"quote:\""
=> 	EOF
<= 	[@0,0:9='"quote:\""',<1>,1:0]
 	[@1,11:10='<EOF>',<-1>,2:0]
```

应当谨慎使用非贪婪的子规则,因为它们会使识别问题复杂化,有时使解密词法分析器如何匹配文本变得棘手。词法分析器如何选择令牌规则:

<ol>
<li>主要目标是匹配可识别最多输入字符的词法分析器规则。

```
INT : [0-9]+ ;
DOT : '.' ; //match period
FLOAT : [0-9]+ '.' ; //match FLOAT upon '34.' not INT then DOT
```
</li>
<li>
如果有多个词法分析器规则与相同的输入序列匹配,则优先级优先于在语法文件中最先出现的规则。

```
DOC : '/**' .*? '*/' ; //both rules match /** foo */, resolve to DOC
CMT : '/*' .*? '*/' ;
```
</li>
<li>
非贪婪子规则匹配的字符数最少,但仍允许周围的词汇规则匹配。

```
/** Match anything except \n inside of double angle brackets */
STRING : '<<' ~'\n'*? '>>' ; //Input '<<foo>>>>' matches STRING then END
END    : '>>' ;
```
</li>
<li>
<p>在通过词汇规则内的非贪婪子规则后,从那时起的所有决策都是“首场比赛获胜”。
</p>
<p>
例如,规则右侧的文字“ ab”(语法片段)“。*”?(“ a” |“ ab”)是无效代码,无法匹配。如果输入为ab,则第一个替代项“ a”与第一个字符匹配,因此成功。规则右侧的(a)| ab本身正确匹配输入ab的第二个替代项。这个怪癖源于一个非贪婪的设计决策,该决策太复杂了,无法在此处进行介绍。</p>
<li>
</ol>

为了说明在词法分析器规则中使用循环的不同方式,请考虑以下语法,该语法具有三个不同的类似于动作的标记(使用不同的定界符,以便它们都适合一个示例语法)。

```
ACTION1 : '{' ( STRING | . )*? '}' ; //Allows {"foo}
ACTION2 : '[' ( STRING | ~'"' )*? ']' ; //Doesn't allow ["foo]; nongreedy *?
ACTION3 : '<' ( STRING | ~[">] )* '>' ; //Doesn't allow <"foo>; greedy *
STRING : '"' ( '\\"' | . )*? '"' ;
```

规则`ACTION1`允许使用未终止的字符串,例如`"foo`,因为输入`"foo`与循环的通配符部分匹配。它不必进入规则`STRING`即可匹配报价。为了解决这个问题,规则`ACTION2`使用`~’"’`匹配除引号之外的任何字符。表达式`~’"’`与结束规则的`’]’`仍然不明确,但子规则是非贪婪的事实意味着词法分析器将在右方括号处退出循环。为了避免出现非贪婪的子规则,请使替代项明确。表达式`~[">]`匹配除引号和直角括号外的所有内容。这是一个示例运行:

```bash
$ antlr4 Actions.g4
$ javac Actions*.java
$ grun Actions tokens -tokens
=> 	{"foo}
=> 	EOF
<= 	[@0,0:5='{"foo}',<1>,1:0]
 	[@1,7:6='<EOF>',<-1>,2:0]
=> 	$ grun Actions tokens -tokens
=> 	["foo]
=> 	EOF
<= 	line 1:0 token recognition error at: '["foo]
 	'
 	[@0,7:6='<EOF>',<-1>,2:0]
=> 	$ grun Actions tokens -tokens
=> 	<"foo>
=> 	EOF
<= 	line 1:0 token recognition error at: '<"foo>
 	'
 	[@0,7:6='<EOF>',<-1>,2:0]
```

## Nongreedy Parser Subrules

非贪婪子规则和通配符在解析器中也非常有用,可以进行“模糊解析”,其目的是从输入文件中提取信息而不必指定完整的语法。与非贪婪的词法分析器决策相反,解析器始终做出全局正确的决策。解析器永远不会做出最终导致有效输入在解析过程中失败的决定。这是中心思想:非贪婪的解析器子规则匹配最短的标记序列,从而保留对有效输入语句的成功解析。

例如,以下关键规则演示了如何从任意Java文件中提取整数常量:

```
grammar FuzzyJava;
 
/** Match anything in between constant rule matches */
file : .*? (constant .*?)+ ;
 
/** Faster alternate version (Gets an ANTLR tool warning about
 * a subrule like .* in parser that you can ignore.)
 */
altfile : (constant | .)* ; //match a constant or any token, 0-or-more times

/** Match things like "public static final SIZE" followed by anything */
constant
    :   'public' 'static' 'final' 'int' Identifier
        {System.out.println("constant: "+$Identifier.text);}
    ;
 
Identifier : [a-zA-Z_$] [a-zA-Z_$0-9]* ; //simplified
```

该语法包含来自真实Java词法分析器的极大简化的词法分析器规则集;整个文件大约60行。识别器仍然需要处理字符串和字符常量以及注释,因此它不会变得不同步,例如尝试匹配字符串内部的常量。唯一不寻常的词法分析器规则执行“匹配与另一个词法分析器规则不匹配的任何字符”?功能:

```
OTHER : . -> skip ;
```

这个包罗万象的词法分析器规则和解析器中的`.*?`子规则是模糊解析的关键要素。

这是一个示例文件,我们可以将其运行到模糊解析器中:

```java
import java.util.*;
public class C {
    public static final int A = 1;
    public static final int B = 1;
    public void foo() { }
    public static final int C = 1;
}
```

这是构建和测试序列:

```bash
$ antlr4 FuzzyJava.g4
$ javac FuzzyJava*.java
$ grun FuzzyJava file C.java
constant: A
constant: B
constant: C
```

请注意,除了`public static final int`声明外,它完全忽略所有内容。所有这一切仅用两个解析器规则进行。

现在,让我们尝试匹配一些简单的类定义,而不必为内部的垃圾构建解析器规则。这里只想抓`A`和`B`:

```
class A {
        String name = "parrt";
}

class B {
        int x;   
        int getDubX() {
                return 2*x;
        }
}
```

这个语法做到了。

```
grammar Island;
file : clazz* ;
clazz : 'class' ID '{' ignore '}' ;
ignore : (method|.)*? ;
method : type ID '()' block ;
type : 'int' | 'void' ;
block : '{' (block | .)*? '}' ;
ID : [a-zA-Z] [a-zA-Z0-9]* ;
WS : [ \r\t\n]+ -> skip ;
ANY : . ;
```

你得到:

<img src = images/nonnested-fuzzy.png width = 450>

现在让我们尝试一些嵌套类

```
class A {
        String name = "parrt";
        class Nested {
            any filthy shite we want in here { }}}}}}
        }
}

class B {
        int x;   
        int getDubX() {
                return 2*x;
        }
}

```

```
grammar Island;
file : clazz* ;
clazz : 'class' ID '{' ignore '}' ;
ignore : (method|clazz|.)*? ; //<- only change is to add clazz alt here
method : type ID '()' block ;
type : 'int' | 'void' ;
block : '{' (block | .)*? '}' ;
ID : [a-zA-Z] [a-zA-Z0-9]* ;
WS : [ \r\t\n]+ -> skip ;
ANY : . ;
```

你得到:

<img src = images/nested-fuzzy.png宽度= 600>