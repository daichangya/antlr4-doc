# Parsing Binary Files

解析二进制文件与解析基于字符的文件没有什么不同,除了“字符”实际上是字节而不是16位无符号短Unicode字符。从词法分析器/解析器的角度来看,没有区别,只是字符可能无法打印。如果要匹配特殊的2字节标记0xCA,然后匹配0xFE,则以下规则就足够了。

```
MARKER : '\u00CA' '\u00FE' ;
```

解析器当然会像其他任何令牌一样引用该令牌。

这是与以下代码段一起使用的示例语法。

```
grammar IP;

file : ip+ (MARKER ip)* ;

ip : BYTE BYTE BYTE BYTE ;

MARKER : '\u00CA' '\u00FE' ;
BYTE : '\u0000'..'\u00FF' ;
```

请注意,`BYTE`使用范围运算符来匹配0到255之间的任何内容。我们自然不能使用像`[a-z]`这样的字符类,因为我们没有解析字符代码。所有字符说明符的最高字节必须为`00`。例如,`\uCAFE`不是有效字符,因为将永远不会从输入流中创建16位值(仅记住字节)。

如果在二进制文件中有像`$`或`!`这样的实际字符被编码为字节,则可以像通常一样通过`'$'`这样的文字来引用它们。参见语法中的`'.'`。
 
## Binary streams

现在有许多目标,所以我不确定它们如何处理文本文件,但是大多数目标会根据计算机的区域设置提取文本。在大多数情况下,这将意味着将文本的UTF-8编码转换为16位Unicode。ANTLR的词法分析器在`int`上运行,因此我们可以处理您想要发送的适合`int`的任何类型的字符。

一旦词法分析器获得了输入流,它就不在乎这些字符来自/表示字节还是实际的Unicode字符。

让我们获取一个名为`ips`的二进制文件,并将其放在资源目录中:

```java
public class WriteBinaryFile {
	public static final byte[] bytes = {
		(byte)172, 0, 0, 1, (byte)0xCA, (byte)0xFE,
		(byte)10, 10, 10, 1, (byte)0xCA, (byte)0xFE,
		(byte)10, 10, 10, 99
	};

	public static void main(String[] args) throws IOException {
		Files.write(new File("/tmp/ips").toPath(), bytes);
	}
}
```

现在我们需要创建一个ANTLR满意的字节流,它很简单:

```java
CharStream bytesAsChar = CharStreams.fromFileName("/tmp/ips", StandardCharsets.ISO_8859_1);
```

`ISO-8859-1`编码只是LATIN-1的8位char编码,它有效地告诉流将每个字节视为一个字符。那就是我们想要的。然后我们有通常的测试装备:


```java
//ANTLRFileStream bytesAsChar = new ANTLRFileStream("/tmp/ips", "ISO-8859-1"); DEPRECATED in 4.7
CharStream bytesAsChar = CharStreams.fromFileName("/tmp/ips", StandardCharsets.ISO_8859_1);
IPLexer lexer = new IPLexer(bytesAsChar);
CommonTokenStream tokens = new CommonTokenStream(lexer);
IPParser parser = new IPParser(tokens);
ParseTree tree = parser.file();
IPBaseListener listener = new MyIPListener();
ParseTreeWalker.DEFAULT.walk(listener, tree);
```

这是监听器:

```java
class MyIPListener extends IPBaseListener {
	@Override
	public void exitIp(IPParser.IpContext ctx) {
		List<TerminalNode> octets = ctx.BYTE();
		short[] ip = new short[4];
		for (int i = 0; i<octets.size(); i++) {
			String oneCharStringHoldingOctet = octets.get(i).getText();
			ip[i] = (short)oneCharStringHoldingOctet.charAt(0);
		}
		System.out.println(Arrays.toString(ip));
	}
}
```

我们不能只打印出文本,因为我们没有阅读文本。我们需要将每个字节作为十进制值发出。运行测试代码时,输出应为以下内容:

```
[172, 0, 0, 1]
[10, 10, 10, 1]
[10, 10, 10, 99]
```

## Custom stream

(* ANTLRFileStream在4.7中已弃用*)

如果您想在流中玩转,可以。这是一个示例,该示例更改了从字节流中计算“文本”的方式(这也更改了令牌打印其文本的方式):

```java
/** make a stream treating file as full of single unsigned byte characters */
class BinaryANTLRFileStream extends ANTLRFileStream {
	public BinaryANTLRFileStream(String fileName) throws IOException {
		super(fileName, "ISO-8859-1");
	}

	/** Print the decimal value rather than treat as char */
	@Override
	public String getText(Interval interval) {
		StringBuilder buf = new StringBuilder();
		int start = interval.a;
		int stop = interval.b;
		if(stop >= this.n) {
			stop = this.n - 1;
		}

		for (int i = start; i<=stop; i++) {
			int v = data[i];
			buf.append(v);
		}
		return buf.toString();
	}
}
```

新的测试代码如下所示开始:

```java
ANTLRFileStream bytesAsChar = new BinaryANTLRFileStream("/tmp/ips");
IPLexer lexer = new IPLexer(bytesAsChar);
...
```

这样可以简化我们的监听器:

```java
class MyIPListenerCustomStream extends IPBaseListener {
	@Override
	public void exitIp(IPParser.IpContext ctx) {
		List<TerminalNode> octets = ctx.BYTE();
		System.out.println(octets);
	}
}
```

您应该获得以下增强的输出:

```
[172(0xAC), 0(0x0), 0(0x0), 1(0x1)]
[10(0xA), 10(0xA), 10(0xA), 1(0x1)]
[10(0xA), 10(0xA), 10(0xA), 99(0x63)]
```

## Error handling in binary files

错误处理的过程与其他解析器完全相同。例如,让我们更改二进制文件,使其在第一个IP地址中丢失0之一:

```java
public static final byte[] bytes = {
	(byte)172, 0, 1, (byte)0xCA, (byte)0xFE, //OOOPS
	(byte)10, 10, 10, 1, (byte)0xCA, (byte)0xFE,
	(byte)10, 10, 10, 99
};
```

运行原始测试用例将为我们提供:

```
line 1:4 extraneous input '.' expecting BYTE
line 1:6 mismatched input 'Êþ' expecting '.'
[172, 0, 1, 0]
[10, 10, 10, 1]
[10, 10, 10, 99]
```

`'Êþ'`只是两个字节0xCA和0xFE的字符表示。使用增强的二进制流,我们看到:

```
line 1:4 extraneous input '46(0x2E)' expecting BYTE
line 1:6 mismatched input '202(0xCA)254(0xFE)' expecting '.'
[172(0xAC), 0(0x0), 1(0x1)]
[10(0xA), 10(0xA), 10(0xA), 1(0x1)]
[10(0xA), 10(0xA), 10(0xA), 99(0x63)]
```