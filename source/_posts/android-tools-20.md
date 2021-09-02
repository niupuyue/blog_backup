---
title: android奇技淫巧 20 ProgressBar的使用
date: 2019-10-14 21:55:10
tags:
  - android
---

android中ProgressBar的使用

<!--more-->

ProgressBar是Android下的进度条，也是为数不多的直接继承自View类的空间，直接子类有AbsSeekBar和ContentLoadingProgressBar，其中AbsSeekBar的子类有SeekBar和RatingBar

## ProgressBar的注意

1. ProgressBar有两个进度，一个是android:progress,另一个是android:secondaryProgress.后者主要是用来为缓存需要所涉及的，例如我们在播放视频时会有一个缓存的进度条和一个播放进度条。
2. ProgressBar分为确定的和不确定的，上面说的播放进度，缓存等就是确定的，相反，不确定的就是不清楚，不确定一个操作需要多长时间来完成，这个时候就需要用的不确定的ProgressBar。这个是由android:indeterminate来控制的，如果设置为true，那么ProgressBar就可能是圆形的滚动条或者水平滚动条，默认情况下，如果是水平进度条，那么就是确定的

### ProgressBar的样式

- Widget.ProgressBar.Horizontal
- Widget.ProgressBar.Small
- Widget.ProgressBar.Large
- Widget.ProgressBar.Inverse
- Widget.ProgressBar.Small.Inverse
- Widget.ProgressBar.Large.Inverse

使用的时候可以这样
```
style="@android:style/Widget.ProgressBar.Small"
```
同样我们可以使用系统的attr的方式
```
style="?android:attr/progressBarStyle"
style="?android:attr/progressBarStyleHorizontal"
style="?android:attr/progressBarStyleInverse"
style="?android:attr/progressBarStyleLarge"
style="?android:attr/progressBarStyleLargeInverse"
style="?android:attr/progressBarStyleSmall"
style="?android:attr/progressBarStyleSmallInverse"
style="?android:attr/progressBarStyleSmallTitle"
```

## 常见的几种样式

在布局中设置
```
android:progress="50" // 设置第一显示进度
android:secondaryProgress="80" // 设置第二现实进度
androi:indeterminate="true" // 设置是否精确显示，true表示不精确显示进度，false表示精确显示进度
```

使用Java代码设置
```
setProgress(int) // 设置第一进度
setSecondaryProgress(int) // 设置第二进度
getProgress() // 获取第一进度
getSecondaryProgress() // 获取第二进度
incrementProgressBy(int) // 增加或减少第一进度
incrementSecondaryProgressBy(int) // 增加或减少第二进度
getMax() // 获取最大进度
```

#### 横向progressBarStyleHorizontal
```
<Progress
  style="?android:attr/progressBarStyleHorizontal"
  android:layout_width="240dp"
  android:layout_hegith="wrap_content"
  android:layout_margin_top="10dp"
  android:max="100"
  android:progress="50" />
```
![横向效果图](/assets/tools/progressbar_01.png)

#### 横向Widget.ProgressBar.Horizontal
```
<ProgressBar 
  style="@android:style/Widget.ProgressBar.Horizontal"
  android:layout_width="240"
  android:layout_height="wrap_content"
  android:layout_gravity="center_horizontal"
  android:layout_marginTop="10dp"
  android:max="100"
  android:progress="50" />
```
![横向效果图](/assets/tools/progressbar_02.png)

#### 圆形progressBarStyleLarge
```
<ProgressBar
  android:layout_gravity="center_horizontal"
  android:layout_marginTop="10dp"
  android:id="@+id/progressBar1"
  style="?android:attr/progressBarStyleLarge"
  android:layout_width="wrap_content"
  android:layout_height="wrap_content" />
```
![圆形效果图](/assets/tools/progressbar_03.png)

#### 圆形普通
```
<ProgressBar
  android:layout_marginTop="10dp"
  android:id="@+id/progressBar02"
  android:layout_width="wrap_content"
  android:layout_height="wrap_content" />
```
![圆形普通效果图](/assets/tools/progressbar_04.png)

#### 圆形小型
```
<ProgressBar
  android:layout_marginTop="10dp"
  android:layout_gravity="center_horizontal"
  android:id="@+id/progressBar3"
  android:layout_width="wrap_content"
  android:layout_height="wrap_content"
  style="?android:attr/progressBarStyleSmall" />
```
![圆形小型效果图](/assets/tools/progressbar_05.png)

## 自定义进度条修改进度颜色
在布局文件中的style属性就是设置进度条样式的
```
<ProgressBar
  style="?android:attr/progressBarStyleHorizontal"
  android:layout_width="match_parent"
  android:layout_height="wrap_content" />
```

实际上面的背景文件是位于@android:style/Widget.ProgressBar.Horizontal,即上面的布局可以写成
```
<ProgressBar
  style="@android:style/Widget.ProgressBar.Horizontal"
  android:layout_width="match_parent"
  android:layout_height="wrap_content" />
```
两种方式的写法效果是一样的
查看系统中的水平进度条风格文件
```
<style name="Widget.ProgressBar.Horizontal">
    <item name="indeterminateOnly">false</item>
    <item name="progressDrawable">@drawable/progress_horizontal</item>
    <item name="indeterminateDrawable">@drawable/progress_indeterminate_horizontal</item>
    <item name="minHeight">20dip</item>
    <item name="maxHeight">20dip</item>
    <item name="mirrorForRtl">true</item>
</style>
```
我们会发现在android:progressDrawable属性是设置进度条背景的
```
<?xml version="1.0" encoding="utf-8"?>
<!-- Copyright (C) 2008 The Android Open Source Project

     Licensed under the Apache License, Version 2.0 (the "License");
     you may not use this file except in compliance with the License.
     You may obtain a copy of the License at

          http://www.apache.org/licenses/LICENSE-2.0

     Unless required by applicable law or agreed to in writing, software
     distributed under the License is distributed on an "AS IS" BASIS,
     WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
     See the License for the specific language governing permissions and
     limitations under the License.
-->

<layer-list xmlns:android="http://schemas.android.com/apk/res/android">
    
    <item android:id="@android:id/background">
        <shape>
            <corners android:radius="5dip" />
            <gradient
                    android:startColor="#ff9d9e9d"
                    android:centerColor="#ff5a5d5a"
                    android:centerY="0.75"
                    android:endColor="#ff747674"
                    android:angle="270"
            />
        </shape>
    </item>
    
    <item android:id="@android:id/secondaryProgress">
        <clip>
            <shape>
                <corners android:radius="5dip" />
                <gradient
                        android:startColor="#80ffd300"
                        android:centerColor="#80ffb600"
                        android:centerY="0.75"
                        android:endColor="#a0ffcb00"
                        android:angle="270"
                />
            </shape>
        </clip>
    </item>
    
    <item android:id="@android:id/progress">
        <clip>
            <shape>
                <corners android:radius="5dip" />
                <gradient
                        android:startColor="#ffffd300"
                        android:centerColor="#ffffb600"
                        android:centerY="0.75"
                        android:endColor="#ffffcb00"
                        android:angle="270"
                />
            </shape>
        </clip>
    </item>
    
</layer-list>
```
这里面有三个item标签，分别是进度条，第二进度条，第一进度条背景色。这里我们可以直接将内容复制下来，然后新建一个xml文件，然后修改背景颜色等属性即可
```
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">

    <item android:id="@android:id/background">
        <shape>
            <corners android:radius="5dip" />
            <gradient
                android:startColor="#ffffffff"
                android:centerColor="#ffffffff"
                android:centerY="0.75"
                android:endColor="#ffffffff"
                android:angle="270"
                />
        </shape>
    </item>

    <item android:id="@android:id/progress">
        <clip>
            <shape>
                <corners android:radius="5dip" />
                <gradient
                    android:startColor="#fff15358"
                    android:centerColor="#fff15358"
                    android:centerY="0.75"
                    android:endColor="#fff15358"
                    android:angle="270"
                    />
            </shape>
        </clip>
    </item>

</layer-list>
```
使用的如下所示
```
            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="10dp"
                android:gravity="center_horizontal"
                android:orientation="vertical">

                <ProgressBar
                    style="@android:style/Widget.ProgressBar.Horizontal"
                    android:layout_width="200dp"
                    android:layout_height="wrap_content"
                    android:layout_margin_top="10dp"
                    android:background="@drawable/bg_progress_color"
                    android:max="100"
                    android:progress="50" />

                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_marginTop="5dp"
                    android:text="修改背景颜色/进度颜色" />

            </LinearLayout>
```

效果如下所示
![改变进度条颜色](/assets/tools/progressbar_06.png)

做一个颜色可以渐变的样式
```
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:id="@android:id/background">
        <shape>
            <corners android:radius="360dp" />
            <solid android:color="#e7e7e7" />
        </shape>
    </item>

    <item android:id="@android:id/secondaryProgress">
        <scale android:scaleWidth="100%">
            <shape android:shape="rectangle">
                <corners android:radius="360dp" />
                <gradient
                    android:centerColor="#f9630c"
                    android:endColor="#f04d52"
                    android:startColor="#ffa902" />
            </shape>
        </scale>
    </item>
</layer-list>
```
使用进度条
```

            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="10dp"
                android:gravity="center_horizontal"
                android:orientation="vertical">

                <ProgressBar
                    style="@android:style/Widget.Holo.ProgressBar.Horizontal"
                    android:layout_width="200dp"
                    android:layout_height="20dp"
                    android:layout_margin_top="10dp"
                    android:max="100"
                    android:progress="50"
                    android:progressDrawable="@drawable/bg_progress_colors" />

                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_marginTop="5dp"
                    android:text="一个比较好看的样式" />

            </LinearLayout>
```
![渐变效果样式](/assets/tools/progressbar_07.png)

[Github例子](https://github.com/niupuyue/blog_demo_android/tree/master/ProgressBarDemo)