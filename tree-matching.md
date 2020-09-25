# Parse Tree Matching and XPath

*自ANTLR 4.2起*

ANTLR 4引入了访问者和侦听器机制,使您可以实现树节点的DOM访问或类似于SAX的事件处理。这很好。例如,如果您只关心Java方法声明,请获取`Java.g4`文件,然后在`JavaBaseListener`中覆盖methodDeclaration。从那里,`ParseTreeWalker`可以在遍历树时触发对您的重写方法的调用。简单的事情很容易。

这种机制或多或少地在节点级别上起作用。换句话说,对于每个方法声明子树根,都会调用您的`methodDeclaration()`。在许多情况下,我们更关心子树而不仅仅是节点。我们可能要:

*收集特定上下文中的方法声明(即,嵌套在另一个方法中)或具有特定结构或特定类型的方法(例如,`void <ID>() { }`)。为此,我们将结合`XPath`和树模式匹配。
*通过树中的模式对转换操作进行分组,而不是在侦听器事件方法之间分布操作。
*获取树中任何位置的所有作业的列表。说*去找我全部“ ... = ...;”要容易得多。subtrees *而不是创建类只是为了获取规则分配的侦听器方法,然后将侦听器传递给解析树遍历器。

这里的另一个重要思想是,由于我们所讨论的是解析树而不是抽象语法树,因此我们可以使用具体模式代替树语法。例如,我们可以说`x = 0;`而不是AST `(= x 0)`,其中`;`在进入AST之前可能会剥离。

## Parse tree patterns

为了测试子树是否具有特定的结构,我们使用树型。我们还经常想根据结构从子树中提取后代。一个非常简单的示例是检查子树是否与赋值语句匹配。该模式在您的语言中可能类似于以下内容:

```
<ID> = <expr>;
```

其中尖括号中的“标记”表示关联语法中的标记或规则引用。ANTLR将该字符串转换为带有特殊节点的解析树,这些特殊节点表示任何令牌`ID`和规则`expr`子树。要创建此解析树,模式匹配编译器需要知道模式符合语法中的哪个规则。在这种情况下,可能是声明。这是我们如何测试`t`树以查看其是否与该模式匹配:

```java
ParseTree t = ...; //assume t is a statement
ParseTreePattern p = parser.compileParseTreePattern("<ID> = <expr>;", MyParser.RULE_statement);
ParseTreeMatch m = p.match(t);
if ( m.succeeded() ) {...}
```

我们还可以测试特定的表达式或标记值。例如,以下检查以查看`t`是否为包含添加到0的标识符的表达式:

```java
ParseTree t = ...; //assume t is an expression
ParseTreePattern p = parser.compileParseTreePattern("<ID>+0", MyParser.RULE_expr);
ParseTreeMatch m = p.match(t);
```

我们还可以要求`ParseTreeMatch`结果提取与`<ID>`标签匹配的令牌:

```java
String id = m.get("ID");
```

您可以使用模式匹配器上的方法来更改标签定界符:

```java
ParseTreePatternMatcher m = new ParseTreePatternMatcher();
m.setDelimiters("<<", ">>", "$"); //$ is the escape character
```

这将允许将模式`<<ID>> = <<expr>> ;$<< ick $>>`解释为元素:`ID`,` = `,`expr`和` ;<< ick >>`。

```java
String xpath = "//blockStatement/*";
String treePattern = "int <Identifier> = <expression>;";
ParseTreePattern p =
parser.compileParseTreePattern(treePattern,
JavaParser.RULE_localVariableDeclarationStatement);
List<ParseTreeMatch> matches = p.findAll(tree, xpath);
```

###模式标签

树模式匹配器在对树模式中的标签进行匹配时跟踪树中的节点。这样,我们可以使用`get()`和`getAll()`方法来检索匹配的子树的组件。例如,对于模式`<ID>`,`get("ID")`返回与该`ID`匹配的节点。如果有多个节点与指定的标记或规则标记匹配,则仅返回第一个匹配项。如果没有与标签关联的节点,则返回null。

您也可以使用标识符来标记标签。如果标签是语法中解析器规则或标记的名称,则`getAll()`(或`get()`的节点)的结果列表将包含解析树匹配规则或标记有标签的标签以及与解析器规则或标记的模式中与已标记和未标记标签匹配的完整解析树集。例如,如果label为`foo`,则结果将包含以下所有内容。

*解析与`<foo:anyRuleName>`和`<foo:AnyTokenName>`形式的标签匹配的树节点。
*解析与`<anyLabel:foo>`形式的标签匹配的树节点。
*解析与`<foo>`形式的标签匹配的树节点。

###使用模式匹配器创建解析树

您可以使用解析树模式编译器为部分输入片段创建解析树。只需使用方法`ParseTreePattern.getPatternTree()`。

参见[TestParseTreeMatch.java](https://github.com/antlr/antlr4/blob/master/tool-testsuite/test/org/antlr/v4/test/tool/TestParseTreeMatcher.java)。

## Using XPath to identify parse tree node sets

XPath路径是表示您要在解析树中选择的节点或子树的字符串。收集要处理的分析树的子集很有用。例如,您可能想知道所有赋值在方法中或初始化的所有变量声明中的位置。

路径是一系列节点名称,带有以下分隔符。

|表达式|说明|
| --------- | ----------- |
|nodename|具有令牌或规则名称nodename的节点
|/|根节点,但`/X`与`X`相同,因为您假定传递给xpath的树为根。因为看起来更好,所以以`/`(或下面的`//`)开始所有模式。|
| //|树中与路径中下一个元素匹配的所有节点。例如,`//ID`查找树中的所有`ID`令牌节点。|
|!|除路径中的下一个元素外的任何节点。例如,`/classdef/!field`应该找到不是`field`子树的`classdef`根节点的所有子节点。|

例子:

```
/prog/func, -> all funcs under prog at root
/prog/*, -> all children of prog at root
/*/func, -> all func kids of any root node
prog, -> prog must be root node
/prog, -> prog must be root node
/*, -> any root
*, -> any root
//ID, -> any ID in tree
//expr/primary/ID, -> any ID child of a primary under any expr
//body//ID, -> any ID under a body
//'return', -> any 'return' literal in tree
//primary/*, -> all kids of any primary
//func/*/stat, -> all stat nodes grandkids of any func node
/prog/func/'def', -> all def literal kids of func kid of prog
//stat/';', -> all ';' under any stat node
//expr/primary/!ID, -> anything but ID under primary under any expr node
//expr/!primary, -> anything but primary under any expr node
//!*, -> nothing anywhere
/!*, -> nothing at root
```

给定一个解析树,访问这些节点的典型机制是以下循环:

```java
for (ParseTree t : XPath.findAll(tree, xpath, parser) ) {
    ... process t ...
}
```

例如,这是一个通用公式,用于列出与由路径规范标识的每个节点关联的文本:

```java
List<String> nodes = new ArrayList<String>();
for (ParseTree t : XPath.findAll(tree, xpath, parser) ) {
    if ( t instanceof RuleContext) {
        RuleContext r = (RuleContext)t;
        nodes.add(parser.getRuleNames()[r.getRuleIndex()]);    }      
    else { 
        TerminalNode token = (TerminalNode)t;
        nodes.add(token.getText());
    }      
}
```

## Combining XPath and tree pattern matching

自然地,您可以结合使用XPath来找到一组根节点,然后使用树模式匹配来识别这些根节点的特定子集并提取组件节点。

```java
//assume we are parsing Java
ParserRuleContext tree = parser.compilationUnit();
String xpath = "//blockStatement/*"; //get children of blockStatement
String treePattern = "int <Identifier> = <expression>;";
ParseTreePattern p =
    parser.compileParseTreePattern(treePattern,   
        ExprParser.RULE_localVariableDeclarationStatement);
List<ParseTreeMatch> matches = p.findAll(tree, xpath);
System.out.println(matches);
```

参见[TestXPath.java](https://github.com/antlr/antlr4/blob/master/tool-testsuite/test/org/antlr/v4/test/tool/TestXPath.java)。