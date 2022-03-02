---
title: 重拾android路(四十一) SurfaceView和TextureView
date: 2020-08-22 16:35:51
tags:
  - android
  - surface
---
SurfaceView和TextureView
<!--more-->

> SurfaceView和TextureView都继承子android.view.View，属于Android提供的控件体系中的一员。与普通的View不同，他们都可以在独立线程中绘制和渲染。所以相比较于普通的ImageView，他们的性能更高，所以会应用到一些对绘制速率要求较高的场景中，用来解决普通View因为绘制耗时而带来的掉帧问题，比如相机预览，视频播放等

## Surface

> 简单理解就是在内存中的一段绘图缓冲区，在SDK文档中，对Surface的描述为”有屏幕显示内容合成器所管理的原生缓冲器句柄“。也就是说：通过Surface可以获取原生缓冲器以及其内容；原生缓冲器是用于保存当前窗口的像素数据

- Surface对应了一块屏幕缓冲区，每个Window对应一个Surface，任何View都是画在Surface上的，传统的View铜像一块屏幕缓冲区，所有的绘制都必须在UI线程中进行，不能直接操作Surface示例，要通过SurfaceHolder，在SurfaceView中可以通过**getHolder()**方法获取SurfaceHolder实例
- Surface是一个用来画图形的地方，但是我们的画图都是在一个Canvas对象上进行的，Surface中的Canvas成，是专门提供画图的地方，就像黑板一样，其中的原始缓冲区是用来保存数据的地方
- Surface本上的作用类似一个句柄，得到了这个句柄就可以得到其中的Canvas，原始缓冲区以及其他方面的内容，所以简单来说，Surface是用来管理数据的(句柄)

## SurfaceView应用

#### SurfaceView介绍

1. SurfaceView就是Surface的View里面嵌套了一个转本用于绘制的Surface，SurfaceView控制这个Surface的格式和尺寸以及绘制位置
2. SurfaceView就是在Window上挖一个洞，他就是显示在这个洞里，其他的View是显示在Window上，所以View可以显示在SurfaceView上，我们也可以添加一些层在SurfaceView上

```
if (mWindow == null) {  
    mWindow = new MyWindow(this);  
    mLayout.type = mWindowType;  
    mLayout.gravity = Gravity.LEFT|Gravity.TOP;  
    mSession.addWithoutInputChannel(mWindow, mWindow.mSeq, mLayout,  
    mVisible ? VISIBLE : GONE, mContentInsets);  
}
```

每个SurfaceVIew创建的时候都会创建一个MyWindow，其中在new这个对象时传递的**this**就是SurfaceView本身，因此将SurfaceView和Window丙丁在一起，而每个Window对应一个Surface
所以SurfaceView也就嵌套了一个自己的Surface，可以认为SurfaceView是来控制Surface的位置和尺寸。

