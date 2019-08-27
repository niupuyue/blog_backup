---
title: android 音视频开发(六)
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

## 使用OpenGL绘制一些图像

### OpenGL 环境搭建
我们想要通过OpenGL来绘制一个图像，就必须先获取到OpenGL的容器，一般情况最简单最直接的办法就是直接使用GLSurfaceView和GLSurfaceView.Rendered。
GLSurfaceVIew是绘制图形的容器，GLSurfaceView.Rendered是控制绘制图形的内容

#### 在Manfiest文件中声明OpenGL
为了能够使用OpenGL 2.0 我们必须在清单配置文件中声明如下的内容
```
<uses-feature android:glEsVersion="0x00020000" android:required="true" />
```
如果我们的应用需要使用纹理压缩，那么我们也要为应用声明需要使用哪种压缩格式。
```
<supports-gl-texture android:name="GL_OES_compressed_ETC1_RGB8_texture" />
<supports-gl-texture android:name="GL_OES_compressed_paletted_texture" />
```

#### 创建一个Activity用于展示OpenGL
在一个使用了OpenGL的Activity和普通的Activity没什么不同，如果非要说不同，那么就是Activity的布局不同了。
这里跟大多数的blog是一样的，我们先在Activity中设置一个GLSurfaceView，然后让他展示黑色的背景样式

先来看Activity中的代码
```
public class OpenGLSimpleActivity extends AppCompatActivity {

    private MyGLSurfaveView surfaceView;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        surfaceView = new MyGLSurfaveView(this);
        setContentView(surfaceView);
    }
}
```
这段代码比较简单，这里我们声明了一个GLSurfaceVIew，这个GLSurfaceView是我自己定义的，然后创建GLSurfaceVIew实体对象，并且把这个GLSurfaceView对象作为布局设置给当前的Activity

其次是MyGLSurfaceView
```
public class MyGLSurfaveView extends GLSurfaceView {

    private MyGLSurfaceViewRendered renderer;

    public MyGLSurfaveView(Context context) {
        super(context);
        init();
    }

    private void init() {
        // 设置我们使用OpenGL的版本  2
        setEGLContextClientVersion(2);
        renderer = new MyGLSurfaceViewRendered();
        setRenderer(renderer);
    }
}
```
在这段代码中，我们创建了一个名为MyGLSurfaceView的类，并且让这个类继承了GLSurfaceView，在构造方法中我们需要设置使用的OpenGL的版本，并且创建一个Rendered对象，并且将这个对象设置成当前GLSurfaceView的Render。这里的Rendered对象是我们自己创建的，这样我们可以规定当前样式需要显示的方式，并且更好的管理各个不同的状态

最后是MyGLSurfaceViewRendered
```
public class MyGLSurfaceViewRendered implements GLSurfaceView.Renderer {
    public void onSurfaceCreated(GL10 unused, EGLConfig config) {
        // Set the background frame color
        GLES20.glClearColor(0.0f, 0.0f, 0.0f, 1.0f);
    }

    public void onDrawFrame(GL10 unused) {
        // Redraw background color
        GLES20.glClear(GLES20.GL_COLOR_BUFFER_BIT);
    }

    public void onSurfaceChanged(GL10 unused, int width, int height) {
        GLES20.glViewport(0, 0, width, height);
    }

}
```
在这个类里，我们让他实现了GLSurfaceView.Rendered接口，并且重写了三个方法，这三个方法中我们写的代码很少，主要实现是一个跟当前屏幕大小一样的黑色背景样式

#### OpenGL ES 定义形状

上面我们配置好了OpenGL的基本环境，但是想要进行进步的操作，其实是很困难的，我有很多次学习OpenGL都是卡在这一步，然后，就没有然后了。。。手动滑稽

首先我们来看一下我们需要了解的一些基本知识，这些知识包括OpenGL ES坐标系统，定义形状，环状面等

##### 定义三角形

OpenGL ES允许我们使用三维空间坐标系定义绘制图形，所以我们在绘制三角形之前需要先了解一下坐标系.
在OpenGL中，典型方法是为坐标定义浮点数的顶点数组。为了有更好的执行效率，我们可以将这些坐标写入byteBuffer，并且传递给OpenGL ES图形管道进行处理






# 参考资料
[OpenGL ES API](https://www.jianshu.com/p/87abc92134dd)