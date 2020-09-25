# ANTLR Tool Command Line Options

如果您在不使用命令行参数的情况下调用ANTLR工具,则会获得帮助消息:

```bash
$ antlr4
ANTLR Parser Generator  Version 4.7.1
 -o ___              specify output directory where all output is generated
 -lib ___            specify location of grammars, tokens files
 -atn                generate rule augmented transition network diagrams
 -encoding ___       specify grammar file encoding; e.g., euc-jp
 -message-format ___ specify output style for messages in antlr, gnu, vs2005
 -long-messages      show exception details when available for errors and warnings
 -listener           generate parse tree listener (default)
 -no-listener        don't generate parse tree listener
 -visitor            generate parse tree visitor
 -no-visitor         don't generate parse tree visitor (default)
 -package ___        specify a package/namespace for the generated code
 -depend             generate file dependencies
 -D<option>=value    set/override a grammar-level option
 -Werror             treat warnings as errors
 -XdbgST             launch StringTemplate visualizer on generated code
 -XdbgSTWait         wait for STViz to close before continuing
 -Xforce-atn         use the ATN simulator for all predictions
 -Xlog               dump lots of logging info to antlr-timestamp.log
 -Xexact-output-dir  all output goes into -o dir regardless of paths/package
```

以下是有关这些选项的更多详细信息:

## `-o outdir`

默认情况下,ANTLR在当前目录中生成输出文件。此选项指定ANTLR应在其中生成解析器,侦听器,访问者和令牌文件的输出目录。
  
```bash
$ antlr4 -o /tmp T.g4
$ ls /tmp/T*
/tmp/T.tokens /tmp/TListener.java
/tmp/TBaseListener.java /tmp/TParser.java
```

## `-lib libdir`

当寻找标记文件和导入的语法时,ANTLR通常会在当前目录中寻找。该选项指定要查找的目录。它仅用于解析import语句和tokenVocab选项的语法引用。主语法的路径必须始终完全指定。
  
	$ cat /tmp/B.g4
	
	parser grammar B;
	
	x : ID ;
	
	$ cat A.g4
	
	grammar A;
	
	import B;
	
	s : x ;
	
	ID : [a-z]+ ;
	
	$ antlr4 -lib /tmp A.g4

## `-atn`

生成DOT图形文件,这些文件表示ANTLR用来表示语法的内部ATN(增强过渡网络)数据结构。文件以Grammar.rule .dot的形式出现。如果文法是组合文法,则词法分析器规则将命名为语法Lexer.rule .dot。
  
	$ cat A.g4
	
	grammar A;
	
	s : b ;
	
	b : ID ;
	
	ID : [a-z]+ ;
	
	$ antlr4 -atn A.g4
	
	$ ls *.dot
	
	A.b.dot A.s.dot ALexer.ID.dot


## `-encoding encodingname`

默认情况下,ANTLR使用UTF-8编码加载语法文件,这是一种非常常见的字符文件编码,对于适合一个字节的字符,该文件会退化为ASCII。世界各地有许多字符文件编码。如果该语法文件不是您的语言环境的默认编码,则需要此选项,以便ANTLR可以正确解释语法文件。这不会影响到生成的解析器的输入,只会影响语法本身的编码。

## `-message-format format`

ANTLR使用来自目录tool/resources/org/antlr/v4/tool/templates/messages/formats的模板生成警告和错误消息。默认情况下,ANTLR使用antlr.stg(StringTemplate组)文件。您可以将其更改为gnu或vs2005,以使ANTLR生成适用于Emacs或Visual Studio的消息。要创建自己的X,请创建资源org/antlr/v4/tool/templates/messages/formats/X并将其放在CLASSPATH中。

## `-listener`

此选项告诉ANTLR生成解析树侦听器,并且是默认选项。

## `-no-listener`

此选项告诉ANTLR不要生成解析树侦听器。

## `-visitor`

默认情况下,ANTLR不会生成解析树访问者。此选项打开该功能。ANTLR可以生成解析树侦听器和访客。此选项和-listener并不互斥。

## `-no-visitor`

告诉ANTLR不要生成解析树访问者;这是默认值。

## `-package`

使用此选项可以为ANTLR生成的文件指定包或名称空间。另外,您可以添加一个@header {...}动作,但会将语法与特定语言联系在一起。如果使用此选项和@header,请确保header操作不包含程序包规范,否则生成的代码将包含其中两个。

## `-depend`

而不是生成解析器和/或词法分析器,而是生成文件依赖关系列表,每行一个。输出显示每个语法所依赖的内容及其生成的内容。这对于需要了解ANTLR语法依赖性的构建工具很有用。这是一个示例:
  
```bash
$ antlr4 -depend T.g	
T.g: A.tokens
TParser.java : T.g
T.tokens : T.g
TLexer.java : T.g
TListener.java : T.g
TBaseListener.java : T.g
```

如果将-lib libdir与-depend和语法选项tokenVocab = A一起使用,则依赖项还包括库路径:Tg:libdir/A.tokens。输出对-o outdir选项也很敏感:outdir/TParser.java:Tg

## `-D<option>=value`

使用此选项可以覆盖或设置一个或多个指定语法中的语法级别选项。此选项对于在不更改语法本身的情况下以不同的语言生成解析器很有用。(我预计不久的将来还会有其他目标。)
  
```bash
$ antlr4 -Dlanguage=Java T.g4 # default
$ antlr4 -Dlanguage=C T.g4
error(31): ANTLR cannot generate C code as of version 4.0b3
```

## `-Werror`

作为大型构建的一部分,ANTLR警告消息可能不会被注意。打开此选项可将警告视为错误,从而使ANTLR工具将故障报告回调用命令行外壳程序。
还有一些扩展选项,主要用于调试ANTLR本身:

## `-Xsave-lexer`

ANTLR从组合语法中生成解析器和词法分析器。为了创建词法分析器,ANTLR从组合语法中提取词法分析器语法。有时,如果不清楚ANTLR正在创建什么令牌规则,则可以查看其外观。这不会影响生成的解析器或词法分析器。

## `-XdbgST`

对于构建代码生成目标的用户,此选项将打开一个窗口,其中显示了生成的代码以及用于生成该代码的模板。它调用StringTemplate检查器窗口。

## `-Xforce-atn`

ANTLR通常会构建传统的“打开令牌类型”。尽可能进行决策(一个先行标记足以区分决策中的所有备选方案)。要将这些简单的决定强制纳入自适应LL(*)机制,请使用此选项。

## `-Xlog`

此选项会在处理语法时创建一个包含来自ANTLR的大量信息消息的日志文件。如果您想了解ANTLR如何转换您的左递归规则,请打开此选项并查看生成的日志文件。
  
```bash
$ antlr4 -Xlog T.g4 	
wrote ./antlr-2012-09-06-17.56.19.log
```

## `-Xexact-output-dir`

(*请参阅[讨论](https://github.com/antlr/antlr4/pull/2065)*)。

所有输出都进入`-o`目录,与路径/包无关。

*输出`-o`目录说明符是包含输出的确切目录。以前,出于打包目的,它将包括在语法本身上指定的相对路径。

**新**:`-o /tmp subdir/T.g4` => `/tmp/subdir/T.java`
**旧**:`-o /tmp subdir/T.g4` => `/tmp/T.java`

*以前,我们在`-lib`目录或输出目录中查找vocab文件。**新增**:还要在包含语法的目录中查找,特别是如果它是用路径指定的。

###输出目录示例(4.7)

这是现有的4.7功能。

(对于这些示例,假设a4.7和a4.7.1是ANTLR的`org.antlr.v4.Tool`正确版本的别名。)

```bash
$ cd /tmp/parrt
$ tree
.
├── B.g4
└── src
    └── pkg
        └── A.g4
$ a4.7 -o /tmp/build src/pkg/A.g4
$ tree /tmp/build
/tmp/build/
└── src
    └── pkg
        ├── A.tokens
        ├── ABaseListener.java
        ├── ALexer.java
        ├── ALexer.tokens
        ├── AListener.java
        └── AParser.java
```

现在,让我们构建一个位于当前目录中的语法:

```bash
$ a4.7 -o /tmp/build B.g4
$ tree /tmp/build
/tmp/build
├── B.tokens
├── BBaseListener.java
├── BLexer.java
├── BLexer.tokens
├── BListener.java
├── BParser.java
└── src
    └── pkg
        ├── A.tokens
        ├── ABaseListener.java
        ├── ALexer.java
        ├── ALexer.tokens
        ├── AListener.java
        └── AParser.java
```

最后,如果不指定输出目录,它将注意输入语法中指定的相对路径:

```bash
$ a4.7 src/pkg/A.g4
$ tree
.
├── B.g4
└── src
    └── pkg
        ├── A.g4
        ├── A.tokens
        ├── ABaseListener.java
        ├── ALexer.java
        ├── ALexer.tokens
        ├── AListener.java
        └── AParser.java
```

###输出目录示例(带有-Xexact-output-dir的4.7.1)

现在,无论语法上的相对路径如何,输出目录都是生成输出的确切目录

```bash
$ cd /tmp/parrt
$ a4.7.1 -Xexact-output-dir  -o /tmp/build src/pkg/A.g4
$ tree /tmp/build
/tmp/build
├── A.tokens
├── ABaseListener.java
├── ALexer.java
├── ALexer.tokens
├── AListener.java
└── AParser.java
```

如果使用package选项,则使用`-o`仍不会更改生成输出的位置

```bash
$ a4.7.1 -Xexact-output-dir -package pkg -o /tmp/build src/pkg/A.g4
$ tree /tmp/build
/tmp/build
├── A.tokens
├── ABaseListener.java
├── ALexer.java
├── ALexer.tokens
├── AListener.java
└── AParser.java
```

但是,4.7.1确实将软件包规范添加到了生成的文件中:

```bash
$ grep package /tmp/build/A*.java
/tmp/build/ABaseListener.java:package pkg;
/tmp/build/ALexer.java:package pkg;
/tmp/build/AListener.java:package pkg;
/tmp/build/AParser.java:package pkg;
```

将此与4.7进行比较:

```bash
$ a4.7 -package pkg -o /tmp/build src/pkg/A.g4
beast:/tmp/parrt $ tree /tmp/build
/tmp/build
└── src
    └── pkg
        ├── A.tokens
        ├── ABaseListener.java
        ├── ALexer.java
        ├── ALexer.tokens
        ├── AListener.java
        └── AParser.java
```

###在哪里寻找代币vocab的示例

在4.7版中,我们遇到了一个明显的错误,应该可以正常工作:

```bash
$ cd /tmp/parrt
$ tree
.
└── src
    └── pkg
        ├── L.g4
        └── P.g4
$ a4.7 -o /tmp/build src/pkg/*.g4
error(160): P.g4:2:21: cannot find tokens file /tmp/build/L.tokens
warning(125): P.g4:3:4: implicit definition of token A in parser
```

在4.7.1中,它也在包含语法的目录中查找:

```bash
$ a4.7.1 -o /tmp/build src/pkg/*.g4
$ tree /tmp/build
/tmp/build
├── L.java
├── L.tokens
├── P.java
├── P.tokens
├── PBaseListener.java
├── PListener.java
└── src
    └── pkg
        ├── L.java
        └── L.tokens
```