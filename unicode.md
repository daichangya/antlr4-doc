# Lexers and Unicode text

在ANTLR 4.7之前,大多数目标中生成的词法分析器仅支持Unicode标准的一部分(代码点最多为`U+FFFF`)。从ANTLR 4.7开始,所有语言运行时中的词法分析器都支持最多`U+10FFFF`的所有Unicode代码点。

C ++,Python,Go和Swift API不需要更改任何API即可支持Unicode代码点,因此我们决定将这些类接口保持原样。 

Java,C#和JavaScript运行时需要进行更改,因此我们不建议使用它们,而不是破坏以前的接口。(* Java-target *不推荐使用的`ANTLRInputStream`和`ANTLRFileStream` API仅支持Unicode代码点,最高支持`U+FFFF`。)现在,这些目标必须使用#从输入创建`CharStream` s。 #7 ##,`CharStreams.fromFileName()`等...

向本·汉密尔顿(github bhamiltoncx)大声喊叫他的超人
跨所有目标的努力,以获得对U + 10FFFF代码点的真正支持。

## Example

Java,C#和JavaScript运行时使用新的工厂样式流创建接口。例如,下面是一些使用`CharStreams.fromPath()`的示例Java代码:

```java
public static void main(String[] args) {
  CharStream charStream = CharStreams.fromPath(Paths.get(args[0]));
  Lexer lexer = new UnicodeLexer(charStream);
  CommonTokenStream tokens = new CommonTokenStream(lexer);
  tokens.fill();
  for (Token token : tokens.getTokens()) {
    System.out.println("Got token: " + token.toString());
  }
}
```

# Unicode Code Points in Lexer Grammars

引用Unicode [代码点](https://en.wikipedia.org/wiki/Code_point)
在词法分析器语法中,使用`\u`字符串转义符以及最多4个十六进制数字。例如创建
通过创建一个范围从一个西里尔字母的词法则规则
`U+0400`至`U+04FF`:

```ANTLR
CYRILLIC : '\u0400'..'\u04FF' ; //or [\u0400-\u04FF] without quotes
```

大于U + FFFF的Unicode文字必须使用扩展的`\u{12345}`语法。例如,为选择笑脸创建词法分析器规则
摘自[Emoticons Unicode块](http://www.unicode.org/charts/PDF/U1F600.pdf):

```ANTLR
EMOTICONS : ('\u{1F600}' | '\u{1F602}' | '\u{1F615}') ; //or [\u{1F600}\u{1F602}\u{1F615}]
```

最后,词法分析器字符集可以包含Unicode属性。每个Unicode代码点至少具有一个描述其所属类型组的属性(例如,字母,数字,标点符号)。其他属性可以是语言脚本或特殊的二进制属性和Unicode代码块。但是,这意味着,属性指定一组代码点,因此仅在lexer字符集中允许使用它们。

```ANTLR
EMOJI : [\p{Emoji}] ;
JAPANESE : [\p{Script=Hiragana}\p{Script=Katakana}\p{Script=Han}] ;
NOT_CYRILLIC : [\P{Script=Cyrillic}] ;
```

有关Unicode的更多详细信息,请参见[lexer-rules.md](lexer-rules.md#lexer-rule-elements)
在词法分析器规则中转义。

## Migration


** 4.6 **的代码如下所示:


```java
CharStream input = new ANTLRFileStream("myinputfile");
JavaLexer lexer = new JavaLexer(input);
CommonTokenStream tokens = new CommonTokenStream(lexer);
```

(尽管文档先前曾说过,但默认情况下它并未使用UTF-8;它实际上取决于默认的调用环境。)

** 4.7 **的代码默认情况下采用UTF-8,如下所示:

```java
CharStream input = CharStreams.fromFileName("inputfile");
JavaLexer lexer = new JavaLexer(input);
CommonTokenStream tokens = new CommonTokenStream(lexer);
```

或者,如果您想指定文件编码:

```java
CharStream input = CharStreams.fromFileName("inputfile", Charset.forName("windows-1252"));
```

###动机

在[热烈讨论](https://github.com/antlr/antlr4/pull/1771)之后,我(parrt)决定不简单地将4.6 `ANTLRFileStream`和`ANTLRInputStream`融合到新版本中U + 10FFFF功能。我决定*弃用*旧界面,并建议使用新界面以免造成混淆。我的推理总结为:

*我不喜欢破坏所有4.6代码的想法。为了使先前的流正确支持> 16位Unicode,将需要对方法签名进行大量更改。
*使用`int`缓冲区元素类型会使在内存中保存流所需的内存大小加倍,因为我们缓冲了所有内容(并且我不想更改流的这一方面)。
*新的工厂样式接口支持根据输入流中找到的Unicode代码点创建最小的代码点缓冲区元素。这意味着使用一半的内存
就像旧的{@link ANTLRFileStream}(假定16位字符)用于ASCII文本。
*通过一些[认真的测试和性能调整](https://github.com/antlr/antlr4/pull/1781),新流的运行速度比4.6流更快或更快。

**警告**。*您应该避免在同一应用程序中同时使用已弃用的流和新的流*,因为您会看到
非凡的性能下降。这种速度的打击是因为
`Lexer`的内部代码从单态变为大态
动态分配以从输入流中获取字符。Java的
动态编译器(JIT)无法执行相同的优化
因此,如果性能出色,请坚持使用旧的或新的流
一个主要的问题。请参阅在我们的计时装置中识别此问题所需的[极端调试和说明](https://github.com/antlr/antlr4/pull/1781)。

###使用替代代码单元的旧语法

自己进行过UTF-16代理代码单元匹配的旧语法将需要继续使用`ANTLRInputStream`(Java目标),直到解析器应用程序代码可以升级到`CharStreams`接口为止。然后,应该从语法中删除替代代码单元匹配,以便让新流进行解码。  

在4.7之前的版本中,应用程序代码可以直接将`Token.getStartIndex()`和`Token.getStopIndex()`传递给Java和C#String API(因为两者都使用UTF-16代码单元作为长度的基本单位)。使用新的流,客户将不得不从代码点索引转换为UTF-16代码单元索引。这是一些(Java)代码,向您展示必要的逻辑:

```java
public final class CodePointCounter {
  private final String input;
  public int inputIndex = 0;
  public int codePointIndex = 0;
  
  public int advanceToIndex(int newCodePointIndex) {
    assert newCodePointIndex >= codePointIndex;
    while (codePointIndex < newCodePointOffset) {
        int codePoint = Character.codePointAt(input, inputIndex);
        inputIndex += Character.charCount(codePoint);
        codePointIndex++;
    }
    return inputIndex;
  }
}
```

###字符缓冲,未缓冲的流

创建时,ANTLR字符流仍会缓冲所有输入
就像他们过去20年来所做的一样。 

如果您需要无缓冲
访问,请注意,创建它变得充满挑战
解析树。解析树必须指向令牌,
指向未缓冲流中的陈旧位置,否则您必须复制
字符从缓冲区移到令牌中。达不到目的
无缓冲输入。请参阅[ANTLR 4书](https://www.amazon.com/Definitive-ANTLR-4-Reference/dp/1934356999)“ 13.8未缓冲的字符和令牌流”。未缓冲的流主要是
在解析期间用于处理无限流非常有用,并且需要您手动缓冲字符。使用`UnbufferedCharStream`和`UnbufferedTokenStream`。

```java
CharStream input = new UnbufferedCharStream(is);
CSVLexer lex = new CSVLexer(input); //copy text out of sliding buffer and store in tokens
lex.setTokenFactory(new CommonTokenFactory(true));
TokenStream tokens = new UnbufferedTokenStream<CommonToken>(lex);
CSVParser parser = new CSVParser(tokens);
parser.setBuildParseTree(false);
parser.file();
```

您的语法需要具有嵌入式操作,这些操作需要在令牌创建时访问令牌,但要在令牌消失并进行垃圾回收之前对其进行访问。例如,

```
data : a=INT {int x = Integer.parseInt($a.text);} ;
```

从`CommonTokenFactory`的代码注释中:

> `new CommonTokenFactory(true)`中的`true`表示是否应在之后调用`CommonToken.setText` 
构造标记以显式设置文本。这在情况下很有用
输入流可能无法提供任意子字符串的地方
词法分析器创建标记后(例如,
在`CharStream.getText`中的实现
`UnbufferedCharStream`抛出一个
`UnsupportedOperationException`)。明确设置令牌文本
允许随时调用`Token.getText`,无论
输入流实现。

*目前,只有Java,C ++和C#实施了这些未缓冲的流*。