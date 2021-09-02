---
title: android 音视频开发(六)
date: 2019-08-26 22:50:10
tags:
  - android
  - 音视频
---
学习Android平台的OpenGL ES API，了解OpenGL开发的基本流程，使用OpenGL绘制三角形
<!--more-->
先看效果图
![使用OpenGL ES绘制三角形](/assets/tools/tools-opengl-04.png)

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

##### 顶点坐标系

!(顶点坐标系)[/assets/tools/tools-opengl-02.png]

在如上图所示的内容中，我们会发现在OpenGL中原点就是屏幕的中心(0,0),最左边是(-1,0),最右边是(1,0),最上面是(0,1),最下面是(0,-1).这里我们需要现绘制一个三角形，所以我们让这个三角形的三个坐标分别如下所示

!(绘制三角形坐标)[/assets/tools/tools-opengl-03.png]

三角形的三个坐标分别是(-1,0),(1,0),(0,1)

##### 定义三角形

在绘制三角形之前，我们知道java有JVM虚拟机，在虚拟机中有一个非常重要的机制，叫做垃圾回收机制(GC)。在我们使用java语言开发应用时，所有的内存地址都是由JVM统一管理的。但是这样做的好处是方便我们上手，我们只需要注重业务逻辑，而不需要过多的思考内存情况。但是因为GC的清除不受我们代码的控制，所以会出现一种情况，当我们的程序运行时，GC可能会将三角形的顶点回收掉，这样会导致OpenGL找不到三角形的顶点，从而导致程序错误。为了避免这样情况的发生，我们需要给三角形的顶点分配内存(也就跳过了JVM分配内存的这一步).这一步很关键，网上的blog都把这步叫做顶点的本地化。

顶点的本地化分为两步：
1. 使用float数组来存放我们的三角形顶点
```
float[] vertexData = {
    -1,0,// 左下角
    1,0,// 右下角
    0,1// 顶角
}
```
2. 根据顶点分配底层内存地址，因为需要本地化，所以内存地址分配和C++差不多。这里我们需要使用的是ByteBuffer这个类
```
public class Triangle {

    private FloatBuffer vertexBuffer;

    // 数组中每个顶点的坐标数(其实就是在三维模型中，我们要有x,y,z三个坐标)
    private static final int COORDS_PER_VERTEX = 3;
    // 声明三角形三个顶点的位置
    private static float triangleCoords[] = {
            -1f,0f,// 左下角
            1f,0f,// 右下角
            0f,1f// 顶角
    };

    // 设置颜色，分别为red,green,blue和alpha
    private float color[] = {0.63671875f, 0.76953125f, 0.22265625f, 1.0f};

    // 构造方法
    public Triangle(){
        // 为存放三角形坐标，初始化顶点字节缓存
        ByteBuffer bb = ByteBuffer.allocateDirect(4 * triangleCoords.length);// 分配内存空间
        // 内存bit的排序方式和本地机器一致
        bb.order(ByteOrder.nativeOrder());
        // 转换成float的buffer，因为这里我们存放的是float类型的顶点
        vertexBuffer = bb.asFloatBuffer();
        // 将数据放入内存中
        vertexBuffer.put(triangleCoords);
        // 把索引指针指向开头位置
        vertexBuffer.position(0);
    }

}
```

- 方法allocateDirect是用来分配内存大小的，其大小为float数组长度*float所占的字节数，而float是占有4个字节的。
- 方法order使用配置内存中字节的排列方式的，这里我们默认和本机的排列方式一致即可
- 方法asFloatBuffer是将ByteBuffer转换成floatBuffer
- 方法put是将float数组作为原数据放在floatBuffer中
- 方法position是设置从头开始访问这块内存地址

> 在android开发中如果我们想要设置颜色一般都会调用setxxxColor这样的方法去执行。但是在OpenGL中，我们需要编写自己的着色器，然后OpenGL会用GPU去执行这个着色器，最终将反馈的结果返回给我们。这里着色器的编写需要使用到glsl语言来编写。

##### 顶点着色器

编写顶点着色器，新建一个名为vertex_shader.glsl的文件，并且把这个文件放在res/raw文件夹下面
```
attribute vec4 av_Position; // 用于在Java代码中获取属性
void main(){
    gl_Position = av_Position;// gl_Position是内置变量，OpenGL绘制顶点就是根据这个值绘制的。所以我们需要将java代码中的值赋值给它
}
```
代码很少，但是理解起来去不那么容易，如果第一次看不懂也没有关系，后面我们会继续学习。

1. 先来看看第一行：attribute vec4 av_Position;
- attribute表示顶点属性，只能用在顶点坐标中。java代码会回去这个变量，并且为它赋值
- vec4表示包含了(x,y,z,w)四个值的向量，其中x和y表示的平面，z表示的是3D，w表示的是摄像头的距离。因为我们这里只需要绘制2D图形，所以不需要设置z和w的值，OpenGL会填写默认值1.

> 所以这句话的意思就是我们声明了一个名为av_Position的拥有四个值(x,y,z,w)并且数据类型是attribute的向量，我们在java代码中会找到这个变量，并且把顶点(FloatBuffer)的值赋值给它。

这样OpenGL在执行着色器代码时就会获取到三角形三个顶点，进而绘制出我们想要的三角形。

2. void main(){}：这个没啥好说的，就是一个函数
3. gl_Position = av_Position: gl_Position就是glsl中内置的最终顶点变量，其实就是这个变量决定了我们将顶点绘制到哪个地方，所以我们需要将av_Position赋值给它

##### 片元着色器

上面的代码中我们只是绘制了点，那么绘制完点之后我们需要给三角形一个填充颜色，这时候就要使用到片元着色器。

编写片元着色器，需要在res/raw文件夹下新建一个名为fragment_shader.glsl的文件
```
precision mediump float;// 声明我们使用中等精度 float
uniform vec4 af_Color;// 用于在java代码中传递颜色数值
void main(){
    gl_FragColor = af_Color;// gl_FragColor内置变量，OpenGL在填充颜色时使用该变量
}
```

1. 第一行代码：precision mediump float  表示用中等精度float类型来保存变量数值。
2. 第二行代码：uniform vec4 af_Color: 
- uniform:声明变量的类型。uniform是用于Java代码向顶点和片元着色器传递数据。和attribute的区别在于，attribute只能用于顶点着色器的应用程序中并且包含具体的顶点数据，每次执行都要从顶点内存中获取新的值，而uniform则始终都是一个值。
- vec4 af_Color也是声明一个4个分量的变量af_Color，这个里面保存的是颜色的值了（rgba四个分量)。
3. void main(){}一个函数，没啥好说的
4. gl_FragColor = af_Color: gl_FragColor是一个内置变量，用于最终颜色填充的赋值。

上面的两步都是生命着色器的，下面我们需要将着色器引入到java代码中

##### 加载并编译着色器语言

1. 通过GLES20.glCreateShader(shaderType)创建(顶点/片元)类型的代码
```
// 创建顶点类型着色器
int vertexShader = GLES20.glCreateShader(GL_VERTEX_SHADER);
// 创建片源类型着色器
int fragmentShader = GLES20.glCreateShader(GL_FRAGMENT_SHADER);
```
2. 加载shader源码并且编译shader
```
// 其实两种着色器我们要写两种，为了方便，后面我们会写一个方法，然后在这个方法里统一执行
GLES20.glShaderSource(shader,source);// 根据我们的类型加载不同的着色器
GLES20.glCompileShader(shader);// 编译我们自己写的着色器代码程序
```
3. 实际创建并添加到渲染程序中
```
int program = GLES20.glCreateProgram();// 创建一个program程序
```
4. 将着色器添加到渲染程序中
```
GLES20.glAttachShader(program,vertexShader);// 把顶点着色器加入到渲染程序中
GLES20.glAttachShader(program.fragmentShader);// 把片元着色器加入到渲染程序中
```
5. 链接源程序
```
GLES20.glLinkProgram(program);// 最终链接顶点着色器和片元着色器，后面再program中就可以方位顶点着色器和片元着色器的属性
```

##### 传递顶点坐标和颜色给着色器程序

1. 获取顶点变量
```
// 获取顶点属性，后面我们会给他赋值
int position = GLES20.glGetAttributeLocation(programId,"av_Position");
```
2. 获取颜色变量
```
// 获取片元变量
int color = GLES20.glGetUniformLocation(programId,"af_Color");
```
3. 开始执行着色器程序
```
// 在开始绘制之前想将当前的programId赋值出去
GLES20.glUseProgram(programId);
```
4. 激活顶点属性
```
// 先激活顶点属性数组，之后才能对他进行赋值
GLES20.glEnableVertexAttribArray(position);
```
5. 向顶点属性传递值
```
GLES20.glVertexAttribPointer(position,2,GLES20.GL_FLOAT，false,8,vertexBuffer);
```
- 第一个参数是顶点属性的句柄
- 第二个参数表示我们使用几个分量来表示一个点，很明显，2个(x,y)
- 第三个参数表示顶点的数据类型
- 第四个参数表示是否要做归一化处理，如果我们的坐标不再(-1,1)之间，就需要
- 第五个参数表示每个点所占据的空间大小，因为每个点都是(x,y),每个值都是4个字节，所以写8
- 第六个参数表示OpenGL要从哪个内存中获取点数据

6. 绘制
```
// 绘制三角形，从我们顶点数组中0的位置开始，绘制顶点个数为3个
GLES20.glDrawArrays(GLES20.GL_TRIANGLES,0,3);
```
- 第一个参数表示绘制的方式，GL_TRIANGLES表示绘制三角形，还有其他的方式
- 第二个参数表示从哪个位置开始绘制，因为顶点坐标里只有三个点所以从0开始绘制
- 第三个参数表示绘制多少个点


综上所述，绘制的过程大致分为以下几个内容
坐标点(顶点/纹理) -->> 编写着色器程序 -->> 加载着色器程序并生成program -->> 获取program中的变量 -->> program变量的赋值 -->> 最终绘制


### 完整代码如下
<p style="color:red">activity</p>

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

<p style="color:red">自定义GLSurfaceView</p>

```
public class MyGLSurfaveView extends GLSurfaceView {

    private MyGLSurfaceViewRendered renderer;
    private Context context;

    public MyGLSurfaveView(Context context) {
        super(context);
        this.context = context;
        init();
    }

    private void init() {
        // 设置我们使用OpenGL的版本  2
        setEGLContextClientVersion(2);
        renderer = new MyGLSurfaceViewRendered(context);
        setRenderer(renderer);
    }

}
```

<p style="color:red">创建ShaderUtils工具类</p>

```
public class ShaderUtils {

    /**
     * 将glsl数据内容转换成String类型
     *
     * @param context
     * @param rawId
     * @return
     */
    public static String readRawText(Context context, int rawId) {
        InputStream is = context.getResources().openRawResource(rawId);
        BufferedReader br = new BufferedReader(new InputStreamReader(is));
        StringBuffer sb = new StringBuffer();
        String line;
        try {
            while ((line = br.readLine()) != null) {
                sb.append(line).append("\n");
            }
            br.close();
        } catch (Exception ex) {
            ex.printStackTrace();
        }
        return sb.toString();
    }

    public static int loadShader(int shaderType, String source) {
        int shader = GLES20.glCreateShader(shaderType);
        if (shader != 0) {
            GLES20.glShaderSource(shader, source);
            GLES20.glCompileShader(shader);
            int[] compile = new int[1];
            GLES20.glGetShaderiv(shader, GLES20.GL_COMPILE_STATUS, compile, 0);
            if (compile[0] != GLES20.GL_TRUE) {
                // 生成shader失败
                GLES20.glDeleteShader(shader);
                shader = 0;
            }
        }
        return shader;
    }

    public static int createProgram(String vertexSource, String fragmentSource) {
        int vertexShader = loadShader(GLES20.GL_VERTEX_SHADER, vertexSource);
        if (vertexShader == 0) {
            return 0;
        }
        int fragmentShader = loadShader(GLES20.GL_FRAGMENT_SHADER, fragmentSource);
        if (fragmentShader == 0) {
            return 0;
        }
        int program = GLES20.glCreateProgram();
        if (program != 0) {
            GLES20.glAttachShader(program, vertexShader);
            GLES20.glAttachShader(program, fragmentShader);
            GLES20.glLinkProgram(program);
            int[] linsStatus = new int[1];
            GLES20.glGetProgramiv(program, GLES20.GL_LINK_STATUS, linsStatus, 0);
            if (linsStatus[0] != GLES20.GL_TRUE) {
                // 生成program社保
                GLES20.glDeleteProgram(program);
                program = 0;
            }
        }
        return program;
    }

    public static FloatBuffer fBuffer(float[] a) {
        FloatBuffer floatBuffer;
        // 先初始化buffer,数组的长度*4,因为一个float占4个字节
        ByteBuffer mbb = ByteBuffer.allocateDirect(a.length * 4);
        // 数组排列用nativeOrder
        mbb.order(ByteOrder.nativeOrder());
        floatBuffer = mbb.asFloatBuffer();
        floatBuffer.put(a);
        floatBuffer.position(0);
        return floatBuffer;
    }

}
```

<p style="color:red">自定义Rendered</p>

```
public class MyGLSurfaceViewRendered implements GLSurfaceView.Renderer {

    private Context context;
    private final float[] vertexData = {
            -1f, 0f,// 左下角
            1f, 0f,// 右下角
            0f, 1f// 顶角
    };

    private FloatBuffer vertexBuffer;
    private int program;
    private int position;
    private int color;

    public MyGLSurfaceViewRendered(Context context) {
        this.context = context;
        vertexBuffer = ByteBuffer.allocateDirect(vertexData.length * 4)
                .order(ByteOrder.nativeOrder())
                .asFloatBuffer()
                .put(ShaderUtils.fBuffer(vertexData));
        vertexBuffer.position(0);
    }

    public void onSurfaceCreated(GL10 unused, EGLConfig config) {
        String vertexSource = ShaderUtils.readRawText(context, R.raw.vertex_shader);
        String fragmentSource = ShaderUtils.readRawText(context, R.raw.fragment_shader);
        program = ShaderUtils.createProgram(vertexSource, fragmentSource);
        if (program > 0) {
            position = GLES20.glGetAttribLocation(program, "av_Position");
            color = GLES20.glGetUniformLocation(program, "af_Color");
        }
    }

    public void onSurfaceChanged(GL10 unused, int width, int height) {
        GLES20.glViewport(0, 0, width, height);
    }

    public void onDrawFrame(GL10 unused) {
        GLES20.glClear(GLES20.GL_COLOR_BUFFER_BIT);
        GLES20.glClearColor(1.0f, 1.0f, 1.0f, 1.0f);
        GLES20.glUseProgram(program);
        GLES20.glUniform4f(color, 1f, 0f, 0f, 1f);
        GLES20.glEnableVertexAttribArray(position);

        GLES20.glVertexAttribPointer(position, 2, GLES20.GL_FLOAT, false, 8, vertexBuffer);
        GLES20.glDrawArrays(GLES20.GL_TRIANGLES, 0, 3);
    }
}
```
[demo地址github](https://github.com/xiaoniudadi/android-media-demo/tree/master/06-android-media-opengl)

# 参考资料
[OpenGL ES API](https://www.jianshu.com/p/87abc92134dd)
[OpenGL 绘制三角形](https://blog.csdn.net/ywl5320/article/details/80964212)
[OpenGL 学习](https://www.jianshu.com/u/eb01968a6673)