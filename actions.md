# Actions and Attributes

在第10章中,属性和动作,我们学习了如何在语法中嵌入动作,并研究了最常见的标记和规则属性。本节总结了该章中的重要语法和语义,并提供了所有可用属性的完整列表。(您可以从有关听众和动作的免费摘录中了解有关语法中动作的更多信息。)

动作是用目标语言编写的文本块,并用大括号括起来。识别器根据它们在语法中的位置触发它们。例如,以下规则在解析器看到有效声明后发出“ found a decl”:

```
decl: type ID ';' {System.out.println("found a decl");} ;
type: 'int' | 'float' ;
```

最常见的是,操作访问令牌和规则引用的属性:

```
decl: type ID ';'
      {System.out.println("var "+$ID.text+":"+$type.text+";");}
    | t=ID id=ID ';'
      {System.out.println("var "+$id.text+":"+$t.text+";");}
    ;
```

## Token Attributes

所有令牌都有一组预定义的只读属性。这些属性包括有用的令牌属性,例如令牌类型和与令牌匹配的文本。动作可以通过`$label.attribute`访问这些属性,其中标签标记了令牌引用的特定实例(以下示例中的`a`和`b`在动作代码中用作`$a`和`$b`)。通常,一个特定的令牌在规则中仅被引用一次,在这种情况下,令牌名称本身可以在操作代码中明确使用(令牌`INT`可以在操作中用作`$INT`)。以下示例说明了令牌属性表达式的语法:

```
r : INT {int x = $INT.line;}
    ( ID {if ($INT.line == $ID.line) ...;} )?
    a=FLOAT b=FLOAT {if ($a.line == $b.line) ...;}
  ;
```

`(...)?`子规则中的操作可以在外部级别看到匹配的`INT`令牌。

因为有两个对`FLOAT`令牌的引用,所以在动作中对`$FLOAT`的引用不是唯一的。您必须使用标签来指定您感兴趣的令牌引用。

不同替代方案中的令牌引用是唯一的,因为任何规则调用都只能匹配其中一个。例如,在以下规则中,两种选择中的操作都可以直接引用`$ID`,而无需使用标签:

```
 	r : ... ID {System.out.println($ID.text);}
 	| ... ID {System.out.println($ID.text);}
 	;
```

要访问与文字匹配的标记,必须使用标签:

```
 	stat: r='return' expr ';' {System.out.println("line="+$r.line);} ;
```

大多数情况下,您访问令牌的属性,但是有时访问令牌对象本身很有用,因为它会聚合所有属性。此外,您可以使用它来测试可选子规则是否与令牌匹配:

```
 	stat: 'if' expr 'then' stat (el='else' stat)?
 	{if ( $el!=null ) System.out.println("found an else");}
 	| ...
 	;
```

`$T`和`$L`对令牌名称`T`和令牌标签`L`的`Token`对象求值。列表标签`ll`的`$ll`计算结果为`List<Token>`。`$T.attr`计算下表中为属性`attr`指定的类型和值:


|属性|类型|描述|
| --------- | ---- | ----------- |
|text|String|与令牌匹配的文本;转换为对getText的调用。例如:$ ID.text.|
|type|int|令牌的令牌类型(非零正整数),例如INT;转换为对getType的调用。示例:$ ID.type.|
|line|int|令牌出现的行号,从1开始计数;转换为对getLine的调用。示例:$ ID.line.|
|pos|int|令牌的第一个字符出现的行中的字符位置从零开始计数;转换为对togetCharPositionInLine的调用。示例:$ ID.pos.|
|index|int|该令牌在令牌流中的总索引,从零开始计数;转换为对getTokenIndex的调用。示例:$ ID.index.|
|channel|int|令牌的通道号。解析器仅调整到一个通道,有效地忽略了通道外令牌。默认通道为0(Token.DEFAULT_CHANNEL),默认隐藏通道为Token.HIDDEN_CHANNEL。转换为对getChannel的调用。示例:$ ID.channel.|
|int|int|此令牌保存的文本的整数值;它假定文本是有效的数字字符串。方便构建计算器等。转换为Integer.valueOf(令牌文本)。示例:$ INT.int.|

## Parser Rule Attributes

ANTLR预定义了与可用于操作的解析器规则引用关联的许多只读属性。操作只能访问操作之前的引用的规则属性。规则名称为`r`或分配给规则引用的标签的语法为`$r.attr`。例如,`$expr.text`返回与规则`expr`的先前调用匹配的完整文本:

```
returnStat : 'return' expr {System.out.println("matched "+$expr.text);} ;
```

使用规则标签如下所示:

```
returnStat : 'return' e=expr {System.out.println("matched "+e.text);} ;
```

您还可以使用`$ followed by the name of the attribute to access the value associated with the currently executing rule. For example, ` $ start`是当前规则的起始标记。

```
returnStat : 'return' expr {System.out.println("first token "+$start.getText());} ;
```

`$r`和`$rl`的规则名称`r`和规则标签`rl`的类型为`RContext`的`ParserRuleContext`个对象。规则列表标签`rll`的`$rll`计算结果为`List<RContext>`。`$r.attr`计算得出下表中为属性`attr`指定的类型和值:

|属性|类型|描述|
| --------- | ---- | ----------- |
|text|String|与规则匹配的文本,或者从规则开始到`$text`表达式求值为止的文本。请注意,这包括所有标记的文本,包括隐藏通道上的标记,这是您想要的,因为通常具有所有的空白和注释。当引用当前规则时,此属性在任何操作(包括任何异常操作)中均可用。|
|start|Token|第一个可能与主令牌通道上的规则匹配的令牌;换句话说,此属性永远不会是隐藏的标记。对于最终没有匹配令牌的规则,此属性指向该规则可能已匹配的第一个令牌。引用当前规则时,此属性可用于规则内的任何操作。|
|stop|Token|规则要匹配的最后一个非隐藏通道令牌。引用当前规则时,此属性仅对after和finally操作可用。|
|ctx|ParserRuleContext|与规则调用关联的规则上下文对象。通过该属性可以使用所有其他属性。例如,`$ctx.start`访问当前规则上下文对象内的开始字段。与`$start`。|相同

## Dynamically-Scoped Attributes

您可以使用参数和返回值在规则之间传递信息,就像通用编程语言中的函数一样。但是,编程语言不允许函数访问局部变量或调用函数的参数。例如,以下对嵌套方法调用中的局部变量`x`的引用在Java中是非法的:

```java
void f() {
	int x = 0;
	g();
}
void g() {
	h();
}
void h() {
	int y = x; //INVALID reference to f's local variable x
}
```

变量`x`仅在`f`范围内可用,该范围是用大括号按词法分隔的文本。因此,据说Java使用词法作用域。词法作用域是大多数编程语言的规范。那些允许调用链中更下游的方法访问之前定义的局部变量的语言被称为使用动态作用域。术语动态是指编译器无法静态确定可见变量集的事实。这是因为方法可见的变量集会根据谁调用该方法而发生变化。

事实证明,在语法领域中,遥远的规则有时需要彼此通信,主要是为了向规则调用链中下方匹配的规则提供上下文信息。(自然,这是假定您直接在语法中使用动作而不是解析树侦听器事件机制。)ANTLR允许动态范围界定,因为动作可以使用语法`$r::x`访问其中调用规则的属性,其中## 59# #是规则名称,而`x`是该规则内的属性。程序员应确保`r`实际上是当前规则的调用规则。如果在访问`$r::x`时当前调用链中没有`r`,则会发生运行时异常。

为了说明动态作用域的用法,请考虑定义变量并确保定义表达式中的变量的实际问题。以下语法定义了符号属性在块规则中的位置,但在规则`decl`中为其添加了变量名称。然后,规则`stat`查阅列表以查看是否已定义变量。

```
grammar DynScope;
 
prog: block ;
 
block
	/* List of symbols defined within this block */
	locals [
	List<String> symbols = new ArrayList<String>()
	]
	: '{' decl* stat+ '}'
	//print out all symbols found in block
	//$block::symbols evaluates to a List as defined in scope
	{System.out.println("symbols="+$symbols);}
	;
 
/** Match a declaration and add identifier name to list of symbols */
decl: 'int' ID {$block::symbols.add($ID.text);} ';' ;
 
/** Match an assignment then test list of symbols to verify
 * that it contains the variable on the left side of the assignment.
 * Method contains() is List.contains() because $block::symbols
 * is a List.
 */
stat: ID '=' INT ';'
	{
	if ( !$block::symbols.contains($ID.text) ) {
	System.err.println("undefined variable: "+$ID.text);
	}
	}
	| block
	;
 
ID : [a-z]+ ;
INT : [0-9]+ ;
WS : [ \t\r\n]+ -> skip ;
```

这是一个简单的构建和测试序列:

```bash
$ antlr4 DynScope.g4
$ javac DynScope*.java
$ grun DynScope prog
=> 	{
=> 	int i;
=> 	i = 0;
=> 	j = 3;
=> 	}
=> 	EOF
<= 	undefined variable: j
 	symbols=[i]
```

`@members`操作中的简单字段声明与动态作用域之间存在重要区别。symbol是一个局部变量,因此每次调用规则`block`都有一个副本。这正是嵌套块所需要的,以便我们可以在内部块中重用相同的输入变量名称。例如,以下嵌套代码块在内部范围中重新定义了`i`。此新定义必须将定义隐藏在外部范围内。

```
{
	int i;
	int j;
	i = 0;
	{
		int i;
		int x;
		x = 5;
	}
	x = 3;
}
```

这是DynScope为该输入生成的输出:

```bash
$ grun DynScope prog nested-input
symbols=[i, x]
undefined variable: x
symbols=[i, j]
```

引用`$block::symbols`访问最近调用的`block`规则上下文对象的`symbols`字段。如果您需要从更远的调用链的规则调用中访问符号实例,则可以从当前上下文`$ctx`开始向后走。使用`getParent`上链。