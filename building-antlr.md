# Building ANTLR

大多数程序员不需要此页面上的信息,因为他们只需下载适当的jar或通过maven(通过ANTLR的antlr4-maven-plugin)使用ANTLR。如果您想分叉项目并修复错误或调整运行时代码的生成,那么几乎可以肯定,您需要构建ANTLR本身。有两个部分:

 1.以一种目标语言将语法编译成解析器和词法分析器的工具
 1.这些生成的解析器和词法分析器使用的运行时。

为了说明如何在本文档中构建ANTLR,我将假定根目录为`/tmp`。

*从4.6开始,ANTLR工具和Java目标运行时需要Java7。*

# Get the source

第一步是从github上的ANTLR 4存储库获取Java源代码。您可以从github下载存储库,但是最简单的方法就是在本地磁盘上克隆存储库:

```bash
$ cd /tmp
/tmp $ git clone https://github.com/antlr/antlr4.git
Cloning into 'antlr4'...
remote: Counting objects: 61480, done.
remote: Total 61480 (delta 0), reused 0 (delta 0), pack-reused 61480
Receiving objects: 100% (61480/61480), 31.24 MiB | 7.18 MiB/s, done.
Resolving deltas: 100% (32970/32970), done.
Checking connectivity... done.
Checking out files: 100% (1427/1427), done.
```

# Compile

```bash
$ cd /tmp/antlr4
$ export MAVEN_OPTS="-Xmx1G"   # don't forget this on linux
$ mvn clean                    # must be separate, not part of install/compile
$ mvn -DskipTests install
...
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Summary:
[INFO] 
[INFO] ANTLR 4 ............................................ SUCCESS [  0.287 s]
[INFO] ANTLR 4 Runtime .................................... SUCCESS [  4.915 s]
[INFO] ANTLR 4 Tool ....................................... SUCCESS [  1.315 s]
[INFO] ANTLR 4 Maven plugin ............................... SUCCESS [  2.393 s]
[INFO] ANTLR 4 Runtime Test Annotations ................... SUCCESS [  0.078 s]
[INFO] ANTLR 4 Runtime Test Processors .................... SUCCESS [  0.019 s]
[INFO] ANTLR 4 Runtime Tests (2nd generation) ............. SUCCESS [  1.986 s]
[INFO] ANTLR 4 Tool Tests ................................. SUCCESS [  0.513 s]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 12.005 s
[INFO] Finished at: 2016-11-21T11:42:42-08:00
[INFO] Final Memory: 52M/434M
[INFO] ------------------------------------------------------------------------
```

**注意:**作为工具测试,我们执行`install`而不是`compile`,这些是指必须从maven安装本地缓存中提取的模块。

# Installing libs to mvn cache locally

要跳过测试(要求安装所有目标语言)并**安装到本地存储库** `~/.m2/repository/org/antlr`,请执行以下操作:

```bash
$ export MAVEN_OPTS="-Xmx1G"     # don't forget this on linux
$ mvn install -DskipTests=true   # make sure all artifacts are visible on this machine
```

您应该看到这些罐子(构建4.6-SNAPSHOT时):

```bash
/Users/parrt/.m2/repository/org/antlr $ find antlr4* -name '*.jar'
antlr4-maven-plugin/4.6-SNAPSHOT/antlr4-maven-plugin-4.6-SNAPSHOT.jar
antlr4-runtime-test-annotation-processors/4.6-SNAPSHOT/antlr4-runtime-test-annotation-processors-4.6-SNAPSHOT.jar
antlr4-runtime-test-annotations/4.6-SNAPSHOT/antlr4-runtime-test-annotations-4.6-SNAPSHOT.jar
antlr4-runtime-testsuite/4.6-SNAPSHOT/antlr4-runtime-testsuite-4.6-SNAPSHOT-tests.jar
antlr4-runtime-testsuite/4.6-SNAPSHOT/antlr4-runtime-testsuite-4.6-SNAPSHOT.jar
antlr4-runtime/4.6-SNAPSHOT/antlr4-runtime-4.6-SNAPSHOT.jar
antlr4-tool-testsuite/4.6-SNAPSHOT/antlr4-tool-testsuite-4.6-SNAPSHOT.jar
antlr4/4.6-SNAPSHOT/antlr4-4.6-SNAPSHOT-tests.jar
antlr4/4.6-SNAPSHOT/antlr4-4.6-SNAPSHOT.jar
```

请注意,ANTLR本身就是编写的,这就是为什么maven为增强4.6-SNAPSHOT目的而下载antlr4-4.5.jar的原因。

# Testing tool and targets

请参阅[ANTLR项目单元测试](antlr-project-testing.md)。


# Building without testing

要在不运行测试的情况下进行构建(节省大量时间),请执行以下操作:

```bash
$ mvn -DskipTests install
```

## Building ANTLR in Intellij IDE

下载ANTLR源代码后,只需“从现有资源导入项目”,然后单击IDE右侧装订线中的“ Maven项目”选项卡。它应该在后台自动构建东西,如下所示:

<img src = images/intellij-maven.png宽度= 200>