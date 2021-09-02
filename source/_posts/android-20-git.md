---
title: 重拾android路(十九) git
date: 2018-2-15 14:27:11
tags:
  - android
  - git
---

git介绍
<!--more-->
版本控制是一种记录一个或若干文件内容变化，以便将来查阅特定版本修订情况的系统。 除了项目源代码，你可以对任何类型的文件进行版本控制
有了它你就可以将某个文件回溯到之前的状态，甚至将整个项目都回退到过去某个时间点的状态，你可以比较文件的变化细节，查出最后是谁修改了哪个地方，从而找出导致怪异问题出现的原因，又是谁在何时报告了某个功能缺陷等等

# git命令
git的命令并不算多，而且很好记，并且在已有命令的基础上我们可以实现简写命令或者自定义命令
![git总览](/assets/git/git01.png)

# git使用
一般我们都是需要以客户端的形式完成git的学习的，只有少数情况需要我们自己去创建一个git仓库。这里我们一GitHub为例，帮我们实现一些基本的操作。

## git的安装
git的安装非常简单，Windows可以直接去公司官网下载安装[git官网](https://git-scm.com/),安装的过程全部采用默认安装的方式即可，不需要执行太多的操作。安装完成之后，就已经将Git添加的环境变量中，我们可以通过命令行检查当前git版本信息等操作
![git的Windows安装结果](/assets/git/git02.png)

Mac的安装也是找到当前的终端工具，在终端工具中输入```git --version```就会显示当前的git版本信息。mac默认是已经安装了git

## git的初始化操作
git的初始化分为两种方式，第一种是自己在本地创建新项目，然后通过git的初始化等操作，最终将git提交到GitHub中；第二种是将已有项目发布到GitHub中。其实还有另外一种，将服务器或GitHub中的项目拷贝到本地，而这个比较简单，自行Google即可。

## 获取Git仓库
1. 在现有目录中初始化仓库，进入项目目录中运行```git init```命令，该命令将创建一个名为.git的子目录
![git init](/assets/git/git03.png)
2. 从吴福气克隆一个现有的Git仓库：```git clone [url] ```

## 记录每次更新到仓库
1. 检查当前文件状态：```git status```
2. 提出更改(把他们添加到暂存区):```git add filename(针对指定文件)```,```git add .(所有文件)```,```git add *.txt(支持通配符)```
3. 忽略文件: ```.gitignore```文件
4. 提交更新: ```git commit-m "提交代码信息" ```
5. 跳过使用暂存区域更新的方式: ``` git commit -a -m "提交代码信息"```,这里的```-a```其实就是省略了```git add```命令
6. 移除文件：```git rm filename``` (从暂存区域移除，然后提交)
7. 对文件重命名： ```git mv README.md READ```

## 推送改动待远程仓库
- 如果你还没有克隆现有仓库，并欲将你的仓库连接到某个远程服务器，你可以使用如下命令添加：```·git remote add origin <server> ```,比如我们要让本地的一个仓库和 Github 上创建的一个仓库关联可以这样```git remote add origin https://github.com/Snailclimb/test.git```
- 将这些改动提交到远端仓库：```git push origin master ```(可以把 master 换成你想要推送的任何分支)

如此你就能够将你的改动推送到所添加的服务器上去了

## 远程仓库的移除和重命名
- 将test重命名为test1：```git remote rename test test1```
- 移除远程仓库test1：```git remote rm test1```

## 查看提交历史
在提交若干次更新之后，我们可能需要回顾一下提交历史，我们需要使用的命令是```git log```
当然如果我们只想看某一个人的，可以使用下面的命令
```
git log --author=paulniu
```

## 撤销操作
有时候我们提交完发现漏掉了几个文件没有添加，或作者提交信息写错了，这是我们可以用```---amend```选项的提交命令尝试重新提交
```
git log --amend
```
取消暂存的文件
```
git reset filename
```
撤销对文件的修改
```
git checkout --filename
```
如果我们想要丢弃本地的所有改动并且提交，可以得到服务器上的最新版本，并且将本地主分支指向该版本
```
git fetch origin
git reset --hard origin/master
```

## 分支
分支是用来将特性开发绝缘开来的。在你创建仓库的时候，master 是“默认的”分支。在其他分支上进行开发，完成后再将它们合并到主分支上。
我们通常在开发新功能、修复一个紧急 bug 等等时候会选择创建分支。单分支开发好还是多分支开发好，还是要看具体场景来说

创建一个名为dev的分支
```
git branch dev
```
切换当前分支到 test（当你切换分支的时候，Git 会重置你的工作目录，使其看起来像回到了你在那个分支上最后一次提交的样子。 Git 会自动添加、删除、修改文件以确保此时你的工作目录和这个分支最后一次提交时的样子一模一样）
```
git checkout dev
```
我们可以采用合并的写法
```
git checkout -b dev
```
常用的命令
1. 切换到主分支
```
git checkout master
```
2. 合并分支
```
git merge master(当前分支是dev分支，将master分支上的内容合并到dev分支上)
```
3. 推送到远程服务器上
```
git push origin
```
4. 查看远程分支
```
git remote -v
```
5. 更改远程仓库地址
```
git remote set-url origin 新地址
```

# 参考资料
[git入门](https://github.com/pengMaster/BestNote/blob/master/docs/tools/Git.md#git-%E4%BD%BF%E7%94%A8%E5%BF%AB%E9%80%9F%E5%85%A5%E9%97%A8)
