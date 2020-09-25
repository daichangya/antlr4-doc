# Installation

请仔细阅读:[ANTLR v4入门](https://raw.githubusercontent.com/antlr/antlr4/master/doc/getting-started.md)。

## Why can't ANTLR (grun) find my lexer or parser?

如果看到“无法将Hello加载为词法分析器或解析器”,那是因为您没有'。'。(当前目录)在CLASSPATH中。

```bash
$ alias antlr4='java -jar /usr/local/lib/antlr-4.2.2-complete.jar'
$ alias grun='java org.antlr.v4.runtime.misc.TestRig'
$ export CLASSPATH="/usr/local/lib/antlr-4.2.2-complete.jar"
$ antlr4 Hello.g4
$ javac Hello*.java
$ grun Hello r -tree
Can't load Hello as lexer or parser
$
```

对于mac/linux,使用:

```bash
export CLASSPATH=".:/usr/local/lib/antlr-4.2.2-complete.jar:$CLASSPATH"
```

或Windows:

```
SET CLASSPATH=.;C:\Javalib\antlr4-complete.jar;%CLASSPATH%
```

**看到开头的点吗?**这很关键。

## Why can't I run the ANTLR tool?

如果出现找不到类定义的错误,则说明`CLASSPATH`中缺少ANTLR jar(或者您可能只有运行时jar):

```bash
/tmp $ java org.antlr.v4.Tool Hello.g4
Exception in thread "main" java.lang.NoClassDefFoundError: org/antlr/v4/Tool
Caused by: java.lang.ClassNotFoundException: org.antlr.v4.Tool
    at java.net.URLClassLoader$1.run(URLClassLoader.java:202)
    at java.security.AccessController.doPrivileged(Native Method)
    at java.net.URLClassLoader.findClass(URLClassLoader.java:190)
    at java.lang.ClassLoader.loadClass(ClassLoader.java:306)
    at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:301)
    at java.lang.ClassLoader.loadClass(ClassLoader.java:247)
```

## Why doesn't my parser compile?

如果看到这些类型的错误,那是因为您的CLASSPATH中没有运行时或完整的ANTLR库。

```bash
/tmp $ javac Hello*.java
HelloBaseListener.java:3: package org.antlr.v4.runtime does not exist
import org.antlr.v4.runtime.ParserRuleContext;
                           ^
...
```