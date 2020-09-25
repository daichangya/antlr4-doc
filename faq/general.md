# General

## Why do we need ANTLR v4?

* Oliver Zeigermann向我询问了有关v4的一些问题。这是我们的谈话。*

*请参见[书中的序言](http://media.pragprog.com/titles/tpantlr2/preface.pdf)*

**问:为什么新版本的ANTLR也称为“蜜badge” **

ANTLR v4被称为YouTube感动的无所畏惧英雄The Crazy Nastyass Honey Badger。

**问:为什么您要创建新版本的ANTLR?**

好吧,我开始创建一个新版本,因为v3的内部非常混乱,并且还依赖于ANTLR v2编写的语法。不幸的是,v2的开源许可证尚不清楚,因此诸如Eclipse之类的项目由于不依赖v2而不能包含v3。最后,Sam Harwell将所有v2语法都转换为v3,以便将v3本身编写出来。由于v3拥有非常干净的BSD许可证,因此Eclipse项目可以在2011年夏季纳入该项目。

在重写ANTLR时,我想尝试LL(\ *)解析算法的新变体。幸运的是,我想出了一个很酷的新版本,称为自适应LL(\ *),它将所有语法分析工作推向了运行时。解析器像Java一样通过其即时JIT编译器进行预热。运行时间越长,代码就会变得越来越快。好处是自适应算法比v3中的静态LL(\ *)语法分析算法强得多。Honey Badger会采用您提供的任何语法;它只是不该死。(v4甚至接受左递归语法,但x调用y并调用x的间接左递归语法除外)。

v4是对解析器和解析器生成器进行25年研究的高潮。我想我终于知道我要构建什么。:)

**问:是什么让您对ANTLR4感到兴奋?**

最大的事情是新的自适应分析策略,该策略使我们能够接受我们希望编写的任何语法。这极大地提高了生产率,因为我们现在可以编写更多自然的表达规则(几乎在所有语法中都存在)。例如,自下而上的解析器生成器(例如yacc)可让您编写非常自然的语法,如下所示:

```
e : e '*' e
  | e '+' e
  | INT
  ;
```

ANTLR v4现在也将采用该语法,将其秘密翻译为非左递归版本。

v4的另一大优点是我的目标已从性能转移到易用性。例如,ANTLR可以自动为您构建解析树并生成侦听器和访问者。这不仅是巨大的生产力提升,而且是构建不依赖嵌入式动作的语法方面的重要一步。这些嵌入式动作(原始Java代码或其他任何动作)将语法锁定为仅使用一种语言。如果我们将所有动作放在语法之外,然后将它们放入外部访问者,则可以重用相同的语法,以使用我们有ANTLR目标的任何语言生成代码。

**问:您认为人们在ANTLR3中遇到什么问题?**

最大的问题是弄清楚为什么ANTLR不喜欢他们的语法。静态分析通常无法弄清楚如何为语法生成解析器。蜂蜜with完全解决了这个问题,因为它几乎可以不加任何提示地带给您任何东西。

**问:那其他的编译器生成器工具又如何呢?**

对于普通从业者而言,最大的问题是大多数解析器生成器不会生成可加载到调试器中并逐步执行的代码。这立即使普通程序员不再考虑自底向上的解析器生成器和真正强大的GLR解析器生成器。还有一些其他工具可以像ANTLR一样生成源代码,但是它们没有v4的自适应LL(\ *)解析器。您将不得不扭曲语法以适应工具较弱的需求,例如LL(k)解析策略。基于PEG的工具有许多缺点,但要提及的是,它们基本上没有错误恢复,因为它们无法报告错误,并且必须先解析整个输入。

**问:ANTLR4的主要设计决策是什么?**

易用性超过性能。稍后我会担心性能。复杂性上的简单性。例如,我删除了明确的/手动的AST构造工具和树语法工具。20年来,我一直在努力使人们朝着这个方向发展,但是自那时以来,我一直认为这是一个错误。最好给人们一个可以自动构建树的解析器生成器,然后让他们使用纯代码来完成他们想要的任何树遍历。例如,人们对访客极为熟悉和舒适。

**问:您认为人们最喜欢ANTLR4吗?**

通过ANTLR运行语法时没有错误。自动树结构和侦听器/访问者生成。

**您认为人们会尝试使用ANTLR4解决哪些问题?**

以我的经验,几乎没有人使用解析器生成器来构建商业编译器。因此,人们将ANTLR用于日常工作,构建从配置文件到小的脚本语言的所有内容。

为了回答来自stackoverflow.com的有关此条目的问题:我相信编译器开发人员非常关心解析速度,错误报告和错误恢复。为此,他们希望对解析器有绝对的控制权。而且,某些语言非常复杂,例如C ++,以至于解析器生成器的解析器生成速度可能比编译器开发人员所希望的慢。编译器开发人员还喜欢使用递归下降解析器来控制解析,以处理上下文敏感的构造,例如C ++中的`T(i)`。

也有一种可能认为解析是构建编译器的容易部分,因此它们不会立即自动跳转到解析器生成器。我认为这也是上一代解析器生成器的功能。McPeak的基于Elkhound GLR的解析器生成器足够强大和快速,可以在知道他们正在做什么的人的手中,适合于编译器。我还可以证明ANTLR v4现在足够强大和快速,足以与手工解析器竞争。例如,预热后,现在只需1秒即可解析整个JDK java/\ *库。

## What is the difference between ANTLR 3 and 4?

ANTLR 3和4之间的最大区别是ANTLR 4可以采用您提供的任何语法,除非该语法具有间接左递归。这意味着我们不需要语法谓词或回溯,因此ANTLR 4不支持该语法。您会收到使用警告。ANTLR 4允许直接向左递归,因此表达诸如算术表达式语法之类的内容非常简单自然:

```
expr : expr '*' expr
     | expr '+' expr
     | INT
     ;
```

ANTLR 4会自动为您构造解析树,并且不再可以选择抽象语法树(AST)构造。另请参见[例如,如果我不需要AST而不为编译器解析树怎么办?](https://github.com/antlr/antlr4/blob/master/doc/faq/parse-trees.md#what-if-我需要asts不解析树为一个编译器的例子)。

另一个很大的不同是,我们不鼓励直接在语法中使用动作,因为ANTLR 4会自动为您生成[听众和访问者](https://github.com/antlr/antlr4/blob/master/doc/listeners.md)在解析后的树行走过程中识别出某些感兴趣的短语时使用该触发器方法调用。另请参见[解析树匹配和XPath](https://github.com/antlr/antlr4/blob/master/doc/tree-matching.md)。

解析器和词法分析器规则中仍然允许语义谓词作为我们的操作。为了提高效率,请将语义谓词保持在词汇规则的右边。

没有树语法,因为我们改为使用侦听器和访客。

## Why is my expression parser slow?

确保使用两阶段解析。请参阅[错误报告](https://github.com/antlr/antlr4/issues/374)中的示例。

```Java

CharStream input = CharStreams.fromPath(Paths.get(args[0]));
ExprLexer lexer = new ExprLexer(input);
CommonTokenStream tokens = new CommonTokenStream(lexer);
ExprParser parser = new ExprParser(tokens);
parser.getInterpreter().setPredictionMode(PredictionMode.SLL);
try {
    parser.stat();  //STAGE 1
}
catch (Exception ex) {
    tokens.reset(); //rewind input stream
    parser.reset();
    parser.getInterpreter().setPredictionMode(PredictionMode.LL);
    parser.stat();  //STAGE 2
    //if we parse ok, it's LL not SLL
}
```