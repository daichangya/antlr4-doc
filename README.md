## 微信公众号

扫码关注微信公众号，分布式编程。
![分布式编程](http://www.images.mdan.top/qrcode_for_gh_1e2587cc42b1_258_1587996055777.jpg)

[https://zthinker.com/](https://zthinker.com/)

中文文档网址：https://daichangya.github.io/antlr4-doc/#/


# ANTLR 4 Documentation

在对stackoverflow或antlr讨论列表提出问题之前,请检查[常见问题(FAQ)](faq/index.md)。

笔记:
<ul>
<li>要添加或改进本文档,请<a href=https://help.github.com/articles/fork-a-repo> fork </a> <a href = https://github.com/antlr/antlr4> antlr/antlr4存储库</a>,然后更新该“ doc/index.md”或该目录中的文件。提交<a href=https://help.github.com/articles/creating-a-pull-request>拉请求</a>,以将您的更改合并到主存储库中。不要在示例拉取请求中混合使用代码和文档更新。<b>如果您之前没有签署过请求,则必须在您的拉取请求上签名原始的contributors.txt。</b> </li>

<li>版权所有©2012,实用书架。实用书架授予非独家,不可撤销,免版税的全球许可,用于复制,分发,准备衍生作品,或将其用作ANTLR项目和相关文档的一部分。</li>

<li>尽管其中的大部分内容是在<a href=http://pragprog.com/book/tpantlr2/the-definitive-antlr-4-reference> Definitive ANTLR 4 Reference </a>的允许下复制的。随着工具的更改,它会随着时间的推移而变形。</li>
</ul>

文档中的链接引用了本书的各个部分,但已重定向到出版商网站上的常规书籍页面。出版商网站上有两个摘录,可能对您有用,而不必购买该书:[Let's get Meta](http://media.pragprog.com/titles/tpantlr2/picture.pdf)和[Build a Translator (使用侦听器)](http://media.pragprog.com/titles/tpantlr2/listener.pdf)。您还应该考虑阅读以下书籍(vid描述了参考书):

<a href=""> <img src = images/tpantlr2.png宽度= 120> </a>
<a href=""> <img src = images/tpdsl.png宽度= 120> </a>
<a href="https://www.youtube.com/watch?v=OAoA3E-cyug"> <img src = images/teronbook.png宽度= 250> </a>

本文档是参考,概述了语法语法和ANTLR语法的关键语义。本书的所有示例(不仅是本章)的源代码都可以在出版商的网站上免费获得。以下视频是ANTLR 4的一般概览,并描述了如何使用解析树侦听器轻松处理Java文件:

<a href="https://vimeo.com/59285751"> <img src = images/tertalk.png宽度= 200> </a>

对于使用Java的用户,这是Andreas Stefik撰写的[Intellij笔记中的ANTLR集](https://docs.google.com/document/d/1gQ2lsidvN2cDUUsHEkT05L-wGbX5mROB7d70Aaj3R64/edit#heading=h.xr0jj8vcdsgc)。

## Sections

* [ANTLR v4入门](getting-started.md)

* [语法词典](lexicon.md)

* [语法结构](grammars.md)

* [解析器规则](parser-rules.md)

* [左递归规则](left-recursion.md)

* [动作和属性](actions.md)

* [Lexer规则](lexer-rules.md)

* [通配符和非贪婪子规则](wildcard.md)

* [解析树侦听器](listeners.md)

* [解析树匹配和XPath](tree-matching.md)

* [语义谓词](predicates.md)

* [选项](options.md)

* [ANTLR工具命令行选项](tool-options.md)

* [运行时库和代码生成目标](targets.md)

* [Unicode U + FFFF,U + 10FFFF字符流](unicode.md)

* [解析二进制流](parsing-binary-files.md)

* [不区分大小写的Lexing](case-insensitive-lexing.md)

* [解析器和词法分析器解释器](interpreters.md)

* [资源](resources.md)

# Building/releasing ANTLR itself

* [构建ANTLR本身](building-antlr.md)

* [为ANTLR贡献](/CONTRIBUTING.md)

* [剪切ANTLR版本](releasing-antlr.md)

* [ANTLR项目单元测试](antlr-project-testing.md)

* [创建ANTLR语言目标](creating-a-language-target.md)