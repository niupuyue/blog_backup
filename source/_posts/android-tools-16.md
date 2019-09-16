---
title: android奇技淫巧 16 Android App 不显示在最近使用过的应用列表中
date: 2019-09-12 21:55:10
tags:
  - android
---

android App最近使用过的应用列表

<!--more-->

最近在一次使用脉脉的时候发现了一个比较骚的操作。我明明是有运行脉脉的，但是当我把脉脉切换到后台之后，在最近使用的软件列表中，没有找到脉脉这个软件。
我当时就觉得这个功能虽然可有可无，因为当我执行清除后台应用程序的时候，实际上也是将脉脉后台进程杀死了的，但是总感觉以后可能会用上这样的功能，比如当你浏览一些敏感信息的时候，切换到后台却不想让别人看到，就可以将后台进程设置为不可见状态。

好了，废话不多说，直接看一下如何实现这样的效果。

想要实现这样的效果只需要在清单配置文件中填写如下的内容即可

```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.paulniu.nobackrunningstate">

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        <activity android:name=".MainActivity"
            android:excludeFromRecents="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>
```
嗯，就这么多