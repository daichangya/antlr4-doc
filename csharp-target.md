# C&sharp;

## Which frameworks are supported?

C#运行时符合CLS,仅需要相应的3.5 .Net框架。

实际上,已经针对以下方面对运行时进行了广泛的测试:

* Microsoft .Net 3.5框架
* Mono .Net 3.5框架

没有发现任何问题,因此您应该发现运行时在任何最新的.Net框架中都可以正常工作。

## How do I get started?

您可以在[ANTLR C#运行时的Git存储库页面](https://github.com/antlr/antlr4/tree/master/runtime/CSharp)中找到完整的说明。
 
## How do I use the runtime from my project?

(即,如何运行生成的词法分析器和/或解析器?)

假设您的语法命名为`MyGrammar`。该工具将为您生成以下文件:

* MyGrammarLexer.cs
* MyGrammarParser.cs
* MyGrammarListener.cs(如果尚未激活-no-listener选项)
* MyGrammarBaseListener.cs(如果尚未激活-no-listener选项)
* MyGrammarVisitor.cs(如果您已激活-visitor选项)
* MyGrammarBaseVisitor.cs(如果您已激活-visitor选项)

现在,功能完整的代码可能类似于启动规则`StartRule`的以下代码:

```csharp
using Antlr4.Runtime;
using Antlr4.Runtime.Tree;
     
public void MyParseMethod() {
      String input = "your text to parse here";
      ICharStream stream = CharStreams.fromstring(input);
      ITokenSource lexer = new MyGrammarLexer(stream);
      ITokenStream tokens = new CommonTokenStream(lexer);
      MyGrammarParser parser = new MyGrammarParser(tokens);
      parser.BuildParseTree = true;
      IParseTree tree = parser.StartRule();
}
```

该程序将起作用。除非您执行以下操作之一,否则它将不会有用:

*您使用自定义侦听器访问解析树
*您使用自定义访问者访问解析树
*您的语法包含生产代码(例如AntLR3)

(请注意,生产代码是特定于目标的,因此您不能拥有包含生产代码的多目标语法)
 
## How do I create and run a custom listener?

假设您的MyGrammar语法包含2条规则:“键”和“值”。

antlr4工具将生成以下侦听器(此处仅显示部分代码): 

```csharp
interface IMyGrammarParserListener : IParseTreeListener {
      void EnterKey (MyGrammarParser.KeyContext context);
      void ExitKey (MyGrammarParser.KeyContext context);
      void EnterValue (MyGrammarParser.ValueContext context);
      void ExitValue (MyGrammarParser.ValueContext context);
}
```
 
为了提供自定义行为,您可能需要创建以下类:
 
```csharp
class KeyPrinter : MyGrammarBaseListener {
    //override default listener behavior
    void ExitKey (MyGrammarParser.KeyContext context) {
        Console.WriteLine("Oh, a key!");
    }
}
```
   
为了执行此侦听器,您只需将以下行添加到上面的代码中:
 
 
```csharp
...
IParseTree tree = parser.StartRule() - only repeated here for reference
KeyPrinter printer = new KeyPrinter();
ParseTreeWalker.DEFAULT.walk(printer, tree);
```
        
可以从《权威ANTLR参考》书中找到更多信息。

ANTLR的C#实现与Java尽可能接近,因此您不应该发现很难为C#改编示例。另请参阅[Sam Harwell的替代C#目标](https://github.com/tunnelvisionlabs/antlr4cs)