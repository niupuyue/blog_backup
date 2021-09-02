---
title: android奇技淫巧 04 超简单实现Android自定义Toast
date: 2019-07-01 20:44:10
tags:
  - android
---
Android自定义Toast
<!--more-->
Bamboy的自定义Toast，（

以下称作“BToast”）
特点在于使用简单，
并且自带两种样式：
1)普通的文字样式；
2)带图标样式。
其中图标有√和×两种图标。

BToast还有另外一个特点就是：
系统自带Toast采用的是队列的方式，
等当前Toast消失后，
下一个Toast才能显示出来；
而BToast会把当前Toast顶掉，
直接显示最新的Toast。

简单三步，
我们现在就开始自定义一下吧！

（一）、Layout：
要自定义Toast，
首先我们需要一个XML布局。

但是在布局之前我们需要三个资源文件，
分别是背景、√和×。

背景可以用XML画出来：
toast_back.xml

```
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android" >
    <corners android:radius="12dp" />

    <solid android:color="#CC000000"/>

</shape>
```
√和×就最好用图片啦，
源码里面有这两张图片，
这里就不贴出来了。

现在就可以写布局了：
toast_layout.xml

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
  android:layout_width="wrap_content"
  android:layout_height="wrap_content"
  android:background="@drawable/toast_back"
  android:gravity="center_vertical"
  android:padding="13dp"
  android:orientation="vertical" >

    <ImageView
        android:id="@+id/toast_img"
        android:layout_width="50dp"
        android:layout_height="50dp"
        android:background="@drawable/toast_y"
        android:layout_gravity="center_horizontal"
        android:layout_marginBottom="5dp" />

    <TextView
        android:id="@+id/toast_text"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center_horizontal"
        android:layout_marginLeft="10dp"
        android:layout_marginRight="10dp"
        android:textColor="#FFFFFF"
        android:gravity="center"
        android:textSize="17sp" />

</LinearLayout>
```

所需要的XML现在已经OK，
剩下的就是Java部分了。

（二）、Java：
写一个BToast类，继承Toast、
成员变量自身单例、
还有构造函数：

```
public class BToast extends Toast {
    /**
     * Toast单例
     */
    private static BToast toast;

    /**
     * 构造
     *
     * @param context
     */
    public BToast(Context context) {
        super(context);
    }

}
```

为了实现可以吧当前Toast顶下去的需求，
我们需要重写几个方法

```
 /**
     * 隐藏当前Toast
     */
    public static void cancelToast() {
        if (toast != null) {
toast.cancel();
        }
    }

    public void cancel() {
        try {
super.cancel();
        } catch (Exception e) {

        }
    }

    @Override
    public void show() {
        try {
super.show();
        } catch (Exception e) {

        }
    }
```

现在我们就可以写我们的逻辑了，

首先当然是引入我们的布局咯：

```
/**
     * 初始化Toast
     *
     * @param context 上下文
     * @param text    显示的文本
     */
    private static void initToast(Context context, CharSequence text) {
        try {
cancelToast();

toast = new BToast(context);

// 获取LayoutInflater对象
LayoutInflater inflater = 
    (LayoutInflater) context.getSystemService(Context.LAYOUT_INFLATER_SERVICE);

// 由layout文件创建一个View对象
View layout = inflater.inflate(R.layout.toast_layout, null);

// 吐司上的图片
toast_img = (ImageView) layout.findViewById(R.id.toast_img);

// 吐司上的文字
TextView toast_text = (TextView) layout.findViewById(R.id.toast_text);
toast_text.setText(text);
toast.setView(layout);
toast.setGravity(Gravity.CENTER, 0, 70);
        } catch (Exception e) {
e.printStackTrace();
        }
    }
```

一切准备工作都已就绪，
接下来就是显示Toast的方法了：

```
/**
     * 图标状态 不显示图标
     */
    private static final int TYPE_HIDE = -1;
    /**
     * 图标状态 显示√
     */
    private static final int TYPE_TRUE = 0;
    /**
     * 图标状态 显示×
     */
    private static final int TYPE_FALSE = 1;

    /**
     * 显示Toast
     *
     * @param context 上下文
     * @param text    显示的文本
     * @param time    显示时长
     * @param imgType 图标状态
     */
    private static void showToast(Context context, CharSequence text, int time, int imgType) {
        // 初始化一个新的Toast对象
        initToast(context, text);

        // 设置显示时长
        if (time == Toast.LENGTH_LONG) {
toast.setDuration(Toast.LENGTH_LONG);
        } else {
toast.setDuration(Toast.LENGTH_SHORT);
        }

        // 判断图标是否该显示，显示√还是×
        if (imgType == TYPE_HIDE) {
toast_img.setVisibility(View.GONE);
        } else {
if (imgType == TYPE_TRUE) {
    toast_img.setBackgroundResource(R.drawable.toast_y);
} else {
    toast_img.setBackgroundResource(R.drawable.toast_n);
}
toast_img.setVisibility(View.VISIBLE);

// 动画
ObjectAnimator.ofFloat(toast_img, "rotationY", 0, 360).setDuration(1700).start();
        }

        // 显示Toast
        toast.show();
    }
```

就是这么简单。

细心的朋友可能发现了，
这个方法是private的，
先别产生疑虑，
听我慢慢道来。

写到这里，
其实你可以直接把这个方法改成Public，
这样的话现在就已经大功告成了，
但是这样的话与原生Toast使用起来有什么区别？
还是需要写那么长一串参数，
唯一的好处就是不用写.show()了。

咱们现在做的事情叫“自定义”，
“自定义”的意思就是我们自己定义规则，
既然如此，
我们何不提升一下“用户体验”呢？
何况这个“用户”还是我们自己。

废话不多说，
我们开始进行最后一步。


(三）、升华：
```
 /**
     * 显示一个纯文本吐司
     *
     * @param context 上下文
     * @param text    显示的文本
     */
    public static void showText(Context context, CharSequence text) {
        showToast(context, text, Toast.LENGTH_SHORT, TYPE_HIDE);
    }

    /**
     * 显示一个带图标的吐司
     *
     * @param context   上下文
     * @param text      显示的文本
     * @param isSucceed 显示【对号图标】还是【叉号图标】
     */
    public static void showText(Context context, CharSequence text, boolean isSucceed) {
        showToast(context, text, Toast.LENGTH_SHORT, isSucceed ? TYPE_TRUE : TYPE_FALSE);
    }

    /**
     * 显示一个纯文本吐司
     *
     * @param context 上下文
     * @param text    显示的文本
     * @param time    持续的时间
     */
    public static void showText(Context context, CharSequence text, int time) {
        showToast(context, text, time, TYPE_HIDE);
    }

    /**
     * 显示一个带图标的吐司
     *
     * @param context   上下文
     * @param text      显示的文本
     * @param time      持续的时间
     * @param isSucceed 显示【对号图标】还是【叉号图标】
     */
    public static void showText(Context context, CharSequence text, int time, boolean isSucceed) {
        showToast(context, text, time, isSucceed ? TYPE_TRUE : TYPE_FALSE);
    }
```
简简单单几个方法，
用户体验瞬间直线飙升，
来看一下使用的时候：

```
public void click(View view) {
        switch (view.getId()) {
case R.id.btn_text:
    BToast.showText(this, "简单提示");
    break;

case R.id.btn_text_true:
    BToast.showText(this, "简单提示 正确图标", true);
    break;

case R.id.btn_text_false:
    BToast.showText(this, "简单提示 错误图标", false);
    break;

case R.id.btn_text_long:
    BToast.showText(this, "简单提示 长~ ", Toast.LENGTH_LONG);
    break;

case R.id.btn_text_true_long:
    BToast.showText(this, "简单提示 正确图标 长~ ", Toast.LENGTH_LONG, true);
    break;

case R.id.btn_text_false_long:
    BToast.showText(this, "简单提示 错误图标 长~ ", Toast.LENGTH_LONG, false);
    break;
        }
    }
```


