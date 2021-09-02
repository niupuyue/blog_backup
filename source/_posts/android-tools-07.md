---
title: android奇技淫巧 07 Android APK体积优化
date: 2019-07-04 21:44:10
tags:
  - android
---

一般APK体积优化可以从以下几个方面入手
<!--more-->
1. svg的使用和优化
2. Tint着色器的使用和优化
3. 资源打包配置优化
4. 动态库的打包配置优化
5. 移除无用的资源(物理删除和非物理删除)
6. 代码混淆
7. webp转化(api等级18)
8. 资源混淆

# SVG的使用和优化

> 首先我们下来了解一下什么是svg，svg其实就是可以缩放的缩略图。其实就是在不同地方显示不同大小

在自己做一些小项目的时候，我都会从[阿里矢量图](https://www.iconfont.cn/)中去查找，至于具体怎么下载，大家可以去尝试一下。下载下来之后就是一个.svg的文件。我们可以通过Android studio引入矢量图。

具体的操作步骤如下：
res --> new --> vector Asset

如图所示：

![将矢量图引入到Android studio中](/assets/tools/tools-apk-01.png)

之后就会有一个弹窗，如图

![矢量图引入弹窗](/assets/tools/tools-apk-02.png)

在Asset Type中第一个选项表示的是系统图标，第二个表示的是本地图标。这样我们就会生成一个.xml结尾的图标。里面的代码大致是这个样子的(不同的图片会有差别)
```
// 矢量图生成的代码
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="24dp"
    android:height="24dp"
    android:viewportWidth="24.0"
    android:viewportHeight="24.0">
    <path
        android:fillColor="#FF000000"
        android:pathData="M20,11H7.83l5.59,-5.59L12,4l-8,8 8,8 1.41,-1.41L7.83,13H20v-2z" />
</vector>
```
具体里面的语法结构我也不太清楚，不过能够大致猜出一些标签的含义。
在使用的时候跟之前使用图片的方式也不太一样。

```
// 矢量图的使用
app:srcCompat="@drawable/ic_arror_back_black"
```

首先svg是可缩放矢量图，所以我们在项目中只要添加一张svg，就可以在不同的地方使用，并且在使用的时候可以根据不同的大小显示出不同的样式，这样可以大大减少图片资源带来的APK体积增加。

但是只有这些是不行的，因为我们的Android要适配，我们在app->build.gradle中的defaultConfig标签中添加如下的内容

```
 //5.0的兼容适配
        //5.0以下 将svg图片生成指定维度的png图片,下面写几个就会生成几个相应的图片
        vectorDrawables.generatedDensities('xhdpi','xxhdpi')
        //5.0以上 以上使用support-v7进行兼容
        vectorDrawables.useSupportLibrary = true
```

这个是我找到的解决方案，但是我编译了一下试了试。如果我单写顶上那一句，会在相应的文件夹下生成出图片，但是加上后面这句，相应的图片就没有了！我就好奇了，为什么呢？然后我找到了相应的手机试了一下，加不加上面这句没有什么卵用！我是在19版本上测试的！找这样的手机真心费劲，要不是我父母我还真找不到！！！所以呢？大家斟酌一下吧！！！

这个问题，大神们早就帮我们解决了！！！

下面这个是一个批量转换工具！话说没有什么事情能难倒程序员！！！(对我失效)
MegatronKing/SVG-Android
下载这个jar包->svg2vector-cli-1.0.1.jar

然后一波小命令！！！咔咔咔

```
java -jar svg2vector-cli-1.0.1.jar -d D:\svg -o D:\vector

    -d 指定svg文件所在目录
    -f 指定当个svg文件
    -h 设置转换后svg高
    -w 设置转换后svg宽
    -o 输出android vector图像目录
```
然后转换完成，然后复制就好了！！！

# Tint着色器的使用与优化

> 大家在开发的时候不知道有没有过这种体验！在使用状态选择器的时候，需要使用两张一样颜色不同的相同图片？其实使用tint属性完全可以搞定(但是这里指的是纯色的那种图片，那种花花绿绿的你还是乖乖弄吧，除非你想把他变成纯色！)

其实很简单，在展示图片的地方添加

```
app:tint="颜色值"
```

这样就可以改变图片的颜色，那么状态选择器呢？怎么用呢？其实很简单了！下面我们来看代码！

```
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:drawable="@drawable/ic_arrow_back_black_24dp" android:state_pressed="true" />
    <item android:drawable="@drawable/ic_arrow_back_black_24dp" android:state_pressed="false" />
</selector>
```

这里我们会发现在摁下和松开的两种状态使用的同样的资源文件，别着急，我们接着往下看

```
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:color="@android:color/white" android:state_pressed="true" />
    <item android:color="@android:color/black" />
</selector>
```

其实原理是这样的，状态选择器的话呢？只要你通过tint的状态选择器改变图片的颜色就可以了！

但是这里面有几个点需要注意下：

颜色的那个状态选择器要方法color文件夹下；
设置tint的时候要使用app为前缀，否则5.0以下的会报错；
如果你设置的是svg的图片要使用srcCompat如果是正常图片使用src就好了。
基本上这层优化就到这里了！

# 资源打包配置优化

其实这个标题说的有点大，其实就是删除不必要的语言！！！
可能你们没有留意过，在你用Android Studio查看你的apk的时候，会看到这样的东西



