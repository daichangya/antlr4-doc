# ANTLR project unit tests

## Introduction

由于ANTLR支持多种目标语言,因此单元测试分为两组:用于测试工具本身的单元测试(在`tool-testsuite`中)和用于测试解析器运行时的单元测试(在`antlr4/runtime-testsuite`中)。工具测试很简单,因为它们是Java代码测试Java代码。请参阅此文件底部的部分。

必须以通用方式指定运行时测试才能跨语言目标工作。此外,我们必须测试Java的各种目标。这通常意味着Java启动进程来编译C ++并运行解析器。

从4.6开始,我们使用[Java描述符对象](https://github.com/antlr/antlr4/blob/master/runtime-testsuite/test/org/antlr/v4/test/runtime/RuntimeTestDescriptor.java)描述每个运行时测试。单元测试分为[ParserExecDescriptors](https://github.com/antlr/antlr4/blob/master/runtime-testsuite/test/org/antlr/v4/test/runtime/descriptors/ParserExecDescriptors.java ),它具有多个嵌套的描述符对象,每个测试一个。例如,这是该文件的开头:

```java
public class ParserExecDescriptors {
    public static class APlus extends BaseParserTestDescriptor {
        public String input = "a b c";
        public String output = "abc\n";
        public String errors = "";
        public String startRule = "a";
        public String grammarName = "T";

        /**
         grammar T;
         a : ID+ {
         <writeln("$text")>
         };
         ID : 'a'..'z'+;
         WS : (' '|'\n') -> skip;
         */
        @CommentHasStringValue
        public String grammar;
    }
```

神秘的`@CommentHasStringValue`注释有点像个hack,它允许Java中使用多行字符串。需要此功夫,以便我们可以使用Java类而不是StringTemplate组文件来指定运行时测试(传统系统使用了这些测试,很难正确地进行测试)。这是按组组织的所有[运行时测试描述符](https://github.com/antlr/antlr4/tree/master/runtime-testsuite/test/org/antlr/v4/test/runtime/descriptors)。

语法是表示StringTemplates的字符串(`ST`对象),因此在生成单元测试文件(`Test.java`,`Test.cs`,...)时,`<writeln("$text")>`将被替换。必须为每个目标定义`writeln`模板。这是所有的
[运行时测试的目标模板](https://github.com/antlr/antlr4/tree/master/runtime-testsuite/resources/org/antlr/v4/test/runtime/templates)。

## Requirements

为了对所有目标语言执行测试,您需要安装以下语言:

*非Windows机器上的`mono`(例如`brew install mono`)(在Windows上使用Microsoft .net堆栈)。在运行测试之前,还必须[`xbuild`运行时](https://github.com/antlr/antlr4/blob/master/doc/releasing-antlr.md);见下文
* `nodejs`
* Python 2.7
* Python 3.6
* 走
* Swift 4(通过XCode 10.x)目前仅通过osx测试
* lang(用于C ++目标) 
* 
要**安装到本地存储库** `~/.m2/repository/org/antlr`,请执行以下操作:

```bash
$ export MAVEN_OPTS="-Xmx1G"     # don't forget this on linux
$ mvn install -DskipTests=true   # make sure all artifacts are visible on this machine
```

现在,确保在本地构建和安装C#运行时。

```bash
cd ~/antlr/code/antlr4/runtime/CSharp/runtime/CSharp
# kill previous ones manually as "xbuild /t:Clean" didn't seem to do it
find . -name '*.dll' -exec rm {} \;
# build
xbuild /p:Configuration=Release Antlr4.Runtime/Antlr4.Runtime.mono.csproj
```

C ++测试装备会在测试期间自动构建C ++运行时。其他人则不需要预先构建的库。


## Running the runtime tests

使用[junit参数化测试](https://github.com/junit-team/junit4/wiki/parameterized-tests)机制,单个测试平台就足以针对所有描述符测试所有目标。但是,这很不方便,因为我们经常只想测试单个目标,或者甚至只测试单个目标的单个组中的单个测试。我自动生成了一堆
[目标运行时测试装置](https://github.com/antlr/antlr4/tree/master/runtime-testsuite/test/org/antlr/v4/test/runtime),使开发人员具有这种灵活性。例如,这是intellij中的Python3测试平台:

<img src = images/testrigs.png宽度= 300>

测试整个子目录的结果:

<img src = images/python3-tests.png宽度= 400>

在命令行中的`mvn`,您将看到:

```bash
$ cd antlr4
$ mvn test
...
-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Running org.antlr.v4.test.runtime.csharp.TestCompositeLexers
dir /var/folders/s1/h3qgww1x0ks3pb30l8t1wgd80000gn/T/TestCompositeLexers-1446068612451
Starting build /usr/bin/xbuild /p:Configuration=Release /var/folders/s1/h3qgww1x0ks3pb30l8t1wgd80000gn/T/TestCompositeLexers-1446068612451/Antlr4.Test.mono.csproj
dir /var/folders/s1/h3qgww1x0ks3pb30l8t1wgd80000gn/T/TestCompositeLexers-1446068615081
Starting build /usr/bin/xbuild /p:Configuration=Release /var/folders/s1/h3qgww1x0ks3pb30l8t1wgd80000gn/T/TestCompositeLexers-1446068615081/Antlr4.Test.mono.csproj
Tests run: 2, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 3.451 sec
Running org.antlr.v4.test.runtime.csharp.TestCompositeParsers
dir /var/folders/s1/h3qgww1x0ks3pb30l8t1wgd80000gn/T/TestCompositeParsers-1446068615864
antlr reports warnings from [-visitor, -Dlanguage=CSharp, -o, /var/folders/s1/h3qgww1x0ks3pb30l8t1wgd80000gn/T/TestCompositeParsers-1446068615864, -lib, /var/folders/s1/h3qgww1x0ks3pb30l8t1wgd80000gn/T/TestCompositeParsers-1446068615864, -encoding, UTF-8, /var/folders/s1/h3qgww1x0ks3pb30l8t1wgd80000gn/T/TestCompositeParsers-1446068615864/M.g4]
...
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Summary:
[INFO] 
[INFO] ANTLR 4 ............................................ SUCCESS [  0.445 s]
[INFO] ANTLR 4 Runtime .................................... SUCCESS [  3.392 s]
[INFO] ANTLR 4 Tool ....................................... SUCCESS [  1.373 s]
[INFO] ANTLR 4 Maven plugin ............................... SUCCESS [  1.519 s]
[INFO] ANTLR 4 Runtime Test Annotations ................... SUCCESS [  0.086 s]
[INFO] ANTLR 4 Runtime Test Processors .................... SUCCESS [  0.014 s]
[INFO] ANTLR 4 Runtime Tests (2nd generation) ............. SUCCESS [06:39 min]
[INFO] ANTLR 4 Tool Tests ................................. SUCCESS [  6.922 s]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 06:53 min
[INFO] Finished at: 2016-11-16T15:36:56-08:00
[INFO] Final Memory: 44M/458M
[INFO] ------------------------------------------------------------------------
```

注意:这实际上是运行速度快得多的结果:

```bash
mvn -Dparallel=methods -DthreadCount=4 install
```

## Running test subsets

*来自`runtime-testsuite`目录*

###跨目标运行一个测试组

```bash
$ cd runtime-testsuite
$ export MAVEN_OPTS="-Xmx1G"     # don't forget this on linux
$ mvn -Dtest=TestParserExec test
-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Running org.antlr.v4.test.runtime.cpp.TestParserExec
...
Tests run: 32, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 114.283 sec
Running org.antlr.v4.test.runtime.csharp.TestParserExec
...
```

或运行所有与lexer相关的测试:

```
$ cd runtime-testsuite
$ mvn -Dtest=Test*Lexer* test
-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Running org.antlr.v4.test.runtime.cpp.TestCompositeLexers
...
```

###针对单个目标运行所有测试

```bash
$ cd runtime-testsuite
$ mvn -Dtest=java.* test
...
```

或仅在Java目标中运行所有与lexer相关的测试:

```bash
$ cd runtime-testsuite
$ mvn -Dtest=java.*Lexer* test
...
-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Running org.antlr.v4.test.runtime.java.TestCompositeLexers
Tests run: 2, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 1.277 sec
Running org.antlr.v4.test.runtime.java.TestLexerErrors
Tests run: 12, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 2.376 sec
Running org.antlr.v4.test.runtime.java.TestLexerExec
Tests run: 38, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 10.07 sec
Running org.antlr.v4.test.runtime.java.TestSemPredEvalLexer
Tests run: 7, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 1.255 sec

Results :

Tests run: 59, Failures: 0, Errors: 0, Skipped: 0
```

## Testing in parallel

使用它来并行运行测试:

```bash
$ export MAVEN_OPTS="-Xmx1G"
$ mvn -Dparallel=methods -DthreadCount=4 test
...
-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Concurrency config is parallel='methods', perCoreThreadCount=true, threadCount=4, useUnlimitedThreads=false
...
```

可以与上面的其他`-D`结合使用。

## Adding a runtime test

要添加新的运行时测试,请首先确定哪个[测试组](https://github.com/antlr/antlr4/blob/master/runtime-testsuite/test/org/antlr/v4/test/runtime/descriptors)它属于。然后,通过子类化以下方法之一,添加一个新的[RuntimeTestDescriptor](https://github.com/antlr/antlr4/blob/master/runtime-testsuite/test/org/antlr/v4/test/runtime/RuntimeTestDescriptor.java)实现。 :

* [BaseParserTestDescriptor](https://github.com/antlr/antlr4/blob/master/runtime-testsuite/test/org/antlr/v4/test/runtime/BaseParserTestDescriptor.java);参见示例[APlus](https://github.com/antlr/antlr4/blob/master/runtime-testsuite/test/org/antlr/v4/test/runtime/descriptors/ParserExecDescriptors.java#L7)。
* [BaseDiagnosticParserTestDescriptor](https://github.com/antlr/antlr4/blob/master/runtime-testsuite/test/org/antlr/v4/test/runtime/BaseDiagnosticParserTestDescriptor);如果要测试解析器诊断输出;参见[示例输出](https://github.com/antlr/antlr4/blob/master/runtime-testsuite/test/org/antlr/v4/test/runtime/descriptors/FullContextParsingDescriptors.java#L16)。
* [BaseCompositeParserTestDescriptor](https://github.com/antlr/antlr4/blob/master/runtime-testsuite/test/org/antlr/v4/test/runtime/BaseCompositeParserTestDescriptor.java);参见示例[BringInLiteralsFromDelegate](https://github.com/antlr/antlr4/blob/master/runtime-testsuite/test/org/antlr/v4/test/runtime/descriptors/CompositeParsersDescriptors.java#L11)
* [BaseLexerTestDescriptor](https://github.com/antlr/antlr4/blob/master/runtime-testsuite/test/org/antlr/v4/test/runtime/BaseLexerTestDescriptor.java);参见示例[ActionPlacement](https://github.com/antlr/antlr4/blob/master/runtime-testsuite/test/org/antlr/v4/test/runtime/descriptors/LexerExecDescriptors.java#L12)。
* [BaseCompositeLexerTestDescriptor](https://github.com/antlr/antlr4/blob/master/runtime-testsuite/test/org/antlr/v4/test/runtime/BaseCompositeLexerTestDescriptor.java);参见示例[LexerDelegatorInvokesDelegateRule](https://github.com/antlr/antlr4/blob/master/runtime-testsuite/test/org/antlr/v4/test/runtime/descriptors/CompositeLexersDescriptors.java#L11)


每个描述符对象都描述了以下用于测试的必需元素:

 *测试类型
 * 语法 
 *起始规则
 *要解析或输入的输入文本
 *预期输出
 *预期错误

最好的选择是在适当的组中找到一个相似的测试,然后复制并粘贴描述符对象,在测试组类中创建一个新的嵌套类。修改字段定义以适合您的新问题。

如果您需要创建一个全新的测试组,则需要一个新的描述符类。称之为`XDescriptors`。然后,在每个[目标子目录](https://github.com/antlr/antlr4/tree/master/runtime-testsuite/test/org/antlr/v4/test/runtime)中,您需要创建一个新的测试装备`TestX.java`文件:

```java
package org.antlr.v4.test.runtime.java;

import org.antlr.v4.test.runtime.BaseRuntimeTest;
import org.antlr.v4.test.runtime.RuntimeTestDescriptor;
import org.antlr.v4.test.runtime.descriptors.ListenersDescriptors;
import org.junit.runner.RunWith;
import org.junit.runners.Parameterized;

@RunWith(Parameterized.class)
public class TestX extends BaseRuntimeTest {
	public TestX(RuntimeTestDescriptor descriptor) {
		super(descriptor,new Base<TARGET>Test());
	}

	@Parameterized.Parameters(name="{0}")
	public static RuntimeTestDescriptor[] getAllTestDescriptors() {
		return BaseRuntimeTest.getRuntimeTestDescriptors(XDescriptors.class, "<TARGET>");
	}
}
```

其中,在各个子目录中,`<TARGET>`被Java,Cpp,CSharp,Python2等替换。
 
###忽略测试

为了关闭针对特定目标的测试,我们需要使用`ignore`方法。给定目标名称,描述符对象可以决定是否忽略测试。这并不总是很方便,但是它是完全通用的,并且对于我们现在不得不忽略除JavaScript之外的所有目标中的`Visitor`测试的一种情况,效果很好。

###目标API /库测试

需要使用专门用目标语言编写的代码来测试运行时API的某些部分。例如,您可以在此处查看所有Java运行时API测试:

[https://github.com/antlr/antlr4/tree/master/runtime-testsuite/test/org/antlr/v4/test/runtime/java/api](https://github.com/antlr/antlr4/树/主/运行时测试套件/测试/组织/antlr/v4 /测试/运行时/java/api)

请注意,它在`api`目录下。上面的目录是所有`Test*`文件所在的目录。

###语法中嵌入的跨语言操作

要得到:

```
System.out.println($set.stop);
```

改用与语言无关的语言:

```
<writeln("$set.stop")>
```

模板文件[runtime-testsuite/resources/org/antlr/v4/test/runtime/templates/Java.test.stg](https://github.com/antlr/antlr4/tree/master/runtime-testsuite/resources/org/antlr/v4/test/runtime/templates/Java.test.stg)具有以下模板:

```
writeln(s) ::= <<System.out.println(<s>);>>
```

将通用运算转换为特定于目标的语言语句或表达式。

## Adding an ANTLR tool unit test

只需进入dir [antlr4/tool-testsuite/test/org/antlr/v4/test/tool]目录中的相应Java测试类即可(https://github.com/antlr/antlr4/tree/master/tool-testsuite/测试/组织/antlr/v4 /测试/工具)并添加您的单元测试。