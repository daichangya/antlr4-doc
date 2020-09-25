# C++

C ++目标支持可以运行MS Visual Studio 2013(或更高版本),XCode 7(或更高版本)或CMake(需要C ++ 11)的所有平台。所有构建工具都可以创建静态或动态库(均为64位或32位架构)。此外,XCode可以创建一个iOS库。另请参阅[使用CMake的C ++的Antlr4:一个实际示例](http://blorente.me//Antlr-,-C++-and-CMake-Wait-what.html)。

## How to create a C++ lexer or parser?
这与创建Java词法分析器或解析器几乎相同,除了需要指定语言目标外,例如:

```
$ antlr4 -Dlanguage=Cpp MyGrammar.g4
```

您将看到此调用生成了一大堆文件。如果不抑制访问者或侦听器(这是默认设置),则会得到:

* MyGrammarLexer.h + MyGrammarLexer.cpp
* MyGrammarParser.h + MyGrammarParser.cpp
* MyGrammarVisitor.h + MyGrammarVisitor.cpp
* MyGrammarBaseVisitor.h + MyGrammarBaseVisitor.cpp
* MyGrammarListener.h + MyGrammarListener.cpp
* MyGrammarBaseListener.h + MyGrammarBaseListener.cpp

## Where can I get the runtime?

生成词法分析器和/或解析器代码后,您需要下载或构建运行时。可在ANTLR网站上获得针对Windows(Visual Studio 2013/2015),OSX/macOS和iOS的预构建C ++运行时二进制文件:

* http://www.antlr.org

使用CMake构建Linux库(也可在OSX上运行,但不适用于iOS库)。

除了下载预构建的二进制文件,您还可以轻松地在OSX或Windows上构建自己的库。只需使用提供的XCode或Visual Studio项目并进行构建即可。应该开箱即用,没有任何其他依赖性。


## How do I run the generated lexer and/or parser?

将所有内容放在一起以获得一个有效的解析器真的很容易。在[runtime/Cpp/demo](../runtime/Cpp/demo)文件夹中查找一个简单的示例。此处的[README](../runtime/Cpp/demo/README.md)简短描述了如何在OSX,Windows或Linux上构建和运行演示。

## How do I create and run a custom listener?

上面的生成步骤为您创建了一个侦听器和基础侦听器类。侦听器类是一个抽象接口,它为您的每个解析器规则声明enter和exit方法。基本侦听器以空主体实现所有这些抽象方法,因此,如果您只想实现一个功能,则不必自己做。因此,将此基本侦听器用作您的自定义侦听器的基类:

```c++
#include <iostream>

#include "antlr4-runtime.h"
#include "MyGrammarLexer.h"
#include "MyGrammarParser.h"
#include "MyGrammarBaseListener.h"

using namespace antlr4;

class TreeShapeListener : public MyGrammarBaseListener {
public:
  void enterKey(ParserRuleContext *ctx) override {
	//Do something when entering the key rule.
  }
};


int main(int argc, const char* argv[]) {
  std::ifstream stream;
  stream.open(argv[1]);
  ANTLRInputStream input(stream);
  MyGrammarLexer lexer(&input);
  CommonTokenStream tokens(&lexer);
  MyGrammarParser parser(&tokens);

  tree::ParseTree *tree = parser.key();
  TreeShapeListener listener;
  tree::ParseTreeWalker::DEFAULT->walk(&listener, tree);

  return 0;
}

```
 
本示例假定您的语法包含名为`key`的解析器规则,并为该规则生成了enterKey函数。

## Specialities of this ANTLR target

只有C ++ ANTLR目标需要处理几件事。它们在这里描述。

###构建方面
代码生成(通过运行ANTLR4 jar)允许指定2个值,您可能会发现它们对于将生成的文件更好地集成到应用程序中很有用(两个都是可选的):

*命名空间**:使用** `-package` **参数指定所需的命名空间。
*导出宏**:特别是在VC ++中,需要额外的工作才能从DLL中导出类。这通常由具有不同值的宏来完成,具体取决于您是创建DLL还是导入DLL。ANTLR4运行时本身也为其类使用了一个:

```c++
  #ifdef ANTLR4CPP_EXPORTS
    #define ANTLR4CPP_PUBLIC __declspec(dllexport)
  #else
    #ifdef ANTLR4CPP_STATIC
      #define ANTLR4CPP_PUBLIC
    #else
      #define ANTLR4CPP_PUBLIC __declspec(dllimport)
    #endif
  #endif
```
就像这里的`ANTLR4CPP_PUBLIC`宏一样,您可以使用** `-DexportMacro=...` **命令行参数为生成的类指定自己的宏,或者
语法文件中的语法选项`options {exportMacro='...';}`。

为了在Visual Studio中创建静态库,除了必须为静态库设置的项目设置(如果您自己编译运行时)之外,还定义`ANTLR4CPP_STATIC`宏。

对于gcc和clang,可以使用`-fvisibility=hidden`设置隐藏除默认可见(在运行时中为所有公共类定义的符号)以外的所有符号。

### 内存管理
由于C ++没有内置的内存管理,因此我们需要格外小心。为此,我们主要依靠智能指针,但是如果不谨慎使用,可能会导致时间浪费或内存副作用(例如循环引用)。但是目前,内存家庭看起来非常稳定。通常,当您在代码中看到原始指针时,请在此处将其视为已托管。您永远不应尝试管理此类指针(删除,分配给智能指针等)。

### Unicode支持
编码主要是输入问题,即当词法分析器将文本输入转换为词法分析器标记时。解析器完全不知道编码。

C ++目标始终希望输入UTF-8(以字符串或流的形式),然后将其转换为UTF-32(char32_t数组)并提供给词法分析器。

###命名动作
为了帮助自定义生成的文件,还有许多其他的所谓的“命名动作”。这些操作与生成的代码中的特定区域紧密相关,并允许添加自定义(特定于目标)的代码。所有目标都支持这些行动

* @parser :: header
* @parser :: members
* @lexer :: header
* @lexer ::成员

(以及它们的无范围替代方法`@header`和`@members`),其中header并不意味着C/C ++头文件,而是代码文件的顶部。标头操作的内容出现在所有生成的文件的第一行。因此,对于诸如许可/版权信息之类的东西来说,它是很好的。

* members *动作的内容位于lexer或解析器类声明的public部分中。因此,它可以用于语法谓词中的公共变量或谓词函数。由于所有目标都支持* header * + * members *,因此它们是在其他语言的生成文件中也应有的东西的最佳位置。

除此之外,C ++目标还支持更多此类命名动作。不幸的是,不可能定义新的范围(例如,除* parser *之外的* listener *),因此必须将它们定义为现有范围的一部分(* lexer *或* parser *)。演示应用程序中的语法还包含所有命名的动作,以供参考。这是清单:

* ** @ lexer :: preinclude **-放在第一个#include之前(例如,对于必须首先出现的标头,系统标头等有用)。出现在lexer h和cpp文件中。
* ** @ lexer :: postinclude **-放在最后一个#include之后,但在任何类代码之前(例如,用于其他名称空间)。出现在lexer h和cpp文件中。
* ** @ lexer :: context **-放在lexer类声明之前。用于其他类型,别名,前向声明等。出现在lexer h文件中。
* ** @ lexer :: declarations **-放在lexer声明的私有部分中(所有类中生成的部分都严格遵循以下模式:public,protected,privat,从上到下)。将此用于私有变量等。
* ** @ lexer :: definitions **-放置在cpp文件中其他实现之前(但** @ postinclude *之后)。用它来实现例如私有类型。

对于解析器,有与上述词法分析器相同的操作。除此之外,访问者和侦听器类还有更多操作:

* ** @ parser :: listenerpreinclude **
* ** @ parser :: listenerpostinclude **
* ** @ parser :: listenerdeclarations **
* ** @ parser :: listenermembers **
* ** @ parser :: listenerdefinitions **
* 
* ** @ parser :: baselistenerpreinclude **
* ** @ parser :: baselistenerpostinclude **
* ** @ parser :: baselistener声明**
* ** @ parser :: baselistenermembers **
* ** @ parser :: baselistener定义**
* 
* ** @ parser :: visitorpreinclude **
* ** @ parser :: visitorpostinclude **
* ** @ parser ::访客声明**
* ** @ parser :: visitormembers **
* ** @ parser :: visitordefinitions **
* 
* ** @ parser :: basevisitorpreinclude **
* ** @ parser :: basevisitorpostinclude **
* ** @ parser :: basevisitor声明**
* ** @ parser :: basevisitormembers **
* ** @ parser :: basevisitor定义**

并且现在应该可以自我解释。注意:没有针对监听者或访问者的* context *操作,仅仅是因为它们比其他操作要少得多,并且已经有很多此类操作了。