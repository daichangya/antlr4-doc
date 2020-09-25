# Parser Rules

解析器由解析器或组合语法中的一组解析器规则组成。Java应用程序通过调用由ANTLR生成的与所需的启动规则关联的规则函数来启动解析器。最基本的规则只是规则名称,后跟一个以分号结尾的替代名称:

```
 	/** Javadoc comment can precede rule */
 	retstat : 'return' expr ';' ;
```

规则也可以用|分隔替代项 

```
operator:
 	stat: retstat
 	| 'break' ';'
 	| 'continue' ';'
 	;
```

备选方案是规则元素列表或为空。例如,这是一个规则,其中有一个空的替代项,使整个规则可选:

```
superClass
 	: 'extends' ID
 	| //empty means other alternative(s) are optional
 	;
```

## Alternative Labels

如我们在第7.4节“为精确事件方法标记规则替代项”中所看到的,通过使用#运算符标记规则的最外层替代项,我们可以获得更精确的解析树侦听器事件。规则中的所有替代品都必须加标签,或者没有标签。这是两个带有替代标记的规则。

```
grammar T;
stat: 'return' e ';' # Return
 	| 'break' ';' # Break
 	;
e   : e '*' e # Mult
    | e '+' e # Add
    | INT # Int
    ;
```

替代标签不必在行的末尾,并且#符号后不必有空格。
ANTLR为每个标签生成一个规则上下文类定义。例如,这是ANTLR生成的侦听器:

```java
public interface AListener extends ParseTreeListener {
 	void enterReturn(AParser.ReturnContext ctx);
 	void exitReturn(AParser.ReturnContext ctx);
 	void enterBreak(AParser.BreakContext ctx);
 	void exitBreak(AParser.BreakContext ctx);
 	void enterMult(AParser.MultContext ctx);
 	void exitMult(AParser.MultContext ctx);
 	void enterAdd(AParser.AddContext ctx);
 	void exitAdd(AParser.AddContext ctx);
 	void enterInt(AParser.IntContext ctx);
 	void exitInt(AParser.IntContext ctx);
}
```

存在与每个标记的替代方法关联的进入和退出方法。这些方法的参数特定于替代方法。

您可以在多个替代方案上重复使用相同的标签,以指示解析树walker应该为这些替代方案触发相同的事件。例如,这是上面语法A对规则e的一种变体:

```
 	e : e '*' e # BinaryOp
 	| e '+' e # BinaryOp
 	| INT # Int
 	;
```

ANTLR将为e生成以下侦听器方法:

```java
 	void enterBinaryOp(AParser.BinaryOpContext ctx);
 	void exitBinaryOp(AParser.BinaryOpContext ctx);
 	void enterInt(AParser.IntContext ctx);
 	void exitInt(AParser.IntContext ctx);
 ```

如果备用名称与规则名称冲突,则ANTLR会给出错误。这是规则e的另一种重写,其中两个
替代标签与规则名称冲突:

```
 	e : e '*' e # e
 	| e '+' e # Stat
 	| INT # Int
 	;
```

从规则名称和标签生成的上下文对象被大写,因此标签Stat与规则stat冲突:

```bash
 	$ antlr4 A.g4
 	error(124): A.g4:5:23: rule alt label e conflicts with rule e
 	error(124): A.g4:6:23: rule alt label Stat conflicts with rule stat
 	warning(125): A.g4:2:13: implicit definition of token INT in parser
```

## Rule Context Objects

ANTLR生成用于访问与每个规则引用关联的规则上下文对象(解析树节点)的方法。对于具有单个规则引用的规则,ANTLR生成不带参数的方法。考虑以下规则。

```
 	inc : e '++' ;
```

ANTLR生成此上下文类:

```java
public static class IncContext extends ParserRuleContext {
 	public EContext e() { ... } //return context object associated with e
 	...
}
```

当对规则的引用不止一个时,ANTLR还提供对访问上下文对象的支持:

```
field : e '.' e ;
```

ANTLR生成带有索引以访问第ith个元素的方法,以及获取对该规则的所有引用的上下文的方法:

```java
public static class FieldContext extends ParserRuleContext {
 	public EContext e(int i) { ... } //get ith e context
 	public List<EContext> e() { ... } //return ALL e contexts
 	...
}
```

如果我们有另一个规则s引用该字段,则嵌入式操作可以访问该字段执行的e条规则匹配的列表:

```
s : field
 	{
 	List<EContext> x = $field.ctx.e();
 	...
 	}
;
```

听众或访客可以做同样的事情。给定指向FieldContext对象的指针,f,fe()将返回List <EContext>。

## Rule Element Labels

您可以使用=运算符标记规则元素,以将字段添加到规则上下文对象中:

```
stat: 'return' value=e ';' # Return
 	| 'break' ';' # Break
 	;
```

这里的value是规则e返回值的标签,它在其他地方定义。
标签成为适当的解析树节点类中的字段。在这种情况下,由于Return替代标签,标签值成为ReturnContext中的一个字段:

```java
public static class ReturnContext extends StatContext {
 	public EContext value;
 	...
}
```

跟踪许多令牌通常很方便,您可以使用+ =“列表标签”来完成此任务。操作员。例如,以下规则创建与简单数组构造匹配的Token对象的列表:

```
 	array : '{' el+=INT (',' el+=INT)* '}' ;
```

ANTLR在适当的规则上下文类中生成一个List字段:

```
 	public static class ArrayContext extends ParserRuleContext {
 	public List<Token> el = new ArrayList<Token>();
 	...
 	}
```

这些列表标签也可用于规则引用:

```
 	elist : exprs+=e (',' exprs+=e)* ;
```

ANTLR生成一个包含上下文对象列表的字段:

```
 	public static class ElistContext extends ParserRuleContext {
 	public List<EContext> exprs = new ArrayList<EContext>();
 	...
 	}
```

## Rule Elements

规则元素指定解析器在给定时刻应该执行的操作,就像编程语言中的语句一样。元素可以是规则,令牌,字符串文字(例如表达式,ID和“返回”)。这是规则元素的完整列表(稍后我们将详细介绍操作和谓词):

<表格>
<tr>
<th>语法</th> <th>说明</th>
</tr>
<tr>
<td> T </td> <td>
在当前输入位置匹配令牌T。令牌始终以大写字母开头。</td>
</tr>
<tr>
<td>â€™literalâ€™</td> <td>
在当前输入位置匹配字符串文字。字符串文字只是具有固定字符串的标记。</td>
</tr>
<tr>
<td> r </td> <td>
在当前输入位置匹配规则r,相当于像调用函数一样调用规则。解析器规则名称始终以小写字母开头。</td>
</tr>
<tr>
<td> r [Â«argsÂ»] </td> <td>
在当前输入位置匹配规则r,像函数调用一样传入参数列表。方括号内的参数使用目标语言的语法,并且通常是逗号分隔的表达式列表。</td>
</tr>
<tr>
<td> {Â«actionÂ»} </td> <td>
在前一个替代元素之后和后一个替代元素之前立即执行操作。该动作符合目标语言的语法。ANTLR将动作代码逐字复制到生成的类中,除了替换属性和令牌引用(例如$ x和$ xy </td>)
</tr>
<tr>
<td> {Â«pÂ»}?</td> <td>
评估语义谓词“ p”。如果«p»在运行时评估为false,请不要继续解析超过谓词。在预测过程中遇到的谓词,当ANTLR区分替代项时,启用或禁用围绕谓词的替代项。</td>
</tr>
<tr>
<td>。</td> <td>
匹配文件令牌末尾以外的任何单个令牌。“点” 运算符称为通配符。</td>
</tr>
</table>

当您要匹配除特定令牌或一组令牌以外的所有内容时,请使用`~`“ not”?操作员。该运算符很少在解析器中使用,但可用。`~INT`匹配除`INT`令牌以外的任何令牌。`~’,’`匹配除逗号以外的任何令牌。`~(INT|ID)`与除INT或ID之外的任何令牌匹配。

令牌,字符串文字和语义谓词规则元素可以采用选项。请参阅规则元素选项。

## Subrules

规则可以包含称为子规则的替代块(如扩展BNF表示法EBNF所允许)。子规则就像缺少名称的规则,并用括号括起来。子规则在括号内可以有一个或多个替代项。子规则不能使用局部变量定义属性,而返回规则则可以。有四种子规则(x,y和z代表语法片段):

<表格>
<tr>
<th>语法</th> <th>说明</th>
</tr>
<tr>
<td> <img src = images/xyz.png> </td> <td>(x|y|z)。
将子规则中的任何替代项精确匹配一次。例:
<br>
<tt>
returnType:(类型|'void');
</tt>
</td>
</tr>
<tr>
<td> <img src = images/xyz_opt.png> </td> <td>(x|y|z)?
在子规则内不匹配任何内容或任何其他选项。例:
<br>
<tt> 
classDeclaration
    :“类” ID(typeParameters)?(“扩展”类型)?
      (“实现” typeList)?
     classBody
    ;
</tt>
<tr>
<td> <img src = images/xyz_star.png> </td> <td>(x|y|z)*
在子规则内匹配替代项零次或多次。例:
<br>
<tt>
注解名称:ID('。'ID)*;
</tt>
</tr>
<tr> 
<td> <img src = images/xyz_plus.png> </td> <td>(x|y|z)+
在子规则内匹配替代项一次或多次。例:
<br>
<tt>
注释:(annotation)+;
</tt>
</td>
</tr>
</table>

您可以在`?`,`*`和`+`子规则运算符后加上nongreedy运算符,这也是一个问号:`??`,`*?`和# #36 ##。请参见第15.6节“通配符运算符和非贪婪子规则”。

简而言之,您可以省略由具有单个规则元素引用的单个替代项组成的子规则的括号。例如,`annotation+`与`(annotation)+`相同,而`ID+`与`(ID)+`相同。标签也可用于速记。`ids+=INT+`列出`INT`令牌对象。

## Catching Exceptions

当规则中发生语法错误时,ANTLR会捕获异常,报告错误,尝试进行恢复(可能通过消耗更多标记),然后从规则中返回。每个规则都包含在`try/catch/finally`语句中:

```
void r() throws RecognitionException {
 	try {
 		rule-body
 	}
 	catch (RecognitionException re) {
	 	_errHandler.reportError(this, re);
	 	_errHandler.recover(this, re);
 	}
 	finally {
		exitRule();
 	}
}
```

在第9.5节“更改ANTLR的错误处理策略”中,我们看到了如何使用策略对象来更改ANTLR的错误处理。但是,替换策略会更改所有规则的策略。要更改单个规则的异常处理,请在规则定义后指定一个异常:

```
r : ...
  ;
  catch[RecognitionException e] { throw e; }
```

该示例说明了如何避免默认错误报告和恢复。r抛出异常,当高层规则报告错误更有意义时,此异常很有用。指定任何异常子句可防止ANTLR生成子句来处理`RecognitionException`。

您还可以指定其他例外:

```
r : ...
  ;
  catch[FailedPredicateException fpe] { ... }
  catch[RecognitionException e] { ... }
```

花括号内的代码段和“参数”异常 动作必须以目标语言编写;Java,在这种情况下。
当即使发生异常也需要执行操作时,请将其放入`finally`子句中:

```
r : ...
  ;
  //catch blocks go first
  finally { System.out.println("exit rule r"); }
```

finally子句在规则返回之前触发`exitRule`之前立即执行。如果要在规则与替代项完全匹配之后但在其清除工作之前执行操作,请使用`after`操作。

这是完整的例外列表:

<表格>
<tr>
<th>异常名称</th> <th>描述</th>
</tr>
<tr>
<td> RecognitionException </td> <td>
ANTLR生成的识别器抛出的所有异常的超类。它是RuntimeException的子类,可以避免检查异常的麻烦。此异常记录了识别器(词法分析器或解析器)在输入中的位置,在ATN(代表语法的内部图形数据结构)中的位置,规则调用堆栈以及发生了什么类型的问题。</td>
</tr>
<tr>
<td> NoViableAltException </td> <td>
指示解析器无法通过查看其余输入来决定采用两个或更多路径中的哪一条。此异常跟踪有问题的输入的起始标记,并且还知道发生错误时解析器在各个路径中的位置。</td>
</tr>
<tr>
<td> LexerNoViableAltException </td> <td>
等效于NoViableAltException,但仅适用于词法分析器。</td>
</tr>
<tr>
<td> InputMismatchException </td> <td>
当前输入的令牌与解析器预期的不匹配。</td>
</tr>
<tr>
<td> FailedPredicateException </td> <td>
在预测期间评估为假的语义谓词使周围的选择不可行。当规则预测采用哪种替代方法时,就会进行预测。如果所有可行路径均消失,则解析器将抛出NoViableAltException。在匹配标记和调用规则的正常解析过程中,当语义谓词在预测之外评估为false时,该谓词就会被解析器抛出。</td>
</tr>
</table>

## Rule Attribute Definitions

有许多与动作相关的语法元素与规则相关,需要注意。规则可以具有参数,返回值和局部变量,就像编程语言中的函数一样。(规则可以在规则元素中嵌入动作,如我们在15.4节,动作和属性中所看到的。)ANTLR收集您定义的所有变量,并将它们存储在规则上下文对象中。这些变量通常称为属性。这是显示所有可能的属性定义位置的常规语法:

```
rulename[args] returns [retvals] locals [localvars] : ... ;
```

在这些[...]中定义的属性可以像其他任何变量一样使用。这是将参数复制到返回值的示例规则:

```
//Return the argument plus the integer value of the INT token
add[int x] returns [int result] : '+=' INT {$result = $x + $INT.int;} ;
```

args,locals和return `[...]`通常使用目标语言,但有一些限制。`[...]`字符串是逗号分隔的声明列表,带有前缀或后缀类型表示法或无类型表示法。元素可以具有`[int x = 32, float y]`之类的初始值设定项,但不要太过疯狂,因为我们在[ScopeParser](https://github.com/antlr/antlr4/blob/master/tool/src/org/antlr/v4/parse/ScopeParser.java)。  

* Java,CSharp,C ++使用`int x`表示法,但是C ++必须对数组引用`int[] x`使用稍有改动的表示法,以适合* type * * id *语法。
* Go和Swift在变量名后给出类型,但是Swift在两者之间需要`:`。去`i int`,迅速`i:int`。对于Go目标,您必须使用`int i`或`i:int`。
* Python和JavaScript没有指定静态类型,因此动作只是标识符列表,例如`[i,j]`。

从技术上讲,任何目标都可以使用任何一种表示法。有关示例,请参见[TestScopeParsing](https://github.com/antlr/antlr4/blob/master/tool-testsuite/test/org/antlr/v4/test/tool/TestScopeParsing.java)。

与语法级别一样,您可以指定规则级别的命名操作。对于规则,有效名称为`init`和`after`。顾名思义,解析器会在尝试匹配关联规则之前立即执行初始化操作,并在匹配规则后立即执行操作。操作后的ANTLR不会作为生成的规则函数的final代码块的一部分执行。使用ANTLR最终动作将代码放置在生成的规则函数“最终代码”块中。

这些操作在任何参数,返回值或局部属性定义操作之后进行。第10.2节“访问令牌和规则属性”中的`row`规则序言很好地说明了该语法:
动作/CSV.g4

```
/** Derived from rule "row : field (',' field)* '\r'? '\n' ;" */
row[String[] columns]
   returns [Map<String,String> values]
   locals [int col=0]
	@init {
	$values = new HashMap<String,String>();
	}
	@after {
	if ($values!=null && $values.size()>0) {
	System.out.println("values = "+$values);
	}
	}
	: ...
	;
```

规则行接受参数列,返回值,并定义局部变量col。“行动” 方括号中的内容直接复制到生成的代码中:

```java
public class CSVParser extends Parser {
	...
	public static class RowContext extends ParserRuleContext {
		public String [] columns;
		public Map<String,String> values;
		public int col=0;
		...
	}
	...
}
```

生成的规则函数还将规则自变量指定为函数自变量,但是它们很快被复制到本地RowContext对象中:

```java
public class CSVParser extends Parser {
 	...
 	public final RowContext row(String [] columns) throws RecognitionException {
	 	RowContext _localctx = new RowContext(_ctx, 4, columns);
	 	enterRule(_localctx, RULE_row);
	 	...
 	}
 	...
}
```

ANTLR跟踪动作中嵌套的`[...]`,以便正确解析`String[]`列。它还跟踪尖括号,以使通用类型参数内的逗号不表示另一个属性的开始。`Map<String,String>`值是一种属性定义。

每个操作中可以有多个属性,即使对于返回值也是如此。使用逗号分隔同一操作中的属性:

```
a[Map<String,String> x, int y] : ... ;
```

ANTLR解释该动作以定义两个参数x和y:

```java
public final AContext a(Map<String,String> x, int y)
	throws RecognitionException
{
	AContext _localctx = new AContext(_ctx, 0, x, y);
	enterRule(_localctx, RULE_a);
	...
}
```

## Start Rules and EOF

起始规则是解析器首先使用的规则;它是语言应用程序调用的规则函数。例如,解析为Java代码的语言应用程序可能会为名为`parser`的`JavaParser`对象调用`parser.compilationUnit()`。语法中的任何规则都可以充当开始规则。

启动规则不一定会消耗所有输入。它们仅消耗与匹配规则替代所需的尽可能多的输入。例如,考虑以下规则,该规则根据输入匹配一个,两个或三个标记。

```
s : ID
  | ID '+'
  | ID '+' INT
  ;
```

在`a+3`上,规则`s`与第三个替代匹配。在`a+b`上,它与第二个替代项匹配,并忽略最终的`b`令牌。在`a b`上,它与第一个选择匹配,而忽略`b`令牌。在后两种情况下,解析器不会使用完整的输入,因为规则`s`并未明确指出必须在匹配规则的替代项后出现文件结尾。

此默认功能对于构建类似IDE的东西非常有用。想象一下IDE想要解析一个大Java文件中间某处的方法。调用规则`methodDeclaration`应该尝试仅匹配一个方法,而忽略接下来的内容。

另一方面,描述整个输入文件的规则应引用特殊的预定义令牌`EOF`。如果不这样做,您可能会想一会儿,想知道为什么启动规则不管输入内容如何都不会报告任何输入错误。这是一条规则,它是读取配置文件的语法的一部分:

```
config : element*; //can "match" even with invalid input.
```

无效的输入将导致`config`立即返回而不匹配任何输入,也不会报告错误。这是正确的规范:

```
file : element* EOF; //don't stop early. must match all input
```