---
title: android奇技淫巧 10 SoftInputMode属性介绍
date: 2019-07-12 22:42:10
tags:
  -android
---

android软键盘属性SoftInputMode属性设置介绍
<!--more-->
SoftInputMode用来设置软键盘的各种属性.有两种方式可以设置，第一种是在清单配置文件也就是AndroidManifest.xml文件，另一种也就是在java代码中动态设置

```
// 第一种  xml清单文件中进行设置
 android:windowSoftInputMode="adjustResize"

//第二种  代码设置 (在Activity对象中  可以调用  getWindow()  方法   )
getWindow().setSoftInputMode(WindowManager.LayoutParams.SOFT_INPUT_ADJUST_RESIZE | WindowManager.LayoutParams.SOFT_INPUT_STATE_HIDDEN );
```
windowSoftInputMode共有9个取值：
stateUnspecified，stateUnchanged，stateHidden，stateAlwaysHidden，stateVisible，stateAlwaysVisible，adjustUnspecified，adjustResize，adjustPan。
这9个取值可以划分成两大类 ： StateXXX 类型，和AdjustXXX类型。
其中StateXXX类型的设置主要与软键盘的显示有关（例如 刚进入界面的时候软键盘是否显示）
而AdjustXXX类型主要涉及到软键盘弹出时候界面的调整（例如 在回复评论信息的时候我们希望软键盘弹出的时候能够把底下的EditText输入框也顶上去）

## stateXXX各种类型

#### stateUnspecified

这是系统默认的设置，系统会根据界面要求来显示软键盘，一般来说软键盘是不会自动弹出的，但是当有获得焦点的输入框有滚动需求时候就会自动弹出软键盘（例如EditText外面嵌套了ScrollView，并且EditText获得焦点）。

####  stateUnchanged

Unchanged就是不改变的意思，在这里的使用的一种情况就是，当前的activiy已经弹出了软键盘，当进入到新界面时候，软键盘仍然是弹出的状态。如果之前是隐藏的那么跳转的新界面软键盘也是隐藏的，保持不变（Unchanged）。

#### stateHidden

这种设置下，进入界面时候，软键盘都是隐藏的。

#### stateAlwaysHidden

#### stateVisible

在一般情动情况下会弹出软键盘，计时没有输入框EditText

#### stateAlwaysVisible

这个属性与上一个<pre>stateAlwaysVisible</pre>类似，不过有一些小区别，设置为stateVisible时候，如果从一个没有软键盘弹出的界面返回到当前界面时候是不会弹出软键盘的，但是设置为stateAlwaysVisible模式，即使返回之前的那个界面没有软键盘弹出，退回到当前界面的时候软键盘也是会弹起的，这也就是  AlwaysVisible 。

## adjustXXX各种类型

上面的是关于软键盘弹出的时机，也就是说控制软键盘是否弹出来的。现在的<pre>adjustXXX</pre>表示的是软键盘显示的时候对于整个页面的影响。

#### adjustUnspecified

这是系统默认的设置，系统会根据界面选择不同的模式。如果界面里面有可以滚动的控件，比如ScrowView，系统会减小可以滚动的界面的大小，从而保证即使软键盘显示出来了，也能够看到所有的内容。如果布局里面没有滚动的控件，那么软键盘可能就会盖住一些内容。

#### justResize

这是比较常用的用来调整输入框位置的属性，一般设置为这个属性就可以实现底部输入框被顶上去的效果。
这是由于软键盘是一个Dialog，也就是一个Window，Window可以说是View的一个容器，我们自己编写的布局文件所生成的View都是在与Activity绑定的那个Window之中，当软键盘弹出的时候，整个屏幕就被切分成了两部分，上面的View布局和下面的软键盘，这时候上面的根View会调整自己的大小（压缩自己）来适应这种变化，之后会从View的根部重新请求测量、布局、绘制。这时候软键盘的上面就成了新的底部，输入框在软键盘上面重新绘制自己，直观上就变成了输入框被软键盘给顶上去了。

#### justPan

这种模式与adajustResize的区别就在于视图View不会重新调整自己的高度，而是选择向上滚动一段距离，来让下面的输入框显示出来。当然带来的问题可能就是顶部的ActionBar等View会被顶出视线外。



# 参考资料

[SoftInputMode属性介绍](https://www.jianshu.com/p/3d5d5d60d336)
