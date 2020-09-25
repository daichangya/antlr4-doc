# Lexical analysis

## How can I parse non-ASCII text and use characters in token rules?

请参阅[在令牌规则中使用非ASCII字符](http://stackoverflow.com/questions/28126507/antlr4-using-non-ascii-characters-in-token-rules/28129510#28129510)。

## How do I replace escape characters in string tokens?

不幸的是,操纵与词法规则匹配的标记的文本很麻烦(从4.2开始)。您必须建立一个缓冲区,然后在最后设置文本。词法分析器中的动作就像在解析器中一样,在输入中的关联位置执行。这是一个在字符串中转义字符的示例。它不是很漂亮,但是可以。

```
grammar Foo;
 
@members {
StringBuilder buf = new StringBuilder(); //can't make locals in lexer rules
}
 
STR :   '"'
        (   '\\'
            (   'r'     {buf.append('\r');}
            |   'n'     {buf.append('\n');}
            |   't'     {buf.append('\t');}
            |   '\\'    {buf.append('\\');}
            |   '\"'   {buf.append('"');}
            )
        |   ~('\\'|'"') {buf.append((char)_input.LA(-1));}
        )*
        '"'
        {setText(buf.toString()); buf.setLength(0); System.out.println(getText());}
    ;
```

返回原始输入字符串,然后在解析树遍历或其他过程中稍后使用一个小函数重写该字符串更加容易和高效。但是,这是从词法分析器内部执行操作的方法。

Lexer动作在解释器中不起作用,该解释器包括xpath和树模式。

有关反对在词法分析器中执行复杂操作的论点的更多信息,请参见[相关的词法分析器作用问题,位于github](https://github.com/antlr/antlr4/issues/483#issuecomment-37326067)。

## Why are my keywords treated as identifiers?

像`begin`这样的关键字在词法上也是有效的标识符,因此输入是不明确的。为了解决歧义,ANTLR优先使用首先指定的词汇规则。这意味着您必须将标识符规则放在所有关键字之后:

```
grammar T;
 
decl : DEF 'int' ID ';'
 
DEF : 'def' ;   //ambiguous with ID as is 'int'
ID  : [a-z]+ ;
```

请注意,文字`'int'`在物理上也位于ID规则之前,并且也会获得优先权。

## Why are there no whitespace tokens in the token stream?

词法分析器没有将空白发送给解析器,这意味着重写流也无法访问令牌。这是因为skip lexer命令:

```
WS : [ \t\r\n\u000C]+ -> skip
   ;
```

您必须将所有这些更改为`-> channel(HIDDEN)`,这会将它们发送到其他通道上的解析器,从而使它们在令牌流中可用,但对于解析器而言是不可见的。