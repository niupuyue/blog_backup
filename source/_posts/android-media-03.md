---
title: android 音视频开发(三)
date: 2019-08-04 21:51:10
tags:
  - android
  - 音视频
---

实现音频的采集和播放
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

完整代码

```
/**
 * 用于实现录音、暂停、继续、停止、播放
 * 最近看了下pcm和wav，内容真多，要是有一些参数不理解的，可以查阅资料
 * PCM BufferSize = 采样率 * 采样时间 * 采样位深 / 8 * 通道数（Bytes）
 */
public class AudioRecorder {
    private static AudioRecorder audioRecorder;
    //音频输入-麦克风
    private final static int AUDIO_INPUT = MediaRecorder.AudioSource.MIC;
    /**
     * 采样率即采样频率，采样频率越高，能表现的频率范围就越大
     * 设置音频采样率，44100是目前的标准，但是某些设备仍然支持22050，16000，11025
     */
    private final static int AUDIO_SAMPLE_RATE = 16000;
    // 设置音频的录制的声道CHANNEL_IN_STEREO为双声道，CHANNEL_CONFIGURATION_MONO为单声道
    private final static int AUDIO_CHANNEL = AudioFormat.CHANNEL_IN_MONO;

    /**
     * 位深度也叫采样位深，音频的位深度决定动态范围
     * 音频数据格式:PCM 16位每个样本。保证设备支持。PCM 8位每个样本。不一定能得到设备支持。
     */
    private final static int AUDIO_ENCODING = AudioFormat.ENCODING_PCM_16BIT;
    // 缓冲区字节大小
    private int bufferSizeInBytes = 0;

    //录音对象
    private AudioRecord audioRecord;

    /**
     * 播放声音
     * 一些必要的参数，需要和AudioRecord一一对应，否则声音会出错
     */
    private AudioTrack audioTrack;

    //录音状态,默认未开始
    private AudioStatus status = AudioStatus.STATUS_NO_READY;

    //文件名
    private String fileName;

    //录音文件集合
    private List<String> filesName = new ArrayList<>();

    //用来回调，转码后的文件绝对路径
    private static IAudioCallback iAudioCallback;
    /**
     * 创建带有缓存的线程池
     * 当执行第二个任务时第一个任务已经完成，会复用执行第一个任务的线程，而不用每次新建线程。
     * 如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程。
     * 一开始选择错误，选用newSingleThreadExecutor，导致停止后在录制，出现一堆问题
     */
    private ExecutorService cachedThreadPool = Executors.newCachedThreadPool();

    /**
     * 重置，删除所有的pcm文件
     */
    private boolean isReset = false;

    private AudioRecorder() {
    }

    public void setReset() {
        isReset = true;
    }

    /**
     * 单例，双重检验
     *
     * @param iAudio 用于合成后回调
     * @return
     */
    public static AudioRecorder getInstance(IAudioCallback iAudio) {
        if (audioRecorder == null) {
            synchronized (AudioRecord.class) {
                if (audioRecorder == null) {
                    audioRecorder = new AudioRecorder();
                    iAudioCallback = iAudio;
                }
            }
        }
        return audioRecorder;
    }

    /**
     * 创建默认的录音对象
     *
     * @param fileName 文件名
     */
    @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
    public void createDefaultAudio(String fileName) {
        // 获得缓冲区字节大小
        bufferSizeInBytes = AudioRecord.getMinBufferSize(AUDIO_SAMPLE_RATE, AUDIO_CHANNEL, AUDIO_ENCODING);
        audioRecord = new AudioRecord(AUDIO_INPUT, AUDIO_SAMPLE_RATE, AUDIO_CHANNEL, AUDIO_ENCODING, bufferSizeInBytes);
        this.fileName = fileName;
        status = AudioStatus.STATUS_READY;

        AudioAttributes audioAttributes = new AudioAttributes.Builder().setUsage(AudioAttributes.USAGE_MEDIA)
                .setContentType(AudioAttributes.CONTENT_TYPE_MUSIC).build();

        AudioFormat audioFormat = new AudioFormat.Builder().setSampleRate(AUDIO_SAMPLE_RATE)
                .setEncoding(AUDIO_ENCODING).setChannelMask(AudioFormat.CHANNEL_OUT_MONO).build();

        audioTrack = new AudioTrack(audioAttributes, audioFormat, bufferSizeInBytes,
                AudioTrack.MODE_STREAM, AudioManager.AUDIO_SESSION_ID_GENERATE);
    }

    /**
     * 开始录音
     */
    public void startRecord() {
        if (status == AudioStatus.STATUS_NO_READY || TextUtils.isEmpty(fileName)) {
            throw new IllegalStateException("请检查录音权限");
        }
        if (status == AudioStatus.STATUS_START) {
            throw new IllegalStateException("正在录音");
        }
        audioRecord.startRecording();
        cachedThreadPool.execute(new Runnable() {
            @Override
            public void run() {
                writeDataTOFile();
            }
        });
    }

    /**
     * 暂停录音
     */
    public void pauseRecord() {
        if (status != AudioStatus.STATUS_START) {
            throw new IllegalStateException("没有在录音");
        } else {
            audioRecord.stop();
            status = AudioStatus.STATUS_PAUSE;
        }
    }

    /**
     * 停止录音
     */
    public void stopRecord() {
        if (status == AudioStatus.STATUS_NO_READY || status == AudioStatus.STATUS_READY) {
            throw new IllegalStateException("录音尚未开始");
        } else {
            audioRecord.stop();
            status = AudioStatus.STATUS_STOP;
            release();
        }
    }

    /**
     * 释放资源
     */
    public void release() {
        //假如有暂停录音
        try {
            if (filesName.size() > 0) {
                List<String> filePaths = new ArrayList<>();
                for (String fileName : filesName) {
                    filePaths.add(FileUtils.getPcmFileAbsolutePath(fileName));
                }
                //清除
                filesName.clear();
                if (isReset) {
                    isReset = false;
                    FileUtils.clearFiles(filePaths);
                } else {
                    //将多个pcm文件转化为wav文件
                    pcmFilesToWavFile(filePaths);
                }
            }
        } catch (IllegalStateException e) {
            throw new IllegalStateException(e.getMessage());
        }

        if (audioRecord != null) {
            audioRecord.release();
            audioRecord = null;
        }
        status = AudioStatus.STATUS_NO_READY;
    }

    /**
     * 播放合成后的wav文件
     *
     * @param filePath 文件的绝对路径
     */
    public void play(final String filePath) {
        audioTrack.play();

        cachedThreadPool.execute(new Runnable() {
            @Override
            public void run() {
                File file = new File(filePath);
                FileInputStream fis = null;
                try {
                    fis = new FileInputStream(file);
                } catch (FileNotFoundException e) {
                    e.printStackTrace();
                }
                byte[] buffer = new byte[bufferSizeInBytes];
                while (fis != null) {
                    try {
                        int readCount = fis.read(buffer);
                        if (readCount == AudioTrack.ERROR_INVALID_OPERATION || readCount == AudioTrack.ERROR_BAD_VALUE) {
                            continue;
                        }
                        if (readCount != 0 && readCount != -1) {
                            audioTrack.write(buffer, 0, readCount);
                        }
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
            }
        });
    }

    /**
     * 释放audioTrack
     */
    public void releaseAudioTrack(){
        if (audioTrack == null) {
            return;
        }
        if (audioTrack.getPlayState() != AudioTrack.PLAYSTATE_STOPPED) {
            audioTrack.stop();
        }
        audioTrack.release();
        audioTrack = null;
    }

    /**
     * 将音频信息写入文件
     */
    private void writeDataTOFile() {
        // new一个byte数组用来存一些字节数据，大小为缓冲区大小
        byte[] audioData = new byte[bufferSizeInBytes];
        FileOutputStream fos = null;
        int readSize = 0;
        try {
            String currentFileName = fileName;
            if (status == AudioStatus.STATUS_PAUSE) {
                //假如是暂停录音 将文件名后面加个数字,防止重名文件内容被覆盖
                currentFileName += filesName.size();
            }
            filesName.add(currentFileName);
            File file = new File(FileUtils.getPcmFileAbsolutePath(currentFileName));
            if (file.exists()) {
                file.delete();
            }
            // 建立一个可存取字节的文件
            fos = new FileOutputStream(file);
        } catch (IllegalStateException e) {
            e.printStackTrace();
            throw new IllegalStateException(e.getMessage());
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        }
        //将录音状态设置成正在录音状态
        status = AudioStatus.STATUS_START;
        while (status == AudioStatus.STATUS_START) {
            readSize = audioRecord.read(audioData, 0, bufferSizeInBytes);
            if (AudioRecord.ERROR_INVALID_OPERATION != readSize && fos != null) {
                try {
                    fos.write(audioData);
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
        try {
            if (fos != null) {
                fos.close();// 关闭写入流
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    /**
     * 将pcm合并成wav
     *
     * @param filePaths pcm文件的绝对路径
     */
    private void pcmFilesToWavFile(final List<String> filePaths) {
        cachedThreadPool.execute(new Runnable() {
            @Override
            public void run() {
                String filePath = FileUtils.getWavFileAbsolutePath(fileName);
                if (PcmToWav.mergePCMFilesToWAVFile(filePaths, filePath)) {
                    //合成后回调
                    if (iAudioCallback != null) {
                        iAudioCallback.showPlay(filePath);
                    }
                } else {
                    throw new IllegalStateException("合成失败");
                }
                fileName = null;
            }
        });
    }

    /**
     * 获取录音对象的状态
     */
    public AudioStatus getStatus() {
        return status;
    }
}
```

# MediaRecord

完整代码

```
/**
 * Coder: niupuyue
 * Date: 2019/8/6
 * Time: 16:29
 * Desc: 用于实现录音，暂停，继续，停止，播放
 * 统一将语音录制成wav格式
 * Version:
 */
public class IMediaRecorder {

    // 默认采样率
    private static final int RECORDER_SIMPLE_RATE = 8000;
    // 最大采样率
    private static final int RECORDER_SIMPLE_BIG_RATE = 67000;

    private static IMediaRecorder mIMediaRecorder;

    public static IMediaRecorder getInstance() {
        if (mIMediaRecorder == null) {
            synchronized (IMediaRecorder.class) {
                if (mIMediaRecorder == null) {
                    mIMediaRecorder = new IMediaRecorder();
                }
            }
        }
        return mIMediaRecorder;
    }

    // 默认输出文件
    private File audioFile;
    // 语音录制系统封装好的对象
    private MediaRecorder mediaRecorder;
    // 语音播放系统封装好的对象
    private MediaPlayer mediaPlayer;
    // 语音播放管理对象
    private AudioManager audioManager;
    // 声明一个带有缓存的线程池
    private ExecutorService thread = Executors.newCachedThreadPool();


    /**
     * 开始语音录制
     * 语音的录制放在子线程中
     */
    public void startRecorder() {
        thread.execute(new Runnable() {
            @Override
            public void run() {
                audioFile = FileUtils.createMediaRecordCacheFile(new SimpleDateFormat("yyyyMMddhhmmss", Locale.CHINA).format(new Date()));
                try {
                    if (mediaRecorder == null) {
                        mediaRecorder = new MediaRecorder();
                    }
                    mediaRecorder.setAudioSource(MediaRecorder.AudioSource.MIC);
                    mediaRecorder.setOutputFormat(MediaRecorder.OutputFormat.RAW_AMR);
                    mediaRecorder.setAudioEncoder(MediaRecorder.AudioEncoder.AMR_NB);
                    mediaRecorder.setAudioSamplingRate(RECORDER_SIMPLE_RATE);
                    mediaRecorder.setAudioEncodingBitRate(RECORDER_SIMPLE_BIG_RATE);
                    mediaRecorder.setOutputFile(audioFile.getAbsolutePath());
                    mediaRecorder.prepare();

                    mediaRecorder.start();
                } catch (Exception ex) {
                    if (mediaRecorder != null) {
                        mediaRecorder.reset();
                        mediaRecorder.release();
                        mediaRecorder = null;
                    }
                    ex.printStackTrace();
                    return;
                }
            }
        });
    }

    /**
     * 停止录制语音
     */
    public void stopRecorder() {
        if (mediaRecorder != null) {
            try {
                mediaRecorder.stop();
            } catch (Exception ex) {
                ex.printStackTrace();
            }
            mediaRecorder.release();
            mediaRecorder = null;
        }
    }

    /**
     * 播放语音
     */
    public void playRecorder() {
        if (audioFile != null && audioFile.exists()) {
            if (mediaPlayer == null) {
                mediaPlayer = new MediaPlayer();
            }
            // 语音播放完成回调
            mediaPlayer.setOnCompletionListener(new MediaPlayer.OnCompletionListener() {
                @Override
                public void onCompletion(MediaPlayer mediaPlayer) {
                    Log.e("npl", "语音播放完成");
                }
            });
            // 设置语音播放失败监听回调
            mediaPlayer.setOnErrorListener(new MediaPlayer.OnErrorListener() {
                @Override
                public boolean onError(MediaPlayer mediaPlayer, int what, int extra) {
                    if (what == MediaPlayer.MEDIA_ERROR_UNKNOWN) {
                        Log.e("NPL", "语音播放失败");
                    } else {
                        Log.e("NPL", "取消语音播放");
                    }
                    return false;
                }
            });
            if (!mediaPlayer.isPlaying()) {
                try {
                    mediaPlayer.reset();
                    mediaPlayer.setDataSource(audioFile.getAbsolutePath());
                    // 异步操作播放语音
                    mediaPlayer.prepareAsync();
                    // 语音播放准备完毕监听回调
                    mediaPlayer.setOnPreparedListener(new MediaPlayer.OnPreparedListener() {
                        @Override
                        public void onPrepared(MediaPlayer mediaPlayer) {
                            // 开始播放
                            mediaPlayer.start();
                        }
                    });
                } catch (Exception ex) {
                    ex.printStackTrace();
                }
            }
        }
    }

}
```

[demo代码](https://github.com/xiaoniudadi/android-media-demo/tree/master/02-android-media-audio-collection)

# 参考博客
[AudioRecord 录音详解](https://blog.csdn.net/pangpang123654/article/details/82657795)

