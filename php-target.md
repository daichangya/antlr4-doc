# ANTLR4 Runtime for PHP

### 第一步

#### 1.安装ANTLR4

[入门指南](https://github.com/antlr/antlr4/blob/master/doc/getting-started.md) 
应该让您开始。

#### 2.安装PHP ANTLR运行时

ANTLR的每种目标语言都有一个用于运行解析器的运行时包 
由ANTLR4生成。运行时提供了一组用于使用解析器的通用工具。

使用Composer安装运行时:

```bash
composer install antlr/antlr4
```

#### 3.生成解析器

您使用ANTLR4“工具”来生成解析器。这些将引用ANTLR
运行时,安装在上面。

假设您使用的是UNIX系统,并为ANTLR4工具设置了别名 
如[入门指南](https://github.com/antlr/antlr4/blob/master/doc/getting-started.md)中所述。 
要生成PHP解析器,请运行以下命令:

```bash
antlr4 -Dlanguage=PHP MyGrammar.g4
```

有关antlr4工具选项的完整列表,请访问 
[工具文档页面](https://github.com/antlr/antlr4/blob/master/doc/tool-options.md)。

###完整的例子

假设您正在使用https://github.com/antlr/grammars-v4/tree/master/json中的JSON语法。

然后,调用`antlr4 -Dlanguage=PHP JSON.g4`。这样的结果是
`parser`目录中的`.php`文件的集合,包括:
```
JsonParser.php
JsonBaseListener.php
JsonLexer.php
JsonListener.php
```

ANTLR工具的另一个常见选项是`-visitor`,它会生成一个解析 
树访客,但我们不会在这里这样做。有关antlr4工具的完整列表
选项,请访问[工具文档页面](tool-options.md)。

我们将编写一个小的主函数来调用生成的解析器/词法分析器 
(假设它们是分开的)。这写出遇到的
`ParseTreeContext`:

```php
<?php

namespace JsonParser;

use Antlr\Antlr4\Runtime\CommonTokenStream;
use Antlr\Antlr4\Runtime\Error\Listeners\DiagnosticErrorListener;
use Antlr\Antlr4\Runtime\InputStream;
use Antlr\Antlr4\Runtime\ParserRuleContext;
use Antlr\Antlr4\Runtime\Tree\ErrorNode;
use Antlr\Antlr4\Runtime\Tree\ParseTreeListener;
use Antlr\Antlr4\Runtime\Tree\ParseTreeWalker;
use Antlr\Antlr4\Runtime\Tree\TerminalNode;

final class TreeShapeListener implements ParseTreeListener {
    public function visitTerminal(TerminalNode $node) : void {}
    public function visitErrorNode(ErrorNode $node) : void {}
    public function exitEveryRule(ParserRuleContext $ctx) : void {}

    public function enterEveryRule(ParserRuleContext $ctx) : void {
        echo $ctx->getText();
    }
}

$input = InputStream::fromPath($argv[1]);
$lexer = new JSONLexer($input);
$tokens = new CommonTokenStream($lexer);
$parser = new JSONParser($tokens);
$parser->addErrorListener(new DiagnosticErrorListener());
$parser->setBuildParseTree(true);
$tree = $parser->json();

ParseTreeWalker::default()->walk(new TreeShapeListener(), $tree);
```

创建一个`example.json`文件:
```json
{"a":1}
```

解析输入文件:

```
php json.php example.json
```

预期的输出是:

```
{"a":1}
{"a":1}
"a":1
1
```