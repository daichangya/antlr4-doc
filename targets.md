# Runtime Libraries and Code Generation Targets

此页面列出了可用的和即将推出的ANTLR运行时。请注意,您在这里找不到特定于语言的代码生成器。这是因为只有一种用Java编写的工具能够通过命令行选项为所有目标生成词法分析器和解析器代码。该工具可以从命令行或流行的IDE和构建系统的任何集成插件中调用:Eclipse,IntelliJ,Visual Studio,Maven。因此,无论您的环境和目标是什么,您都应该能够运行该工具并以目标语言生成代码。在撰写本文时,可用的目标如下:

* [Java](java-target.md)。[ANTLR v4书籍](http://pragprog.com/book/tpantlr2/the-definitive-antlr-4-reference)具有运行时库的详细介绍。自从本书印制以来,我们已经添加了有用的XPath功能,使您可以选择一些解析树。请参阅[运行时API](http://www.antlr.org/api/Java/index.html)和[ANTLR v4入门](getting-started.md)
* [C#](csharp-target.md)
* [Python](python-target.md)(2和3)
* [JavaScript](javascript-target.md)
* [开始](go-target.md)
* [C ++](cpp-target.md)
* [Swift](swift-target.md)
* [PHP](php-target.md)

## Target feature parity

新功能通常会出现在Java目标中,然后迁移到其他目标中,但是这些其他目标并不总是在同一总体工具版本中得到更新。本部分尝试确定尚未添加到其他目标的已添加到Java的功能。

|功能|Java|C&sharp; |Python2|Python3|JavaScript|Go|C ++ |Swift|PHP
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
|树木模糊的结构|4.5.1|-|-|-|-|-|-|-|-|-|