# Python (2 and 3)

ANTLR 4本书中转换为Python的示例在[here](https://github.com/jszheng/py3antlr4book)。

有2个Python目标:`Python2`和`Python3`。这是因为这两个版本的语言之间的兼容性有限。有关完整的详细信息,请参考[Python文档](https://wiki.python.org/moin/Python2orPython3)。

如何创建Python词法分析器或解析器?
这与创建Java词法分析器或解析器几乎相同,除了需要指定语言目标外,例如:

```
$ antlr4 -Dlanguage=Python2 MyGrammar.g4
```

要么

```
$ antlr4 -Dlanguage=Python3 MyGrammar.g4
```

有关antlr4工具选项的完整列表,请访问工具文档页面。

## Where can I get the runtime?

生成词法分析器和/或解析器代码后,您需要下载运行时。可以从PyPI获得Python运行时:

* https://pypi.python.org/pypi/antlr4-python2-runtime/
* https://pypi.python.org/pypi/antlr4-python3-runtime/

运行时以源代码的形式提供,因此不需要其他安装。

我们这里将不介绍如何从您的Python项目中引用运行时,因为这取决于您的项目类型和IDE会有很大不同。 

## How do I run the generated lexer and/or parser?

假设您的语法如上所述被命名为“ MyGrammar”。让我们假设该解析器包含一个名为“ startRule”的规则。该工具将为您生成以下文件:

* MyGrammarLexer.py
* MyGrammarParser.py
* MyGrammarListener.py(如果尚未激活-no-listener选项)
* MyGrammarVisitor.py(如果您已激活-visitor选项)

(习惯于Java/C#AntLR的开发人员会注意到没有生成任何基本的侦听器或访问者,这是因为Python不支持接口,因此生成的侦听器和访问者是完全成熟的类)

现在,功能完整的脚本可能如下所示:
 
```python
import sys
from antlr4 import *
from MyGrammarLexer import MyGrammarLexer
from MyGrammarParser import MyGrammarParser
 
def main(argv):
    input_stream = FileStream(argv[1])
    lexer = MyGrammarLexer(input_stream)
    stream = CommonTokenStream(lexer)
    parser = MyGrammarParser(stream)
    tree = parser.startRule()
 
if __name__ == '__main__':
    main(sys.argv)
```

该程序将起作用。除非您执行以下操作之一,否则它将不会有用:

*您使用自定义侦听器访问解析树
*您使用自定义访问者访问解析树
*您的语法包含生产代码(例如ANTLR3)

(请注意,生产代码是特定于目标的,因此,除了非常有限的用例之外,您不能拥有包含生产代码的多目标语法,请参见下文)
 
## How do I create and run a custom listener?

假设您的MyGrammar语法包含2条规则:“键”和“值”。antlr4工具将生成以下侦听器:

```python
class MyGrammarListener(ParseTreeListener):
    def enterKey(self, ctx):
        pass
    def exitKey(self, ctx):
        pass
    def enterValue(self, ctx):
        pass
    def exitValue(self, ctx):
        pass
```
 
为了提供自定义行为,您可能需要创建以下类:
  
```python
class KeyPrinter(MyGrammarListener):     
    def exitKey(self, ctx):         
        print("Oh, a key!") 
```
 
为了执行此侦听器,您只需将以下行添加到上面的代码中:
 
```
       ...
       tree = parser.startRule() - only repeated here for reference
   printer = KeyPrinter()
   walker = ParseTreeWalker()
   walker.walk(printer, tree)
```
 
可以从ANTLR 4最终指南中找到更多信息。

ANTLR的Python实现与Java尽可能接近,因此您不应该发现很难将示例改编成Python。

## Target agnostic grammars

如果您的语法仅针对Python,则可以忽略以下内容。但是,如果您的目标是使Java解析器也可以在Python中运行,那么您可能会发现它很有用。

1.不要将生产代码嵌入语法中。这不是便携式的,也不会。将所有代码移至侦听器或访客。
1.语法必不可少的唯一生产代码应该是语义谓词,例如:
```
ID {$text.equals("test")}?
```

不幸的是,这不是便携式的,但是您可以解决它。诀窍包括:

*从您提供的解析器(例如BaseParser)派生您的解析器
*在此BaseParser中实现实用程序方法,例如“ isEqualText”
*在Java/C#BaseParser中添加一个“ self”字段,并使用“ this”对其进行初始化

由于上述原因,您应该能够如下重写上述语义谓词:

```
ID {$self.isEqualText($text,"test")}?
```