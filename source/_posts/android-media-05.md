---
title: android 音视频开发(五)
date: 2019-08-12 18:50:10
tags:
  - android
  - 音视频
---

学习Android平台的MediaExtractor和MediaMuxer API，知道如何解析和封装mp4文件
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

### 主要的API介绍
1. setDataSource(String path)  可以设置本地文件又可以设置为网络文件
2. getTrackCount()             得到源文件的通道数
3. getTrackFormat(int index)   获取指定(index)的通道格式
4. getSampleTime()             返回当前的时间戳
5. readSampleData(ByteBuffer byteBuf,int offset)    把指定通道中的数据偏移量读取到ByteBuffer中
6. advance()                   读取下一帧数据
7. release()                   读取结束后释放资源

```
MediaExtractor extractor = new MediaExtractor();
 extractor.setDataSource(...);
 int numTracks = extractor.getTrackCount();
 for (int i = 0; i < numTracks; ++i) {
   MediaFormat format = extractor.getTrackFormat(i);
   String mime = format.getString(MediaFormat.KEY_MIME);
   if (weAreInterestedInThisTrack) {
     extractor.selectTrack(i);
   }
 }
 ByteBuffer inputBuffer = ByteBuffer.allocate(...)
 while (extractor.readSampleData(inputBuffer, ...) >= 0) {
   int trackIndex = extractor.getSampleTrackIndex();
   long presentationTimeUs = extractor.getSampleTime();
   ...
   extractor.advance();
 }

 extractor.release();
 extractor = null;
```

# MediaMuxer API

作用：生成一个音频或者视频文件，还可以吧音频和视频混合成一个音视频文件

### 如何使用

1. 因为生成一个文件，所以构造一个MediaMuxer的时候需要传入文件的路径，和文件的格式，如：new MediaMuxer(String filePath,int format);其中格式一般为MediaMuxer.OutputFormat.MUXER_OUTPUT_MPEG_4
2. 然后添加通道，记录一个数据通道的格式(MediaExtracktor第3步)addTrack(MediaFormat format)并且会得到一个trackIndex之后用这个判断使用哪个通过到写入数据
3. start():开始合成文件
4. 每当MediaExtracktor的第5步之后，用writeSampleData(int trackIndex,ByteBuffer byteBuf,MediaCodec.BufferInfo bufferInfo):把ByteBuffer中的数据写入之前设置的文件中
5. 数据写入完成，stop():停止合成文件  release()释放资源

### 主要API介绍

1. MediaMuxer(String path, int format)：path:输出文件的名称  format:输出文件的格式；当前只支持MP4格式；
2. addTrack(MediaFormat format)：添加通道；我们更多的是使用MediaCodec.getOutpurForma()或Extractor.getTrackFormat(int index)来获取MediaFormat;也可以自己创建；
3. start()：开始合成文件
4. writeSampleData(int trackIndex, ByteBuffer byteBuf, MediaCodec.BufferInfo bufferInfo)：把ByteBuffer中的数据写入到在构造器设置的文件中；
5. stop()：停止合成文件
6. release()：释放资源

```
MediaMuxer muxer = new MediaMuxer("temp.mp4", OutputFormat.MUXER_OUTPUT_MPEG_4);
 // More often, the MediaFormat will be retrieved from MediaCodec.getOutputFormat()
 // or MediaExtractor.getTrackFormat().
 MediaFormat audioFormat = new MediaFormat(...);
 MediaFormat videoFormat = new MediaFormat(...);
 int audioTrackIndex = muxer.addTrack(audioFormat);
 int videoTrackIndex = muxer.addTrack(videoFormat);
 ByteBuffer inputBuffer = ByteBuffer.allocate(bufferSize);
 boolean finished = false;
 BufferInfo bufferInfo = new BufferInfo();

 muxer.start();
 while(!finished) {
   // getInputBuffer() will fill the inputBuffer with one frame of encoded
   // sample from either MediaCodec or MediaExtractor, set isAudioSample to
   // true when the sample is audio data, set up all the fields of bufferInfo,
   // and return true if there are no more samples.
   finished = getInputBuffer(inputBuffer, isAudioSample, bufferInfo);
   if (!finished) {
     int currentTrackIndex = isAudioSample ? audioTrackIndex : videoTrackIndex;
     muxer.writeSampleData(currentTrackIndex, inputBuffer, bufferInfo);
   }
 };
 muxer.stop();
 muxer.release();
```

# 使用场景
从MP4文件中提取视频，并生成新的视频文件

```
public class MainActivity extends AppCompatActivity {

    private static final String SDCARD_PATH = Environment.getExternalStorageDirectory().getPath();

    private MediaExtractor mMediaExtractor;
    private MediaMuxer mMediaMuxer;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        // 获取权限
        int checkWriteExternalPermission = ContextCompat.checkSelfPermission(this, Manifest.permission.WRITE_EXTERNAL_STORAGE);
        int checkReadExternalPermission = ContextCompat.checkSelfPermission(this, Manifest.permission.READ_EXTERNAL_STORAGE);if (checkWriteExternalPermission != PackageManager.PERMISSION_GRANTED ||
                checkReadExternalPermission != PackageManager.PERMISSION_GRANTED) {

            ActivityCompat.requestPermissions(this, new String[]{
                    Manifest.permission.WRITE_EXTERNAL_STORAGE,
                    Manifest.permission.READ_EXTERNAL_STORAGE}, 0);
        }

        setContentView(R.layout.activity_main);
        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    process();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }

    private boolean process() throws IOException {
        mMediaExtractor = new MediaExtractor();
        mMediaExtractor.setDataSource(SDCARD_PATH + "/ss.mp4");

        int mVideoTrackIndex = -1;
        int framerate = 0;
        for (int i = 0; i < mMediaExtractor.getTrackCount(); i++) {
            MediaFormat format = mMediaExtractor.getTrackFormat(i);
            String mime = format.getString(MediaFormat.KEY_MIME);
            if (!mime.startsWith("video/")) {
                continue;
            }
            framerate = format.getInteger(MediaFormat.KEY_FRAME_RATE);
            mMediaExtractor.selectTrack(i);
            mMediaMuxer = new MediaMuxer(SDCARD_PATH + "/ouput.mp4", MediaMuxer.OutputFormat.MUXER_OUTPUT_MPEG_4);
            mVideoTrackIndex = mMediaMuxer.addTrack(format);
            mMediaMuxer.start();
        }

        if (mMediaMuxer == null) {
            return false;
        }

        MediaCodec.BufferInfo info = new MediaCodec.BufferInfo();
        info.presentationTimeUs = 0;
        ByteBuffer buffer = ByteBuffer.allocate(500 * 1024);
        int sampleSize = 0;
        while ((sampleSize = mMediaExtractor.readSampleData(buffer, 0)) > 0) {

            info.offset = 0;
            info.size = sampleSize;
            info.flags = MediaCodec.BUFFER_FLAG_SYNC_FRAME;
            info.presentationTimeUs += 1000 * 1000 / framerate;
            mMediaMuxer.writeSampleData(mVideoTrackIndex, buffer, info);
            mMediaExtractor.advance();
        }

        mMediaExtractor.release();

        mMediaMuxer.stop();
        mMediaMuxer.release();

        return true;
    }
}
```

# 参考资料
[Android 音视频开发(五)：使用 MediaExtractor 和 MediaMuxer API 解析和封装 mp4 文件](https://www.cnblogs.com/renhui/p/7474096.html)