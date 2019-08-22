---
title: 拥抱Python（九） 字典
date: 2019-08-05 21:51:10
tags:
  - Python
---

字典
<!--more-->
字典是另一种可变容器模型，且可存储任意类型对象
字典的每个键值(key=value)对用冒号(:)分割，每个对之间用逗号隔开，字典是放在花括号里面的
键必须是唯一的，但值不必
值可以取任何数据类型，但键必须是不可变的，如字符串，数字，元组

```
dic = {"a":123,"b":"123",100:101,(1,2,3):"hello world"}
print(dic)
print(dic["a"])
print(dic[(1,2,3)])
```

### 修改字典

向字典添加新内容的方法是增加新的键值对，修改或删除已有的键值对

```
dict = {"name":"paulniu","age":27,"city":"BJ"}
dict["age"] = 28
dict['name'] = "牛谱乐"
print(dict)
```

### 删除字典元素

```
dict = {1:"hello",2:"world",3:9,"name":23,(1,2,3):[4,5,6]}
del dict[1]
print(dict)
# dict.clear()
del dict
print(dict)
```

### 字典键的特性

1. 不允许同一个键出现两次，创建时如果同一个键被赋值了两次，前面的值会被覆盖
2. 键必须不可变，所以只能是数字，字符串或者元祖，列表就是不行

### 字典内置函数

|函数|描述|
|:---|:---|
|len(dict)|计算字典元素个数，即键的总数|
|str(dict)|输出字典，以可打印的字符串表示|
|type(dict)|返回输入的变量类型，如果变量是字典则返回字典类型|
|dict.clear()|删除字典内所有元素|
|dict.copy()|返回一个字典的浅复制|
|dict.fromkeys()|创建一个薪字典，以序列seq中元素作为字典的键，val为字典所有键对应的初始值|
|dict.get(ke,default=None)|返回指定键的值，如果值不在字典中返回default值|
|key in dict| 如果键在字典dict中返回True，否则返回false|
|dict.items()|以列表返回可遍历的(键，值)元组数组||
|dict.keys()|返回一个迭代器，keyishiyonglist()来转换成列表|
|dict.setdefault(key,default=None)|和get()类似，但是如果键不存在于字典中，将会添加键并将值设置为default|
|dict.update(dict2)|把字典dict2的键值对更新到dict中|
|dict.values()|返回一个迭代器，可以shiyonglist()来转换成列表|
|dict.pop(key[,default])|删除字典给定键key所有对应的值，返回值为被删除的值，key值必须给出，否则返回default值|
|dict.popitem()|随机返回并删除字典中的一对键值(一般删除末尾)|

```
dict1 = {"name":"paulniu","age":27,"city":"beijing","home":"henan",(1,2,3):['a','b',("hello",["world"])]}
dict2 = {"name":"牛谱乐","school":"Guangzhou Un"}
tunple = ("address","phone",1,(1,2,3,4))
print("字典的长度是",len(dict1))
print(str(dict1))
print(type(dict1))
print("字典的浅复制",dict1.copy())
print("formkeys=",dict1.fromkeys(tunple))
print(dict1)
print("返回键为name的值",dict1.get("name"))
print('age' in dict1)
print(dict1.items())
print(dict1.keys())
dict1.update(dict2)
print(dict1)
print(dict1.values())
dict1.popitem()
print(dict1)
dict1.pop((1,2,3))
print(dict1)
```

字典是支持无线嵌套的，但是最好别太多层，比较难分析

### 例子
```
students = {}

write = 1
while write:
    name = str(input("请输入学生姓名"))
    age = int(input("请输入学生年龄"))
    students[str(name)] = age
    write = int(input("是否继续? \n 1/继续  0/结束 \n"))
# 输出最终结果
for key, value in students.items():
    if value > 90:
        print("%s    %s    A".center(20, "-") % (key, value))
    elif value >= 60 and value <= 90:
        print("%s    %s    B".center(20, "-") % (key, value))
    elif value < 60:
        print("%s    %s    C".center(20, "-") % (key, value))

```        