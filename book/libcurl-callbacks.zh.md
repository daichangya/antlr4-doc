
## 回调

LIUCURL中的许多操作是通过使用*回调*. 回调是一个函数指针,它提供给libcurl,LIcCURL然后在某个时间点调用以完成特定的任务.

每个回调都有其特定的文档化用途,并且它要求您使用准确的函数原型来编写回调,以便接受正确的参数,并返回文档化的返回代码和返回值,以便libcurl按照您希望的方式执行.

每个回调选项也有一个同伴选项,用来设置关联的"用户指针".这个用户指针是一个libcurl不触碰或关心的指针,只是作为一个参数传递给回调.例如,这允许您一路传递指向本地数据的指针到回调函数.

除非在libcurl函数文档中明确声明,否则从libcurl回调中调用libcurl函数是不合法的.