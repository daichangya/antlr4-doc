# Translation

## ASTs vs parse trees

我曾经做专门的AST(抽象语法树)节点,而不是(具体的)解析树,因为我曾经更多地考虑编译和生成字节码/汇编代码。当我开始更多地考虑翻译时,我开始使用解析树。对于v4,我意识到我主要从事翻译工作。我想我想说的是,解析树可能不如AST生成字节码那样好。就个人而言,我宁愿看到`(+ 3 4)`而不是`(expr 3 + 4)`来生成字节码,但这不是世界末日。(*有人可以填写吗?*)

## Decoupling input walking from output generation

我建议创建一个代表您的输出的中间模型。您遍历解析树以收集信息并创建模型。然后,您几乎可以肯定会自动遍历此内部模型,以根据与内部模型的类名匹配的字符串模板生成输出。换句话说,定义一个特殊的`IFStatement`对象,该对象具有所需的所有字段,然后在遍历解析树时创建它们。输入与输出的这种解耦非常强大。仅仅因为我们有一个分析树侦听器,并不意味着分析树本身必然是保存生成代码所需的所有信息的最佳数据结构。想象一下输出与输入完全相反的情况。在那种情况下,您真的想遍历输入只是为了收集数据。