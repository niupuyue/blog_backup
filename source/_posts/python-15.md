---
title: 拥抱Python（十五） 小例子--格式化文件中的字符串
date: 2019-11-14 22:06:10
tags:
  - Python
---

昨天公司的同事问我会不会python语言，帮他写一个脚本，然后可以将文本中的数据转换成另外一种格式，并且生成新的文件。我想自己学习python语言也有一段时间了，但是却从来没有能够应用到实际中，这是一个机会，于是便应承下来了。然后自己发现，不管你觉得你的基础知识学习的如何，在没有应用到实际之前，都不要夸下海口，或者自以为是。因为能够将一个语言，解决实际的问题，这才是我们真正需要做的事情。

<!--more-->

下面就将具体需要做的事情做一下总结：
需要的是将一个文件夹中所有的文本文件中的字符串，转换成响应的特殊格式
原有的数据格式
```
{type:dislikeJob,jobID:ILDivIuNu369Y3Uo,jobName:文职培训岗绝不加班丨无责6.8K丨入职6险一金,tags:薪资不适合,content:测试一下,createTime:1572263526664}
```
最终解析的数据结果
```
dislikeJob	ILDivIuNu369Y3Uo	文职培训岗绝不加班丨无责6.8K丨入职6险一金	薪资不适合	测试一下	157226352666
```
中间使用tab键

代码：
```
# conding=utf8
import os

def formatValue(line):
    result = ""
    start = int(line.find("type") + 5)
    end = int(line.find("jobID") - 1)
    type = line[start:end]
    start = int(line.find("jobID") + 6)
    end = int(line.find("jobName") - 1)
    jobID = line[start:end]
    start = int(line.find("jobName") + 8)
    end = int(line.find("tags") - 1)
    jobName = line[start:end]
    start = int(line.find("tags") + 5)
    end = int(line.find("content") - 1)
    tags = line[start:end]
    start = int(line.find("content") + 8)
    end = int(line.find("createTime") - 1)
    content = line[start:end]
    start = int(line.find("createTime") + 11)
    end = int(len(line) - 2)
    createTime = line[start:end]
    result = type + '\t' + jobID + '\t' + jobName + '\t' + tags + '\t' + content + '\t' + createTime
    return result

# 修改需要遍历的文件夹目录
g = os.walk("D:\\files\\001")
# 创建一个新目录用来存放输出结果
outRoot = os.path.exists("D:\\outputfiles")
if not outRoot:
    # 如果新目录不存在，则创建新目录
    os.mkdir("D:\\outputfiles")
for path, dir_list, file_list in g:
    for file_name in file_list:
        # 判断文件夹在目标目录中是否存在
        file = open(os.path.join(path, file_name), encoding='utf-8', errors='ignore')
        line = file.readline()
        while line:
            # 此处line就是每一行所代表的的内容
            # result最后返回的数据，需要写入到新的文件中
            result = formatValue(line)
            # 需要数据写入到新的文件中
            # 生成新的文件名称
            outNewFileName = "survery-" + file_name + ".txt"
            newFile = open("D:\\outputfiles" + "\\" + outNewFileName, "a",encoding='utf-8')
            newFile.writelines(result)
            newFile.write("\n")
            line = file.readline()

```
