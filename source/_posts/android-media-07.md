---
title: android 音视频开发(七)
date: 2019-08-26 22:50:10
tags:
  - android
  - 音视频
---
学习Android平台的OpenGL ES API，了解OpenGL开发的基本流程，使用OpenGL绘制三角形
<!--more-->
先来看一张图
![Android架构图](/assets/tools/tools-opgl-01.png)

这里我们可以找到Libraries里面有我们目前要接触的库，即OpenGL ES。
根据上图可以知道Android 目前是支持使用开放的图形库的，特别是通过OpenGL ES API来支持高性能的2D和3D图形。OpenGL是一个跨平台的图形API。为3D图形处理硬件指定了一个标准的软件接口。OpenGL ES 是适用于嵌入式设备的OpenGL规范。
Android 支持OpenGL ES API版本的详细状态是：

> OpenGL ES 1.0 和 1.1 能够被Android 1.0及以上版本支持
> OpenGL ES 2.0 能够被Android 2.2及更高版本支持
> OpenGL ES 3.0 能够被Android 4.3及更高版本支持
> OpenGL ES 3.1 能够被Android 5.0及以上版本支持

## OpenGL ES的使用
这里有两个非常重要的概念需要先声明一下

### GLSurfaceView

GLSurfaceView从名字就可以看出，它是一个SurfaceView。看源码可知，GLSurfaceView继承自SurfaceView，并增加了Renderer，它的作用就是专门为OpenGL显示渲染使用的

### GLSurfaceView.Renderer

此接口定义了在GLSurfaceView中绘制图形所需的方法。您必须将此接口的实现作为单独的类提供，并使用GLSurfaceView.setRenderer()将其附加到您的GLSurfaceView实例。

GLSurfaceView.Renderer要求实现以下方法

1. onSurfaceCreated()：创建GLSurfaceView时，系统调用一次该方法。使用此方法执行只需要执行一次的操作，例如设置OpenGL环境参数或初始化OpenGL图形对象。
2. onDrawFrame()：系统在每次重画GLSurfaceView时调用这个方法。使用此方法作为绘制（和重新绘制）图形对象的主要执行方法。
3. onSurfaceChanged()：当GLSurfaceView的发生变化时，系统调用此方法，这些变化包括GLSurfaceView的大小或设备屏幕方向的变化。例如：设备从纵向变为横向时，系统调用此方法。我们应该使用此方法来响应GLSurfaceView容器的改变。

SurfaceView使用的一般步骤

1. 创建一个GlSurfaceView
2. 为这个GlSurfaceView设置渲染
3. 在GlSurfaceView.renderer中绘制处理显示数据



# 参考资料
[OpenGL ES API](https://www.jianshu.com/p/87abc92134dd)