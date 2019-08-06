---
title: android 音视频开发(四)
date: 2019-08-06 20:00:10
tags:
  - android
  - 音视频
---

# Camera2 API 采集视频并SurfaceView、TextureView 预览

<!--more-->

从5.0开始（API Level 21），可以完全控制安卓设备相机的新api Camera2(android.hardware.Camera2)被引入了进来。在以前的Camera api(android.hardware.Camera)中，对相机的手动控制需要更改系统才能实现，而且api也不友好。不过老的Camera API在5.0上已经过时，如今Android推荐使用Camera2采集视频，借着写这篇记录的过程，熟悉和理解Camera2流程。





