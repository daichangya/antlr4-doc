# Parse Tree Listeners

*部分取自公开可见的内容[摘自ANTLR 4书](http://media.pragprog.com/titles/tpantlr2/picture.pdf)*

默认情况下,ANTLR生成的解析器构建一个称为解析树或语法树的数据结构,该数据结构记录解析器如何识别输入语句和组成短语的结构。

<img src = images/process.png>

语法分析树的内部节点是短语名称,用于分组和标识其子级。根节点是最抽象的短语名称,在这种情况下为`stat`(语句的缩写)。解析树的叶子始终是输入标记。解析树位于语言识别器和解释器或翻译器的实现之间。它们是非常有效的数据结构,因为它们包含所有输入内容,并且具有解析器如何将符号分组为短语的完整知识。更好的是,它们易于理解,并且解析器会自动生成它们(除非您使用`parser.setBuildParseTree(false)`将其关闭)。

因为我们用一组规则指定短语结构,所以分析树子树的根与语法规则名称相对应。ANTLR有一个ParseTreeWalker,它知道如何遍历这些解析树并触发可创建的侦听器实现对象中的事件。除非您使用命令行选项将其关闭,否则ANTLR工具也会为您生成侦听器接口。您也可以让它产生访客。例如,从Java.g4语法中,ANTLR生成:

```java
public interface JavaListener extends ParseTreeListener<Token> {
  void enterClassDeclaration(JavaParser.ClassDeclarationContext ctx);
  void exitClassDeclaration(JavaParser.ClassDeclarationContext ctx);
  void enterMethodDeclaration(JavaParser.MethodDeclarationContext ctx);
 ...
}
```

解析器语法中每个规则都有一个输入和退出方法。ANTLR还会使用所有侦听器接口方法的空实现来生成基本侦听器,在本例中称为JavaBaseListener。您可以通过继承此基础并覆盖感兴趣的方法来构建您的侦听器。

假设您已经创建了一个名为`MyListener`的侦听器对象,以下是调用Java解析器并遍历解析树的方法:

```java
JavaLexer lexer = new JavaLexer(input);
CommonTokenStream tokens = new CommonTokenStream(lexer);
JavaParser parser = new JavaParser(tokens);
JavaParser.CompilationUnitContext tree = parser.compilationUnit(); //parse a compilationUnit

MyListener extractor = new MyListener(parser);
ParseTreeWalker.DEFAULT.walk(extractor, tree); //initiate walk of tree with listener in use of default walker
```

侦听器和访问者之所以出色,是因为它们将特定于应用程序的代码排除在语法之外,从而使语法更易于阅读,并防止它们与特定应用程序纠缠在一起。

请参阅本书,以获取有关听众的更多信息并了解如何使用访问者。(侦听器和访客机制之间的最大区别在于,侦听器方法是由ANTLR提供的walker对象独立调用的,而访客方法则必须通过显式的访问调用来遍历其子代。忘记在节点的子代上调用访客方法,表示这些子树不会被访问。)