# Parser and Lexer Interpreters

*自ANTLR 4.2起*

对于小型解析任务,有时在解释模式下使用ANTLR较方便,而不是在特定目标中生成解析器,将其编译并作为应用程序的一部分运行。这是一些示例代码,它们创建词法分析器和解析器语法对象,然后创建解释器。一旦有了ParserInterpreter,我们就可以使用它从我们喜欢的任何规则开始解析,并提供规则索引(语法+解析器可以提供)。

## Action Code

由于解释器不使用生成的解析器+词法分析器,因此它们无法执行任何操作代码(包括谓词)。这意味着解释器运行起来就好像根本没有谓词一样。如果您的语法需要操作代码才能正确解析,您将无法使用此方法对其进行测试。

## Java Target Interpreter Setup

```java
LexerGrammar lg = new LexerGrammar(
    "lexer grammar L;\n" +
    "A : 'a' ;\n" +
    "B : 'b' ;\n" +
    "C : 'c' ;\n");
Grammar g = new Grammar(
    "parser grammar T;\n" +
    "s : (A|B)* C ;\n",
    lg);   
LexerInterpreter lexEngine =
    lg.createLexerInterpreter(new ANTLRInputStream(input));
CommonTokenStream tokens = new CommonTokenStream(lexEngine);
ParserInterpreter parser = g.createParserInterpreter(tokens);
ParseTree t = parser.parse(g.rules.get(startRule).index);
```

您还可以从文件中加载组合语法:

```java
public static ParseTree parse(String fileName,
                              String combinedGrammarFileName,
                              String startRule)
    throws IOException
{
    final Grammar g = Grammar.load(combinedGrammarFileName);
    LexerInterpreter lexEngine = g.createLexerInterpreter(CharStreams.fromPath(Paths.get(fileName)));
    CommonTokenStream tokens = new CommonTokenStream(lexEngine);
    ParserInterpreter parser = g.createParserInterpreter(tokens);
    ParseTree t = parser.parse(g.getRule(startRule).index);
    System.out.println("parse tree: "+t.toStringTree(parser));
    return t;
}
```

然后:

```java
ParseTree t = parse("T.om",
                    MantraGrammar,
                    "compilationUnit");
```
 
要加载单独的词法分析器/解析器语法,请执行以下操作:

```java
public static ParseTree parse(String fileNameToParse,
                              String lexerGrammarFileName,
                              String parserGrammarFileName,
                              String startRule)
    throws IOException
{
    final LexerGrammar lg = (LexerGrammar) Grammar.load(lexerGrammarFileName);
    final Grammar pg = Grammar.load(parserGrammarFileName, lg);
    CharStream input = CharStreams.fromPath(Paths.get(fileNameToParse));
    LexerInterpreter lexEngine = lg.createLexerInterpreter(input);
    CommonTokenStream tokens = new CommonTokenStream(lexEngine);
    ParserInterpreter parser = pg.createParserInterpreter(tokens);
    ParseTree t = parser.parse(pg.getRule(startRule).index);
    System.out.println("parse tree: " + t.toStringTree(parser));
    return t;
}
```

然后:

```java
ParseTree t = parse(fileName, XMLLexerGrammar, XMLParserGrammar, "document");
```

这也是我们将即时解析集成到ANTLRWorks2和开发环境插件中的方式。

请参阅[TestParserInterpreter.java](../tool-testsuite/test/org/antlr/v4/test/tool/TestParserInterpreter.java)。

## Non-Java Target Interpreter Setup
ANTLR4运行时不包含任何语法分析类(它们在ANTLR4工具jar中)。因此,我们不能使用`LexerGrammar`和`Grammar`来解析解释器的语法。相反,我们直接实例化`LexerInterpreter`和`ParserInterpreter`对象。它们需要一些数据(即符号信息和ATN),只有ANTLR4工具才能提供给我们。但是,在每一代运行时,ANTLR不仅会生成您的解析器和词法分析器文件,而且还会生成解释器数据文件(* .interp),其中包含提供解释器所需的全部内容。

为了方便起见,支持类(`InterpreterDataReader`)用于加载数据,这使它非常易于使用。顺便说一句。甚至Java目标也采用这种方式,而不是使用非运行时类`Grammar`和`LexerGrammar`。有时,出于任何原因使用工具罐可能都不可行。

这是设置的样子(C ++示例):

```cpp
/**
 * sourceFileName - name of the file with content to parse
 * lexerName - the name of your lexer (arbitrary, that's what is used in error messages)
 * parserName - ditto for the parser
 * lexerDataFileName - the lexer interpeter data file name (e.g. `<path>/ExprLexer.interp`)
 * parserDataFileName - ditto for the parser (e.g. `<path>/Expr.interp`)
 * startRule - the name of the rule to start parsing at
 */
void parse(std::string const& sourceFileName,
  std::string const& lexerName, std::string const& parserName,
  std::string const& lexerDataFileName, std::string const& parserDataFileName,
  std::string const& startRule) {
  
    InterpreterData lexerData = InterpreterDataReader::parseFile(lexerDataFileName);
    InterpreterData parserData = InterpreterDataReader::parseFile(parserDataFileName);

    ANTLRFileStream input(sourceFileName);
    LexerInterpreter lexEngine(lexerName, lexerData.vocabulary, lexerData.ruleNames,
      lexerData.channels, lexerData.modes, lexerData.atn, &input);
    CommonTokenStream tokens(&lexEngine);

    /* Remove comment to print the tokens.
    tokens.fill();
    std::cout << "INPUT:" << std::endl;
    for (auto token : tokens.getTokens()) {
      std::cout << token->toString() << std::endl;
    }
    */

    ParserInterpreter parser(parserName, parserData.vocabulary, parserData.ruleNames,
      parserData.atn, &tokens);
    tree::ParseTree *tree = parser.parse(parser.getRuleIndex(startRule));

    std::cout << "parse tree: " << tree->toStringTree(&parser) << std::endl;
}
```