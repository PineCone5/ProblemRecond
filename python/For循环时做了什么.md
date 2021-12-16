# For循环时做了什么

使用for循环的形式一般为`for i in x:...`

执行这条语句时，运行顺序为：
<img src="image\for循环时程序流程图.png" width="90%">
> 注：从技术上讲，`for` 循环内部的调用情况等价于 `it_x.__next__`，而不是 `next(it_x)`，尽管二者非常相似。
