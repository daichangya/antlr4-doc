# Creating an ANTLR Language Target

本文档介绍了如何使ANTLR以新的语言* X *生成解析器。

## Overview

创建新目标涉及以下关键要素:

1.对于该工具,在包`org.antlr.v4.codegen.target`中创建类* X * Target作为类`Target`的子类。此类描述有关转义字符和字符串等的语言特定的详细信息。通常,这里很少要做。
1.在目录tool/resources/org/antlr/v4/tool/templates/codegen/* X */* X * .stg中创建* X * .stg。这是一个[StringTemplate](http://www.stringtemplate.org/)组文件(`.stg`),该文件告诉ANTLR如何表达生成代码所需的所有解析元素。您将看到名为`ParserFile`,`Parser`,`Lexer`,`CodeBlockForAlt`,`AltBlock`等的模板。这些模板中的每一个都必须描述如何构建所示的模板代码块。最好的选择是找到最接近的现有目标,复制该模板文件,并进行调整以适合需要。
1.创建一个运行时库以支持ANTLR生成的解析器。在目录runtime/* X *下,您可以完全控制该目录结构,这取决于该目标语言的常用用法。例如,Java具有:`runtime/Java/lib`和`runtime/Java/src`目录。在`src`下,您将找到软件包`org.antlr.v4.runtime`及其以下的目录结构。
1.创建用于运行时测试的模板文件。您所要做的就是提供一些模板,这些模板指示如何打印值和声明变量。我们在dir `runtime-testsuite`中的运行时测试机制将使用这些模板为每个目标自动生成代码,并检查测试结果。它需要知道如何定义各种类字段,比较成员等。您必须在[runtime-testsuite/resources/org/antlr/v4/test/runtime]下创建一个* X * .test.stg文件(https://github.com/antlr/antlr4/tree/master/runtime-testsuite/resources/org/antlr/v4/test/runtime)。同样,最好的选择是将模板从最接近的语言复制到您的目标,然后对其进行调整以适应需要。

## Getting started

1.在github上将`antlr/antlr4`存储库分叉给您自己的用户,以便您拥有存储库`username/antlr4`。
2.将分支的存储库`username/antlr4`克隆到本地磁盘。您的远程`origin`将成为GitHub上的分叉存储库。将远程`upstream`添加到原始`antlr/antlr4`存储库(URL `https://github.com/antlr/antlr4.git`)。您想回馈给项目的更改是通过[pull request](https://help.github.com/articles/using-pull-requests/)完成的。
3.尝试做任何事情之前先构建它
```bash
$ mvn compile
```
那应该成功。有关更多详细信息,请参见[Building ANTLR](building-antlr.md)。