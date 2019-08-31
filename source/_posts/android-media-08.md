---
title: android 音视频开发(八)
date: 2019-08-31 09:30:10
tags:
  - android
  - 音视频
---

使用OpenGL ES绘制立方体，并且设置为彩色的

<!--more-->

先来看一下效果，因为我之前设置的颜色看起来并不是非常的明显，所以我在网上找了一个颜色
![绘制立方体](/assets/tools/tools-opengl-09.png)
下面我们将一步步的实现一个立方体效果

#### 构建立方体

从我们之前学习过的数学知识，我们知道立方体(这里以正方体为例)是有六个面的，那么就对应着六个定点，所以第一步，我们需要先把定点生命出来
```
private float[] vertexCoords = {
            -0.5f, 0.5f, 1f,
            -0.5f, -0.5f, 1f,
            0.5f, -0.5f, 1f,
            0.5f, 0.5f, 1f,
            -0.5f, 0.5f, 0f,
            -0.5f, -0.5f, 0f,
            0.5f, -0.5f, 0f,
            0.5f, 0.5f, 0f
    };
```

设置完六个定点之后，我们需要设置这六个点的索引，所谓的索引就是要依次找到各个点可能组成的三角形，如下所示
```
private static short[] indexCoords = {
            0, 1, 2, 0, 2, 3,// 上面
            0, 4, 5, 0, 5, 1,// 左面
            0, 4, 7, 0, 7, 3,// 后面
            4, 5, 7, 4, 5, 6,// 底面
            1, 2, 5, 2, 5, 6,// 前面
            2, 6, 7, 2, 3, 7// 右面
    };
```

如果我们只是用单一颜色，那么这个立方体看起来会比较怪异，所以我们这里给立方体设置了多个颜色
```
private float[] colorCoords = {
            0f,1f,0f,1f,
            0f,1f,0f,1f,
            0f,1f,0f,1f,
            0f,1f,0f,1f,
            1f,0f,0f,1f,
            1f,0f,0f,1f,
            1f,0f,0f,1f,
            1f,0f,0f,1f,
    };
```

#### 绘制立方体

设置完成定点和颜色之后，我们需要设置顶点着色器和片元着色器。

步骤其实就是那么四步

- 初始化坐标数据、索引数据、颜色数据，具体操作为将坐标数据、颜色数据分别写入到独自的FloatBuffer中，将索引数据写入到ShortBuffer中
- 创建OpenGL2.0程序，将顶点着色器和片元着色器加入到程序中，并链接程序。
- 使用创建的OpenGLES2.0程序，写入变换矩阵、顶点坐标数据及颜色数据。
- 索引法绘制出所有顶点坐标组成的三角形，得到一个立方体。

如果我们仅仅只做了以上事情，往往我们得不到一个正确的立方里，反而会出现比较奇怪的立方体，所以我们一般情况下是需要做一个深度测试的

这是因为我们没有开启深度测试GLES20.glEnable(GLES20.GL_DEPTH_TEST)，并在绘制前清除深度缓存导致GLES20.glClear(GLES20.GL_DEPTH_BUFFER_BIT)的。
加入后，我们即可得到正常的立方体


#### 深度测试
1. 什么是深度？
深度其实就是该象素点在3d世界中距离摄象机的距离（绘制坐标），深度缓存中存储着每个象素点（绘制在屏幕上的）的深度值！
深度值（Z值）越大，则离摄像机越远。
深度值是存贮在深度缓存里面的，我们用深度缓存的位数来衡量深度缓存的精度。深度缓存位数越高，则精确度越高，目前的显卡一般都可支持16位的Z Buffer，一些高级的显卡已经可以支持32位的Z Buffer，但一般用24位Z Buffer就已经足够了。
2. 为什么需要深度？
在不使用深度测试的时候，如果我们先绘制一个距离较近的物体，再绘制距离较远的物体，则距离远的物体因为后绘制，会把距离近的物体覆盖掉，这样的效果并不是我们所希望的。而有了深度缓冲以后，绘制物体的顺序就不那么重要了，都能按照远近（Z值）正常显示，这很关键。
实际上，只要存在深度缓冲区，无论是否启用深度测试，OpenGL在像素被绘制时都会尝试将深度数据写入到缓冲区内，除非调用了glDepthMask(GL_FALSE)来禁止写入。这些深度数据除了用于常规的测试外，还可以有一些有趣的用途，比如绘制阴影等等。
3. 启用深度测试
使用 glEnable(GL_DEPTH_TEST);
在默认情况是将需要绘制的新像素的z值与深度缓冲区中对应位置的z值进行比较，如果比深度缓存中的值小，那么用新像素的颜色值更新帧缓存中对应像素的颜色值。
但是可以使用glDepthFunc(func)来对这种默认测试方式进行修改。
其中参数func的值可以为GL_NEVER（没有处理）、GL_ALWAYS（处理所有）、GL_LESS（小于）、GL_LEQUAL（小于等于）、GL_EQUAL（等于）、GL_GEQUAL（大于等于）、GL_GREATER（大于）或GL_NOTEQUAL（不等于），其中默认值是GL_LESS。
一般来将，使用glDepthFunc(GL_LEQUAL);来表达一般物体之间的遮挡关系。
4. 启用了深度测试，那么这就不适用于同时绘制不透明物体。

实现代码：
```
public class MyOpenGLRenderCube implements GLSurfaceView.Renderer {

    private Context context;

    private int program;

    private int positionHandler;
    private int colorHandler;
    private int matrixHandler;

    private FloatBuffer vertexBuffer;
    private FloatBuffer colorBuffer;
    private ShortBuffer indexBuffer;

    private float[] projectMatrix = new float[16];
    private float[] viewMatrix = new float[16];
    private float[] mvpMatrix = new float[16];

    private float[] colorCoords = {
            0f,1f,0f,1f,
            0f,1f,0f,1f,
            0f,1f,0f,1f,
            0f,1f,0f,1f,
            1f,0f,0f,1f,
            1f,0f,0f,1f,
            1f,0f,0f,1f,
            1f,0f,0f,1f,
    };

    private float[] vertexCoords = {
            -0.5f, 0.5f, 1f,
            -0.5f, -0.5f, 1f,
            0.5f, -0.5f, 1f,
            0.5f, 0.5f, 1f,
            -0.5f, 0.5f, 0f,
            -0.5f, -0.5f, 0f,
            0.5f, -0.5f, 0f,
            0.5f, 0.5f, 0f
    };

    private static short[] indexCoords = {
            0, 1, 2, 0, 2, 3,// 上面
            0, 4, 5, 0, 5, 1,// 左面
            0, 4, 7, 0, 7, 3,// 后面
            4, 5, 7, 4, 5, 6,// 底面
            1, 2, 5, 2, 5, 6,// 前面
            2, 6, 7, 2, 3, 7// 右面
    };

    public MyOpenGLRenderCube(Context context) {
        this.context = context;

        // 设置顶点
        ByteBuffer bb = ByteBuffer.allocateDirect(vertexCoords.length * 4);
        bb.order(ByteOrder.nativeOrder());
        vertexBuffer = bb.asFloatBuffer();
        vertexBuffer.put(vertexCoords);
        vertexBuffer.position(0);

        // 设置颜色
        ByteBuffer cc = ByteBuffer.allocateDirect(colorCoords.length * 4);
        cc.order(ByteOrder.nativeOrder());
        colorBuffer = cc.asFloatBuffer();
        colorBuffer.put(colorCoords);
        colorBuffer.position(0);

        // 设置索引
        ByteBuffer dd = ByteBuffer.allocateDirect(indexCoords.length * 2);
        dd.order(ByteOrder.nativeOrder());
        indexBuffer = dd.asShortBuffer();
        indexBuffer.put(indexCoords);
        indexBuffer.position(0);
    }

    @Override
    public void onSurfaceCreated(GL10 gl10, EGLConfig eglConfig) {
        String vertexSource = ShaderUtils.readRawText(context, R.raw.vertex_shader_7);
        String fragmentSource = ShaderUtils.readRawText(context, R.raw.fragment_shader_7);
        program = ShaderUtils.createProgram(vertexSource, fragmentSource);
        if (program > 0) {
            positionHandler = GLES20.glGetAttribLocation(program, "position");
            colorHandler = GLES20.glGetAttribLocation(program, "color");
            matrixHandler = GLES20.glGetUniformLocation(program, "matrix");
        }
        // 开启深度测试
        GLES20.glEnable(GLES20.GL_DEPTH_TEST);
    }

    @Override
    public void onSurfaceChanged(GL10 gl10, int i, int i1) {
        float ratio = (float)i/i1;
        Matrix.frustumM(projectMatrix,0,-ratio,ratio,-1,1,3,20);
        Matrix.setLookAtM(viewMatrix,0,5.0f,5.0f,10.0f,0f,0f,0f,0f,1f,0f);
        Matrix.multiplyMM(mvpMatrix,0,projectMatrix,0,viewMatrix,0);
    }

    @Override
    public void onDrawFrame(GL10 gl10) {
        GLES20.glClear(GLES20.GL_COLOR_BUFFER_BIT | GLES20.GL_DEPTH_BUFFER_BIT);
        GLES20.glClearColor(0.5f,0.5f,0.5f,1f);

        GLES20.glUseProgram(program);

        GLES20.glEnableVertexAttribArray(positionHandler);
        GLES20.glVertexAttribPointer(positionHandler,3,GLES20.GL_FLOAT,false,0,vertexBuffer);

        GLES20.glEnableVertexAttribArray(colorHandler);
        GLES20.glVertexAttribPointer(colorHandler,4,GLES20.GL_FLOAT,false,0,colorBuffer);

        GLES20.glUniformMatrix4fv(matrixHandler,1,false,mvpMatrix,0);

        GLES20.glDrawElements(GLES20.GL_TRIANGLES,indexCoords.length,GLES20.GL_UNSIGNED_SHORT,indexBuffer);
    }
}
```
