---
title: 重拾android路(二) 手机适配
date: 2016-01-09 16:40:42
tags:
  - android
---

随着android智能手机的发展和普及，各种各样的大小和尺寸的android智能机不断的退出，通过各种各样的设备机型，我们能够让自己的APP接触到广大的用户。为了能在各种android平台上使用，我们的APP需要兼容不同的设备类型，比如语言，屏幕尺寸，android系统的版本等重要的变量因素。希望能够通过这次总结，复习符合使用基础的平台功能，利用替代资源和其他功能，使APP仅用一个程序包，就能想用android兼容设备的用户提供更好的用户体验
<!--more-->
# 设配不同的语言

把UI中的字符串存储到外部文件，通过代码提取这种方式可以实现。当我们通过AS创建一个项目工程的时候，就会在工程的根目录中创建一个res的目录，目录中包含所有的资源类型的子目录。其中就包好工程的默认文件res/values/strings.xml，用于保存字符串值。

## 创建区域设置目录及字符串文件

为了支持多国语言，在res中创建一个额外的values目录，以连字符和ISO国家代码结尾来命名，如果决定支持某种语言，则需要创建资源子目录和字符串资源文件

![支持了不同国家语言](/assets/android/lang-1.png)

添加不同区域语言的字符串值到相应的文件

android系统运行时，会根据当前用户的区域设置，使用相应的字符串资源。

如下几个不同语言对应的不同字符串资源

英语(默认区域语言) values/strings.xml

```
<resources>
    <string name="app_name">Android_Demo_01</string>
</resources>
```
西班牙语: values-es/strings.xml
```
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="title">Mi Aplicación</string>
    <string name="hello_world">Hola Mundo!</string>
</resources>
```
法语: values-fr/strings.xml
```
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="title">Mon Application</string>
    <string name="hello_world">Bonjour le monde !</string>
</resources>
```

## 使用字符资源

我们可以通过源代码和其他XML文件总通过<pre><string></pre>元素的name属性来引入自己的字符串资源

在源代码中可以通过R.string.<string_name>语法来引入一个字符串资源，很多方法都可以通过这种方式来接受字符串

```
String ss = getResources().getString(R.string.app_name);
```

在其他的XML文件中，每当XML属性需要接受一个字符串值时，我们可以通过@string/<string_name>的方式来引入
```
<TextView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@string/hello_world" />
```

# 屏幕适配

## 定义

使得某一元素在android不同尺寸，不同分辨率的手机上具备相同的显示效果

### 屏幕尺寸

- 含义: 手机对角线的物理尺寸
- 单位: 英寸 1英寸 = 2.54厘米

> 目前来说android常见的尺寸有5寸，5.5寸，6寸

### 屏幕分辨率

- 含义: 手机荷香，纵向上像素点数的综合

> 一般描述成屏幕的"宽x高"=AxB
> 含义: 屏幕在横向上有A个像素点，在纵向上有B个像素点
> 例如:1080x1920 即宽度上有1080个像素点，高度上有1920个像素点

- 单位：px 1px = 1像素

- android常见的分辨率是320x480 480x800 720x1280 1080x1920

### 屏幕像素密度

- 含义: 每英寸上的像素点数
- 单位: dpi

> 假设设备每英寸有160个像素，该设备的屏幕像素度是160dpi

- android手机对于每类手机屏幕大小都有一个相应的屏幕像素密度

| 密度类型        | 代表的分辨率（px） | 屏幕像素密度（dpi）|
| -------------    |:-------------:|
| 低密度（ldpi）    | 240x320       | 120 |
| 中密度（mdpi）    | 320x480       | 160 |
| 高密度（hdpi）    | 480x800       | 240|
| 超高密度（xhdpi） | 720x1280      | 320|
| 超超高密度（xxhdpi） | 1080x1920   | 480 |

### 密度无关像素

- 含义: 与终端上的实际物理像素点无关
- 单位: dp 可以保证在不同屏幕像素密度的设备上显示相同的效果

> 1. android开发室用dp而不是px单位设置图片大小，是android特有的单位
> 2. 场景：假如同样都是画一条长度是屏幕一半的线，如果使用px作为计量单位，那么在480x800分辨率手机上设置应为240px；在320x480的手机上应设置为160px，二者设置就不同了；如果使用dp为单位，在这两种分辨率下，160dp都显示为屏幕一半的长度

- dp和px的转换
因为ui设计师给你的设计图是以px为单位的，Android开发则是使用dp作为单位的，那么我们需要进行转换

| 密度类型 | 代表的分辨率（px） | 屏幕密度（dpi）|换算（px/dp） |比例|
| ------------- |:-------------:| -------------:| -------------:|
| 低密度（ldpi） | 240x320 | 120 |1dp=0.75px|3|
| 中密度（mdpi） | 320x480 | 160 |1dp=1px|4|
| 高密度（hdpi） | 480x800 | 240|1dp=1.5px|6|
| 超高密度（xhdpi） | 720x1280 | 320|1dp=2px|8|
| 超超高密度（xxhdpi） | 1080x1920 | 480 |1dp=3px|12|

在android中，规定以160dpi(即分辨率为320x480为基准),1dp = 1px

### 独立比例像素

- 含义: sp
- 单位：sp

> 1. Android开发时用此单位设置文字大小，可根据字体大小首选项进行缩放
> 2. 荐使用12sp、14sp、18sp、22sp作为字体设置的大小，不推荐使用奇数和小数，容易造成精度的丢失问题；小于12sp的字体会太小导致用户看不清

<font color=gray size=72>以上是基本概念，需要牢记</font>

## 屏幕适配的本质

- 使得布局，不及组件，图片资源，用户界面流程匹配不同的屏幕尺寸

> 使得布局，布局组件自适应屏幕尺寸
> 根据屏幕的配置来加载相应的UI布局和用户界面流程

- 使得图片资源匹配不同的屏幕密度

### 布局匹配

#### 使得布局元素自适应屏幕尺寸

- 做法:使用相对布局(relativeLayout)，禁止使用绝对布局(AbsoluteLayout)

> android中的布局样式分为4种
> 1. 线性布局
> 2. 相对布局
> 3. 帧布局
> 4. 绝对布局

针对不同的布局我们需要在不同的场景下使用

1. 相对布局  布局的子控件之间使用相对位置的方式排列，因为RelativeLayout讲究的是相对位置，即使屏幕的大小改变，视图之前的相对位置都不会变化，与屏幕大小无关，灵活性很强
2. 线性布局  通过多层嵌套LinearLayout和组合使"wrap_content"和"match_parent"已经可以构建出足够复杂的布局。但是LinearLayout无法准确地控制子视图之间的位置关系，只能简单的一个挨着一个地排列

#### 最小宽度(Smallest-width)限定符

- 定义: 通过制定某个最小宽度(以dp为单位)来精确定位屏幕从而家在不同的UI资源
- 例子: 使用layout-sw 600dp的最小宽度限定符，即无论是宽度还是高度，只要大于600dp，就采用layout-sw 600dp目录中的布局文件

代码展示:
- 适配手机单面板(默认) 布局文件路径: res/layout/main.xml
```
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <fragment android:id="@+id/headlines"
              android:layout_height="fill_parent"
              android:name="com.example.android.newsreader.HeadlinesFragment"
              android:layout_width="match_parent" />
</LinearLayout>
```
- 适配尺寸较大的平板电脑 布局路径: res/layout-sw600dp/main.xml
```
LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
    android:orientation="horizontal">
    <fragment android:id="@+id/headlines"
              android:layout_height="fill_parent"
              android:name="com.example.android.newsreader.HeadlinesFragment"
              android:layout_width="400dp"
              android:layout_marginRight="10dp"/>
    <fragment android:id="@+id/article"
              android:layout_height="fill_parent"
              android:name="com.example.android.newsreader.ArticleFragment"
              android:layout_width="fill_parent" />
</LinearLayout>
```

当屏幕最小宽度大于或者等于600dp的设备，系统会自动加载layout-sw600dp、main.xml布局。否则系统就会选择默认布局

> 注意，这里的最小宽度限定符仅用于android3.2以后的版本

#### 使用布局别名

像上面的例子中，我们已经拥有了两个不同的布局文件，那么为了能够能够适配不同尺寸，我们可以通过使用布局别名的方式

- 适配手机单页面(比较小的尺寸)  res/layout/main.xml
- 适配手机尺寸比较大的  res/layout/main_twopanes.xml

然后加入以下的两个文件，

1. res/values-large/layout.xml (默认)
```
<resources>
    <item name="main" type="layout">@layout/main_twopanes</item>
</resources>
```
2. res/values-sw600dp/layout.xml
```
<resources>
<item name="main" type="layout">@layout/main_twopanes</item>
</resources>
```

a. 版本低于3.2的会自动匹配large的文件
b. 版本高于3.2的会自动匹配sw600dp的文件

> 以上的两个layout.xml文件可以引入相同的文件,避免了重复定义布局文件的情况

#### 屏幕方向限定符

使用场景: 根据屏幕方向进行布局调整

> 小屏幕, 竖屏: 单面板
> 小屏幕, 横屏: 单面板
> 7 英寸平板电脑，纵向：单面板，带操作栏
> 7 英寸平板电脑，横向：双面板，宽，带操作栏
> 10 英寸平板电脑，纵向：双面板，窄，带操作栏
> 10 英寸平板电脑，横向：双面板，宽，带操作栏
> 电视，横向：双面板，宽，带操作栏

方法是
1. 先定义类别: 单/双面板,是否带操作栏，宽/窄
> 定义在res/layout/目录下的某个XML文件中
2. 在进行相应的匹配: 屏幕尺寸(小屏 7寸 10寸) 方向(横，纵)
> 使用布局别名进行匹配

步骤：
1. 在res/layout/目录下的某个XML文件中定义所需要的布局类型
res/layout/onepane.xml
```
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"  
    android:orientation="vertical"  
    android:layout_width="match_parent"  
    android:layout_height="match_parent">  
  
    <fragment android:id="@+id/headlines"  
              android:layout_height="fill_parent"  
              android:name="com.example.android.newsreader.HeadlinesFragment"  
              android:layout_width="match_parent" />  
</LinearLayout>
```
res/layout/onepane_with_bar.xml
```
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"  
    android:orientation="vertical"  
    android:layout_width="match_parent"  
    android:layout_height="match_parent">  
    <LinearLayout android:layout_width="match_parent"   
                  android:id="@+id/linearLayout1"    
                  android:gravity="center"  
                  android:layout_height="50dp">  
        <ImageView android:id="@+id/imageView1"   
                   android:layout_height="wrap_content"  
                   android:layout_width="wrap_content"  
                   android:src="@drawable/logo"  
                   android:paddingRight="30dp"  
                   android:layout_gravity="left"  
                   android:layout_weight="0" />  
        <View android:layout_height="wrap_content"   
              android:id="@+id/view1"  
              android:layout_width="wrap_content"  
              android:layout_weight="1" />  
        <Button android:id="@+id/categorybutton"  
                android:background="@drawable/button_bg"  
                android:layout_height="match_parent"  
                android:layout_weight="0"  
                android:layout_width="120dp"  
                style="@style/CategoryButtonStyle"/>  
    </LinearLayout>  
  
    <fragment android:id="@+id/headlines"   
              android:layout_height="fill_parent"  
              android:name="com.example.android.newsreader.HeadlinesFragment"  
              android:layout_width="match_parent" />  
</LinearLayout>
```
res/layout/twopanes.xml
```
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
    android:orientation="horizontal">
    <fragment android:id="@+id/headlines"
              android:layout_height="fill_parent"
              android:name="com.example.android.newsreader.HeadlinesFragment"
              android:layout_width="400dp"
              android:layout_marginRight="10dp"/>
    <fragment android:id="@+id/article"
              android:layout_height="fill_parent"
              android:name="com.example.android.newsreader.ArticleFragment"
              android:layout_width="fill_parent" />
</LinearLayout>
```
res/layout/twopanes_narrow.xml(双面板，窄布局)
```
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
    android:orientation="horizontal">
    <fragment android:id="@+id/headlines"
              android:layout_height="fill_parent"
              android:name="com.example.android.newsreader.HeadlinesFragment"
              android:layout_width="200dp"
              android:layout_marginRight="10dp"/>
    <fragment android:id="@+id/article"
              android:layout_height="fill_parent"
              android:name="com.example.android.newsreader.ArticleFragment"
              android:layout_width="fill_parent" />
</LinearLayout>
```

2. 使用布局别名进行相应的匹配
res/values/layouts.xml
```
<resources>  
    <item name="main_layout" type="layout">@layout/onepane_with_bar</item>  
    <bool name="has_two_panes">false</bool>  
</resources>
```
> 可为resources设置bool，通过获取其值来动态判断目前已处在哪个适配布局

res/values-sw600dp-land/layouts.xml
大屏、横向、双面板、宽-Andorid 3.2版本后
```
<resources>
    <item name="main_layout" type="layout">@layout/twopanes</item>
    <bool name="has_two_panes">true</bool>
</resources>
```
res/values-sw600dp-port/layouts.xml
（大屏、纵向、单面板带操作栏-Andorid 3.2版本后）
```
<resources>
    <item name="main_layout" type="layout">@layout/onepane</item>
    <bool name="has_two_panes">false</bool>
</resources>
```
res/values-large-land/layouts.xml
（大屏、横向、双面板、宽-Andorid 3.2版本前）
```
<resources>
    <item name="main_layout" type="layout">@layout/twopanes</item>
    <bool name="has_two_panes">true</bool>
</resources>
```
res/values-large-port/layouts.xml
（大屏、纵向、单面板带操作栏-Andorid 3.2版本前）
```
<resources>
    <item name="main_layout" type="layout">@layout/onepane</item>
    <bool name="has_two_panes">false</bool>
</resources>
```

> 这里并没有写完，可自行补充

### 布局组件匹配

使得布局组件自适应屏幕尺寸
使用wrap_content,match_parent和weight来控制试图组件的宽度和高度

> 关于weigth权重的问题，这边有个小哥哥，写的很好，可供参考[androidweight权重](http://mobile.51cto.com/abased-375428.htm)

### 图片资源匹配

使得图片资源在不同屏幕密度上显示相同的像素效果

- 使用自动拉伸位图:Nine-patch的图片类型
假设需要匹配不同屏幕大小，你的图片资源也必须自动适应各种屏幕尺寸
> 例如：一个按钮的背景图片必须随着按钮的大小改变而改变
> 解决方案: 使用自动拉伸位图,后缀名.9.png，它是一种被特殊处理过的png图片，设计师可以指定图片的拉伸区域和非拉伸区域，使用时，系统会根据空间大小自动的拉伸你想要拉伸的部分

### 创建不同的bitmap

我们应该为四种普通分辨率:高，中，低，超高精度，都提供相应设配的bitmap资源
要生成这些图像，应该从原始的矢量图像资源着手，然后根据下列尺寸比例，生成各种密度的图像

- xhdpi:2.0
- hdpi:1.5
- mdpi:1.0
- ldpi:0.75

这也就意味着，如果针对xhdpi的设备生成了一张200x200的图像，那么应该为hdpi生成150x150,为mdpi生成100x100, 和为ldpi生成75x75的图片资源
然后在将这些文件放在相应的drawable资源目录中
[](/assets/android/screen-1.png)

任何时候，当引用@drawable/ic_launch是系统就会自动使用恰当的bitmap

# 适配不同的系统版本

新的Android版本会为我们的app提供更棒的APIs，但我们的app仍应支持旧版本的Android，直到更多的设备升级到新版本为止
当我们通过AS创建一个应用时，有一些地方的数据信息对我们开发APP应用是有很大的帮助的
[](/assets/android/androidversion-1.png)
所以，在一般情况下，在更新APP到最新版本的android时，最好保证新版本的APP可以支持90%

## 指定最小和目标API级别

android的清单配置文件中描述了我们的APP的细节，以及APP支持哪些android版本。具体说<uses-sdk>元素中的minSdkVersion和targetSdkVersion属性，表明了在设计和测试APP是最低兼容API的级别和最高使用的API级别
当然现在这个属性设置在了gradle中
```
android {
    compileSdkVersion 26
    buildToolsVersion "26.0.1"
    defaultConfig {
        applicationId "com.paul.android_demo_01"
        minSdkVersion 18
        targetSdkVersion 26
        versionCode 1
        versionName "1.0"
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}
```
随着新版本Android的发布，一些风格和行为可能会改变，为了能使app能利用这些变化，而且能适配不同风格的用户的设备，我们应该将targetSdkVersion的值尽量的设置与最新可用的Android版本匹配

## 运行时检查系统版本

android在build常量类中提供了对每一个版本的唯一代号，在我们的APP中使用这些代号可以建立条件，保证依赖于高级别的API代码，只会在这些API在当前系统中可用
```
private void setUpActionBar(){
        //确保我们当前运行的版本是大于等于Honeycomb版本(sdk11)
        if(Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB){
            android.app.ActionBar actionBar = getActionBar();
            actionBar.setDisplayHomeAsUpEnabled(true);
        }
    }
}
```

### 使用平台风格和主题

Android提供了用户体验主题，为app提供基础操作系统的外观和体验。这些主题可以在manifest文件中被应用于app中。通过使用内置的风格和主题，我们的app自然地随着Android新版本的发布，自动适配最新的外观和体验.

例如
使activity看起来像一个对话框:
```
<activity android:theme="@android:style/Theme.Dialog">
```
使Activity看起来透明的背景
```
<activity android:theme="@android:style/Theme.Translucent">
```
应用在/res/values/styles.xml中定义的自定义主题:
```
<activity android:theme="@style/CustomTheme">
```
使整个app应用一个主题(全部activities)在元素中添加android:theme属性:
```
<application android:theme="@style/CustomTheme">
```



未完待续···


# 参考资料

[android官方培训课-胡凯大神翻译整理](http://hukai.me/android-training-course-in-chinese/index.html)
[stormzhang:android屏幕设配](http://blog.csdn.net/guolin_blog/article/details/8830286)
[鸿祥:android屏幕适配方案](http://blog.csdn.net/lmj623565791/article/details/45460089)
[郭霖大神:android官方提供的支持不同屏幕大小的方法](http://blog.csdn.net/guolin_blog/article/details/8830286)
