# Integrating ANTLR JavaScript parsers with ACE editor

具备解析JavaScript以外的代码的能力非常棒,但是如今,用户希望能够使用出色的编辑功能(例如关键字突出显示,缩进和大括号匹配)以及高级功能(例如语法检查)来编辑代码。

我已经完成了将ANTLR解析器与ACE集成的过程,ACE是基于Web的代码编辑的主要代码编辑器。有关ACE的信息可以在其网站上找到。

此页面描述了我的经验,并谦虚地帮助您入门。但是,它不是参考指南,也不提供支持。

## Architecture

ACE编辑器的组织方式如下

1.编辑器本身是<div>,一旦初始化,它就会包含许多元素。此UI元素负责显示以及编辑事件的生成。
1.编辑器依赖于一个会话,该会话管理事件和配置。
1.代码本身存储在文档中。文本的任何插入或删除都会反映在文档中。
1.将关键字突出显示,缩进和大括号匹配委托给一种模式。在ANTLR中没有直接等效的ACE模式。尽管关键字等效于ANTLR词法分析器标记,但缩进和括号匹配是编辑任务,而不是解析任务。给定的ACE编辑器只能具有一种模式,该模式对应于正在编辑的语言。无需ANTLR集成即可支持关键字突出显示,缩进和括号匹配。
1.语法检查委托给工作人员。这是需要ANTLR集成的地方。如果启用了语法检查,ACE将要求该模式创建工作程序。在JavaScript中,工作程序完全隔离运行,即他们不与其他工作程序或HTML页面本身共享代码或变量。
1.下图描述了整个系统的工作方式。绿色是您需要提供的组件。您会注意到,无需在HTML页面本身中加载ANTLR。您还将注意到,ACE在每个线程中维护一个文档。这是通过ACE会话发送给工作人员的低级事件来完成的,这些事件描述了增量。一旦应用于工作人员文档,便会触发一个高级事件,该事件很容易处理,因为此时工作人员文档是UI文档的完美副本。  

<img src = images/ACE-Architecture.001.png>

## Step-by-step guide

首先要做的是在您的html页面中创建一个编辑器。ACE文档中对此进行了详细描述,因此我们在这里进行总结:

```xml
<script src="../js/ace/ace.js" type="text/javascript" charset="utf-8"></script>
<script>
    var editor = ace.edit("editor");
</script>
```

这应该给您一个工作的编辑器。您可能要使用CSS控制其大小。我个人将编辑器加载到iframe中,并将其样式设置为position:绝对,top:0,left:0等...,但是我敢肯定,您比我更了解如何获得结果。

第二件事是将ACE编辑器配置为使用您的模式,即语言配置。一个好的开始是从内置TextMode继承。以下是一个非常简单的示例,该示例仅满足注释,文字和分隔符和关键字的有限子集的要求:

```javascript
ace.define('ace/mode/my-mode',["require","exports","module","ace/lib/oop","ace/mode/text","ace/mode/text_highlight_rules", "ace/worker/worker_client" ], function(require, exports, module) {
    var oop = require("ace/lib/oop");
    var TextMode = require("ace/mode/text").Mode;
    var TextHighlightRules = require("ace/mode/text_highlight_rules").TextHighlightRules;

    var MyHighlightRules = function() {
        var keywordMapper = this.createKeywordMapper({
            "keyword.control": "if|then|else",
            "keyword.operator": "and|or|not",
            "keyword.other": "class",
            "storage.type": "int|float|text",
            "storage.modifier": "private|public",
            "support.function": "print|sort",
            "constant.language": "true|false"
  }, "identifier");
        this.$rules = {
            "start": [
                { token : "comment", regex : "//" },
                { token : "string",  regex : '["](?:(?:\\\\.)|(?:[^"\\\\]))*?["]' },
                { token : "constant.numeric", regex : "0[xX][0-9a-fA-F]+\\b" },
                { token : "constant.numeric", regex: "[+-]?\\d+(?:(?:\\.\\d*)?(?:[eE][+-]?\\d+)?)?\\b" },
                { token : "keyword.operator", regex : "!|%|\\\\|/|\\*|\\-|\\+|~=|==|<>|!=|<=|>=|=|<|>|&&|\\|\\|" },
                { token : "punctuation.operator", regex : "\\?|\\:|\\,|\\;|\\." },
                { token : "paren.lparen", regex : "[[({]" },
                { token : "paren.rparen", regex : "[\\])}]" },
                { token : "text", regex : "\\s+" },
                { token: keywordMapper, regex: "[a-zA-Z_$][a-zA-Z0-9_$]*\\b" }
            ]
        };
    };
    oop.inherits(MyHighlightRules, TextHighlightRules);

    var MyMode = function() {
        this.HighlightRules = MyHighlightRules;
    };
    oop.inherits(MyMode, TextMode);

    (function() {

        this.$id = "ace/mode/my-mode";

    }).call(MyMode.prototype);

    exports.Mode = MyMode;
});
```

现在,如果将以上内容存储在名为“ my-mode.js”的文件中,则设置ACE Editor变得很简单:

```xml
<script src="../js/ace/ace.js" type="text/javascript" charset="utf-8"></script>
<script src="../js/my-mode.js" type="text/javascript" charset="utf-8"></script>
<script>
    var editor = ace.edit("editor");
    editor.getSession().setMode("ace/mode/my-mode");
</script>
```

此时,您应该有一个工作的编辑器,能够突出显示关键字。您可能想知道为什么在ANTLR词法分析器语法中已经设置了令牌后,为什么需要设置令牌。首先,ACE需要ANTLR中不存在的分类(控件,运算符,类型...)。其次,由于ACE带有自己的词法分析器,因此ANTLR无需实现这一目标。

好的,现在我们有了一个工作的编辑器,我们需要进行语法验证。这是工作人员出现在照片中的位置。

创建工作人员是您提供的模式的责任。因此,您需要使用以下内容来增强它:
 
```javascript
var WorkerClient = require("ace/worker/worker_client").WorkerClient;
this.createWorker = function(session) {
    this.$worker = new WorkerClient(["ace"], "ace/worker/my-worker", "MyWorker", "../js/my-worker.js");
    this.$worker.attachToDocument(session.getDocument());

    this.$worker.on("errors", function(e) {
        session.setAnnotations(e.data);
    });

    this.$worker.on("annotate", function(e) {
        session.setAnnotations(e.data);
    });

    this.$worker.on("terminate", function() {
        session.clearAnnotations();
    });

    return this.$worker;

};
```

上面的代码需要放在以下位置的现有工作程序中: 

```javascript
this.$id = "ace/mode/my-mode";
```

请注意,模式代码在UI端而不是工作端端运行。这里的事件处理程序用于工作者发送的事件,而不是发送给工作者的事件。

显然,以上内容无法立即使用,因为您需要提供“ my-worker.js”文件。

从头开始创建工人不是我尝试过的事情。简而言之,您的工作人员需要使用该模式创建的WorkerClient处理ACE发送的所有消息。这不是一个简单的任务,最好将其委托给现有的ACE代码,因此我们可以专注于特定于我们语言的任务。

我所做的是从“ mode-json.js”开始的,它是ACE附带的一个相当简单的工作程序,从中删除了所有与JSON验证相关的内容,并将其余代码保存在文件名“ worker-base.js”中。您可以在[此处](resources/worker-base.js)中找到它。完成此操作后,我就可以创建一个简单的工作程序,如下所示:

```javascript
importScripts("worker-base.js");
ace.define('ace/worker/my-worker',["require","exports","module","ace/lib/oop","ace/worker/mirror"], function(require, exports, module) {
    "use strict";

    var oop = require("ace/lib/oop");
    var Mirror = require("ace/worker/mirror").Mirror;

    var MyWorker = function(sender) {
        Mirror.call(this, sender);
        this.setTimeout(200);
        this.$dialect = null;
    };

    oop.inherits(MyWorker, Mirror);

    (function() {

        this.onUpdate = function() {
            var value = this.doc.getValue();
            var annotations = validate(value);
            this.sender.emit("annotate", annotations);
        };

    }).call(MyWorker.prototype);

    exports.MyWorker = MyWorker;
});

var validate = function(input) {
    return [ { row: 0, column: 0, text: "MyMode says Hello!", type: "error" } ];
};
```

此时,您应该有一个编辑器,该编辑器在第一行旁边显示一个错误图标。当您将鼠标悬停在错误图标上时,它将显示:MyMode说Hello!。那不是一个友好的工人吗?好吃

剩下要做的是让我们的validate函数实际验证输入。最后,ANTLR出现在图片中!

首先,让我们加载ANTLR和解析器,侦听器等。 

加载解析器代码的首选方法是捆绑解析器,[如此处所述](javascript-target.md)。
然后,可以在工作程序代码的开头将其作为importScripts指令的一部分进行加载。

另一种方法是使用“ require”加载它。容易,因为您可以编写:

```js
var antlr4 = require('antlr4/index');
```

这可能有效,但实际上是不可靠的。原因是ACE附带的'require'函数使用的语法与ANTLR使用的'require'函数的语法不同,后者遵循NodeJS的'require'约定。
因此,我们需要引入一个符合NodeJS语法的兼容NodeJS的“ require”函数。我个人使用的是Torben Haase的Honey项目中的项目,您可以在li/require.js中找到它。
但是,嘿,现在我们将拥有2个互不兼容的'require'函数!的确,这就是为什么您需要特别注意以下原因:

```js
//load nodejs compatible require
var ace_require = require;
require = undefined;
var Honey = { 'requirePath': ['..'] }; //walk up to js folder, see Honey docs
importScripts("../lib/require.js");
var antlr4_require = require;
require = ace_require;
```
现在可以安全地加载antlr和为您的语言生成的解析器。 
假设您的语言文件(生成的或手工生成的)位于一个index.js文件的文件夹中,每个文件都需要调用该文件,解析器的加载代码可以很简单:
```js
//load antlr4 and myLanguage
var antlr4, mylanguage;
try {
    require = antlr4_require;
    antlr4 = require('antlr4/index');
    mylanguage = require('mylanguage/index');
} finally {
    require = ace_require;
}
```
请注意try-finally结构。ANTLR同步使用“ require”,因此在运行ANTLR代码时忽略ACE“ require”是绝对安全的。ACE本身不保证同步执行,因此始终将“ require”切换回“ ace_require”更加安全。

现在,检测代码中的深层语法错误是ANTLR侦听器或访问者或您委托它的任何代码段的任务。我们将不再在此进行描述,因为这可能需要您的语言知识。但是,ANTLR可以很好地检测语法语法错误(这不是您为什么首先选择ANTLR的原因吗?)。因此,我们将在此处说明如何报告语法语法错误。毫无疑问,您将可以从那里扩展验证器以满足您的特定需求。
只要ANTLR遇到意外令牌,就会触发错误。默认情况下,错误被路由到错误侦听器,该侦听器仅写入控制台。
我们需要做的是用我们自己的侦听器替换此侦听器,这样我们就可以将错误路由到ACE编辑器。首先,让我们创建一个这样的监听器:
```js
//class for gathering errors and posting them to ACE editor
var AnnotatingErrorListener = function(annotations) {
    antlr4.error.ErrorListener.call(this);
    this.annotations = annotations;
    return this;
};

AnnotatingErrorListener.prototype = Object.create(antlr4.error.ErrorListener.prototype);
AnnotatingErrorListener.prototype.constructor = AnnotatingErrorListener;

AnnotatingErrorListener.prototype.syntaxError = function(recognizer, offendingSymbol, line, column, msg, e) {
    this.annotations.push({
        row: line - 1,
        column: column,
        text: msg,
        type: "error"
 });
};
```
这样,剩下要做的就是在解析代码时插入侦听器。这是我的方法:
```js
var validate = function(input) {
    var stream = CharStreams.fromString(input);
    var lexer = new mylanguage.MyLexer(stream);
    var tokens = new antlr4.CommonTokenStream(lexer);
    var parser = new mylanguage.MyParser(tokens);
    var annotations = [];
    var listener = new AnnotatingErrorListener(annotations)
    parser.removeErrorListeners();
    parser.addErrorListener(listener);
    parser.parseMyRule();
    return annotations;
};
```
你知道吗?而已!现在,您有了一个使用ANTLR进行语法验证的ACE编辑器!我希望您觉得这很有用,并且足够简单,可以开始使用。
现在等等,嘿!您如何调试呢?好吧,像往常一样,使用Chrome,因为没有其他浏览器能够调试工作程序代码。多可惜...