# Cutting an ANTLR Release

## Github

在github上创建预发行版或完整发行版;[示例4.5-rc-1](https://github.com/antlr/antlr4/releases/tag/4.5-rc-1)。

###删除现有的发行标签

破坏现有的标签,因为mvn会创建一个,如果已经存在,它将失败。

```
$ git tag -d 4.8
$ git push origin :refs/tags/4.8
$ git push upstream :refs/tags/4.8
```

###创建发布候选标签

```bash
$ git tag -a 4.8-rc1 -m 'heading towards 4.8'
$ git push origin 4.8-rc1
$ git push upstream 4.8-rc1
```

## Update submodules

确保您告诉git引入子模块(对于您对antlr4做的每个克隆):

```bash
git submodule init
```

在`runtime/PHP/src/RuntimeMetaData.php`中也将版本提高到4.8。

通过运行以下命令来更新运行时子模块:

```bash
git submodule update --recursive
git submodule update --remote --merge # might only need this last one but do both
```

确保这些更改返回到antlr4存储库:

```bash
git add runtime/PHP
git commit -m "Update PHP Runtime to latest version"
```

## Bump version

编辑存储库以查找4.5或其他内容并进行更新。以下文件中的凹凸版本:

 *运行时/Java/src/org/antlr/v4/runtime/RuntimeMetaData.java
 * runtime/Python2/setup.py
 * runtime/Python2/src/antlr4/Recognizer.py
 * runtime/Python3/setup.py
 *运行时/Python3/src/antlr4/Recognizer.py
 *运行时/CSharp /运行时/CSharp/Antlr4.Runtime/Properties/AssemblyInfo.cs
 *运行时/CSharp/runtime/CSharp/Antlr4.Runtime/Antlr4.Runtime.dotnet.csproj
 * runtime/JavaScript/package.json
 *运行时/JavaScript/src/antlr4/Recognizer.js
 *运行时/Cpp/VERSION
 * runtime/Cpp/runtime/src/RuntimeMetaData.cpp
 *运行时/Cpp/cmake/ExternalAntlr4Cpp.cmake
 *运行时/Cpp/demo/generate.cmd
 *运行时/Go/antlr/recognizer.go
 *运行时/Swift/Antlr4/org/antlr/v4/runtime/RuntimeMetaData.swift
 *工具/src/org/antlr/v4/codegen/target/GoTarget.java
 *工具/src/org/antlr/v4/codegen/target/CppTarget.java
 *工具/src/org/antlr/v4/codegen/target/CSharpTarget.java
 *工具/src/org/antlr/v4/codegen/target/JavaScriptTarget.java
 *工具/src/org/antlr/v4/codegen/target/Python2Target.java
 *工具/src/org/antlr/v4/codegen/target/Python3Target.java
 *工具/src/org/antlr/v4/codegen/target/SwiftTarget.java
 *工具/src/org/antlr/v4/codegen/Target.java
 *工具/资源/org/antlr/v4/tool/templates/codegen/Swift/Swift.stg
 
这是一个简单的脚本,用于显示关键文件中包含`4.5`的任何行:

```bash
find tool runtime -type f -exec grep -l '4\.6' {} \;
```

提交到存储库。

## Building

啊。显然,您必须先`mvn install`然后再`mvn compile`或某些此类或子目录pom.xml不会看到最新的运行时版本。

## Maven Repository Settings

首先,请确保您已设置maven与登台服务器等进行通信。...创建文件`~/.m2/settings.xml`并使用适当的用户名/密码登台服务器,并使用gpg.keyname/passphrase进行签名。确保它对您只有严格的可见性特权。在Unix上,它看起来像:

```bash
beast:~/.m2 $ ls -l settings.xml 
-rw-------  1 parrt  staff  914 Jul 15 14:42 settings.xml
```

这是文件模板

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--
  User-specific configuration for maven. Includes things that should not
  be distributed with the pom.xml file, such as developer identity, along with
  local settings, like proxy information.
-->
<settings>
   <servers>
        <server>
          <id>sonatype-nexus-staging</id>
          <username>sonatype-username</username>
          <password>XXX</password>
        </server>
        <server>
          <id>sonatype-nexus-snapshots</id>
          <username>sonatype-username</username>
          <password>XXX</password>
        </server>
   </servers>
    <profiles>
            <profile>
              <activation>
                    <activeByDefault>false</activeByDefault>
              </activation>
              <properties>
                    <gpg.keyname>UUU</gpg.keyname>
                    <gpg.passphrase>XXX</gpg.passphrase>
              </properties>
            </profile>
    </profiles>
</settings>
```

## Maven deploy snapshot

目标是将快照(例如`4.8-SNAPSHOT`)获取到登台服务器:[antlr4工具](https://oss.sonatype.org/content/repositories/snapshots/org/antlr/antlr4)和[ antlr4 Java运行时](https://oss.sonatype.org/content/repositories/snapshots/org/antlr/antlr4-runtime)。

做这个:

```bash
$ mvn deploy -DskipTests
...
[INFO] --- maven-deploy-plugin:2.7:deploy (default-deploy) @ antlr4-tool-testsuite ---
Downloading: https://oss.sonatype.org/content/repositories/snapshots/org/antlr/antlr4-tool-testsuite/4.8-SNAPSHOT/maven-metadata.xml
Uploading: https://oss.sonatype.org/content/repositories/snapshots/org/antlr/antlr4-tool-testsuite/4.8-SNAPSHOT/antlr4-tool-testsuite-4.8-20161211.173752-1.jar
Uploaded: https://oss.sonatype.org/content/repositories/snapshots/org/antlr/antlr4-tool-testsuite/4.8-SNAPSHOT/antlr4-tool-testsuite-4.8-20161211.173752-1.jar (3 KB at 3.4 KB/sec)
Uploading: https://oss.sonatype.org/content/repositories/snapshots/org/antlr/antlr4-tool-testsuite/4.8-SNAPSHOT/antlr4-tool-testsuite-4.8-20161211.173752-1.pom
Uploaded: https://oss.sonatype.org/content/repositories/snapshots/org/antlr/antlr4-tool-testsuite/4.8-SNAPSHOT/antlr4-tool-testsuite-4.8-20161211.173752-1.pom (3 KB at 6.5 KB/sec)
Downloading: https://oss.sonatype.org/content/repositories/snapshots/org/antlr/antlr4-tool-testsuite/maven-metadata.xml
Downloaded: https://oss.sonatype.org/content/repositories/snapshots/org/antlr/antlr4-tool-testsuite/maven-metadata.xml (371 B at 1.4 KB/sec)
Uploading: https://oss.sonatype.org/content/repositories/snapshots/org/antlr/antlr4-tool-testsuite/4.8-SNAPSHOT/maven-metadata.xml
Uploaded: https://oss.sonatype.org/content/repositories/snapshots/org/antlr/antlr4-tool-testsuite/4.8-SNAPSHOT/maven-metadata.xml (774 B at 1.8 KB/sec)
Uploading: https://oss.sonatype.org/content/repositories/snapshots/org/antlr/antlr4-tool-testsuite/maven-metadata.xml
Uploaded: https://oss.sonatype.org/content/repositories/snapshots/org/antlr/antlr4-tool-testsuite/maven-metadata.xml (388 B at 0.9 KB/sec)
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Summary:
[INFO] 
[INFO] ANTLR 4 ............................................ SUCCESS [  4.073 s]
[INFO] ANTLR 4 Runtime .................................... SUCCESS [ 13.828 s]
[INFO] ANTLR 4 Tool ....................................... SUCCESS [ 14.032 s]
[INFO] ANTLR 4 Maven plugin ............................... SUCCESS [  6.547 s]
[INFO] ANTLR 4 Runtime Test Annotations ................... SUCCESS [  2.519 s]
[INFO] ANTLR 4 Runtime Test Processors .................... SUCCESS [  2.385 s]
[INFO] ANTLR 4 Runtime Tests (2nd generation) ............. SUCCESS [ 15.276 s]
[INFO] ANTLR 4 Tool Tests ................................. SUCCESS [  2.233 s]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 01:01 min
[INFO] Finished at: 2016-12-11T09:37:54-08:00
[INFO] Final Memory: 44M/470M
[INFO] ------------------------------------------------------------------------
```

## Maven release

分阶段进行的Maven部署生命周期将ANTLR项目的工件和poms部署到[sonatype远程登台服务器](https://oss.sonatype.org/content/repositories/snapshots/)。

```bash
export JAVA_HOME=`/usr/libexec/java_home -v 1.7`; mvn deploy -DskipTests
```

使用JDK 1.7(不是6或8),请执行以下操作:

```bash
export JAVA_HOME=`/usr/libexec/java_home -v 1.7`; mvn release:prepare -Darguments="-DskipTests"
```

嗯...每https://github.com/keybase/keybase-issues/issues/1712我们都需要这样做才能使gpg正常工作:

```bash
export GPG_TTY=$(tty)
```

在OS X上设置JDK 1.7的旁注:

```bash
alias java='/Library/Java/JavaVirtualMachines/jdk1.7.0_21.jdk/Contents/Home/bin/java'
alias javac='/Library/Java/JavaVirtualMachines/jdk1.7.0_21.jdk/Contents/Home/bin/javac'
alias javadoc='/Library/Java/JavaVirtualMachines/jdk1.7.0_21.jdk/Contents/Home/bin/javadoc'
alias jar='/Library/Java/JavaVirtualMachines/jdk1.7.0_21.jdk/Contents/Home/bin/jar'
export JAVA_HOME=`/usr/libexec/java_home -v 1.7`
```

但是我认为在mvn作品前面只是这样:

```
export JAVA_HOME=`/usr/libexec/java_home -v 1.7`; mvn ...
```

在0xCAFEBABE之后,您应该在生成的.class文件中看到0x33。参见[Java SE 7 = 51(0x33 hex)](https://en.wikipedia.org/wiki/Java_class_file):

```bash
beast:/tmp/org/antlr/v4 $ od -h Tool.class |head -1
0000000      feca    beba    0000    3300    fa04    0207    0ab8    0100
```

首先询问您版本号:

```
...
What is the release version for "ANTLR 4"? (org.antlr:antlr4-master) 4.8: : 4.8
What is the release version for "ANTLR 4 Runtime"? (org.antlr:antlr4-runtime) 4.8: : 
What is the release version for "ANTLR 4 Tool"? (org.antlr:antlr4) 4.8: : 
What is the release version for "ANTLR 4 Maven plugin"? (org.antlr:antlr4-maven-plugin) 4.8: : 
What is the release version for "ANTLR 4 Runtime Test Generator"? (org.antlr:antlr4-runtime-testsuite) 4.8: : 
What is the release version for "ANTLR 4 Tool Tests"? (org.antlr:antlr4-tool-testsuite) 4.8: : 
What is SCM release tag or label for "ANTLR 4"? (org.antlr:antlr4-master) antlr4-master-4.8: : 4.8
What is the new development version for "ANTLR 4"? (org.antlr:antlr4-master) 4.8.1-SNAPSHOT:
...
```

Maven将检查您的pom.xml文件,以将版本从4.8-SNAPSHOT更新到4.8,以供发布,然后在发布后更新至4.8.1-SNAPSHOT,具体操作如下:

```bash
mvn release:perform -Darguments="-DskipTests"
```

Maven将使用git推送pom.xml更改。

现在,去这里:

&nbsp;&nbsp;&nbsp; [https://oss.sonatype.org/#welcome](https://oss.sonatype.org/#welcome)

然后在左侧单击“登台存储库”。您单击暂存库并关闭它,然后刷新,单击并释放它。当您在这里看到它时,它就完成了:

&nbsp;&nbsp;&nbsp; [https://oss.sonatype.org/service/local/repositories/releases/content/org/antlr/antlr4-runtime/4.8-1/antlr4-runtime-4.8-1.jar ](https://oss.sonatype.org/service/local/repositories/releases/content/org/antlr/antlr4-runtime/4.8-1/antlr4-runtime-4.8-1.jar)

所有版本都应该在这里:https://repo1.maven.org/maven2/org/antlr/antlr4-runtime/

将罐子复制到antlr.org网站并更新download/index.html

```bash
cp ~/.m2/repository/org/antlr/antlr4-runtime/4.8/antlr4-runtime-4.8.jar ~/antlr/sites/website-antlr4/download/antlr-runtime-4.8.jar
cp ~/.m2/repository/org/antlr/antlr4/4.8/antlr4-4.8-complete.jar ~/antlr/sites/website-antlr4/download/antlr-4.8-complete.jar
cd ~/antlr/sites/website-antlr4/download
git add antlr-4.8-complete.jar
git add antlr-runtime-4.8.jar 
```

现场更新:

* download.html
* index.html
* api/index.html
*下载/index.html
*脚本/topnav.js

```
git commit -a -m 'add 4.8 jars'
git push origin gh-pages
```

## Deploying Targets

### JavaScript

```bash
cd runtime/JavaScript
# git add, commit, push
```

**推送至npm **

```bash
cd runtime/JavaScript
npm login
npm publish antlr4
```

将目标移至网站

```bash
npm run build
cp /dist/antlr4.js ~/antlr/sites/website-antlr4/download
```

### CSharp

现在,我们有了[appveyor创建工件](https://ci.appveyor.com/project/parrt/antlr4/build/artifacts)。转到[nuget](https://www.nuget.org/packages/manage/upload)上传`.nupkg`。

###从Windows发布到Nuget

**安装前提条件**

当然,您需要安装Mono和`nuget`。在Mac上:

-.NET构建工具-可以从[此处](https://www.visualstudio.com/downloads/)加载
-nuget-下载[nuget.exe](https://www.nuget.org/downloads)
-dotnet-遵循[此处的说明](https://www.microsoft.com/net/core)

或者,您可以安装Visual Studio 2017,并确保选中.NET Core SDK的复选框。

您还需要在Windows“程序和功能”中启用.NET Framework 3.5支持。

如果一切正常,则以下命令将还原nuget软件包,为.NET Standard和.NET 3.5构建Antlr并创建nuget软件包:

```PS
msbuild /target:restore /target:rebuild /target:pack /property:Configuration=Release .\Antlr4.dotnet.sln /verbosity:minimal
```

这应该显示如下: 

**创建和包装装配**

```
Microsoft (R) Build Engine version 15.4.8.50001 for .NET Framework
Copyright (C) Microsoft Corporation. All rights reserved.

  Restoring packages for C:\Code\antlr4-fork\runtime\CSharp\runtime\CSharp\Antlr4.Runtime\Antlr4.Runtime.dotnet.csproj...
  Generating MSBuild file C:\Code\antlr4-fork\runtime\CSharp\runtime\CSharp\Antlr4.Runtime\obj\Antlr4.Runtime.dotnet.csproj.nuget.g.props.
  Generating MSBuild file C:\Code\antlr4-fork\runtime\CSharp\runtime\CSharp\Antlr4.Runtime\obj\Antlr4.Runtime.dotnet.csproj.nuget.g.targets.
  Restore completed in 427.62 ms for C:\Code\antlr4-fork\runtime\CSharp\runtime\CSharp\Antlr4.Runtime\Antlr4.Runtime.dotnet.csproj.
  Antlr4.Runtime.dotnet -> C:\Code\antlr4-fork\runtime\CSharp\runtime\CSharp\Antlr4.Runtime\lib\Release\netstandard1.3\Antlr4.Runtime.Standard.dll
  Antlr4.Runtime.dotnet -> C:\Code\antlr4-fork\runtime\CSharp\runtime\CSharp\Antlr4.Runtime\lib\Release\net35\Antlr4.Runtime.Standard.dll
  Successfully created package 'C:\Code\antlr4-fork\runtime\CSharp\runtime\CSharp\Antlr4.Runtime\lib\Release\Antlr4.Runtime.Standard.4.8.2.nupkg'.
```

**发布到NuGet **

您需要成为“ ANTLR 4 Standard Runtime”的NuGet所有者。
作为NuGet的注册用户,您可以在此处手动上传软件包:[https://www.nuget.org/packages/manage/upload](https://www.nuget.org/packages/manage/upload)

或者,您可以从cmd行发布。您需要从[https://www.nuget.org/account#](https://www.nuget.org/account#)获取NuGet密钥,然后从cmd行中键入:

```cmd
nuget push Antlr4.Runtime.Standard.<version>.nupkg <your-key> -Source https://www.nuget.org/api/v2/package
```

Nuget软件包也可以作为[AppVeyor构建](https://ci.appveyor.com/project/parrt/antlr4/build/artifacts)的工件进行访问。 

### Python

Python目标通过`setup.py`进行部署。首先,使用严格的权限设置`~/.pypirc`:

```bash
beast:~ $ ls -l ~/.pypirc
-rw-------  1 parrt  staff  267 Jul 15 17:02 /Users/parrt/.pypirc
```

```
[distutils] # this tells distutils what package indexes you can push to
index-servers =
    pypi
    pypitest

[pypi]
username: parrt
password: xxx

[pypitest]
username: parrt
password: xxx
```

然后运行通常的python设置程序:

```bash
cd ~/antlr/code/antlr4/runtime/Python2
# assume you have ~/.pypirc set up
python2 setup.py sdist upload
```

然后针对Python 3目标再次执行

```bash
cd ~/antlr/code/antlr4/runtime/Python3
# assume you have ~/.pypirc set up
python3 setup.py sdist upload
```

[download.html](http://www.antlr.org/download.html)中已经有指向工件的链接。

### C ++

C ++目标是最复杂的目标,因为它针对需要单独处理的多个平台。我们涵盖了4种情况:

* ** Windows **:VC ++运行时2017或2019(对应于Visual Studio 2017或2019)的静态和动态库+头文件。所有这一切都在32位和64位上,调试+发布。
* ** MacOS **:静态和动态版本库+头文件。
* ** iOS **:没有预编译的二进制文件,只是源代码的一个zip,包括用于从源代码构建所有内容的XCode项目。
* ** Linux **:没有预编译的二进制文件,而只是源代码的zip,包括cmake文件,用于从源代码构建所有内容。

从理论上讲,我们也可以为iOS创建一个库,但这需要对其进行签名,这取决于有效的iOS开发者帐户。因此,就像Linux构建一样,我们将其留给ANTLR用户来构建iOS库。

对于每个平台,都有一个部署脚本,该脚本生成zip存档并将其复制到目标文件夹。Windows部署脚本必须在安装了VS 2013 + VS 2015的计算机上运行。Mac脚本必须在装有XCode 7+的计算机上运行。可以在任何Linux或Mac机器上执行源脚本。

在Mac(安装了XCode 7+)上:

```bash
cd runtime/Cpp
./deploy-macos.sh
cp antlr4-cpp-runtime-macos.zip ~/antlr/sites/website-antlr4/download/antlr4-cpp-runtime-4.8-macos.zip
```

在任何Mac或Linux计算机上:

```bash
cd runtime/Cpp
./deploy-source.sh
cp antlr4-cpp-runtime-source.zip ~/antlr/sites/website-antlr4/download/antlr4-cpp-runtime-4.8-source.zip
```

在Windows计算机上,构建脚本检查是否已安装VS 2017和/或VS 2019,并为每个构建二进制文件(如果找到)。此脚本需要安装7z(http://7-zip.org,然后从DOS而不是Powershell中执行`set PATH=%PATH%;C:\Program Files\7-Zip\`)。

```bash
cd runtime/Cpp
deploy-windows.cmd Community
cp antlr4-cpp-runtime-vs2019.zip ~/antlr/sites/website-antlr4/download/antlr4-cpp-runtime-4.8-vs2019.zip
```

将目标移动到网站(如果需要,** _首先重命名为特定的ANTLR版本_ **):

```bash
pushd ~/antlr/sites/website-antlr4/download
# vi index.html
git add antlr4cpp-runtime-4.8-macos.zip
git add antlr4cpp-runtime-4.8-windows.zip
git add antlr4cpp-runtime-4.8-source.zip
git commit -a -m 'update C++ runtime'
git push origin gh-pages
popd
```

## Update javadoc for runtime and tool

首先,gen javadoc:

```bash
$ cd antlr4
$ mvn -DskipTests javadoc:jar install
```

然后复制到网站:

```bash
cd ~/antlr/sites/website-antlr4/api
git checkout gh-pages
git pull origin gh-pages
cd Java
jar xvf ~/.m2/repository/org/antlr/antlr4-runtime/4.8/antlr4-runtime-4.8-javadoc.jar
cd ../JavaTool
jar xvf ~/.m2/repository/org/antlr/antlr4/4.8/antlr4-4.8-javadoc.jar
git commit -a -m 'freshen api doc'
git push origin gh-pages
```

## Update Intellij plug-in

用新的antlr jar重建antlr插件。