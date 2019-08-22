---
title: 拥抱Python（七） 列表
date: 2019-08-03 21:51:10
tags:
  - Python
---
序列是Python中最基本的数据结构，序列中每个元素都分配一个数字--位置或者索引，从0开始计数
<!--more-->
Python有6个序列类型，其中最常见的是列表和元组
序列都可以进行的操作包括索引，切片，加，乘检查成员
此外，Python内置确定序列的长度以及确定最大值最小值元素的方法
列表是最常见的Python数据类型，他可以作为一个方括号内的逗号分隔值出现
列表的数据项不需要具有相同的类型
创建一个列表，只要用逗号隔开不同的数据项使用方括号括起来即可

### 访问列表中的值

使用下标索引来访问列表中的值，也可以使用方括号的形式截取字符

```
list1 = [1, 2, 3, 4]
list2 = ['a', 'b', 'c', 'd']
list3 = [100, 101, 'hello', 'world']
print(list1[1])
print(list3[3])
```

### 更新列表

我们可以对列表的数据项进行修改和更新，我们可以使用append()方法来添加列表项，如下

```
list1 = ['google','paulniu','hello',123,456]
print("第三个元素是",list1[2])
list1[2] = 100
print("更新后第三个元素是",list1[2])
```

### 删除列表元素

可以使用del语句来删除列表的元素

```
list1 = ['google','hello',1000,2000]
print("原来列表：",list1)
del list1[1]
print("删除之后列表：",list1)
```

### Python列表脚本操作符

列表对+和*的操作符和字符串相似，+用于组合列表，*用于重复列表

| Python表达式 | 结果 | 描述 |
| :---| :---| :---|
|len([1,2,3]) | 3 | 长度 |
|[1,2,3] + [4,5,6] | [1,2,3,4,5,6] | 组合 |
|["hello"] * 3 | ["hello","hello","hello"] | 重复 |
|3 in [1,2,3] | True | 判断元素是否在列表中 |
|for x in [1,2,3] :print(x,end=" ") | 1 2 3 | 迭代 |

### 嵌套列表
```
a = [1,2,3]
b= [4,5,6]
c = [a,b]
print("c列表的数据为",c)
print(c[1])
```

### Python列表函数和方法

|函数|作用|
|:---:|:---:|
|len(list)|列表元素的个数|
|max(list)|返回列表元素的最大值|
|min(list)|返回列表元素的最小值|
|list(seq)|将元组转换成列表|
|list.append(obj)|在列表末尾添加新对象|
|list.count(obj)|统计某个元素在列表中出现的次数|
|list.extend(seq)|在列表的末尾一次性追加另一个序列中的多值(用新列表扩展原列表)|
|list.index(obj)|从列表中找出某个值第一个匹配项的索引位置|
|list.insert(index,obj)|将对象插入列表中的某一个位置|
|list.pop([index=-1])|移除列表中的一个元素(默认是最后一个元素)，并且返回该元素的值|
|list.remove(obj)|移出列表中某个值的第一个匹配项|
|list.reverse()|反向列表中的元素|
|list.sort(key=None,reverse=False)|对原列表进行排序|
|list.clear()|清空列表|
|list.copy()|复制列表|

```
list = [1,2,3,4,5,6]
list2 = ['a','b','c','d']
list3 = ['a','1',True]
print("列表的长度是",len(list))
print("列表的最大值是",max(list))
print("列表的最小值是",min(list))
print("列表的最大值是",max(list2))
print("列表的最大值是",max(list3))

list = [1,2,3,4,5,6,7,8,9]
listTemp = ["hello","world"]
list.append(10)
print("新增一个元素",list)
print("统计2在列表中出现的次数",list.count(2))
list.extend(listTemp)
print("在列表末尾追加一个序列",list)
print("查找第一次出现2的下标",list.index(2))
list.insert(3,"new element")
print("在index=3的位置添加新的元素",list)
list.pop(4)
print("移除index=4的元素",list)
list.remove(7)
print("移除元素4",list)
list.reverse()
print("列表翻转",list)
list.clear()
print("清空列表",list)

l = [x for x in range(0,10)]
print(l[::3])

li = [i for i in range(1,10)]
print([x**2 for x in li])
print([x**2 for x in li if x >=5])
print(dict([(x,x*10) for x in li]))
```

