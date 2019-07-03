---
title: Flutter (一)  Flutter基础介绍
date: 2019-07-02 20:31:11
tags:
  - flutter
---

# 简介
Flutter是谷歌的移动UI框架，可以快速在iOS和Android上构建高质量的原生用户界面。 Flutter可以与现有的代码一起工作。在全世界，Flutter正在被越来越多的开发者和组织使用，并且Flutter是完全免费、开源的 
[Flutter Dev](https://flutter.dev)
<!--more-->

# 环境配置
我使用的是Mac系统，其实不管是Windows系统还是Linux系统，配置环境都是比较简单的。可以在网上找到相关的教程，这里我就不再赘述。如果你也是Mac系统，可以参考博客[Flutter的环境配置](https://www.jianshu.com/p/b50a92afbef1)

当然还有另外一种方式，就是首先你的电脑里要安装Android studio，然后再插件中搜索Flutter，安装完成之后，重启Android Studio，你就会在主界面中发现有这么一个创建Flutter的选项，如图

![](/assets/flutter_01/flutter01.png)

选中这个选项，进入到如图所示的页面

![安装FlutterSDK](/assets/flutter_01/flutter02.png)

在这里我们因为没有安装过Flutter的SDK，所以直接选中安装按钮即可，系统会自动帮我们下载好SDK，并且配置好。下载的速度可能会有点慢，当然我们可以使用VPN，这个情况就因人而异了。我使用的是第一种方法，在官网下载好SDK的压缩文件，自己通过解压，配置环境变量，就可以使用了。

安装并且配置好环境之后，在终端中输入: flutter -h 如果现实如下图所示的内容，就表示你的环境已经搭建好了

![检测FlutterSDK](/assets/flutter_01/flutter03.png)

这里的截图比较少，其实后面还有很多的内容，包括Flutter常用的一些命令，都会在这里显示出来。

紧接着我们可以检查一下当前的环境，因为Flutter不仅可以满足Android开发，还可以同时满足IOS开发，那么我们有没有将相应的开发环境配置好呢？需要让Flutter帮我们检测一下，检测的命令就是：flutter doctor

![flutter doctor](/assets/flutter_01/flutter04.png)

这里我的电脑里还有一些配置没有安装完成 有！或者 x 符号的表示内容没有安装完成。

首先运行这句：flutter doctor --android-licenses

![flutter doctor --android-licenses](/assets/flutter_01/flutter05.png)

这里面会有几个地方需要我们同意的，其实直接输入y，即可。

最后的样式就是这样的

![](/assets/flutter_01/flutter06.png)

表示对于Android部分的内容是没问题的了。

紧接着做其他的部分，可能根据每个人电脑的配置而进行修改。或者你可以直接根据提示进行修改如下图：

![需要修改的内容](/assets/flutter_01/flutter07.png)

这个过程可能会比较慢，请耐心等待，或者你可以在晚上的时候让他自己去下载，然后我们去休息。过程中如果出现什么错误，一般会给出修改的提示，然后按照提示修改即可，如果遇到一些比较难缠的问题，只能去google了。




