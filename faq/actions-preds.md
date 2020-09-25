# Actions and semantic predicates

## How do I test if an optional rule was matched?

对于可选的规则引用,例如下面的初始化子句

```
decl : 'var' ID (EQUALS expr)? ;
```

可以使用`$EQUALS!=null`或`$expr.ctx!=null`进行测试以查看该子句是否匹配,其中`$expr.ctx`指向为该规则expr引用创建的上下文或解析树。