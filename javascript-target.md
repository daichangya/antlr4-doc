# JavaScript

## Which browsers are supported?

从理论上讲,所有浏览器都支持ECMAScript 5.1。

实际上,已经对该目标进行了广泛的测试:

* Firefox 34.0.5
* Safari 8.0.2
*铬39.0.2171
*资源管理器11.0.3
 
测试使用硒进行。没发现问题,因此您应该发现运行时几乎可以与任何最新的JavaScript引擎一起工作。

## Is NodeJS supported?

还已针对Node.js 10 LTS对运行时进行了广泛的测试。找不到问题。

## How to create a JavaScript lexer or parser?

这与创建Java词法分析器或解析器几乎相同,除了需要指定语言目标外,例如:

```bash
$ antlr4 -Dlanguage=JavaScript MyGrammar.g4
```

有关antlr4工具选项的完整列表,请访问[工具文档页面](tool-options.md)。

## Where can I get the runtime?

生成词法分析器和/或解析器代码后,您需要下载运行时。

JavaScript运行时[可从npm获得](https://www.npmjs.com/package/antlr4)。

如果您不能使用npm,则还可以从ANTLR网站[下载部分](http://www.antlr.org/download/index.html)获得JavaScript运行时。运行时以源代码的形式提供,因此不需要其他安装。

我们不会在这里记录如何从您的项目中引用运行时,因为这取决于您的项目类型和IDE会有很大差异。 

## How do I get the runtime in my browser?

运行时非常大,目前以大约50个脚本的形式进行维护,这些脚本的结构与其他目标(Java,C#,Python ...)的运行时相同。

这种结构对于保持代码在目标之间的可维护性和一致性至关重要。

但是,将其带入浏览器会有些问题。没有人想写50次:

```
<script src='lib/myscript.js'>
```

为避免这样做,首选方法是使用webpack将antlr4与解析器代码捆绑在一起。

您可以在[此处获取有关webpack的信息](https://webpack.github.io)。

创建解析代码的步骤如下:
 -使用antlr工具生成词法分析器,解析器,侦听器和访问者
 -通过提供您的自定义侦听器或访问者以及相关代码(使用'require'加载antlr)来编写解析树处理代码。
 -创建一个index.js文件,其中包含指向您的解析代码的入口(如果需要,可以输入多个)。
 -使用node.js全面测试您的解析逻辑
 
现在,您可以按如下所示捆绑解析代码:
 -按照webpack规范,创建一个webpack.config文件
 -在`webpack.config`文件中,排除仅使用以下节点的node.js模块:`node: { module: "empty", net: "empty", fs: "empty" }`
 -在cmd行中,导航到包含webpack.config的目录,然后键入:webpack
 
这将产生一个包含所有解析代码的js文件。轻松包含在您的网页中!

## How do I run the generated lexer and/or parser?

假设您的语法如上所述被命名为“ MyGrammar”。让我们假设此解析器包含一个名为“ StartRule”的规则。该工具将为您生成以下文件:

* MyGrammarLexer.js
* MyGrammarParser.js
* MyGrammarListener.js(如果尚未激活-no-listener选项)
* MyGrammarVisitor.js(如果您已激活-visitor选项)
   
(习惯于Java/C#ANTLR的开发人员会注意到,没有生成基本的侦听器或访问者,这是因为JavaScript不支持接口,生成的侦听器和访问者是完全成熟的类)

现在,功能完整的脚本可能如下所示:

```javascript
   var antlr4 = require('antlr4');
   var MyGrammarLexer = require('./MyGrammarLexer').MyGrammarLexer;
   var MyGrammarParser = require('./MyGrammarParser').MyGrammarParser;
   var MyGrammarListener = require('./MyGrammarListener').MyGrammarListener;

   var input = "your text to parse here"
   var chars = new antlr4.InputStream(input);
   var lexer = new MyGrammarLexer(chars);
   var tokens  = new antlr4.CommonTokenStream(lexer);
   var parser = new MyGrammarParser(tokens);
   parser.buildParseTrees = true;
   var tree = parser.MyStartRule();
```

该程序将起作用。除非您执行以下操作之一,否则它将不会有用:

*您使用自定义侦听器访问解析树
*您使用自定义访问者访问解析树
*您的语法包含生产代码(例如AntLR3)
 
(请注意,生产代码是特定于目标的,因此您不能拥有包含生产代码的多目标语法)
 
## How do I create and run a visitor?
```javascript
//test.js
var antlr4 = require('antlr4');
var MyGrammarLexer = require('./QueryLexer').QueryLexer;
var MyGrammarParser = require('./QueryParser').QueryParser;
var MyGrammarListener = require('./QueryListener').QueryListener;


var input = "field = 123 AND items in (1,2,3)"
var chars = new antlr4.InputStream(input);
var lexer = new MyGrammarLexer(chars);
var tokens = new antlr4.CommonTokenStream(lexer);
var parser = new MyGrammarParser(tokens);
parser.buildParseTrees = true;
var tree = parser.query();

class Visitor {
  visitChildren(ctx) {
    if (!ctx) {
      return;
    }

    if (ctx.children) {
      return ctx.children.map(child => {
        if (child.children && child.children.length != 0) {
          return child.accept(this);
        } else {
          return child.getText();
        }
      });
    }
  }
}

tree.accept(new Visitor());
````

## How do I create and run a custom listener?

假设您的MyGrammar语法包含2条规则:“键”和“值”。antlr4工具将生成以下侦听器:

```javascript
   MyGrammarListener = function(ParseTreeListener) {
       //some code here
   }
   //some code here
   MyGrammarListener.prototype.enterKey = function(ctx) {};
   MyGrammarListener.prototype.exitKey = function(ctx) {};
   MyGrammarListener.prototype.enterValue = function(ctx) {};
   MyGrammarListener.prototype.exitValue = function(ctx) {};
```

为了提供自定义行为,您可能需要创建以下类:

```javascript
var KeyPrinter = function() {
    MyGrammarListener.call(this); //inherit default listener
    return this;
};

//continue inheriting default listener
KeyPrinter.prototype = Object.create(MyGrammarListener.prototype);
KeyPrinter.prototype.constructor = KeyPrinter;

//override default listener behavior
KeyPrinter.prototype.exitKey = function(ctx) {
    console.log("Oh, a key!");
};
```

为了执行此侦听器,您只需将以下行添加到上面的代码中:

```javascript
    ...
    tree = parser.StartRule() //only repeated here for reference
var printer = new KeyPrinter();
antlr4.tree.ParseTreeWalker.DEFAULT.walk(printer, tree);
```

## What about TypeScript?

我们目前没有TypeScript目标,但是Sam Harwell正在[在端口上工作](https://github.com/tunnelvisionlabs/antlr4ts)。[此处](https://github.com/ChuckJonas/extract-interface-ts)是[ANTLR 4书](http://a.co/5jUJYmh)的4.3节转换为打字稿的内容。

## How do I integrate my parser with ACE editor?

此特定任务在此[专用页面](ace-javascript-target.md)中进行了描述。
 
## How can I learn more about ANTLR?
 

可以从“权威ANTLR 4参考”一书中找到更多信息。

ANTLR的JavaScript实现与Java尽可能接近,因此您不应该发现很难将本书的示例适应JavaScript。