---
title: 拥抱Python（十四） 数据结构
date: 2019-08-21 21:51:10
tags:
  - Python
---

Python 列表
<!--more-->
Python中列表是可变的，这是它区别于字符串和元组的最重要的特点之一，一句话概括就是：列表可以修改，字符串和元组不能修改(其实这样说并不严谨)

1. list.append(x)  把一个元素添加到列表的结尾
2. list.extend(L)  通过添加置顶列表的所有元素来扩充列表
3. list.insert(i,x)  在置顶位置插入一个元素，i：表示准备插入到列表中的元素的索引，x：表示准备插入到列表中的位置下标
4. list.remove(x)  删除列表中值为x的第一个元素，如果没有这个元素，则会返回一个错误
5. list.pop([i])   从列表的置顶位置移除元素，并将其返回，如果没有指定索引，a.pop()返回最后一个元素。元素随机从列表中移除
6. list.clear()    移出列表中的所有项  等于del a[:]
7. list.index(x)   返回列表中第一个值为x的元素的索引，如果没有匹配到，则会返回一个错误
8. list.count(x)   返回x在列表中出现的次数
9. list.sort()     对列表中的元素进行排序
10. list.reverse()   倒排列表中的元素
11. list.copy()    返回列表的浅复制  等于a[:]

# 将列表当做堆栈使用

列表方法使得列表可以很方便的作为一个堆栈来使用，堆栈作为特定的数据结构，最先进入的元素最后一个被释放，用append()方法可以吧一个元素添加到堆栈顶。用不指定索引的pop()方法可以吧一个元素从堆栈顶释放出来
```
stack = [2, 3, 4]
stack.append(5)
stack.append(6)
print(stack)
stack.pop()
print(stack)

stack.clear()
```

# 将列表当做队列使用

可以将列表当做队列使用，只是队列里第一个加入的元素，第一个取出来，但是用列表这样做的效率不高。在列表的最后添加或者弹出元素的速度快，然而在列表中插入或者从头部弹出速度低

```
from collections import deque

queue = deque(["aaa", "bbb", "ccc"])
queue.append("ddd")
queue.append("eee")
print(queue.popleft())
print(queue.popleft())
print(queue)

queue.clear()
```

# 列表推导式

列表推导式提供了从序列创建列表的简单途径。通常应用程序将一些操作应用于某个序列的每个元素，用其获得的结果作为生成新列表的元素，或者根据确定的确定的判断条件创建子序列
新列表的元素，或者根据确定的判定条件创建子序列
每个列表推导式都在for之后跟一个表达式，然后有零到多个for或者if字句，返回结果是一个根据表达从其后的for和if上下文环境中生出的列表。如果希望表达式推导出一个元组，必须使用括号

```
vec = [2, 3, 4]
vec2 = [3, 4, 5]
print([2 * x for x in vec])
print([[x, x ** 2] for x in vec])

print("-" * 100)

freshfruit = ['  banana', '  loganberry ', 'passion fruit  ']
print([weapon.strip() for weapon in freshfruit])
```

# 我们也可以使用if子句作为过滤器

```
print("-" * 100)
print([3 * x for x in vec if x >= 3])
```

# 再来一个

```
print("-" * 100)
print([x * y for x in vec for y in vec2])
```