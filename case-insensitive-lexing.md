# Case-Insensitive Lexing

在某些语言中,关键字不区分大小写,这意味着`BeGiN`与`begin`或`BEGIN`含义相同。ANTLR具有两种机制来支持此类语言的构建语法:

1.建立匹配大写或小写的词汇规则。
  *优势:无需对ANTLR进行任何更改,从而在语法上清楚地表明这种情况下的语言不敏感。
  * **缺点**:效率成本可能较低,语法较为冗长,编写起来也很麻烦。

2.建立与所有大写字母匹配的关键字的词法规则,然后使用自定义的[字符流](https://github.com/antlr/antlr4/blob/master/runtime/Java/src/org/antlr/v4/runtime/CharStream.java),将所有字符转换为大写,然后再将它们发送到词法分析器(通过`LA()`方法)。必须注意不要将流中的所有字符都转换为大写,因为字符串和注释中的字符应不受影响。我们真正想要的只是诱使词法分析器认为输入全部为大写。
  * **优势**:视实现方式而定,可能具有速度优势,语法无需更改。
  * **缺点**:要求不区分大小写的流和语法正确结合使用,使所有字符在词法分析器中均显示为大写/小写,但某些语法在关键字之外区分大小写,因此会出现新的大小写错误不敏感的流和语言输出目标(java,C#,C ++等)。

对于4.7.1版本,我们在[detail](https://github.com/antlr/antlr4/pull/2046)中讨论了两种方法,甚至可能更改了ANTLR元语言以直接支持不区分大小写的词法化。我们讨论了将不区分大小写的流包含到运行时中,但并非所有内容都将立即得到支持。我决定简单地制作文档,清楚地说明如何处理并包括人们可以在其语法中剪切粘贴的适当片段。

## Case-insensitive grammars

作为专门描述不区分大小写的关键字的语法的主要示例,请参见
[SQLite语法](https://github.com/antlr/grammars-v4/blob/master/sqlite/SQLite.g4)。为了匹配不区分大小写的关键字,有如下规则

```
K_UPDATE : U P D A T E;
```

它将与`UpdaTE`和`upDATE`等匹配为`update`关键字。此规则利用了一些通用的片段规则,您可以将这些规则剪切并粘贴到语法中:

```
fragment A : [aA]; //match either an 'a' or 'A'
fragment B : [bB];
fragment C : [cC];
fragment D : [dD];
fragment E : [eE];
fragment F : [fF];
fragment G : [gG];
fragment H : [hH];
fragment I : [iI];
fragment J : [jJ];
fragment K : [kK];
fragment L : [lL];
fragment M : [mM];
fragment N : [nN];
fragment O : [oO];
fragment P : [pP];
fragment Q : [qQ];
fragment R : [rR];
fragment S : [sS];
fragment T : [tT];
fragment U : [uU];
fragment V : [vV];
fragment W : [wW];
fragment X : [xX];
fragment Y : [yY];
fragment Z : [zZ];
```

使用这种机制无需区分大小写,无需特殊的流。

## Custom character streams approach

另一种方法是使用匹配全部大写或全部小写的词法规则,例如:

```
K_UPDATE : 'UPDATE';
```

然后,在创建要解析的字符流时,我们需要一个自定义类,该类将覆盖词法分析器使用的方法。在下面,您将找到可复制到项目中的许多目标的自定义字符流,但是这里以Java中的流为例:

```java
CharStream s = CharStreams.fromPath(Paths.get('test.sql'));
CaseChangingCharStream upper = new CaseChangingCharStream(s, true);
Lexer lexer = new SomeSQLLexer(upper);
```

以下是各种目标语言中`CaseChangingCharStream`的实现:

* [C#](https://github.com/antlr/antlr4/blob/master/doc/resources/CaseChangingCharStream.cs)
* [开始](https://github.com/antlr/antlr4/blob/master/doc/resources/case_changing_stream.go)
* [Java](https://github.com/antlr/antlr4/blob/master/doc/resources/CaseChangingCharStream.java)
* [JavaScript](https://github.com/antlr/antlr4/blob/master/doc/resources/CaseChangingStream.js)
* [Python2/3](https://github.com/antlr/antlr4/blob/master/doc/resources/CaseChangingStream.py)