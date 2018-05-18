---
title: Android豆瓣电影项目记录
date: 2017-01-13 16:02:41
tags:
 - android
---

最近一段时间，公司要求我把之前上课的例子再重新构建一边。但是最近发现，如果一直用H5的混合开发的方式去构建的话，东西比较少，而且自己也想以后继续做Android开发，所以还是做成Android版本的比较好，这样也算是自己在对自己学过东西的一个回顾。

# 第一版本
第一版本的主体思想是，能在最短的时间内完成想要的功能。页面布局较为简单，没有添加太多的样式，最主要的是完成页面中的数据展示等功能。
使用时光网API，这个API实在GitHub上面找到的。数据请求格式比较简单，所以我就做了个比较简单的版本。
<!--more-->
用到的技术
1. android design 
2. android support-core-ui (recycleview,viewpager,tablayout)
3. <a href="https://github.com/scwang90/SmartRefreshLayout">SmartRefreshLayout</a>
4. <a href="https://github.com/hdodenhof/CircleImageView">circleimageview</a>
