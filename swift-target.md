# ANTLR4 Language Target, Runtime for Swift

## Requirements

ANTLR 4.7.2需要Swift 4.2。它也适用于Swift 4.2.1。

ANTLR 4.7.1需要Swift 4.0,并且不能在Swift 4.2上运行。(状态
Swift 4.1支持未知。)

## Performance Note

要在生产环境中使用ANTLR4 Swift目标,请确保按照[这些说明](https://github.com/apple/swift-package-manager/blob/master/Documentation/Usage.md#build -configuration)(如果您使用SwiftPM构建项目)。如果使用Xcode构建项目,则不太可能不会将`release`构建用于生产构建。

结论是,您需要打开`release`模式(它将为您预先配置所有优化),以便ANTLR4 Swift目标可以具有合理的解析速度。

## Install ANTLR4

确保您有ANTLR
已安装。[入门指南](getting-started.md)应该得到
你开始了。

## Create a Swift lexer or parser 
这与创建Java词法分析器或解析器几乎相同, 
除非您需要指定语言目标,例如:

``` 
$ antlr4 -Dlanguage=Swift MyGrammar.g4 
``` 

如果将此作为Xcode中的构建步骤进行集成,则应使用
“ gnu”消息格式,以使Xcode解析任何错误消息。你可以
还希望使用`-o`选项将自动生成的文件放在
单独的子目录。

```
antlr4 -Dlanguage=Swift -message-format gnu -o Autogen MyGrammar.g4
```

有关antlr4工具选项的完整列表,请访问
[工具文档页面](tool-options.md)。

## Build your Swift project with ANTLR runtime

### 注意

我们使用位于Swift运行时文件夹根目录的__boot.py__脚本
`antlr4/runtime/Swift`为基于Xcode的两者提供额外的支持
项目和基于SPM的项目。以下部分针对以下两个方面进行了组织
口味。如果您想快速入门,请尝试:

```
python boot.py --help
```

有关此脚本的信息。

### Xcode项目

请注意,即使您另外使用二进制分发版的ANTLR,
您应该从源代码编译ANTLR Swift运行时,因为Swift
语言尚没有稳定的ABI。

ANTLR使用Swift Package Manager生成Xcode项目文件。注意
Swift Package Manager当前不支持iOS,watchOS或tvOS,因此
如果您希望使用这些平台,则需要更改项目构建
手动进行适当的设置。

####下载ANTLR的源代码

```
git clone https://github.com/antlr/antlr4
```

####为ANTLR运行时生成Xcode项目

`boot.py`脚本包括`swift package
generate-xcodeproj`的包装。使用它为ANTLR生成`Antlr4.xcodeproj`
Swift运行时。(不建议使用_swift包generate-xcodeproj_)
因为项目依赖于_boot.py_生成的某些解析器文件。

```
cd antlr4/runtime/Swift
python boot.py --gen-xcodeproj
```

####将ANTLR Swift运行时导入项目

在Xcode中打开您自己的项目。

在`runtime/Swift`目录中打开Finder:

```
# From antlr4/runtime/Swift
open .
```

将`Antlr4.xcodeproj`拖到您的项目中。

完成此操作后,您的Xcode项目导航器将类似于
下面的屏幕截图。在此示例中,您自己的项目是“ Smalltalk”,并且您
将能够看到`Antlr4.xcodeproj`显示为包含的项目。

<img src = images/xcodenav.png width ="300">

####如有必要,编辑构建设置

Swift Package Manager当前不支持iOS,watchOS或tvOS。如果
您希望为这些平台构建,则需要更改项目
手动构建设置。

####将生成的解析器和词法分析器添加到项目中

确保解析器/词法分析器
__step 2__中生成的代码已添加到项目中。为此,您可以
将生成的文件从Finder拖到Xcode IDE。记得
检查__如果需要,请复制项目__以确保文件实际上是
已移至项目文件夹,而不是符号链接(请参见
下面的屏幕截图)。移动后,您将可以在其中查看文件
项目导航器。确保目标会员资格设置
对您的项目是正确的。

<img src = images/dragfile.png width ="500">

####将ANTLR Swift运行时添加为依赖项

在Xcode中选择您自己的项目,然后转到“构建阶段”设置面板。
在__Target Dependencies__和__Link Binary With下添加ANTLR运行时
图书馆__。

<img src = images/xcodedep.png width ="800">

####建立您的专案

运行时和生成的语法现在应该正确构建。

### Swift Package Manager项目

由于我们无法为Swift目标提供单独的存储库(请参见问题[#1774](https://github.com/antlr/antlr4/issues/1774)), 
和Swift目前不是ABI稳定。我们目前支持基于SPM的支持
通过创建临时本地存储库来进行项目。

对于使用[Swift软件包管理器](https://swift.org/package-manager/)的用户,
__boot.py__脚本支持生成可以使用的本地存储库
作为对您项目的依赖。只需运行:

```
python boot.py --gen-spm-module
```

提示将显示如下内容:

<img src = images/gen_spm_module.png width ="800">

将包含URL的SPM指令放入临时存储库中 
项目的Package.swift。并在您的项目中运行`swift build`。

如果找到该项目,则在系统的`/tmp/`目录中生成该项目
不便之处,请考虑将生成ANTLR存储库的副本复制到某个位置 
不会自动清除并更新您的`url`参数 
`Package.swift`文件。

## Swift access levels

您可以使用`accessLevel`选项来控制生成的访问级别
码。可以在`-DaccessLevel=value`上指定此选项
`antlr4`命令行,或在您的`.g4`文件中,如下所示:

```
options {
    accessLevel = 'value';
}
```

默认情况下(未指定`accessLevel`选项)生成的代码
使用以下访问级别:

* `open`提供您可以通过子类扩展的所有内容:
生成的解析器,词法分析器和上下文类,侦听器和
访问者基类,及其所有访问器和设置器功能。
* `public`用于不应该被子类化的任何东西,否则为
对客户端代码有用:协议,初始化程序和静态定义,例如
如词法分析器标记,符号名称等。
* `internal`或`private`用于不应直接访问的任何内容。

如果指定`accessLevel = 'public'`,则`open`的所有项目
默认将使用`public`代替。否则,行为与
默认值。

如果指定`accessLevel = ''`或`accessLevel='internal'`,则所有项目
默认为`open`或`public`将使用Swift的默认值(内部)
访问级别。

这些是使用Swift时`accessLevel`唯一受支持的值
代码生成器。

我们建议使用`accessLevel = ''`。即使您正在创建解析器
作为库的一部分,通常需要将其包装在您的API中
拥有并保持ANTLR生成的解析器在模块内部。您
仅在需要公开时才需要使用限制较少的访问级别
解析器直接作为您自己模块的API的一部分。