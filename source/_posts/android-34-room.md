---
title: 重拾android路(三十四) room
date: 2019-12-20 22:35:51
tags:
  - android
  - room数据库
---

关于数据库的部分，之前有写过一篇博客，当时使用的数据库是realm数据库。但是在使用了一段时间之后，发现他的使用比较复杂，而且在我们执行数据库语言的时候，总是不够智能化(难道说我用的是一个假的数据库？)，所以经过深思熟虑，决定使用room数据库作为项目的主要数据库框架。

<!--more-->

关于room数据库，网上有很多这样那样的教程，这里不做过多叙述，关键是如何使用，已经如何在项目中以比较灵活多变的形式使用room

## room数据库的引入

引入比较简单，只需要在gradle文件中添加如下依赖即可
```
// room
    compile "android.arch.persistence.room:runtime:1.1.1"
    annotationProcessor "android.arch.persistence.room:compiler:1.1.1"
```
目前好像room数据库有针对androidX进行更新，不过目前项目中并没有用到，所以也没有过多深入，后面如果有时间，再去学习吧。

引入完成之后，我们这里创建一个类名为AppDataBase的文件。