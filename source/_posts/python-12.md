---
title: 拥抱Python（十二） 迭代器
date: 2019-08-08 21:51:10
tags:
  - Python
---
迭代器
<!--more-->

# 迭代器和生成器

迭代是Python最强大的功能之一，是访问集合元素的一种方式
迭代器是一个可以记住遍历位置的对象
迭代器对象从集合的第一个元素开始访问，知道所有元素被访问结束。迭代器只能往前走不能后退
迭代器有两个基本方法：iter()和next()
字符串，列表或元组对象都可以用于创建迭代器
```
list1 = [1, 2, 3, 4]
it1 = iter(list1)
print(next(it1))
print(next(it1))
```
# 迭代器对象可以使用常规for语句进行遍历
```
list2 = [2, 3, 4, 5]
it2 = iter(list2)
for x in it2:
    print(x, end=" ")
```

## 创建一个迭代器

把一个类作为一个迭代器使用需要在类中实现两个方法__iter__()和__next__()
我们都知道类对象中需要有构造函数，Python的构造函数为__init__(),他会在对象初始化的时候执行
__iter__()方法返回一个特殊的迭代器对象，这个迭代器对象实现了__next__()方法并通过StopIteration异常标识迭代的完成
__next__()方法会返回下一个迭代器对象
```
class Numbers:
    def __iter__(self):
        self.a = 1
        return self
    def __next__(self):
        x = self.a
        self.a += 1
        return x

myNumbers = Numbers()
myIter = iter(myNumbers)
print(next(myIter))
print(next(myIter))
print(next(myIter))
print(next(myIter))
print(next(myIter))
```

## StopIteration
StopIteration异常用于标识迭代的完成，防止出现无限循环的情况，在__next__()方法中我们可以设置在完成指定循环次数后触发StopIteration异常来结束迭代
```
class MyNumbers:
    def __iter__(self):
        self.a = 1
        return self

    def __next__(self):
        if self.a <= 20:
            x = self.a
            self.a += 1
            return x
        else:
            raise StopIteration

myNumber = MyNumbers()
myIter = iter(myNumber)
for x in myIter:
    print(x)
```

## 生成器

在Python中，使用了yield的函数被称为生成器
跟普通函数不同的是，生成器是一个返回迭代器的函数，只能用于迭代操作，更简单的理解生成器是一个迭代器
在调用生成器运行的过程中，每次遇到yield时函数会暂停并保存当前所有的运行信息，返回yield的值，并在下一次执行next()方法时从当前位置继续运行
调用一个生成器函数，返回的是一个迭代器对象
```
import sys
def fibonacci(n):# 生成器函数
    a,b,counter = 0,1,0
    while True:
        if (counter > n):
            return
        yield a
        a,b = b , a+b
        counter += 1
f = fibonacci(10) # f是一个迭代器，由生成器返回生成
while True:
    try:
        print(next(f),end=" ")
    except StopIteration:
        sys.exit()
```

什么时候使用yield
一个函数f，f的返回一个list，这个list是动态计算出来的(不管是数学上的计算还是逻辑上的读取格式化),并且这个list会很大(无论是否是固定很很大还是随着输入参数的增大而增大)，
这个时候，我们希望每次调用这个函数并且使用迭代器进行循环的时候一个一个的得到每个list元素而不是直接得到一个完整的list来节省内存，这个时候就可以使用yield

打个比方，yield有点像断点，加了yield的函数，每次执行到有yield的时候，会返回yield后面的值，并且函数会暂停，直到下次调用或迭代终止

