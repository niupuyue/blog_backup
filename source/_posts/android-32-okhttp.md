---
title: 重拾android路(三十二) OkHttp
date: 2019-08-08 19:36:26
tags:
  - android
  - OkHttp
---
OKHttp网络请求

<!--more-->

目前公司所使用的网络请求框架依然是AsyncHttpClient，虽然说这款老的网络请求框架很好用，很容易理解，也很容易上手，但是开发技术总是在不断的创新和更新的，所以保持较高的对新技术的敏感度是非常重要的事情。
之前自己也有使用过OKHttp，只不过那时候使用的时间很少，大部分的还都是自己在业余时间自己去学习和使用。说到这点，可能别人都不相信一个做开发做了快五年的人，竟然还没有去深入学习过OKHttp，真是汗颜啊。所以我决定通过这一篇blog在学习使用OKHttp网络请求框架的基础功能的同时，深入学习OKHttp的核心内容，并且要能够深挖他的设计模式和设计思路，希望不会浪费自己所付出的努力。

# 基础部分

关于OKHttp的介绍这里就不多少了，[官网](https://square.github.io/okhttp/)上给出了比较详细的解读和分析，这里还有[github](https://github.com/square/okhttp)的网址
到目前为止OKHttp的版本已经更新到了4.x版本，使用的是Kotlin语言，而Kotlin语言我目前也正在学习。所以学习OKHttp的版本还是以3.x为主。

鉴于目前RxJava+OKHttp+Retrofit已经成为基本的网络请求框架，所以在这里我也是使用这样的方式去实现的。

在使用OKHttp基本操作之前，我们还是要先了解一下OKHttp的基本步骤，基本步骤一般是由四部分组成的

1. 创建OKHttpClient对象
2. 创建Request请求对象
3. 创建Call对象
4. 使用同步call.execute()或者一部call.enqueue(callback)开启网络请求


# 高级部分

# 封装OKHttp网络请求库
以上的所有使用都是在我们使用的时候直接创建OKHttpClient对象，那么可能这里会造成内容的泄露。除此之外的话，还有另外一些情况，比如当我们通过OKHttp进行网络请求的时候，可能需要对网络请求进行统一的处理，例如将所有的网络请求全部停止。那么这样我们需要对网络请求进行统一的管理，而显然官方的默认框架不能很好的满足我们的要求。除此之外，我们还可能需要对网络请求的头部需要统一管理，对抓包工具的支持等等。那么我们很有必须写一个自己封装的网络请求框架
> 注：本框架参考了[]()封装库


# 扩展
因为目前公司的项目是老项目，所以使用的是AsyncHttpClient，所以还是有必要将AsyncHttpClient重新整理一下的，我的想法是自己创建一个工具类，方便后面的时候。
工具的模块分为以下几个部分，先看图片
![](/assets/android_http/async_01.png)

主体代码：



# 参考博客
[OKHttp源码解读](https://juejin.im/post/5d6f9ee16fb9a06add4e4c5d?utm_source=gold_browser_extension)
