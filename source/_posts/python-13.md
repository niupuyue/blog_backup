---
title: 拥抱Python（十三） 函数
date: 2019-08-09 21:51:10
tags:
  - Python
---
函数
<!--more-->

函数是组织好的，可以重复使用的，用来实现歹意，或者相关联功能的代码段
函数能提高应用的模块性和代码的重复利用率

## 定义一个函数

我们可以定义一个有自己想要功能的函数，规则如下：
1. 代码块以def关键字开头，后面是函数标识符名称和圆括号()
2. 任何传入参数和自变量必须放在括号中间，括号之间可以用于定义参数
3. 函数的第一行语句可以选择性的使用文档字符串--用于存放函数说明
4. 函数内容以冒号起始，并且缩进
5. return[表达式]结束函数，选择性的返回一个值给调用方，不带表达式的return相当于返回None 

## 语法

Python定义函数使用def关键字，一般格式如下：
def 函数名(参数列表):
    函数体

默认情况下，参数值和参数名称是按照函数生命中定义的顺序匹配起来的
```
def sayHello(name):
    print("hello world , %s" % (name))
sayHello("paulniu")
```

## 函数调用

定义一个函数：给了函数一个名称，指定函数里包含的参数，和代码块节奏
这个函数的基本结构完成以后，我们可以通过另一个函数调用执行，也可以直接从Python命令图书符执行

## 参数传递

在python中，定义属于对象吗，变量是没有类型的

## 可改变和不可改变对象

1. 在Python中，string，tuples和numbers是不可更改对象，而list和dict等正式可以修改的对象
不可变类型：变量赋值 a=5 后再赋值 a=10，这里实际是新生成一个 int 值对象 10，再让 a 指向它，而 5 被丢弃，不是改变a的值，相当于新生成了a。

2. 可变类型：变量赋值 la=[1,2,3,4] 后再赋值 la[2]=5 则是将 list la 的第三个元素值更改，本身la没有动，只是其内部的一部分值被修改了。

python 函数的参数传递：

1. 不可变类型：类似 c++ 的值传递，如 整数、字符串、元组。如fun（a），传递的只是a的值，没有影响a对象本身。比如在 fun（a）内部修改 a 的值，只是修改另一个复制的对象，不会影响 a 本身。
2. 可变类型：类似 c++ 的引用传递，如 列表，字典。如 fun（la），则是将 la 真正的传过去，修改后fun外部的la也会受影响

> Python中一切都是对象，严格意义我们不能说值传递还是引用传递，我们可以说传递对象和传递可变对象
```
def changeInt(a):
    a = 10
b = 2
changeInt(b)
```

示例中int对象的值为2，指向他的变量值b，在传递给changeInt函数时，传值的方式复制了变量b，a和b都指向同一个int对象，在a=10时，则新生成了一个int对象10，并把a指向他

## 传可变对象实例

可变对象在函数中修改了参数，那么在调用这个函数的函数里，原始的参数也会被修改
```
def changeMe(myList):
    myList.append([1,2,3,4,5])
    print("函数内取值为：",myList)
    return
myList = ['a','b','c','d']
changeMe(myList)
print("函数外取值为：",myList)
```

## 参数

以下是调用函数时可以使用的正式参数类型
1. 必须参数
2. 关键字参数
3. 默认参数
4. 不定长参数

### 必须参数

必须参数须以正确的顺序传入函数，调用的时候数量必须和声明的一样
```
def printMe(str):
    print(str)
    return
print("hello world!")
```

### 关键字参数

关键字参数和函数调用关系紧密，函数调用使用关键字参数来确定传入的参数值。使用关键字参数允许函数调用时参数顺序和声明时不一样，因为Python解析器能够用参数名匹配参数值
```
def printMe(str):
    print(str)
    return
printMe(str = "hello")

def printInfo(name, age):
    print("姓名是", name)
    print("年龄是", age)
    return
printInfo(age=28, name="paulniu")
```

### 默认参数

调用函数时，如果没有传递参数，会使用默认参数
```
def printInfo(name,age=18):
    print("姓名是",name)
    print("年龄是",age)
    return
printInfo(name="张三")
```

### 不定长参数

加了星号*的参数会以元组的形式导入，存放在所有未命名的变量参数
```
def printInfo(arg1,*var):
    print("arg1=",arg1)
    print(var)
    return
printInfo("hello world!",'a','b',1,2)
```

还有一种就是参数带两个星号**的变量会以字典的形式导入，基本语法如下：
```
def fucationname([formal_arg,]**var_args_dict):
    function_suite
    return [expression]
```
例子
```
def printInfo(arg1,**vardict):
    print(arg1)
    print(vardict)
    return
printInfo(1,a='a',b='b')
```

# 在使用的时候最好以变量名=变量值的方式调用函数

## 匿名函数

python使用lambda来创建匿名函数
所谓匿名函数，就是不适用关键字def
1. lambda只是一个表达式，函数体比def简单的多
2. lambda的主体是一个表达式，而不是代码块，仅仅能够在lambda表达式中封装有限的逻辑
3. lambda函数拥有自己的命名空间。且不能访问自己参数列表之外或者全局命名空间里的参数
4. 虽然lambda函数只有写一行，却不同于C++的内联函数，后者的目的是调用小函数时不占用栈内存而增加运行效率

```
sum = lambda var1,var2:var1 + var2
print(sum(100,100))
print(sum(200,300))
print(sum) # 错误示范
```
## return语句

return [表达式]语句用于退出函数，选择性的向调用方返回一个表达式，不带参数值的return语句返回None

