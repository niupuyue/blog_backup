---
title: 拥抱Python（六） 字符串
date: 2019-08-01 21:51:10
tags:
  - Python
---

字符串是 Python 中最常用的数据类型。我们可以使用引号( ' 或 " )来创建字符串。
<!--more-->
创建字符串很简单，只要为变量分配一个值即可

# Python中访问字符串的值
```
var1 = "abcdefghjiklmn"
print("var1[2] = " + var1[2])
print("var1[1:3] = " + var1[1:3])
del var1
```

# Python中更新字符串

```
var1 = "hello world!"
print("更新后的内容：" + var1[:6] + "Python!")
del var1
```

# Python的转义字符

```
\ 在行尾 换行符
\\ 反斜杠符号
\' 单引号
\" 双引号
\a 响铃
\b backspace按键
\000 空
\n 换行
\r 回车
\v 纵向制表符
\t 横向制表符
\f 换页
```

# Python字符串格式化

Python 支持格式化字符串的输出 。尽管这样可能会用到非常复杂的表达式，但最基本的用法是将一个值插入到一个有字符串格式符 %s 的字符串中

```
var1 = "ab%sdefghijklmn"
print(var1 % (" hahah "))
```

1. %c 格式化字符串以及ASCII码
2. %s 格式化字符串
3. %d 格式化整数
4. %u 格式化无符号整数
5. %o 格式化无符号八进制数
6. %x 格式化无符号十六进制数
7. %f 格式化浮点数，可指定符号后精确度
8. %e 用科学计数法格式化浮点数
9. %p 用十六进制数格式化地址

# Python的字符串内建函数

1. capitalize() 将字符串首字母变成大写
2. center(width, fillchar)    返回一个指定的宽度 width 居中的字符串，fillchar 为填充的字符，默认为空格。
3. isdigit()    是否只包含数字
4. islower()    如果字符串中包含至少一个区分大小写的字符，并且所有这些(区分大小写的)字符都是小写，则返回 True，否则返回 False
5. isnumeric()  如果字符串中只包含数字字符，则返回 True，否则返回 False
6. len()        返回字符串的长度
7. lstrip()     截掉字符串左边的空格或指定字符。
8. max()        返回字符串 str 中最大的字母。
9. replace(old, new [, max])     把 将字符串中的 str1 替换成 str2,如果 max 指定，则替换不超过 max 次。
10. rstrip()    删除字符串字符串末尾的空格.

