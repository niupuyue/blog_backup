---
title: android奇技淫巧 09 gradle统一配置版本
date: 2018-07-11 22:42:10
tags:
  - gradle配置
---

在项目开发中，随着项目的逐渐庞大，引用的依赖库也会随之增多，那么我们需要有一个文件来统一管理gradl的版本

<!--more-->

在Project/build.gradle中定义，在module/build.gradle中使用

我们可以单独配置一个config.gradle的文件，在这个文件中，将我们所有需要使用到的版本进行统一管理


