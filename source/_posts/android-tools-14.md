---
title: android奇技淫巧 14 Android7.0 桌面长按图标出现快捷方式
date: 2019-08-20 11:55:10
tags:
  - android
---

Android 7.0版本有一个新特性：如果app支持，可以通过长按app图标出现一些快捷操作。一些热门应用举例： 
<!--more-->
![android7.0图标长摁](/assets/tools/tools_7.0.png)

设置起来比较简单，两种设置方式：静态设置和动态设置

# 静态设置

1. 创建shortcuts.xml配置资源文件
2. 在Manifest中添加meta-data配置

### 创建shortcuts.xml文件

在res中创建xml的文件夹，并且在里面新建一个xml文件shortcuts.xml

![创建文件](/assets/tools/tools_7.0_02.png)

在shortcuts.xml文件中配置如下内容：
```
<!xml version="1.0" encoding="utf-8"?>
<shortcuts xmls:android="http://schemas.android.com/apk/res/android">
    <shortcut
        android:shortcutId="background_setting"
        android:enable="true"
        android:icon="drawable/ic_launcher_setting"
        android:shrtcutShortLabel="String/shortcuts_back_short_label"
        android:shortcutLongLabel="@string/shortcuts_back_long_label"
        android:shortcutDisabledMessaeg="@string/shortcuts_back_disable_message">

        <intent
            android:action="android.intent.action.VIEW"
            android:targetPackage="com.paulniu.ying"
            android:targetClass="com.paulniu.ying.ui.SettingActivity" />
        <categories
            android:name="android.shortcut.conversation" />
    </shortcut>
    <shortcut
        android:shortcutId="pip_settings"
        android:enabled="true"
        android:icon="@drawable/ic_launcher"
        android:shortcutShortLabel="@string/shortcuts_pip_short_label"
        android:shortcutLongLabel="@string/shortcuts_pip_long_label"
        android:shortcutDisabledMessage="@string/shortcuts_pip_disable_message">

        <intent
            android:action="android.intent.action.VIEW"
            android:targetPackage="com.paulniu.ying"
            android:targetClass="compaulniu.ying.PipSetting" />
        <categories
            android:name="android.shortcut.conversation" />
</shortcuts>
```

### 在Manifest中配置
注意：只能在有action是android.intent.action.MAIN和category是android.intent.category.LAUNCHER的Activity中配置才算有效

```
<application>
    <activity
        android:name="MainActivity">
        <intent-filter>
            <action android:name="android.intent.action.MAIN" />
            <category android:name="android.intent.category.LAUNCHER" />
        </intent-filter>

        <meta-data
            android:name="anndroid.app.shortcuts"
            android:resource="@xml/shortcuts">
    </activity>    
</application>
```

# 动态配置

### 动态添加
```
fun onClickshortcutsAdd(v: View) {
        var shortcutManager = getSystemService(ShortcutManager::class.java) as ShortcutManager
        var intent = Intent(this, NotificationChannelActivity::class.java)
        intent.action = Intent.ACTION_VIEW
        var shortcut = ShortcutInfo.Builder(this, "noti_channel_demo")
                .setIcon(Icon.createWithResource(this, R.drawable.ic_launcher))
                .setShortLabel("通知渠道")
                .setLongLabel("通知渠道演示")
                .setIntent(intent)
                .build()
        shortcutManager.addDynamicShortcuts(listOf(shortcut))
    }
```

### 动态删除

动态配置的快捷方式也是可以删除的，并且只能删除动态配置，静态配置是不能动态删除的，从sdk的api提示就可以看出，并没有提供静态配置项的接口

删除方式：
```
fun onClickshortcutsDel(v: View) {
        var shortcutManager = getSystemService(ShortcutManager::class.java) as ShortcutManager
        shortcutManager.removeDynamicShortcuts(listOf("noti_channel_demo"))
    }
```

# 参考博客
[Android 7.0新特性——桌面长按图标出现快捷方式](https://blog.csdn.net/LVXIANGAN/article/details/84104786)