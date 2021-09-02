---
title: 拥抱Python（二） 基本语法概述
date: 2019-07-25 22:23:10
tags:
  - Python
---

Python变量
<!--more-->
变量是存储在内存中的值。这就意味着在创建变量的时候会在内存中开辟一个空间。
基于变量的数据类型，解释器会分配指定内存，并决定什么数据可以被存储在内存中。
因此，变量可以指定不同的数据类型，这些变量可以存储整数，小数或字符

# 变量赋值

变量赋值没什么好说的，就是通过<pre>=</pre>去执行赋值语句。这里需要注意的是，Python中有一个新的赋值方法，多个变量赋值，这个还是比较有意思的
```
counter = 100  # 赋值整型变量
mails = 10.01  # 赋值浮点类型
values = 'hello world'  # 赋值字符类型

print counter
print mails
print values

# 多个变量赋值
a, b, c = 1, 10.2, 'paulniu'

print a, b, c
```

# 标准数据类型

在内存中存储的数据可以有多种类型，例如，一个人的年龄可以用数字表示，姓名用字符表示
为了方便，在Python中定义了一些标准类型，用于存储各种类型的数据
1. Numbers(数字)
2. String(字符串)
3. List(列表)
4. Tuple(元组)
5. Dictionary(字典)

## Python数字

数字类型用于存储数值。
他们是不可改变的的数据类型，这意味着改变数字数据类型会分配一个新的对象
当你指定一个值时，Number对象就会被创建：
```
var1 = 1
var2 = 10
```

你也可以del语句删除一些对象的引用。如下所示
```
del var1
```

这个地方的操作和Java语言有着非常大的区别。我们都知道，在Java中如果一个变量不再去使用的时候，我们是不需要手动删除变量引用的。java的JVM会在合适的时间去帮我们清除这个内存碎片。但是在Python中，我们是可以自己去删除一些引用，以保证内存的合理使用。

Python支持的四种数字类型如下：

1. int：有符号整数
2. long：长整数，可以代表八进制和十六进制
3. float：浮点型
4. complex：复数

> 注意：long 类型只存在于 Python2.X 版本中，在 2.2 以后的版本中，int 类型数据溢出后会自动转为long类型。在 Python3.X 版本中 long 类型被移除，使用 int 替代。

## Python字符串

字符串或串(String)是由数字、字母、下划线组成的一串字符。
一般记做如下：
```
s = "abcdefghijklmnnopqrstuvwxyz"
```

这里需要特别说明一点，python的字串列表有2种取值顺序:
1. 从左到右索引默认0开始的，最大范围是字符串长度少1
2. 从右到左索引默认-1开始的，最大范围是字符串开头

![python去字符串列表](/assets/python01/python02.png)

```
ss = "hello world!"
print ss  # 输出完整字符串
print ss[0]  # 输出字符串中第一个字符
print ss[2:5]  # 数组字符串中第三个和第六个之间的字符串
print ss[2:]  # 输出字符串中第三个开始后面的字符串
print ss * 2  # 输出字符串两次
print ss + "test"  # 输出拼接字符串
print ss[-4:]
```

输入结果：
![python运行结果](/assets/python01/python03.png)

除了可以传两个参数之外，还可以传递第三个参数，也就是步长，如下所示：
```
# 设置步长
print("-" * 100)
ss1 = "abcdefghijklmnopqrstuvwxyz"
print(ss1[0:20:2])
```
运行结果：
![python运行结果](/assets/python01/python04.png)

## 列表

List（列表） 是 Python 中使用最频繁的数据类型。
列表可以完成大多数集合类的数据结构实现。它支持字符，数字，字符串甚至可以包含列表（即嵌套）。
列表用 [ ] 标识，是 python 最通用的复合数据类型。
列表中值的切割也可以用到变量 [头下标:尾下标] ，就可以截取相应的列表，从左到右索引默认 0 开始，从右到左索引默认 -1 开始，下标可以为空表示取到头或尾。

```
# 列表拼接

ss2 = ["hello", "world", "!"]
ss3 = [100, "python", "!"]
ss4 = "paulniu"
print("-" * 100)
print(ss2 + ss3)
```

运行结果：
![python运行结果](/assets/python01/python05.png)

## 元组

元组相当于list列表，但是只能赋值一次，不能再次修改

```
print("-" * 100)
ss5 = ("hello", "world", "!", 100);
print(ss5)
```

运行结果：
![python运行结果](/assets/python01/python06.png)

## 字典

字典(dictionary)是除列表以外python之中最灵活的内置数据结构类型。列表是有序的对象集合，字典是无序的对象集合。
两者之间的区别在于：字典当中的元素是通过键来存取的，而不是通过偏移存取。
字典用"{ }"标识。字典由索引(key)和它对应的值value组成。

```
print("-" * 100)
ss6 = {"key1": "value1", "name": "paulniu", "age": 27, 100: "classname", 101: 10001}
print(ss6)
```

运行结果：
![运行结果](/assets/python01/python07.png)

# Python数据类型转换方法

- int(x) 将数据转成int类型
- long(x) 将数据转成long类型
- float(x) 将数据类型转成float类型
- complex(x) 创建一个复数
- str(x) 将数据类型转成String类型
- repr(x) 将对象转化为供解释器读取的形式
- eval() 用来计算在字符串中的有效Python表达式,并返回一个对象
- tuple(s) 将序列 s 转换为一个元组
- list(s) 将序列 s 转换为一个列表
- set(s) 转换为可变集合
- dict(d) 创建一个字典。d 必须是一个序列 (key,value)元组。
- frozenset(s) 转换成不可变
- chr(x) 将一个整数转换为一个字符
- unichr(x) 将一个整数转换为Unicode字符
- ord(x) 将一个字符转换为它的整数值
- hex(x) 将一个整数转换为一个十六进制字符串
- oct(x) 将一个整数转换为一个八进制字符串
