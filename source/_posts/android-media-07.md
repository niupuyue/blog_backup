---
title: android 音视频开发(七)
date: 2019-08-29 12:50:10
tags:
  - android
  - 音视频
---

使用OpenGL ES绘制正方形(多边形)，圆形，并且给这些图形填充颜色

<!--more-->

先看效果图
![使用OpenGL ES绘制多边形](/assets/tools/tools-opengl-05.png)
![使用OpenGL ES绘制圆形](/assets/tools/tools-opengl-06.png)

在开始绘制其他的形状之前，我们有这么几个名字/知识点学要学习的。

#### 矩阵
在数学中，矩阵是一个按照长方阵列排列的复数和实数集合，最早来自于方程组的系数和常数所构成的方阵。

矩阵常被用于图形处理，游戏开发，几何光学，量子态的线性组合以及电子学等。我们现在相当于在图形处理或者游戏开发的领域来使用矩阵。其实大学中我们很多人都学过高数，线性代码等课程，当时觉得没有用武之地，其实都是错误的，好后悔的说。在三维图形学中，一般使用的是四阶矩阵。在DirectX中使用的是行向量，如[xyzw]，所以与矩阵相乘是，向量在前矩阵在后。OpenGL中使用的是列向量，如[xyzw]T,所以与矩阵相乘时，矩阵在前向量在后。

#### 相机

和现实生活中的情况一样，随着我们相机的位置，姿态的不同，拍摄出来的画面也不一样。将相机对应到OpenGL的世界，决定相机拍摄的结果（也就是最终显示在屏幕上的内容），包括相机的位置，相机观察方向以及相机的UP方法

这里有几个名字需要解释一下：
1. 相机位置：这个比较好理解，就是相机在3D空间里面的坐标点
2. 相机观察方法：相机的观察方法，表示的是相机镜头的朝向，我们可以朝前，朝后，总之任何方法都是可以的
3. 相机UP方法：相机的UP方法可以理解为相机顶端指向的方法，，例如我们把相机斜着拿，拍出来的照片就是斜着的。

在OpenGL ES中我们可以使用下面的方法来设置相机
```
Matrix.setLookATM(
  float[] rm, // 接收相机变换矩阵
  int rmOffset, // 变换矩阵的起始位置(偏移量)
  float eysX,float eyeY,float eyeZ,// 相机的位置
  float centerX,float centerY,float centerZ,// 观测点的位置
  float upX,float upY,float upZ// up向量在xyz上的分量
)
```

#### 投影

用相机看到的3D世界，最后还需要呈现在2D平面上，这就是投影。其实投影的意思就是说，我们可以通过一些颜色的变换，让2D图像看起来像是3D的。在OpenGL ES中投影分为两种，正交投影和透视投影

1. 正交投影：物体呈现出来的大小不会随着其距离视点的远近而发生变化，在Android中我们使用以下的方法来设置正交投影
```
Matrix.orthoM(
  float[] m, // 接收正交投影的变换矩阵
  int mOffset, // 变换矩阵的起始位置(偏移量)
  float left, // 相对观察点近面的左边距
  float right,// 相对观察点近面的右边距
  float bottom,// 相对于观察点近面的下边距
  float top,// 相对于观察点的上边距
  float near,// 相对于观察点近面距离
  float far // 相对于观察点远面距离
)
```
2. 透视投影：物体离视点越远，呈现出来的越小，离视点越大，呈现出来的越大。我们使用下面的方法来设置透视投影：
```
Matrix.frustumM(
  float[] m,// 接收透视投影的变换矩阵
  int mOffset, // 变换矩阵的起始位置(偏移量)
  float left, // 相对于观察点近面的左边距
  float right,// 相对观察点近面的右边距
  float bottom,// 相对观察点近面的下边距
  float top,// 相对于观察点近面的上边距
  float near,// 相对观察点近面距离
  float far // 相对观察点远面距离
)
```

#### 使用变换矩阵
实际上相机设置和投影设置并不是真正的设置，而是通过设置参数，得到一个使用相机后顶点坐标的变换矩阵和投影下的顶点坐标变换矩阵，除此之外我们还要把矩阵传递给顶点着色器，在顶点着色器中用传入的矩阵乘以坐标的向量，得到时机展示的坐标向量。

> 注意，是矩阵乘以坐标向量，不是坐标向量乘以矩阵。矩阵乘法是不满足交换律的

通过上面的相机设置和投影设置，我们得到两个矩阵，为了方便，我们需要将相机矩阵和投影矩阵相乘，得到一个实际的变换矩阵，然后再传递给顶点着色器
```
Matrix.multiplyMM(
  float[] result,// 接收相乘结果
  int resultOffset,// 接收矩阵的起始位置(偏移量)
  float[] lhs, // 左矩阵
  int lhsOffset,// 左矩阵的起始位置(偏移量)
  float[] rhs, // 右矩阵
  int rhsOffset // 右矩阵的起始位置(偏移量)
)
```

### 绘制多边形

其实在OpenGL ES中没有多边形和圆形，只有点，线，三角形。三角形就是OpenGL所提供的的最复杂的图元单位。所以我们如果想要绘制多边形和圆形，就要通过三角形来实现

先来看一下代码

GLSurfaceView部分代码
```
public class MyOpenGLSurfaceViewWithSquare extends GLSurfaceView {
    public MyOpenGLSurfaceViewWithSquare(Context context) {
        super(context);
        setEGLContextClientVersion(2);
        setRenderer(new MyOpenGLRenderSquare(context));
    }
}
```
这个部分的代码比较简单，就不多说了

Render部分代码
```
public class MyOpenGLRenderSquare implements GLSurfaceView.Renderer {

    private Context context;
    private FloatBuffer vertexBuffer;
    private FloatBuffer colorBuffer;
    private ShortBuffer indexBuffer;

    private int positionHandler;
    private int colorHandler;
    private int matrixHandler;

    private int program;

    private float[] projectMatrix = new float[16];
    private float[] viewMatrix = new float[16];
    private float[] mvpMatrix = new float[16];

    private float[] squareCoords = {
            -0.3f, 0.28f, 0f,
            -0.28f, -0.35f, 0f,
            0.28f, -0.35f, 0f,
            0.3f, 0.35f, 0f,
            0f, 0.5f, 0f
    };
    private float[] colorCoords = {
            0.0f, 1.0f, 0.0f, 1.0f,
            1.0f, 0.0f, 0.0f, 1.0f,
            0.0f, 0.0f, 1.0f, 1.0f
    };
    private short[] indexCoords = {
            0, 1, 2, 0, 2, 3, 0, 3, 4
    };

    public MyOpenGLRenderSquare(Context context) {
        this.context = context;

        // 设置顶点数据
        ByteBuffer bb = ByteBuffer.allocateDirect(squareCoords.length * 4);
        bb.order(ByteOrder.nativeOrder());
        vertexBuffer = bb.asFloatBuffer();
        vertexBuffer.put(squareCoords);
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
        String vertexSource = ShaderUtils.readRawText(context, R.raw.vertex_shader_5);
        String fragmentSource = ShaderUtils.readRawText(context, R.raw.fragment_shader_5);
        program = ShaderUtils.createProgram(vertexSource, fragmentSource);
        if (program > 0) {
            positionHandler = GLES20.glGetAttribLocation(program, "vPosition");
            colorHandler = GLES20.glGetAttribLocation(program, "aColor");
            matrixHandler = GLES20.glGetUniformLocation(program, "vMatrix");
        }
    }

    @Override
    public void onSurfaceChanged(GL10 gl10, int width, int height) {
        // 计算宽高比
        float ratio = (float) width / height;
        Matrix.frustumM(projectMatrix, 0, -ratio, ratio, -1, 1, 3, 7);
        // 设置相机的位置
        Matrix.setLookAtM(viewMatrix, 0, 0, 0, 7f, 0f, 0f, 0f, 0f, 1f, 0f);
        // 计算变换矩阵
        Matrix.multiplyMM(mvpMatrix, 0, projectMatrix, 0, viewMatrix, 0);
    }

    @Override
    public void onDrawFrame(GL10 gl10) {
        GLES20.glClear(GLES20.GL_COLOR_BUFFER_BIT | GLES20.GL_DEPTH_BUFFER_BIT);
        GLES20.glClearColor(0.5f, 0.5f, 0.5f, 1f);

        GLES20.glUseProgram(program);

        GLES20.glEnableVertexAttribArray(positionHandler);
        GLES20.glVertexAttribPointer(positionHandler, 3, GLES20.GL_FLOAT, false, 12, vertexBuffer);

        GLES20.glEnableVertexAttribArray(colorHandler);
        GLES20.glVertexAttribPointer(colorHandler, 4, GLES20.GL_FLOAT, false, 0, colorBuffer);

        GLES20.glUniformMatrix4fv(matrixHandler, 1, false, mvpMatrix, 0);

        GLES20.glDrawElements(GLES20.GL_TRIANGLES, indexCoords.length, GLES20.GL_UNSIGNED_SHORT, indexBuffer);

//        GLES20.glDrawArrays(GLES20.GL_TRIANGLE_STRIP, 0, 4);
    }
}
```

1. 首先我们声明了三个数组，这三个数组分别是squareCoords,colorCoords,indexCoords。其中squareCoords表示的是多边形所对应的顶点的坐标。这里我们使用的是[xyz]的方式；colorCoords表示的是颜色数组，这里为了方便起见，我直接将三原色的数组拿了过来了，至于三个分别代表什么样的含义，后面会说；indexCoords表示的是顶点的索引
2. 构造方法：在构造方法中我们将Context的上下文对象传递给了全局变量。
 - 初始化顶点数据，初始化颜色设置，初始化索引设置
3. 在onSurfaceCreated方法中创建生成program，顶点句柄，颜色句柄，投影句柄，这三个句柄需要后面去使用
4. 在onSurfaceChanged方法中初始化相机位置，初始化视图位置，计算变换矩阵
5. 在onDrawFrame方法中开始设置背景颜色，设置顶点着色器，片元着色器，设置变换矩阵

### 绘制圆形

圆形的构架相对而言复杂一些，我们可以吧圆形看做一个正多边形，边越多，园就越平滑，如下图所示，正六边形，正八边形，正十六边形和正一百边形
![绘制圆形](/assets/tools/tools-opengl-07.png)

我们一正六边形为例，由012,023,034,056,061六个三角形组成
利用简单的数学知识，我们可以知道，以多边形为中心简历的直角坐标系，得到n边形的顶点坐标可以使用下面的函数求出来
```
/**
     * 创建圆形图形的所有坐标点
     *
     * @return
     */
    private float[] createPositions() {
        ArrayList<Float> datas = new ArrayList<>();
        datas.add(0.0f);
        datas.add(0.0f);
        datas.add(0.5f);
        float angleSpan = 360f / 360;
        for (int i = 0; i < 360 + angleSpan; i += angleSpan) {
            datas.add((float) (1.0f * Math.sin(i * Math.PI / 180f)));
            datas.add((float) (1.0f * Math.cos(i * Math.PI / 180f)));
            datas.add(0.5f);
        }
        float[] res = new float[datas.size()];
        for (int i = 0; i < res.length; i++) {
            res[i] = datas.get(i);
        }
        return res;
    }
```

看一下源码
GLSurfaceView的代码比较简单，这里就不贴出来了

render部分的代码
```
public class MyOpenGLRenderOval implements GLSurfaceView.Renderer {

    private Context context;

    private int positionHandler;
    private int colorHandler;
    private int matrixHandler;

    private FloatBuffer vertexBuffer;
    private FloatBuffer colorBuffer;

    private int program;

    private float[] projectMatrix = new float[16];
    private float[] viewMatrix = new float[16];
    private float[] mvpMatrix = new float[16];

    private float[] colorCoords = {
            1.0f, 0f, 0f, 1f,
            0f, 1f, 0f, 1f,
            0f, 0f, 1f, 1f
    };
    private float[] vertexCoords ;

    public MyOpenGLRenderOval(Context context) {
        this.context = context;

        // 设置顶点数据
        vertexCoords = createPositions();
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
    }

    /**
     * 创建圆形图形的所有坐标点
     *
     * @return
     */
    private float[] createPositions() {
        ArrayList<Float> datas = new ArrayList<>();
        datas.add(0.0f);
        datas.add(0.0f);
        datas.add(0.5f);
        float angleSpan = 360f / 360;
        for (int i = 0; i < 360 + angleSpan; i += angleSpan) {
            datas.add((float) (1.0f * Math.sin(i * Math.PI / 180f)));
            datas.add((float) (1.0f * Math.cos(i * Math.PI / 180f)));
            datas.add(0.5f);
        }
        float[] res = new float[datas.size()];
        for (int i = 0; i < res.length; i++) {
            res[i] = datas.get(i);
        }
        return res;
    }

    @Override
    public void onSurfaceCreated(GL10 gl10, EGLConfig eglConfig) {
        String vertexSource = ShaderUtils.readRawText(context, R.raw.vertex_shader_6);
        String fragmentSource = ShaderUtils.readRawText(context,R.raw.fragment_shader_6);
        program = ShaderUtils.createProgram(vertexSource,fragmentSource);
        if (program > 0){
            positionHandler = GLES20.glGetAttribLocation(program,"position");
            matrixHandler = GLES20.glGetUniformLocation(program,"matrix");
            colorHandler = GLES20.glGetAttribLocation(program,"color");
        }
    }

    @Override
    public void onSurfaceChanged(GL10 gl10, int i, int i1) {
        float ratio = (float) i / i1;
        Matrix.frustumM(projectMatrix,0,-ratio,ratio,-1,1,3,7);
        Matrix.setLookAtM(viewMatrix,0,0,0,7f,0f,0f,0f,0f,1f,0f);
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

        GLES20.glDrawArrays(GLES20.GL_TRIANGLE_FAN,0,vertexCoords.length / 3);
    }
}
```

在这段代码中有几个部分需要我们注意，首先我们将
```
GLES20.glDrawArrays(GLES20.GL_TRIANGLES,0,1);
```
改成了
```
GLES20.glDrawArrays(GLES20.GL_TRIANGLE_FAN,0,vertexCoords.length/3);
```
这里面第一个参数表示要绘制的方式，第二个参数表示绘制的偏移量，第三个参数表示顶点的个数

> 绘制的方式有以下几种
> int GL_POINTS:将传入的顶点坐标作为单独的点绘制
> int GL_LINES:将传入的坐标作为单独的线条绘制，ABCDEFG六个顶点，会绘制AB，CD，EF三条线
> int GL_LINE_STRIP:将传入的顶点作为折现绘制，ABCD四个顶点，绘制AB，BC，CD三条线
> int GL_LINE_LOOP:将传入的顶点作为闭合折现绘制，ABCD四个顶点，绘制AB，BC，CD，DA四条线段
> int GL_TRIANGLES:将传入的顶点作为单独的三角形绘制，ABCDEF绘制ABC，DEF两个三角形
> int GL_TRIANGLE_FAN:将传入的顶点作为扇面绘制，ABCDEF绘制ABC，ACD，ADE，AEF四个三角形
> int GL_TRIANGLE_STRIP:将传入的顶点作为三角条带绘制，ABCDEF绘制ABC，BCD，CDE，DEF四个三角形

其实我们可以这样理解，GLTRIANGLE_STRIP的方式绘制连续三角形，比直接用GL_TRIANGLES的方式绘制三角形稍好多个点，绘制的效率会更高。另外GL_TRIANGLE_STRIP并不是只能绘制连续三角形构成的物体，我们只需要将不需要重复绘制的点重复绘制两次即可。比如我们传入ABCDEFFGH坐标，就会得到ABC，BCD，CDE以及FGH四个三角形

GL_TRIANGLE_FAN：扇面绘制是以第一个零点进行绘制，通常我们绘制圆形，圆锥的锥面都会使用到，值得注意的是，最后一个点的左边应当与第二个点重合，在计算的时候，起点角度为0度，终点角度应包含360度

### 绘制彩色
前几篇blog中设置的都是单一颜色，如果我们想要设置彩色的该怎么去实现？其实这个彩色很难看，只不过是为了多去了解如何设置不同的颜色而已。

先来看一下，我们在绘制一个简单的三角形时所使用的代码

首先我们通过onSurfaceCreated方法获取到了raw中的名为af_Color的颜色数值，这个颜色数值是需要我们自己去赋值的。代码如下
```
public void onSurfaceCreated(GL10 unused, EGLConfig config) {
        String vertexSource = ShaderUtils.readRawText(context, R.raw.vertex_shader);
        String fragmentSource = ShaderUtils.readRawText(context, R.raw.fragment_shader);
        program = ShaderUtils.createProgram(vertexSource, fragmentSource);
        if (program > 0) {
            position = GLES20.glGetAttribLocation(program, "av_Position");
            color = GLES20.glGetUniformLocation(program, "af_Color");
        }
    }
```
其中R.raw.vertex_shader和R.raw.fragment_shader表示的是我们需要引入的文件，这个文件时glsl结尾的文件，之前我们有介绍过。然后我们在onDrawFrame中通过<p style="color:red">GLES20.glUniform4f</p>方法设置三角形的颜色。这时候我们设置颜色的代码是
```
public void onDrawFrame(GL10 unused) {
        GLES20.glClear(GLES20.GL_COLOR_BUFFER_BIT);
        GLES20.glClearColor(1.0f, 1.0f, 1.0f, 1.0f);
        GLES20.glUseProgram(program);
        GLES20.glUniform4f(color, 1f, 0f, 0f, 1f);
        GLES20.glEnableVertexAttribArray(position);

        GLES20.glVertexAttribPointer(position, 2, GLES20.GL_FLOAT, false, 8, vertexBuffer);
        GLES20.glDrawArrays(GLES20.GL_TRIANGLES, 0, 3);
    }
```
因为我们只使用了单一颜色，所以三角形呈现的是红色。

那么当我们想要实现多种不同的颜色的时候，就需要使用float类型的数组来为我们展示多种颜色
如下所示，我们这里想要绘制一个彩色的三角形，那么需要声明一个float类型的数组
```
private float[] colorCoords = {
            0.0f, 1.0f, 0.0f, 1.0f,
            1.0f, 0.0f, 0.0f, 1.0f,
            0.0f, 0.0f, 1.0f, 1.0f
    };
```
设置完成数组之后，我们需要将这个数组在底层的内存空间中申请内存地址,我们把这段代码放在了构造方法中
```
// 设置颜色数据
        ByteBuffer color = ByteBuffer.allocateDirect(colorCoords.length * 4);
        color.order(ByteOrder.nativeOrder());
        colorBuffer = color.asFloatBuffer();
        colorBuffer.put(colorCoords);
        colorBuffer.position(0);
```
因为我们这里使用的是FloatBuffer对象，所以在设置三角形颜色的时候也不能再使用上面的那种方式了。使用的方式如下(我第一次看这部分的时候，是想着应该跟vertexBuffer一样，就照着去写，结果一次就成功了)
```
@Override
    public void onDrawFrame(GL10 gl10) {
        GLES20.glClear(GLES20.GL_COLOR_BUFFER_BIT | GLES20.GL_DEPTH_BUFFER_BIT);
        GLES20.glClearColor(0.5f, 0.5f, 0.5f, 1f);

        GLES20.glUseProgram(program);

        GLES20.glEnableVertexAttribArray(colorHandler);
        GLES20.glVertexAttribPointer(colorHandler, 4, GLES20.GL_FLOAT, false, 0, colorBuffer);

        GLES20.glEnableVertexAttribArray(positionHandler);
        GLES20.glVertexAttribPointer(positionHandler, 3, GLES20.GL_FLOAT, false, 12, vertexBuffer);

        GLES20.glUniformMatrix4fv(matrixHandler, 1, false, mvpMatrix, 0);

        GLES20.glDrawArrays(GLES20.GL_TRIANGLES, 0, 3);
    }
```

完整的代码如下：

Render部分的代码：
```
public class MyOpenGLRenderViewColors implements GLSurfaceView.Renderer {

    private Context context;
    private FloatBuffer vertexBuffer;
    private FloatBuffer colorBuffer;
    private int positionHandler;
    private int colorHandler;
    private int matrixHandler;
    private int program;

    private float[] triangleCoords = {
            0.5f, 0.5f, 0.0f, // top
            -0.5f, -0.5f, 0.0f, // bottom left
            0.5f, -0.5f, 0.0f  // bottom right
    };

    private float[] colorCoords = {
            0.0f, 1.0f, 0.0f, 1.0f,
            1.0f, 0.0f, 0.0f, 1.0f,
            0.0f, 0.0f, 1.0f, 1.0f
    };

    private float[] projectMatrix = new float[16];
    private float[] viewMatrix = new float[16];
    private float[] mvpMatrix = new float[16];

    public MyOpenGLRenderViewColors(Context context) {
        this.context = context;
        // 设置顶点数据
        ByteBuffer vertex = ByteBuffer.allocateDirect(triangleCoords.length * 4);
        vertex.order(ByteOrder.nativeOrder());
        vertexBuffer = vertex.asFloatBuffer();
        vertexBuffer.put(triangleCoords);
        vertexBuffer.position(0);

        // 设置颜色数据
        ByteBuffer color = ByteBuffer.allocateDirect(colorCoords.length * 4);
        color.order(ByteOrder.nativeOrder());
        colorBuffer = color.asFloatBuffer();
        colorBuffer.put(colorCoords);
        colorBuffer.position(0);
    }

    @Override
    public void onSurfaceCreated(GL10 gl10, EGLConfig eglConfig) {
        String vertexSource = ShaderUtils.readRawText(context, R.raw.vertex_shader_4);
        String fragmentSource = ShaderUtils.readRawText(context, R.raw.fragment_shader_4);
        program = ShaderUtils.createProgram(vertexSource, fragmentSource);
        if (program > 0) {
            positionHandler = GLES20.glGetAttribLocation(program, "vPosition");
            colorHandler = GLES20.glGetAttribLocation(program, "aColor");
            matrixHandler = GLES20.glGetUniformLocation(program, "vMatrix");
        }
    }

    @Override
    public void onSurfaceChanged(GL10 gl10, int width, int height) {
        // 计算宽高比
        float ratio = (float) width / height;
        Matrix.frustumM(projectMatrix, 0, -ratio, ratio, -1, 1, 3, 7);
        // 设置相机的位置
        Matrix.setLookAtM(viewMatrix, 0, 0, 0, 7f, 0f, 0f, 0f, 0f, 1f, 0f);
        // 计算变换矩阵
        Matrix.multiplyMM(mvpMatrix, 0, projectMatrix, 0, viewMatrix, 0);
    }

    @Override
    public void onDrawFrame(GL10 gl10) {
        GLES20.glClear(GLES20.GL_COLOR_BUFFER_BIT | GLES20.GL_DEPTH_BUFFER_BIT);
        GLES20.glClearColor(0.5f, 0.5f, 0.5f, 1f);

        GLES20.glUseProgram(program);

        GLES20.glEnableVertexAttribArray(colorHandler);
        GLES20.glVertexAttribPointer(colorHandler, 4, GLES20.GL_FLOAT, false, 0, colorBuffer);

        GLES20.glEnableVertexAttribArray(positionHandler);
        GLES20.glVertexAttribPointer(positionHandler, 3, GLES20.GL_FLOAT, false, 12, vertexBuffer);

        GLES20.glUniformMatrix4fv(matrixHandler, 1, false, mvpMatrix, 0);

        GLES20.glDrawArrays(GLES20.GL_TRIANGLES, 0, 3);
    }
}
```

其中vertex_shader_4的代码如下：
```
attribute vec4 vPosition;
uniform mat4 vMatrix;
varying vec4 vColor;
attribute vec4 aColor;
void main(){
    gl_Position = vMatrix * vPosition;
    vColor = aColor;
}
```

fragment_shader_4的代码如下：
```
// 设置我们使用中等精确度
precision mediump float;
// 声明变量用于传递颜色数值
varying vec4 vColor;
void main() {
    // 将af_Color的颜色属性值赋值给gl_FragColor的内置变量
    gl_FragColor = vColor;
}
```

最终的效果如下图所示
![绘制圆形](/assets/tools/tools-opengl-08.png)

[demo地址github](https://github.com/xiaoniudadi/android-media-demo/tree/master/06-android-media-opengl)
