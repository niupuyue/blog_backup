---
title: android奇技淫巧 15 Android沉浸式状态栏
date: 2019-09-03 21:55:10
tags:
  - android
---

Android 沉浸式状态栏

<!--more-->

最近一段时间一直在学习Android OpenGL的内容，所以这个系列的博客停下来有一整子了。最近的项目中，UI画了一个沉浸式状态栏样式，一时之间我竟无言以对，所以趁着这个时间将沉浸式状态栏的内容总结一下

首先我们需要先明确一个概念，什么是沉浸式？
其实沉浸式一开始是从VR里面传出来的，体验过VR的小伙伴肯定都知道，在VR体验中，我们能看到的都是VR为我们提供的内容，除此之后再无其他，会让我们有一种置身于虚拟世界之中的感觉，这个就是沉浸式。

那么对应到Android系统中，大多数情况下是用不到沉浸式的，只有一些比如玩游戏，看电影的时候才会用到沉浸式，如下图所示：
![沉浸式1](/assets/tools/tools-statusbar-01.png)
![沉浸式2](/assets/tools/tools-statusbar-02.png)
在用户玩游戏或者看电影的时候，不会被状态栏的内容所干扰这才是真正的沉浸式。

不过虽然听上去高大上的沉浸式效果，实际上也就是将内容全屏化了而已，其实Android沉浸式模式的本质就是全屏化。

## 隐藏状态栏
一个Android应用程序的界面其实有很多系统元素，如下
![状态栏内容](/assets/tools/tools-statusbar-03.png)
通过上图我们会发现，有状态栏，ActionBar，导航栏等组件。而打造沉浸式模式的用户体验，就是要将这个系统元素全部隐藏，只留下主体部分。
例如，我这里在布局文件中添加一个ImageView，那么他的样式应该是这样的
```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <ImageView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:src="@mipmap/bg_01" />

</RelativeLayout>
```
![添加图片效果](/assets/tools/tools-statusbar-04.png)
这样的效果很明显不是我们想要的，所以我们一步一步的来。首先隐藏状态栏，在Android4.1以下和4.1以上的版本中隐藏状态栏和ActionBar的方式不一样。不过这里我只考虑了4.1以上的版本。

先来看一下代码
```
public class Layout1Activity extends AppCompatActivity {

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_layout_1);
        // 隐藏状态栏和ActionBar
        View decorView = getWindow().getDecorView();
        int option = View.SYSTEM_UI_FLAG_FULLSCREEN;
        decorView.setSystemUiVisibility(option);
        ActionBar actionBar = getSupportActionBar();
        actionBar.hide();
    }
}
```
效果:
![隐藏状态栏和ActionBar](/assets/tools/tools-statusbar-05.png)

这里我们先通过getWindow().getDecorView()方法获取当前页面的DecorView，然后在调用他的setSystemUiVisibility()方法来设置系统UI元素的可见性。其中SYSTEM_UI_FLAG_FULLSCREEN表示全屏的意思，也就是会将状态栏隐藏掉。另外，根据Android的设计建议，ActionBar是不应该独立于状态栏单独现实的，所以我们果断把ActionBar使用hide()方法将其隐藏掉。

这个看上去有点沉浸式的样子了，而我们的UI小姐姐想让我做出来的是这样的效果
![预览样式](/assets/tools/tools-statusbar-06.png)

其实也很简单，只需要使用另外一种flag就可以了
```
public class Layout1Activity extends AppCompatActivity {

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_layout_1);
        // 隐藏状态栏和ActionBar
//        View decorView = getWindow().getDecorView();
//        int option = View.SYSTEM_UI_FLAG_FULLSCREEN;
//        decorView.setSystemUiVisibility(option);
//        ActionBar actionBar = getSupportActionBar();
//        actionBar.hide();

        // 另一种效果
        if (Build.VERSION.SDK_INT >= 21){
            View decorView = getWindow().getDecorView();
            int option = View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN | View.SYSTEM_UI_FLAG_LAYOUT_STABLE;
            decorView.setSystemUiVisibility(option);
            getWindow().setStatusBarColor(Color.TRANSPARENT);
        }
        ActionBar actionBar = getSupportActionBar();
        actionBar.hide();
    }
}
```
效果如下所示
![预览样式](/assets/tools/tools-statusbar-07.png)

首先我们注意到，这样的样式只有5.0以上的系统才能支持，所以这里我们先判断，只有大于5.0系统才能使用。接下来我们使用SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN和SYSTEM_UI_FLAG_LAYOUT_STABLE，这两个flat必须结合在一起使用，表示会让应用的主体内容占用系统状态栏的空间，最后在调用Window的setStatusBarColor()方法将状态栏设置为透明颜色。

## 隐藏导航栏
现在我们已经隐藏了状态栏效果，不过在屏幕的下方我们会发现导航栏还是存在的，接下来我们就对导航栏进行隐藏。

其实原理是一样的，隐藏导航栏也就是使用了不同的UI flag而已，修改代码中的部分内容即可
```
public class Layout1Activity extends AppCompatActivity {

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_layout_1);
        // 隐藏状态栏和ActionBar
//        View decorView = getWindow().getDecorView();
//        int option = View.SYSTEM_UI_FLAG_FULLSCREEN;
//        decorView.setSystemUiVisibility(option);
//        ActionBar actionBar = getSupportActionBar();
//        actionBar.hide();

        // 另一种效果
//        if (Build.VERSION.SDK_INT >= 21){
//            View decorView = getWindow().getDecorView();
//            int option = View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN | View.SYSTEM_UI_FLAG_LAYOUT_STABLE;
//            decorView.setSystemUiVisibility(option);
//            getWindow().setStatusBarColor(Color.TRANSPARENT);
//        }
//        ActionBar actionBar = getSupportActionBar();
//        actionBar.hide();

        // 隐藏导航栏
        View decorView = getWindow().getDecorView();
        int option = View.SYSTEM_UI_FLAG_HIDE_NAVIGATION | View.SYSTEM_UI_FLAG_FULLSCREEN;
        decorView.setSystemUiVisibility(option);
        ActionBar actionBar = getSupportActionBar();
        actionBar.hide();
    }
}
```

![效果预览](/assets/tools/tools-statusbar-08.png)

这里我们同时使用了SYSTEM_UI_FLAG_HIDE_NAVIGATION和SYSTEM_UI_FLAG_FULLSCREEN，这样就可以将状态栏和导航栏同时隐藏

虽然我们实现了全屏化，但是还是相差比较远，因为再这样的模式下，只要我们随意的触摸屏幕，就会退出全屏模式。

除了隐藏导航栏之外，我们也要实现刚才的透明状态栏的效果，其实就是将两部分代码合并一下。

```
public class Layout1Activity extends AppCompatActivity {

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_layout_1);
        // 隐藏状态栏和ActionBar
//        View decorView = getWindow().getDecorView();
//        int option = View.SYSTEM_UI_FLAG_FULLSCREEN;
//        decorView.setSystemUiVisibility(option);
//        ActionBar actionBar = getSupportActionBar();
//        actionBar.hide();

        // 另一种效果
//        if (Build.VERSION.SDK_INT >= 21){
//            View decorView = getWindow().getDecorView();
//            int option = View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN | View.SYSTEM_UI_FLAG_LAYOUT_STABLE;
//            decorView.setSystemUiVisibility(option);
//            getWindow().setStatusBarColor(Color.TRANSPARENT);
//        }
//        ActionBar actionBar = getSupportActionBar();
//        actionBar.hide();

        // 隐藏导航栏
//        View decorView = getWindow().getDecorView();
//        int option = View.SYSTEM_UI_FLAG_HIDE_NAVIGATION | View.SYSTEM_UI_FLAG_FULLSCREEN;
//        decorView.setSystemUiVisibility(option);
//        ActionBar actionBar = getSupportActionBar();
//        actionBar.hide();

        if (Build.VERSION.SDK_INT >= 21){
            View decorView = getWindow().getDecorView();
            int option = View.SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION
                    | View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN
                    | View.SYSTEM_UI_FLAG_LAYOUT_STABLE;
            decorView.setSystemUiVisibility(option);
            getWindow().setNavigationBarColor(Color.TRANSPARENT);
            getWindow().setStatusBarColor(Color.TRANSPARENT);
        }
        ActionBar actionBar = getSupportActionBar();
        actionBar.hide();
    }
}
```

![效果预览](/assets/tools/tools-statusbar-09.png)

这里我们使用SYSTEM_UI_FLAG_HIDE_NAVIGATION表示会让应用的主体内容占用系统导航栏的空间，然后又调用了setNavigation()方法将导航栏设置为透明颜色

## 真正的沉浸式模式

其实不管我们是否误解了沉浸式模式，但是这种模式的的确确的存在，那么我们应该怎么样才能实现像视频播放或者游戏中这样的模式呢？我们只需要重写Activity中的onWindowFocusChanged()方法，然后加入如下的逻辑即可

```
public class Layout1Activity extends AppCompatActivity {

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_layout_1);
        // 隐藏状态栏和ActionBar
//        View decorView = getWindow().getDecorView();
//        int option = View.SYSTEM_UI_FLAG_FULLSCREEN;
//        decorView.setSystemUiVisibility(option);
//        ActionBar actionBar = getSupportActionBar();
//        actionBar.hide();

        // 另一种效果
//        if (Build.VERSION.SDK_INT >= 21){
//            View decorView = getWindow().getDecorView();
//            int option = View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN | View.SYSTEM_UI_FLAG_LAYOUT_STABLE;
//            decorView.setSystemUiVisibility(option);
//            getWindow().setStatusBarColor(Color.TRANSPARENT);
//        }
//        ActionBar actionBar = getSupportActionBar();
//        actionBar.hide();

        // 隐藏导航栏
//        View decorView = getWindow().getDecorView();
//        int option = View.SYSTEM_UI_FLAG_HIDE_NAVIGATION | View.SYSTEM_UI_FLAG_FULLSCREEN;
//        decorView.setSystemUiVisibility(option);
//        ActionBar actionBar = getSupportActionBar();
//        actionBar.hide();

//        if (Build.VERSION.SDK_INT >= 21){
//            View decorView = getWindow().getDecorView();
//            int option = View.SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION
//                    | View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN
//                    | View.SYSTEM_UI_FLAG_LAYOUT_STABLE;
//            decorView.setSystemUiVisibility(option);
//            getWindow().setNavigationBarColor(Color.TRANSPARENT);
//            getWindow().setStatusBarColor(Color.TRANSPARENT);
//        }
//        ActionBar actionBar = getSupportActionBar();
//        actionBar.hide();
    }

    @Override
    public void onWindowFocusChanged(boolean hasFocus) {
        super.onWindowFocusChanged(hasFocus);
        if (hasFocus && Build.VERSION.SDK_INT >= 19){
            View decorView = getWindow().getDecorView();
            int option = View.SYSTEM_UI_FLAG_LAYOUT_STABLE
                    | View.SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION
                    | View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN
                    | View.SYSTEM_UI_FLAG_HIDE_NAVIGATION
                    | View.SYSTEM_UI_FLAG_FULLSCREEN
                    | View.SYSTEM_UI_FLAG_IMMERSIVE_STICKY;
            decorView.setSystemUiVisibility(option);
        }
    }
}
```
![效果预览](/assets/tools/tools-statusbar-10.png)

沉浸式模式的UI flag就这些了，如果我们真的需要实现沉浸式，直接将上面的代码应用到Activity中即可。需要注意的是，这种沉浸式模式只有在Android4.4以上的版本才支持，所以这里也是要加判断的
如果我们需要横屏展示，只需要在配置文件中加入如下代码即可
```
<activity android:name=".MainActivity"
    android:screenOrientation="landscape" />
```

[demo地址](https://github.com/xiaoniudadi/blog_demo_android/tree/master/ImmersiveStatusBarDemo)


# 参考博客
[Android状态栏微技巧，带你真正理解沉浸式模式](https://blog.csdn.net/guolin_blog/article/details/51763825)
