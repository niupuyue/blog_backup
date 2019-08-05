---
title: android 音视频开发(三)
date: 2019-08-02 21:51:10
tags:
  - android
  - 音视频
---

# 实现音频的采集和播放

<!--more-->

实现语音的采集和播放有三种方式，这里我会把三种方式都分别实现一下。这三种方式分别是AudioRecord，MediaRecord，AudioTrack。
其中MediaRecord是基于文件录音，系统集成，提供了大量的方法，集成了录音，编码，压缩等，支持少量的音频格式文件，操作起来比较简单。而AudioRecord基于字节流录音，Audiotrack更加贴近底层，使用起来更加灵活，也能够实现更多的功能。
先来看一下他们的优缺点吧

#### AudioRecord
优点：可以实现语音的实时处理，进行边录边播，对音频的实时处理AudioTrack更加贴近底层
缺点：输出的是PCM语音数据，如果保存成音频文件是不能被播放器播放的。需要使用AudioTrack进行播放。API不完善，常见的如暂停功能没有实现

#### MediaRecord
优点：系统封装完整，直接调用即可，操作简单，录制的音频文件可以使用系统自带的播放器播放
缺点：无法实现实时处理音频，输出音频格式少，录制的音频文件是经过压缩的，需要设置编码器

在实现进一步操作之前，有几个专有名词需要了解一下

#### 采样率
采样率：采样率即采样频率，指每秒钟取得声音样本的次数，采样频率越高，能表现的频率范围就越大，音质就会越好，声音的还原度也更真实，但此同时带来的弊端是占有的内存资源也会越大。因为人耳的分辨率有限，并不是频率越高越好，44KHz已相当于CD音质了，目前的常用采样频率都不超过48KHz。

#### 声道
声道：这个好理解，生活中也经常听到单声道、双声道等，在Android系统中，可以通过设置音频的录制的声道 CHANNEL_IN_STEREO 为双声道，CHANNEL_CONFIGURATION_MONO 为单声道，双声道音质更加，但同样伴随着内存资源消耗更大的弊端。

#### 采样位深
采样位深：位深度也叫采样位深，音频的位深度决定动态范围，它是用来衡量声音波动变化的一个参数，也可以说是声卡的分辨率。它的数值越大，分辨率也就越高，所发出声音的能力越强。在计算机中采样位数一般有8位和16位之分，即分成2的8次方和2的16次方之分，PCM 16位每个样本，保证设备支持。PCM 8位每个样本，不一定能得到设备支持。

# AudioRecord

## 构造函数
我们先来看构造函数
```
public AudioRecord(int audioSource, int sampleRateInHz, int channelConfig, int audioFormat,
            int bufferSizeInBytes)
```
1. audioSource：录音源，指定声音是从哪里录制的，官网文档参考戳此
2. sampleRateInHz：采样率
3. channelConfig：声道数
4. audioFormat：采样位深
5. bufferSizeInBytes：最小缓冲大小，可以通过 getMinBufferSize 获取。

> 存储量= 采样率 * 采样时间 * 采样位深 / 8 * 声道数（Bytes）。以采样率为44.1kHZ、采样位深为16位、双声道计算，一分钟消耗的内存为10.335M。




# 参考博客
![AudioRecord 录音详解](https://blog.csdn.net/pangpang123654/article/details/82657795)

