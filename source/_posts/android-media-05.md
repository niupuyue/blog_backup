---
title: android 音视频开发(五)
date: 2019-08-12 18:50:10
tags:
  - android
  - 音视频
---

# 学习Android平台的MediaExtractor和MediaMuxer API，知道如何解析和封装mp4文件

<!--more-->

# MediaExtractor API

作用：可以把音视频文件的音频和视频分离，并且抽取相应的数据通道，然后进行操作

### 如何使用

1. 先要知道针对的哪个文件进行操作，所以我们需要使用setDataSource(String filePath)设置目标文件
2. 然后需要知道这个文件所有的通道数，我们可以通过getTrackCount()得到，然后通过遍历，得到需要的通道 int  pipe
3. 根据得到的通道，用getTrackFirmat(int pipe)得到这个通道的数据格式(MediaFormat mediaformat)
4. 然后把MediaExtractor对准这个通道，使用selectTrack(int pipe)读取数据
5. 用readSampleData(ByteBuffer byteBuffer,int offset)把指定通道中的数据按偏移量读取到byteBuffer中，注意，此处只是一帧的数据
6. 有了这个byteBuffer数据，之后的是直接交给MediaMuxer执行操作这一帧，之后调用adVance()获取下一帧，重复第5,6步的操作
7. 操作完成之后释放release()

# MediaMuxer API

作用：生成一个音频或者视频文件，还可以吧音频和视频混合成一个音视频文件

### 如何使用

1. 因为生成一个文件，所以构造一个MediaMuxer的时候需要传入文件的路径，和文件的格式，如：new MediaMuxer(String filePath,int format);其中格式一般为MediaMuxer.OutputFormat.MUXER_OUTPUT_MPEG_4
2. 然后添加通道，记录一个数据通道的格式(MediaExtracktor第3步)addTrack(MediaFormat format)并且会得到一个trackIndex之后用这个判断使用哪个通过到写入数据
3. start():开始合成文件
4. 每当MediaExtracktor的第5步之后，用writeSampleData(int trackIndex,ByteBuffer byteBuf,MediaCodec.BufferInfo bufferInfo):把ByteBuffer中的数据写入之前设置的文件中
5. 数据写入完成，stop():停止合成文件  release()释放资源
