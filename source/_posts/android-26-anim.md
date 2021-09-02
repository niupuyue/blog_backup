---
title: 重拾android路(二十五) Android动画
date: 2019-4-08 11:27:11
tags:
  - android
---

总的来说Android的动画一共分为三种，分别是间补动画，帧动画和属性动画。间补动画和帧动画比较简单，这里会简单介绍，对于属性动画，可能会说的更多一些
<!--more-->
# 间补动画(Tween)
所谓的间补动画，就是指改变了当前图像的显示位置，样式和形式，但是对于组件的本身来说依然是保持原来的样子。举个例子来说就是如果我们通过间补动画将一个ImageView进行了移动，表面上看上去我们的ImageView是发生了移动，但是实际上，只是显示的改变，实际位置还是保留在原地，如果想要针对这个ImageView触发一些点击滑动事件，触发点还在原来的位置上。
> 间补动画分为以下几个内容：平移，旋转，缩放和透明度

位移改变的动画。创建动画时只需要指定动画开始的位置(以x，y坐标展示)，结束时的位置，并指出动画持续的时间即可
这里我们需要知道，基本上间补动画都是通过XML文件完成的操作，所以这里我们直接使用。需要将所需要的XML文件放在res/anim文件夹下

### 淡入淡出
```
<?xml version="1.0" encoding="utf-8"?>
<alpha xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="1000"
    android:fromAlpha="1.0"
    android:interpolator="@android:anim/accelerate_decelerate_interpolator"
    android:toAlpha="0.0" />
```
这里的interpolator 表示插值器，表示的是当前动画变化的速度。可以通过 @android:anim 来选择不同的插值器。

### 放缩
```
<?xml version="1.0" encoding="utf-8"?>
<scale xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="1000"
    android:fromXScale="0.0"
    android:fromYScale="0.0"
    android:pivotX="50%"
    android:pivotY="50%"
    android:toXScale="1.0"
    android:toYScale="1.0"/>
```

### 平移
```
<?xml version="1.0" encoding="utf-8"?>
<rotate xmlns:android="http://schemas.android.com/apk/res/android"
      android:fromDegree="0"
      android:toDegree="1800"
      android:pivotX = "50%"
      android:pivotY="50%"
      android:duration = "3000"
/>
```
pivot 这个属性主要是在translate 和 scale 动画中，这两种动画都牵扯到view 的“物理位置“发生变化，所以需要一个参考点。而pivotX和pivotY就共同决定了这个点；它的值可以是float或者是百分比数值。
以 pivotX 为例，说明其取不同的值的含义：
10:距离动画所在view自身左边缘10像素
10% :距离动画所在view自身左边缘 的距离是整个view宽度的10%
10%p:距离动画所在view父控件左边缘的距离是整个view宽度的10%

### 旋转
```
<?xml version="1.0" encoding="utf-8"?>
 <rotate
        android:fromDegrees="float"
        android:toDegrees="float"
        android:pivotX="50%"
        android:pivotY="50%" />
```

这样定义完成之后，就可以直接使用了

一个小例子
动画资源
```
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android"
 android:interpolator="@android:anim/accelerate_decelerate_interpolator"
    >
    <scale
        android:duration="3000"
        android:fromXScale="0.0"
        android:fromYScale="0.0"
        android:pivotX="50%"
        android:pivotY="50%"
        android:toXScale="1.0"
        android:toYScale="1.0"/>
    <alpha
        android:duration="3000"
        android:fromAlpha="1.0"
        android:toAlpha="0.5" />
    <rotate
        android:fromDegrees="0"
        android:toDegrees="720"
        android:pivotX = "50%"
        android:pivotY="50%"
        android:duration = "3000"
        />
    <translate
        android:fromXDelta="0"
        android:toXDelta="100"
        android:fromYDelta="0"
        android:toYDelta="100" />
</set>
```
也可以使用java代码实现

### 移动
```
Animation translateAnimation = new TranslateAnimation(0，500，0，500);
        // 创建平移动画的对象：平移动画对应的Animation子类为TranslateAnimation
        // 参数分别是：
        // 1. fromXDelta ：视图在水平方向x 移动的起始值
        // 2. toXDelta ：视图在水平方向x 移动的结束值
        // 3. fromYDelta ：视图在竖直方向y 移动的起始值
        // 4. toYDelta：视图在竖直方向y 移动的结束值
        translateAnimation.setDuration(3000);
        // 播放动画直接 startAnimation(translateAnimation)
        //如：
        mButton.startAnimation(translateAnimation);
```
### 缩放
```
Animation scaleAnimation= new ScaleAnimation(0,2,0,2,Animation.RELATIVE_TO_SELF,0.5f,Animation.RELATIVE_TO_SELF,0.5f);
        // 1. fromX ：动画在水平方向X的结束缩放倍数
        // 2. toX ：动画在水平方向X的结束缩放倍数
        // 3. fromY ：动画开始前在竖直方向Y的起始缩放倍数
        // 4. toY：动画在竖直方向Y的结束缩放倍数
        // 5. pivotXType:缩放轴点的x坐标的模式
        // 6. pivotXValue:缩放轴点x坐标的相对值
        // 7. pivotYType:缩放轴点的y坐标的模式
        // 8. pivotYValue:缩放轴点y坐标的相对值
        // pivotXType = Animation.ABSOLUTE:缩放轴点的x坐标 =  View左上角的原点 在x方向 加上 pivotXValue数值的点(y方向同理)
        // pivotXType = Animation.RELATIVE_TO_SELF:缩放轴点的x坐标 = View左上角的原点 在x方向 加上 自身宽度乘上pivotXValue数值的值(y方向同理)
        // pivotXType = Animation.RELATIVE_TO_PARENT:缩放轴点的x坐标 = View左上角的原点 在x方向 加上 父控件宽度乘上pivotXValue数值的值 (y方向同理)
        scaleAnimation.setDuration(3000);
        // 使用
        mButton.startAnimation(scaleAnimation);
```
### 旋转
```
Animation rotateAnimation = new RotateAnimation(0,270,Animation.RELATIVE_TO_SELF,0.5f,Animation.RELATIVE_TO_SELF,0.5f);
        // 1. fromDegrees ：动画开始时 视图的旋转角度(正数 = 顺时针，负数 = 逆时针)
        // 2. toDegrees ：动画结束时 视图的旋转角度(正数 = 顺时针，负数 = 逆时针)
        // 3. pivotXType：旋转轴点的x坐标的模式
        // 4. pivotXValue：旋转轴点x坐标的相对值
        // 5. pivotYType：旋转轴点的y坐标的模式
        // 6. pivotYValue：旋转轴点y坐标的相对值
        // pivotXType = Animation.ABSOLUTE:旋转轴点的x坐标 =  View左上角的原点 在x方向 加上 pivotXValue数值的点(y方向同理)
        // pivotXType = Animation.RELATIVE_TO_SELF:旋转轴点的x坐标 = View左上角的原点 在x方向 加上 自身宽度乘上pivotXValue数值的值(y方向同理)
        // pivotXType = Animation.RELATIVE_TO_PARENT:旋转轴点的x坐标 = View左上角的原点 在x方向 加上 父控件宽度乘上pivotXValue数值的值 (y方向同理)
        rotateAnimation.setDuration(3000);
        mButton.startAnimation(rotateAnimation);
```
### 淡入淡出
```
Animation alphaAnimation = new AlphaAnimation(1,0);
        // 1. fromAlpha:动画开始时视图的透明度(取值范围: -1 ~ 1)
        // 2. toAlpha:动画结束时视图的透明度(取值范围: -1 ~ 1)
        alphaAnimation.setDuration(3000);
        mButton.startAnimation(alphaAnimation);
```
### 组合动画
```
// 组合动画设置
        AnimationSet setAnimation = new AnimationSet(true);

        // 特别说明以下情况
        // 因为在下面的旋转动画设置了无限循环(RepeatCount = INFINITE)
        // 所以动画不会结束，而是无限循环
        // 所以组合动画的下面两行设置是无效的
        setAnimation.setRepeatMode(Animation.RESTART);
        setAnimation.setRepeatCount(1);// 设置了循环一次,但无效

        // 旋转动画
        Animation rotate = new RotateAnimation(0,360,Animation.RELATIVE_TO_SELF,0.5f,Animation.RELATIVE_TO_SELF,0.5f);
        rotate.setDuration(1000);
        rotate.setRepeatMode(Animation.RESTART);
        rotate.setRepeatCount(Animation.INFINITE);

        // 平移动画
        Animation translate = new TranslateAnimation(TranslateAnimation.RELATIVE_TO_PARENT,-0.5f,
                TranslateAnimation.RELATIVE_TO_PARENT,0.5f,
                TranslateAnimation.RELATIVE_TO_SELF,0
                ,TranslateAnimation.RELATIVE_TO_SELF,0);
        translate.setDuration(10000);

        // 透明度动画
        Animation alpha = new AlphaAnimation(1,0);
        alpha.setDuration(3000);
        alpha.setStartOffset(7000);

        // 缩放动画
        Animation scale1 = new ScaleAnimation(1,0.5f,1,0.5f,Animation.RELATIVE_TO_SELF,0.5f,Animation.RELATIVE_TO_SELF,0.5f);
        scale1.setDuration(1000);
        scale1.setStartOffset(4000);

        // 将创建的子动画添加到组合动画里
        setAnimation.addAnimation(alpha);
        setAnimation.addAnimation(rotate);
        setAnimation.addAnimation(translate);
        setAnimation.addAnimation(scale1);
        // 使用
        mButton.startAnimation(setAnimation);
```
有时候我们会有一些需求，如当动画执行完成之后，需要跳转页面，所以我们需要监听动画状态
```
Animation.addListener(new AnimatorListener() {
          @Override
          public void onAnimationStart(Animation animation) {
              //动画开始时执行
          }

           @Override
          public void onAnimationRepeat(Animation animation) {
              //动画重复时执行
          }

         @Override
          public void onAnimationCancel()(Animation animation) {
              //动画取消时执行
          }

          @Override
          public void onAnimationEnd(Animation animation) {
              //动画结束时执行
          }
      });
```
### 自定义间补动画
Android 提供了 Animation 作为补间动画抽象基类，而且为该抽象基类提供了 AlphaAnimation、RotationAnimation、ScaleAnimation、TranslateAnimation 四个实现类，这四个实现类只是补间动画的基本形式：透明度、旋转、缩放、位移。但是要实现复杂的动画，就需要继承 Animation。继承 Animation 类关键是要重写一个方法：

applyTransformation(float interpolatedTime,Transformation t)
interploatedTime: 代表了动画的时间进行比。不管动画实际的持续时间如何，当动画播放时，该参数总是从 0 到 1。

Transformation t:该参数代表了补间动画在不同时刻对图形或组件的变形程度。
在实现自定义动画的关键就是重写 applyTransformation 方法时 根据 interpolatedTime 时间来动态地计算动画对图片或视图的变形程度。
Transformation 代表了对图片或者视图的变形，该对象封装了一个 Matrix 对象，对它所包装了 Matrix 进行位移、倾斜、旋转等变换时，Transformation 将会控制对应的图片或视图进行相应的变换。
为了控制图片或者 View 进行三维空间的变换，还需要借助于 Android 提供的一个 Camera，这个 Camera 并非代表手机摄像头，而是空间变换工具。作用类似于 Matrix，其常用方法如下：
getMatrix(Matrix matrix)：将 Camera 所做的变换应用到指定的  matrix 上。
rotateX(float deg):将组件沿 X 轴旋转。
rotateY(float deg):将组件沿 Y 轴旋转。
rotateZ(float deg):将组件沿 Z 轴旋转。
translate(float x,float y,float z):目标组件在三维空间里变换。
applyToCanvas(Canvas canvas):把 Camera 所做的变换应用到 Canvas 上.
![动画坐标系](/assets/anim/anim01.png)实现自定义间补动画
```
public class CustomAnimation extends Animation {
    private float centerX;
    private float centerY;
    // 定义动画的持续事件
    private int duration;
    private Camera camera = new Camera();
    public CustomAnimation(float x,float y,int duration)
    {
        this.centerX = x;
        this.centerY = y;
        this.duration = duration;
    }

    @Override
    public void initialize(int width, int height, int parentWidth, int parentHeight) {
        super.initialize(width, height, parentWidth, parentHeight);
        //设置动画的持续时间
        setDuration(duration);
        //设置动画结束后保留效果
        setFillAfter(true);
        setInterpolator(new LinearInterpolator());
    }

    @Override
    protected void applyTransformation(float interpolatedTime, Transformation t) {
        //super.applyTransformation(interpolatedTime, t);
        camera.save();
        // 根据 interpolatedTime 时间来控制X,Y,Z 上偏移
        camera.translate(100.0f - 100.f * interpolatedTime,150.0f * interpolatedTime - 150,80.0f - 80.0f * interpolatedTime);
        // 根据 interploatedTime 设置在 X 轴 和 Y 轴旋转
        camera.rotateX(360 * interpolatedTime);
        camera.rotateY(360 * interpolatedTime);
        // 获取 Transformation 参数的 Matrix 对象
        Matrix matrix = t.getMatrix();
        camera.getMatrix(matrix);
        matrix.preTranslate(-centerX,-centerY);
        matrix.postTranslate(centerX,centerY);
        camera.restore();
    }
}
```
使用的时候直接调用即可
```
linearLayout.startAnimation(new CustomAnimation(metrics.xdpi/2,metrics.ydpi/2,3500));
```

# 逐帧动画
逐帧动画的原理就是把一系列的静态动画按照一定的顺序播放，形成的动画效果，这种动画在以前的加载动画中会出现

### 利用XML实现逐帧动画
逐帧动画通常是采用 XML 资源进行定义的，需要在 animation-list .../> 标签下使用 <item .../> 子元素标签定义动画的全部帧，并指定各帧的持续时间。
定义逐帧动画的语法格式如下：
```
<animation-list xmlns:android="http://schemas.android.com/apk/res/android"
                  android:oneshot="true|false">
      <item android:drawable="" android:duration=""/>
 </animation-list>
```
其中android:oneshot控制该动画是否循环播放。如果为true，动画将不会循环播放， 否则该动画将会循环播放

创建一个XML文件放在res/drawable文件夹中
这里定义动画的每一帧，素材图片放在drawable文件中

frame_anim.xml
```
<?xml version="1.0" encoding="utf-8"?>
<animation-list xmlns:android="http://schemas.android.com/apk/res/android"
    android:oneshot="true">
    <item android:drawable="@drawable/frame01" android:duration="100"/>
    <item android:drawable="@drawable/frame02" android:duration="100"/>
    <item android:drawable="@drawable/frame03" android:duration="100"/>
    <item android:drawable="@drawable/frame04" android:duration="100"/>
    <item android:drawable="@drawable/frame05" android:duration="100"/>
    <item android:drawable="@drawable/frame06" android:duration="100"/>
    <item android:drawable="@drawable/frame07" android:duration="100"/>
    <item android:drawable="@drawable/frame08" android:duration="100"/>
    <item android:drawable="@drawable/frame09" android:duration="100"/>
    <item android:drawable="@drawable/frame10" android:duration="100"/>
    <item android:drawable="@drawable/frame11" android:duration="100"/>
    <item android:drawable="@drawable/frame12" android:duration="100"/>
    <item android:drawable="@drawable/frame13" android:duration="100"/>
    <item android:drawable="@drawable/frame14" android:duration="100"/>
    <item android:drawable="@drawable/frame15" android:duration="100"/>
    <item android:drawable="@drawable/frame16" android:duration="100"/>
    <item android:drawable="@drawable/frame17" android:duration="100"/>
    <item android:drawable="@drawable/frame18" android:duration="100"/>

</animation-list>
```
在布局文件中直接将AnimationDrawable作为背景
```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical" android:layout_width="match_parent"
    android:layout_height="match_parent">


    <ImageView
        android:layout_marginTop="50dp"
        android:layout_centerHorizontal="true"
        android:id="@+id/frame_image"
        android:layout_width="200dp"
        android:layout_height="200dp"
        android:background="@drawable/frame_animation"
        />

    <LinearLayout
        android:layout_alignParentBottom="true"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        android:layout_margin="5dp"
        android:layout_marginBottom="20dp"
        >
    <Button
        android:id="@+id/frame_start"
        android:layout_weight="1"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:onClick="start"
        android:text="start"
        />

        <Button
            android:id="@+id/frame_stop"
            android:layout_weight="1"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:onClick="stop"
            android:text="stop"
            />
    </LinearLayout>
</RelativeLayout>
```

在Activity中直接使用即可
```
ImageView frame_image;
    AnimationDrawable animationDrawable;
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_frame_animation);
        frame_image = findViewById(R.id.frame_image);
        // 获取 AnimationDrawable 对象
         animationDrawable = (AnimationDrawable) frame_image.getBackground();
    }


    public void start(View view){
        // 开始播放
        animationDrawable.start();
    }

    public void stop(View view){
        //停止播放
        animationDrawable.stop();
    }
```

同时我们可以通过java代码实现逐帧动画

1. 首先创建AnimationDrawable对象
```
    animationDrawable = new AnimationDrawable();

        for (int i = 1; i < 10; i ++ ){
            int id = getResources().getIdentifier("frame0" + i, "drawable", getPackageName());
            Drawable drawable = getResources().getDrawable(id);
            animationDrawable.addFrame(drawable, 100);
        }

        for (int i = 10; i < 19; i ++){
            int id = getResources().getIdentifier("frame" + i, "drawable", getPackageName());
            Drawable drawable = getResources().getDrawable(id);
            animationDrawable.addFrame(drawable, 100);
        }
```
2. 控制播放和停止
```
    public void start(View view){
        // 开始播放
       // animationDrawable.start();

        animationDrawable.setOneShot(true);
        frame_image.setImageDrawable(animationDrawable);
        // 获取资源对象
        animationDrawable.stop();
        // 特别注意：在动画start()之前要先stop()，不然在第一次动画之后会停在最后一帧，这样动画就只会触发一次
        animationDrawable.start();
        // 启动动画

    }

    public void stop(View view){
        //停止播放
       // animationDrawable.stop();
        animationDrawable.setOneShot(true);
        frame_image.setImageDrawable(animationDrawable);
        animationDrawable.stop();
    }
```
完整代码如下：
```
public class FrameAnimation extends AppCompatActivity {
    ImageView frame_image;
    AnimationDrawable animationDrawable;
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_frame_animation);
        frame_image = findViewById(R.id.frame_image);
        // 获取 AnimationDrawable 对象
        // animationDrawable = (AnimationDrawable) frame_image.getBackground();

        animationDrawable = new AnimationDrawable();

        for (int i = 1; i < 10; i ++ ){
            int id = getResources().getIdentifier("frame0" + i, "drawable", getPackageName());
            Drawable drawable = getResources().getDrawable(id);
            animationDrawable.addFrame(drawable, 100);
        }

        for (int i = 10; i < 19; i ++){
            int id = getResources().getIdentifier("frame" + i, "drawable", getPackageName());
            Drawable drawable = getResources().getDrawable(id);
            animationDrawable.addFrame(drawable, 100);
        }


    }


    public void start(View view){
        // 开始播放
       // animationDrawable.start();

        animationDrawable.setOneShot(true);
        frame_image.setImageDrawable(animationDrawable);
        // 获取资源对象
        animationDrawable.stop();
        // 特别注意：在动画start()之前要先stop()，不然在第一次动画之后会停在最后一帧，这样动画就只会触发一次
        animationDrawable.start();
        // 启动动画

    }

    public void stop(View view){
        //停止播放
       // animationDrawable.stop();
        animationDrawable.setOneShot(true);
        frame_image.setImageDrawable(animationDrawable);
        animationDrawable.stop();
    }
}
```

# 属性动画
从Android3.0之后，Android引入了新的动画系统，也就是属性动画，属性动画可以弥补补间动画的一些不足，基本上可以完全替代补间动画。在补间动画中，我们可以通过android.view.animation包下的类和方法帮我们实现多种操作，其实已经是比较完善的，但是在一些特殊的情况下，补间动画并不能满足我们的实际需要，特别是这里说一个，所有的补间动画，虽然在UI效果上是移动了，但是实际上，他在View树中的位置是不变的，也就是说，如果我们想再给这个组件添加点击事件，当我们的动画执行完成之后，点击组件是得不到事件响应的，只有点击组件原来的位置才能响应事件。也就是我看来补间动画最大的缺陷。而属性动画可以很好的避免这样的情况。注意，此处补间动画所有的操作都是针对View的。也就是说，我们是针对一个Button，一个ImageView进行操作。
属性动画机制已经不再是针对于View来设计的了，也不限定于只能实现移动、缩放、旋转和淡入淡出这几种动画操作，同时也不再只是一种视觉上的动画效果了。它实际上是一种不断地对值进行操作的机制，并将值赋值到指定对象的指定属性上，可以是任意对象的任意属性。所以我们仍然可以将一个View进行移动或者缩放，但同时也可以对自定义View中的Point对象进行动画操作了。我们只需要告诉系统动画的运行时长，需要执行哪种类型的动画，以及动画的初始值和结束值，剩下的工作就可以全部交给系统去完成了

## ValueAnimator
ValueAnimator是整个属性动画机制当中最核心的一个类，前面我们已经提到了，属性动画的运行机制是通过不断地对值进行操作来实现的，而初始值和结束值之间的动画过渡就是由ValueAnimator这个类来负责计算的。它的内部使用一种时间循环的机制来计算值与值之间的动画过渡，我们只需要将初始值和结束值提供给ValueAnimator，并且告诉它动画所需运行的时长，那么ValueAnimator就会自动帮我们完成从初始值平滑地过渡到结束值这样的效果。除此之外，ValueAnimator还负责管理动画的播放次数、播放模式、以及对动画设置监听器等，确实是一个非常重要的类。
一个小例子，我们想要将一个值从0过渡到1，耗时300毫秒
```
ValueAnimator anim = ValueAnimator.ofFloat(0f, 1f);  
anim.setDuration(300);  
anim.start();  
```
我们只需要调用ValueAnimator中的ofFloat方法就可以构建一个ValueAnimator对象，ofFloat方法允许传入多个float类型的值，然后这些值就是我们过度的值。再通过setDuration方法设置过度需要的时间，这样姐可以了。最后再通过start方法启动动画即可。之后我们通过注册一个监听器设置基本的操作
```
ValueAnimator anim = ValueAnimator.ofFloat(0f, 1f);  
anim.setDuration(300);  
anim.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {  
    @Override  
    public void onAnimationUpdate(ValueAnimator animation) {  
        float currentValue = (float) animation.getAnimatedValue();  
        Log.d("TAG", "cuurent value is " + currentValue);  
    }  
});  
anim.start();
```
可以看到，这里我们通过addUpdateListener()方法来添加一个动画的监听器，在动画执行的过程中会不断地进行回调，我们只需要在回调方法当中将当前的值取出并打印出来，就可以知道动画有没有真正运行了
同时我们还可以使用传递多个参数，如传入多个值，那么动画的变化时一样的
```
ValueAnimator anim = ValueAnimator.ofFloat(0f, 5f, 3f, 10f);  
anim.setDuration(5000);  
anim.start();  
```
如果我们只需要执行的是int类型，则可以使用ofInt方法。
除此之外还有一些别的常用方法，如setStartDelay，设置延时开始动画的时间，setRepeatCount设置动画重复的次数，setRepeatMode设置重复的模式，RESTART重头开始，REVERSE倒着播放。

## ObjectAnimator
一般情况下，我们通过使用ValueAnimator对值进行平滑过渡，使用的情况比不是非常多，而ObjectAnimator使用的就比较多了。他可以对任意对象的任意值机型操作。虽然说ObjectAnimator使用的情况更多，但是他也是继承自ValueAnimator对象的。
一个小例子，如果我们想要TextView在5秒钟从常规变到透明，在从透明变到常规，就可以使用下面的代码
```
ObjectAnimator animator = ObjectAnimator.ofFloat(textview, "alpha", 1f, 0f, 1f);  
animator.setDuration(5000);  
animator.start(); 
```
可以看到，我们还是调用了ofFloat()方法来去创建一个ObjectAnimator的实例，只不过ofFloat()方法当中接收的参数有点变化了。这里第一个参数要求传入一个object对象，我们想要对哪个对象进行动画操作就传入什么，这里我传入了一个textview。第二个参数是想要对该对象的哪个属性进行动画操作，由于我们想要改变TextView的不透明度，因此这里传入"alpha"。后面的参数就是不固定长度了，想要完成什么样的动画就传入什么值，这里传入的值就表示将TextView从常规变换成全透明，再从全透明变换成常规。之后调用setDuration()方法来设置动画的时长，然后调用start()方法启动动画
```
ObjectAnimator animator = ObjectAnimator.ofFloat(textview, "rotation", 0f, 360f);  
animator.setDuration(5000);  
animator.start(); 
```
如果我们想要把textView移除屏幕，再移动回来
```
float curTranslationX = textview.getTranslationX();  
ObjectAnimator animator = ObjectAnimator.ofFloat(textview, "translationX", curTranslationX, -500f, curTranslationX);  
animator.setDuration(5000);  
animator.start();  
```
我们还可以对TextView进行缩放操作
```
ObjectAnimator animator = ObjectAnimator.ofFloat(textview, "scaleY", 1f, 3f, 1f);  
animator.setDuration(5000);  
animator.start();
```
其实看到这里我们会发现，其实很多情况下对第二个参数的使用会有一定的疑问，我们这里是用过的有scaleX,scaleY,translationX,translationY,alpha，那么这些字符串是有什么特殊的含义吗？其实是肯定的。查看TextView的源码，我们会发现，在TextView中，我们根本就额没有类似alpha这样的属性，那么到底是如何操作，使TextView实现了透明的效果呢？
> textview对象需要根据alpha属性值的改变来不断刷新界面的显示，从而让用户可以看出淡入淡出的动画效果
> 其实ObjectAnimator内部的工作机制并不是直接对我们传入的属性名进行操作的，而是会去寻找这个属性名对应的get和set方法，因此alpha属性所对应的get和set方法应该就是
> public void setAlpha(float value);  
> public float getAlpha(); 
> 这两个方法是由View对象提供的

## 组合动画
独立的动画能够实现的视觉效果毕竟是相当有限的，因此将多个动画组合到一起播放就显得尤为重要。Android团队在设计属性动画的时候也充分考虑到了组合动画的功能，因此提供了一套非常丰富的API来让我们将多个动画组合到一起

实现组合动画功能主要需要借助AnimatorSet这个类，这个类提供了一个play()方法，如果我们向这个方法中传入一个Animator对象(ValueAnimator或ObjectAnimator)将会返回一个AnimatorSet.Builder的实例，AnimatorSet.Builder中包括以下四个方法
- after(Animator anim)   将现有动画插入到传入的动画之后执行
- after(long delay)   将现有动画延迟指定毫秒后执行
- before(Animator anim)   将现有动画插入到传入的动画之前执行
- with(Animator anim)   将现有动画和传入的动画同时执行

这里我们可以让TextView实现先从屏幕外移动到屏幕内，然后在旋转的过程中完成淡入淡出效果
```
ObjectAnimator moveIn = ObjectAnimator.ofFloat(textview, "translationX", -500f, 0f);  
ObjectAnimator rotate = ObjectAnimator.ofFloat(textview, "rotation", 0f, 360f);  
ObjectAnimator fadeInOut = ObjectAnimator.ofFloat(textview, "alpha", 1f, 0f, 1f);  
AnimatorSet animSet = new AnimatorSet();  
animSet.play(rotate).with(fadeInOut).after(moveIn);  
animSet.setDuration(5000);  
animSet.start();
```
我们也可以换一种思路：
```
public void rotateyAnimRuns(final View view) {
        ObjectAnimator anim = ObjectAnimator.offFloat(view,"npl",1.0f,0.0f);       
　　　　 anim.setDuration(500);        
         anim.start();
        anim.addUpdateListener(new AnimatorUpdateListener() {
            @Override
            public void onAnimationUpdate(ValueAnimator animation) {
                float cVal = (Float) animation.getAnimatedValue();
                view.setAlpha(cVal);
                view.setScaleX(cVal);
                view.setScaleY(cVal);
            }
        });
    }
```
这里我们只需要将属性类型写进去，至于具体的操作，我们是在onAnimationUpdate方法中设置的。
再换一种方法实现：
```
public void propertyValuesHolder(View view) {
        PropertyValuesHolder pvhX = PropertyValuesHolder.ofFloat("alpha", 1f, 0f, 1f);
        PropertyValuesHolder pvhY = PropertyValuesHolder.ofFloat("scaleX", 1f, 0, 1f);
        PropertyValuesHolder pvhZ = PropertyValuesHolder.ofFloat("scaleY", 1f, 0, 1f);
        ObjectAnimator.ofPropertyValuesHolder(view, pvhX, pvhY, pvhZ).setDuration(1000).start();
    }
```

## Animator监听器
在很多时候，我们希望可以监听到动画的各种事件，比如动画何时开始，何时结束，然后在开始或者结束的时候去执行一些逻辑处理。这个功能是完全可以实现的，Animator类当中提供了一个addListener()方法，这个方法接收一个AnimatorListener，我们只需要去实现这个AnimatorListener就可以监听动画的各种事件了
ObjectAnimator是继承自ValueAnimator的，而ValueAnimator又是继承自Animator的，因此不管是ValueAnimator还是ObjectAnimator都是可以使用addListener()这个方法的。另外AnimatorSet也是继承自Animator的，因此addListener()这个方法算是个通用的方法
添加一个监听器的代码如下：
```
anim.addListener(new AnimatorListener() {  
    @Override  
    public void onAnimationStart(Animator animation) {  
    }  
  
    @Override  
    public void onAnimationRepeat(Animator animation) {  
    }  
  
    @Override  
    public void onAnimationEnd(Animator animation) {  
    }  
  
    @Override  
    public void onAnimationCancel(Animator animation) {  
    }  
});
```
我们需要实现接口中的四个方法，onAnimationStart()方法会在动画开始的时候调用，onAnimationRepeat()方法会在动画重复执行的时候调用，onAnimationEnd()方法会在动画结束的时候调用，onAnimationCancel()方法会在动画被取消的时候调用.但是也许很多时候我们并不想要监听那么多个事件，可能我只想要监听动画结束这一个事件，那么每次都要将四个接口全部实现一遍就显得非常繁琐。没关系，为此Android提供了一个适配器类，叫作AnimatorListenerAdapter，使用这个类就可以解决掉实现接口繁琐的问题了，如下所示：
```
anim.addListener(new AdimatorAdapter(){})
```
在这个Adapter中，我们是需要重写我们自己所使用的方法即可。

## 使用XML实现动画效果
我们可以使用代码来编写所有的动画功能，这也是最常用的一种做法。不过，过去的补间动画除了使用代码编写之外也是可以使用XML编写的，因此属性动画也提供了这一功能，即通过XML来完成和代码一样的属性动画功能。

通过XML来编写动画可能会比通过代码来编写动画要慢一些，但是在重用方面将会变得非常轻松，比如某个将通用的动画编写到XML里面，我们就可以在各个界面当中轻松去重用它。

如果想要使用XML来编写动画，首先要在res目录下面新建一个animator文件夹，所有属性动画的XML文件都应该存放在这个文件夹当中。然后在XML文件中我们一共可以使用如下三种标签
- <animator> 对应代码中的ValueAnimator
- <objectAnimator>  对应代码中的ObjectAnimator
- <set>  对应代码中的AnimatorSet

这里我们现实的是从0到100的平滑过渡
```
<animator xmlns:android="http://schemas.android.com/apk/res/android"  
    android:valueFrom="0"  
    android:valueTo="100"  
    android:valueType="intType"/>
```
这里我们实现的是透明度的从100%到0的变化
```
<objectAnimator xmlns:android="http://schemas.android.com/apk/res/android"  
    android:valueFrom="1"  
    android:valueTo="0"  
    android:valueType="floatType"  
    android:propertyName="alpha"/>  
```
同时使用XML进行组合动画操作，将TextView从屏幕外移入，然后旋转，并且在旋转的过程中添加淡入淡出效果
```
<set xmlns:android="http://schemas.android.com/apk/res/android"  
    android:ordering="sequentially" >  
  
    <objectAnimator  
        android:duration="2000"  
        android:propertyName="translationX"  
        android:valueFrom="-500"  
        android:valueTo="0"  
        android:valueType="floatType" >  
    </objectAnimator>  
  
    <set android:ordering="together" >  
        <objectAnimator  
            android:duration="3000"  
            android:propertyName="rotation"  
            android:valueFrom="0"  
            android:valueTo="360"  
            android:valueType="floatType" >  
        </objectAnimator>  
  
        <set android:ordering="sequentially" >  
            <objectAnimator  
                android:duration="1500"  
                android:propertyName="alpha"  
                android:valueFrom="1"  
                android:valueTo="0"  
                android:valueType="floatType" >  
            </objectAnimator>  
            <objectAnimator  
                android:duration="1500"  
                android:propertyName="alpha"  
                android:valueFrom="0"  
                android:valueTo="1"  
                android:valueType="floatType" >  
            </objectAnimator>  
        </set>  
    </set>  
  
</set>
```
最后在我们的XML文件编写完成之后，在使用如下代码即可
```
Animator animator = AnimatorInflater.loadAnimator(context, R.animator.anim_file);  
animator.setTarget(view);  
animator.start();  
```

## 属性动画的高级操作





# 参考资料
[Android三种动画](https://www.cnblogs.com/ldq2016/p/5407061.html)
[Android属性动画](https://www.jianshu.com/p/2412d00a0ce4)
[Android动画总结](https://www.jianshu.com/p/609b6d88798d)
