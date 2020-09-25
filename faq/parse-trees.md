# Parse Trees

## How do I get the input text for a parse-tree subtree?

在ParseTree中,您具有以下方法:

```java
/** Return the combined text of all leaf nodes. Does not get any
 * off-channel tokens (if any) so won't return whitespace and
 * comments if they are sent to parser on hidden channel.
 */
String getText();
```

但是,您可能需要TokenStream中的此方法:

```java
/**
 * Return the text of all tokens in the source interval of the specified
 * context. This method behaves like the following code, including potential
 * exceptions from the call to {@link #getText(Interval)}, but may be
 * optimized by the specific implementation.
 *
 * <p>If {@code ctx.getSourceInterval()} does not return a valid interval of
 * tokens provided by this stream, the behavior is unspecified.</p>
 *
 * <pre>
 * TokenStream stream = ...;
 * String text = stream.getText(ctx.getSourceInterval());
 * </pre>
 *
 * @param ctx The context providing the source interval of tokens to get
 * text for.
 * @return The text of all tokens within the source interval of {@code ctx}.
 */
public String getText(RuleContext ctx);
```

也就是说,执行以下操作:

```
mytokens.getText(mySubTree);
```

## What if I need ASTs not parse trees for a compiler, for example?

对于编写编译器,可以生成[LLVM类型的静态单分配](http://llvm.org/docs/LangRef.html)表单,也可以使用侦听器或访问者从解析树构造AST。或者,在语法中使用动作,关闭自动分析树的构建。

## When do I use listener/visitor vs XPath vs Tree pattern matching?

### XPath

当您需要在特定上下文中查找特定节点时,XPath效果很好。上下文仅限于到树的根的父级。例如,如果要查找所有ID节点,请使用路径`//ID`。如果需要所有变量声明,则可以使用路径`//vardecl`。如果只需要字段声明,则可以通过路径`/classdef/vardecl`使用一些上下文信息,该路径只会找到我们的类定义子级的vardecls。您可以合并多个XPath `findAll()` s的结果,以模拟XPath的集合并集。唯一的警告是,当您合并多个`findAll()`集时,不会保留原始树的顺序。

###树型匹配

当您要查找特定的子树结构(例如,使用模式`x = 0;`分配给0的所有分配)时,请使用树模式匹配。(回想起来,这非常方便,因为您可以使用语法描述的语言的具体语法来指定树结构。)如果要查找所有类型的所有赋值,则可以使用模式`x = <expr>;`,其中`<expr>`将找到任何表达式。这对于匹配特定的子结构非常有用,因此为您提供了更多指定上下文的能力。即,您不仅可以找到所有标识符,还可以在表达式的左侧找到所有标识符。

###听众/访问者

使用侦听器或访客接口可以为您提供最大的功能,但需要实现更多方法。发现一个侦听器的紧急行为可能比一个简单的树模式匹配器更具挑战性,后者说“在节点Y *下找到我X”。

当您想访问树中的许多节点时,侦听器非常有用。

侦听器使您可以计算和保存在各个节点进行处理所必需的上下文信息。例如,在为编译器或翻译器构建符号表管理器时,您需要计算符号范围,例如全局变量,类,函数和代码块。输入类或函数时,可以推入新的作用域,然后在退出该类或函数时将其弹出。看到符号时,需要对其进行定义或在适当的范围内查找它。通过具有Enter/exit侦听器函数推入和弹出作用域,用于定义变量的侦听器函数可以简单地表示以下内容:

```java
scopeStack.peek().define(new VariableSymbol("foo"))
```

这样,每个侦听器功能都不必计算其适当的范围。

示例:[DefScopesAndSymbols.java](https://github.com/mantra/compiler/blob/master/src/java/mantra/semantics/DefScopesAndSymbols.java)和[SetScopeListener.java](https://github.com /mantra/compiler/blob/master/src/java/mantra/semantics/SetScopeListener.java)和[VerifyListener.java](https://github.com/mantra/compiler/blob/master/src/java/mantra/语义/VerifyListener.java)