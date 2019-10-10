---
title: android奇技淫巧 13 Android内部存储和外部存储
date: 2019-08-13 17:33:10
tags:
  - android
---

在Android开发中我们经常会听到这几个概念，内存，外部存储，内部存储，很多时候我们常常把这个概念弄混淆，所以这篇博客就在这里做一下总结
<!--more-->
内存(Menory),内部存储(InternalStorage),外部存储(ExternalStorage)，通过英文翻译很容易混淆
我们可以对Android手机存储空间做以下划分
1. 内存：RAM
2. 内部存储：内部ROM
3. 外部存储：外部ROM和SDCard

其中最容易混淆的就是外部存储，如果说PC上也要区分内部存储和外部存储的话，那么自带硬盘就算是内部存储，U盘或者移动硬盘就属于外部存储，因此我们也很容易带着这样的理解去看待Android手机，认为机身固有的存储就是内部存储，而扩展的T卡就是外部存储。早期Android设备的内部存储确实是固定的，而外部存储确实可以向U盘一样移动。到那时后来的设备中，很多中高端手机将自己的机身存储扩展到了8G以上，他们将存储的概念分为"内部Internal"和"外部External"两部分，但是其实都在手机内部。所以不管Android手机是否有可移动的SDCard，他们总是有外部存储和内部存储。关键的是我们都可以通过相同的Api来访问可移动SDCard或者手机自带的存储。

![](/assets/tools/tools-storage-01.png)

使用Android studio打开手机目录，这里有三个文件夹需要我们注意，data，mnt和storage，data是指内部存储，mnt和storage指外部存储，下面我们来具体分析一下

# 内部存储
内部存储位于系统中很特殊的位置，如果我们想将文件存储在内部存储中，那么文件默认只能放在我们的应用可以访问到，且一个应用创建的所有文件都在和应用包名相同的目录下。也就是说，应用创建内部存储的文件，与这个应用关联起来了。当一个应用被卸载的时候，内部存储中的文件也随之被删除。从技术上来讲如果我们在创建内部存储文件是将这个文件属性设置为可读，那么其他App能够访问自己应用的数据，其实他知道你这个应用的包名，如果这个文件属性为私有的，那么即使知道包名其他应用也无法访问。内部存储控件十分有限，因而显得很重要，另外他也是系统本身和系统应用程序主要的数据存储所在地，一旦内部存储耗尽，手机也将无法使用。
在内部存储中有两个重要的目录
1. app文件夹：如果没有root，我们是无法打开该文件夹的。app文件夹内存放着我们所有安装app的APK文件，当我们调试一个app的时候，可以看到控制台输出的内容，有一项是uploading...就是上传我们的APK到这个文件夹
2. data文件夹：这个文件夹里包含一些包名，打开这些包名，我们就会看到一些文件如
    - data/data/包名/shared_prefs
    - data/data/包名/database
    - data/data/包名/files
    - data/data/包名/cache

我们在使用SharedPerference的时候，将数据90%存储到本地，其实就是存储到这个文件夹中的xml文件中，我们的app里面的数据库文件就存储在databases文件中，还有我们的普通数据存储在files中，缓存在cache文件夹中

![](/assets/tools/tools-storage-02.png)

![](/assets/tools/tools-storage-03.png)

# 外部存储
外部存储就是上面我们所说道的storage文件夹，也有可能是mnt文件夹，在storeage文件中有一个sdcard的文件夹，这个文件夹中的文件分为两类，一类是共有目录，还有一类是私有目录。其中共有目录分为9大类，比如DCIM，Download等这些系统为我们创建的文件夹，私有目录就是android这个文件夹，这个文件夹打开后面有一个data的文件夹，打开这个data文件夹里面包好了许多报名组成的文件夹

![](/assets/tools/tools-storage-04.png)

Android应用程序在运行的过程中需要向手机上保存数据，一般是把数据保存在SDCard中，大部分应用时直接在SDCard的根目录创建一个文件夹，将数据直接保存在该文件夹中，当应用被卸载的时候，这些数据还保留在SDCard，留下了垃圾数据

开发中我们一般都是操作存储空间，Google官方建议我们App的数据应该存在外部存储设备的私有目录中该App包名下，这样当用户卸载App之后，相关数据也会一并删除，如果我们直接在/storage/sdcard目录下创建一个应用的文件夹，那么当删除应用时，该文件夹是不会被删除掉的。

# 大招
说了这么多概念，其实就是想让大家都了解外部存储和内部存储的区别。那么我们怎么才能获取到自己想要的存储路径呢，如下，提供了一些基本常用的方法

|      方法调用      |      路径位置      |
|:------             |:---               |
| context.getFilesDir()   |  外部存储data/data/包名/files目录  |
| context.getCacheDir()   |   外部存储设备data/data/包名/cache目录   |
| context.getExternalStorageDirectory()   | 外部存储设备的根目录   |
| context.getExternalStoragePublicDirectory  | 外部存储设备公有目录 (Environment.DIRECTORY_DCIM)  |
| context.getExternalFilesDir()     |  外部存储私有目录 storage/sdcard/android/data/包名/files  |
| context.getExternalCacheDir()   | 外部存储私有目录 storage/sdcard/android/data/包名/cache   |

通过Context.getExternalFilesDir()方法可以获取到SDCard/Android/data包名/files目录，一般存放一些长时间保存的数据

通过Context.getExternalCacheDir()方法可以获取SDCard/Android/data/包名/chache目录，一般存放一些历史缓存数据

使用上面的方法，当应用被卸载的时候，SDCard/Android/data/包名/目录 下所有的文件都会被删除，不会留下垃圾信息。这两个部分的内容分别对应设置->应用->应用详情里面的  "清除数据"和"清除缓存"