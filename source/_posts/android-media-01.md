---
title: android 音视频开发(一)
date: 2019-08-02 19:31:10
tags:
  - android
  - 音视频
---

随着5G时代的即将到来，以后的手机软件中视频和语音类的软件肯定会迎来一次崛起的机会，然而自己在这方面还没有真正的涉足过，所以在网上找了一些资料，打算好好的学习一下。不过自己真的没有将这些应用于实际开发中，所以应该会有很多不完善和不合理的地方，所以希望能够借着这个机会，查漏补缺，将自己的技术多打磨一下。
<!--more-->
这是音视频开发的第一篇博客，所以具体的技术方面的内容不会涉及太多，更多的还是统计一下需要使用到的技术，算是给自己定下目标，以后就按照这个步骤学下去，希望有所收获吧。

## android 音视频开发任务列表

1. 在android平台绘制一张图片，使用至少三种不同的API，ImageView,SurfaceView,和自定义view
2. 在Android平台中使用AudioRecord和AndroidTrack API完成音频PCM数据的采集和播放，并实现读写音频wav文件
3. 在Android平台使用Camera API进行视频采集，分别使用SurfaceView，TextureView来预览Camera数据，获取到NV21数据回调
4. 学习Android平台的MediaExtractor和MediaMuxer API，知道如何解析和封装mp4文件
5. 学习Android平台的OpenGL ES API，了解OpenGL开发的基本流程，使用OpenGL绘制三角形
6. 学习Android平台的OpenGL ES API，学习纹理绘制，能够使用OpenGL显示一张图片
7. 学习MediaCodec API完成音频ACC硬编，硬解
8. 学习MediaCodec API，完成视频H.264的硬编，硬解
9. 串联整个音视频录制流程，完成音视频的采集，编码，封装成mp4输出
10. 串联整个音视频播放流程，完成mp4的解析，音视频的解码，播放和渲染
11. 进一步学习OpenGL，了解如何实现视频的裁剪，旋转，水印，滤镜，并学习OpenGL的高级特性，如VBO，VAO，FBO
12. 学习Android的图形图像框架，能够使用GLSurfaceView绘制Camera预览画面
13. 深入学习音视频的网络协议，如rtms，hls，以及封装格式：flv，mp4
14. 深入学习一些音频领域的开源框架，如webrtc，ffmpeg，ijkplayer，librtmp等
15. 将ffmpeg移植到Android平台，结合上面积累的经验，编写一款简易的音视频播放器
16. 将x264移植到Android平台，结合上面的经验，完成视频数据的H264软编功能
17. 将librtmp库移植到Android平台，结合上面的经验，完成Android RTMP推流功能
18. 结合上面的经验，做一款短视频App，完成如断点拍摄，添加水印，本地转码，视频剪辑，视频拼接，MV特效等功能


可供参考的资料
[雷霄骅的专栏](http://blog.csdn.net/leixiaohua1020)
[Android音频开发](http://ticktick.blog.51cto.com/823160/d-15)
[FFMPEG Tips](http://ticktick.blog.51cto.com/823160/d-17)
[Learn OpenGL 中文](https://learnopengl-cn.readthedocs.io/zh/latest/)
[Android Graphic 架构](https://source.android.com/devices/graphics/)
[灰色飘零](https://www.cnblogs.com/renhui/)

就按照这个步骤走下去，就一定会有所收获

