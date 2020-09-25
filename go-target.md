# ANTLR4 Language Target, Runtime for Go

### 第一步

#### 1.安装ANTLR4

[入门指南](getting-started.md)应该会让您入门。

#### 2.获取Go ANTLR运行时

ANTLR的每种目标语言都有一个运行时包,用于运行ANTLR4生成的解析器。运行时提供了一组用于使用解析器的通用工具。

获取运行时并将其安装在您的GOPATH上:

```bash
go get github.com/antlr/antlr4/runtime/Go/antlr
```

#### 3.设置发布标签(可选)

`go get`没有本地方法来指定分支或提交。因此,当您运行它时,您将下载最新的提交。这可能不是您的偏好。

您需要使用git来设置发布。例如,要设置版本4.6.0的版本标签:

```bash
cd $GOPATH/src/github.com/antlr/antlr4 # enter the antlr4 source directory
git checkout tags/4.6.0 # the go runtime was added in release 4.6.0
```

完整的发行列表可以在[发行页面](https://github.com/antlr/antlr4/releases)上找到。

#### 4.生成解析器

您使用ANTLR4“工具”来生成解析器。这些将引用上面安装的ANTLR运行时。

假设您正在使用UNIX系统,并且已按照[入门指南](getting-started.md)中所述为ANTLR4工具设置了别名。要生成go解析器,您需要调用:

```bash
antlr4 -Dlanguage=Go MyGrammar.g4
```

有关antlr4工具选项的完整列表,请访问[工具文档页面](tool-options.md)。

###引用Go ANTLR运行时

您可以像这样引用go ANTLR运行时软件包:

```go
import "github.com/antlr/antlr4/runtime/Go/antlr"
```

###完整的例子

假设您正在使用https://github.com/antlr/grammars-v4/tree/master/json中的JSON语法。

然后,调用`antlr4 -Dlanguage=Go JSON.g4`。这样的结果是`parser`目录中的.go文件的集合,包括:
```
json_parser.go
json_base_listener.go
json_lexer.go
json_listener.go
```

ANTLR工具的另一个常见选项是`-visitor`,它会生成一个分析树访问者,但是我们在这里不会这样做。有关antlr4工具选项的完整列表,请访问[工具文档页面](tool-options.md)。

我们将编写一个小的主函数来调用生成的解析器/词法分析器(假设它们是分开的)。这写出遇到的`ParseTreeContext`。假设生成的解析器代码相对于此代码在`parser`目录中:

```
package main

import (
	"github.com/antlr/antlr4/runtime/Go/antlr"
	"./parser"
	"os"
	"fmt"
)

type TreeShapeListener struct {
	*parser.BaseJSONListener
}

func NewTreeShapeListener() *TreeShapeListener {
	return new(TreeShapeListener)
}

func (this *TreeShapeListener) EnterEveryRule(ctx antlr.ParserRuleContext) {
	fmt.Println(ctx.GetText())
}

func main() {
	input, _ := antlr.NewFileStream(os.Args[1])
	lexer := parser.NewJSONLexer(input)
	stream := antlr.NewCommonTokenStream(lexer,0)
	p := parser.NewJSONParser(stream)
	p.AddErrorListener(antlr.NewDiagnosticErrorListener(true))
	p.BuildParseTrees = true
	tree := p.Json()
	antlr.ParseTreeWalkerDefault.Walk(NewTreeShapeListener(), tree)
}
```

这个期望输入在命令行上传递:

```
go run test.go input
```

输出为:

```
{"a":1}
{"a":1}
"a":1
1
```