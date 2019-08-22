---
title: 拥抱Python（五） 数字
date: 2019-07-31 21:51:10
tags:
  - Python
---

Python 数字数据类型用于存储数值。
<!--more-->
数据类型是不允许改变的,这就意味着如果改变数字数据类型的值，将重新分配内存空间。

```
var1 = 10
var2 = 0.01
```

同时我们可以使用del去删除一些数字对象的引用，这里我们如果使用了del ，那么被删除的引用就不能在使用了

```
del var1, var2
```

Python支持三种类型的数字类型
1. 整型
2. 浮点型
3. 复数

var1 = 100
var2 = 10.01
var3 = complex(2, 3)

del var1, var2, var3

Python数据类型的转化

1. int(x) 将x转换成整数
2. float(x) 将x转换成浮点数
3. complex(x) 将x转换成复数，实数部分为x，虚数部分为0
4. complex(x,y) 将x，y转换到一个复数，实数部分为x，虚数部分为y

