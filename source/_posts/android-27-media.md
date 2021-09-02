---
title: 重拾android路(二十六) Android多媒体
date: 2018-04-09 11:27:11
tags:
  - android
  - record
  - video
---

做Android开发好几年了，但是对于Android开发中的语音播放，视频播放，语音录制，视频录制等功能，还是没有接触到过。不过最近公司因为在做IM模块的内容，虽然东西不多，但是确实也是让人费心费力。不过对于我来说最大的收获是对android开发中的语音模块有了更多的了解，虽然在这个了解过的过程，是痛苦的，是委屈的，但是只要最后能学到东西，付出一些代价我是愿意接受的。
<!--more-->
# Record
语音模块，这一部分主要是分为两个内容，录制和播放。录制的时候需要调用系统的麦克风，并且在录制之前需要创建好缓存文件，设定语音录制的缓存路径，语音录制的编码格式等。先来看一下这段代码，这是我参考阿里云旺写的语音录制模块的核心代码
```
/**
 * Desc: 语音录制模块模块
 */
public class RecorderKit {
    // 输出的默认文件
    private File audioFile;
    // 语音录制对象
    private MediaRecorder mRecorder;
    private boolean isStart = false;
    private long mDuration = 0L;
    private static int SECOND_PEROID = 1000; // 录制最小的时间
    private static int RECORDER_SIMPLE_RATE = 8000; // 录制的频率
    private static int RECORDER_SIMPLE_BIG_RATE = 67000; // 录制最大频率
    private Handler mHandler;
    private final IMRecorderKitCallback mCallback;
    private final long mMaxRecordTime;
    private final long mMinRecordTime;
    private final long mVolumnPeriodTime;
    private static String IM_RECORDER_HANDLER = "ChattingRecorder";
    // 超过最大录制时间停止录音
    private Runnable mTimeOut = new Runnable() {
        public void run() {
            stopRecord();
        }
    };

    private Runnable mPeriod = new Runnable() {
        public void run() {
            try {
                if (mCallback != null) {
                    // 录音过程中每个1秒钟发送一次录制状态
                    mCallback.onProgress((int) (System.currentTimeMillis() - mDuration) / SECOND_PEROID);
                }
                // 每个固定时间执行一次检测音量
                mHandler.postDelayed(this, mVolumnPeriodTime);
            } catch (Exception ex) {
                ex.printStackTrace();
            }
        }
    };

    public RecorderKit(IMRecorderKitCallback callback, long maxTime, long minTime, long mPeriodTime) {
        HandlerThread thread = new HandlerThread(IM_RECORDER_HANDLER);
        thread.start();
        this.mHandler = new Handler(thread.getLooper());
        this.mCallback = callback;
        this.mMaxRecordTime = maxTime;
        this.mMinRecordTime = minTime;
        this.mVolumnPeriodTime = mPeriodTime;
    }

    /**
     * 开始录制操作
     */
    public void startRecorder() {
        if (this.mHandler == null) return;
        try {
            this.mHandler.post(new Runnable() {
                public void run() {
                    audioFile = FileUtility.createFile(FileUtility.FILE_UTILITY_TYPE_TEMP, null, "amr");
                    if (audioFile == null) {
                        onError("录制语音错误");
                    } else if (!isStart) {
                        try {
                            if (mRecorder == null) {
                                mRecorder = new MediaRecorder();
                            }

                            mRecorder.setAudioSource(MediaRecorder.AudioSource.MIC);
                            mRecorder.setOutputFormat(MediaRecorder.OutputFormat.RAW_AMR);
                            mRecorder.setAudioEncoder(MediaRecorder.AudioEncoder.AMR_NB);
                            mRecorder.setAudioSamplingRate(RECORDER_SIMPLE_RATE);
                            mRecorder.setAudioEncodingBitRate(RECORDER_SIMPLE_BIG_RATE);
                            mRecorder.setOutputFile(audioFile.getAbsolutePath());
                            mRecorder.prepare();
                        } catch (IllegalStateException var6) {
                            if (mRecorder != null) {
                                mRecorder.reset();
                                mRecorder.release();
                                mRecorder = null;
                            }

                            var6.printStackTrace();
                            onError();
                            return;
                        } catch (IOException var7) {
                            if (mRecorder != null) {
                                mRecorder.reset();
                                mRecorder.release();
                                mRecorder = null;
                            }

                            var7.printStackTrace();
                            onError();
                            return;
                        } catch (RuntimeException var8) {
                            try {
                                if (mRecorder != null) {
                                    mRecorder.reset();
                                    mRecorder.release();
                                }
                            } catch (RuntimeException var4) {

                            }

                            mRecorder = null;
                            var8.printStackTrace();
                            onError();
                            return;
                        }

                        try {
                            if (mRecorder != null) {
                                mDuration = System.currentTimeMillis();
                                mRecorder.start();
                            }
                        } catch (RuntimeException var5) {
                            try {
                                if (mRecorder != null) {
                                    mRecorder.reset();
                                    mRecorder.release();
                                }
                            } catch (RuntimeException var3) {

                            }

                            mRecorder = null;
                            onError();
                            return;
                        }

                        isStart = true;
                    }
                }
            });
            this.mHandler.postDelayed(mTimeOut, mMaxRecordTime);
            this.mHandler.post(this.mPeriod);
        } catch (Exception ex) {
            ex.printStackTrace();
        }
    }

    public void stop() {
        try {
            this.mHandler.post(new Runnable() {
                public void run() {
                    stopRecord();
                }
            });
        } catch (Exception ex) {
            ex.printStackTrace();
        }
    }

    public void cancel() {
        if (!this.isStart) return;
        try {
            this.mHandler.post(new Runnable() {
                public void run() {
                    mHandler.removeCallbacks(mTimeOut);
                    mHandler.removeCallbacks(mPeriod);
                    if (isStart) {
                        if (mRecorder != null) {
                            try {
                                mRecorder.stop();
                            } catch (RuntimeException var2) {
                                var2.printStackTrace();
                            }

                            mRecorder.release();
                            mRecorder = null;
                        }

                        isStart = false;
                        deleteFile();
                    }
                }
            });
        } catch (Exception ex) {
            ex.printStackTrace();
        }
    }

    private void stopRecord() {
        if (!this.isStart) return;
        try {
            this.mHandler.removeCallbacks(this.mTimeOut);
            this.mHandler.removeCallbacks(this.mPeriod);
            if (this.mRecorder != null) {
                try {
                    this.mRecorder.stop();
                } catch (RuntimeException var3) {
                    var3.printStackTrace();
                }

                this.mRecorder.release();
                this.mRecorder = null;
            }

            this.isStart = false;
            long duration = System.currentTimeMillis() - this.mDuration;
            if (duration < this.mMinRecordTime) {
                this.onError("录制语音时间过短");
            } else if (this.audioFile == null) {
                this.onError("语音文件创建失败");
            } else if (this.mCallback != null) {
                this.mCallback.onSuccess(this.audioFile.getAbsolutePath(), (int) (duration / SECOND_PEROID));
            }
        } catch (Exception ex) {
            ex.printStackTrace();
        }

    }

    /**
     * 回收资源 会将looper清除
     */
    public void recycle() {
        if (this.mHandler == null) return;
        try {
            this.mHandler.post(new Runnable() {
                public void run() {
                    if (mHandler != null) {
                        mHandler.getLooper().quit();
                        mHandler = null;
                    }

                }
            });
        } catch (Exception ex) {
            ex.printStackTrace();
        }

    }

    /**
     * 设置音量
     *
     * @return
     */
    public int getAmplitude() {
        if (!this.isStart || this.mRecorder == null) return 0;
        try {
            return this.mRecorder.getMaxAmplitude();
        } catch (Exception ex) {
            ex.printStackTrace();
        }
        return 0;
    }

    /**
     * 删除录音文件
     */
    private void deleteFile() {
        if (this.audioFile == null) return;
        try {
            this.audioFile.delete();
        } catch (Exception ex) {
            ex.printStackTrace();
        }
    }

    private void onError() {
        try {
            deleteFile();
            if (this.mCallback != null) {
                this.mCallback.onError(0, "语音录制失败");
            }
        } catch (Exception ex) {
            ex.printStackTrace();
        }

    }

    private void onError(String info) {
        try {
            deleteFile();
            if (this.mCallback != null) {
                this.mCallback.onError(0, info);
            }
        } catch (Exception ex) {
            ex.printStackTrace();
        }
    }

}
```

内容比较多，我们先来将这段代码分隔成几个部分
1. 构造方法和一些初始化Runnable子线程
2. 语音录制，语音结束，语音取消，资源回收等操作
3. 设置音量(额外内容)

## 构造方法和初始化Runnable子线程
首先我们会发现在构造方法中，我们需要传递几个变量参数。第一个IMRecorderKitCallback接口，这个接口如下所示
```
**
 * Desc:录音状态回调
 */
public interface IMRecorderKitCallback {

    /**
     * 录音成功
     *
     * @param var1 var1 数量存在多个，其中第一个和第二个是文件缓存路径和录音时长
     */
    void onSuccess(Object... var1);

    /**
     * 语音错误
     *
     * @param errorCode 错误代码
     * @param errorMsg  错误信息
     */
    void onError(int errorCode, String errorMsg);

    /**
     * 正在录音
     *
     * @param timeCount 时间计数
     */
    void onProgress(int timeCount);

}
```
这是录音状态的回调，里面分别有三个方法，代表了三种不同的状态，当我们在录制的过程中，根据具体的操作和录音所处的状态，调用响应的方法，实现数据的相互传递。
紧接着需要传递三个long类型的数据，这三个数据分别代表了录音最大时长(我这里规定最大是60秒),最小录音时长(规定是1秒),每个多长时间返回一次音量信息，我这里是规定500毫秒。将这四个变量拿到之后，在赋值给全局变量，方便后面的调用。

其次，我们这里还有几个Runnable子线程，这些子线程主要是完成一些异步操作，防止出现意外崩溃的情况。
1. mTimeOut 超过最大录制时间之后，自动定制录制操作。为了计时使用的
2. mPeriod  录音时的音量变换，并且在这个Runnable中我们会发现每隔固定的时间我们就会自己调用自己一次，为的就是实时返回音量变化，我们通过IMRecorderKitCallback中的onProgress()方法向外界传达当前音量的变化。
这些都是比较简单的部分，暂时就说这么多。

## 语音录制的各种状态
简单来说我们的录制状态一共分别三种，录制中，录制完成，录制退出。乍一听这三个中后面两个象是一样的。其实不是，第二个录制完成，便是的是我们录制语音完成，这时候我们是需要将录制语音的缓存文件保存下来的，第三个是录制退出，表示我们想要取消录制，语音文件缓存要删除掉。那么我们其实也可以分别这么几个状态
1. 准备语音录制文件，语音录制设置
2. 开始录制语音，并且将语音写入本地缓存中
3. 实时发送语音音量信息
4. 录制完成，将缓存保存到本地存储空间
5. 取消录制，将缓存文件删除
6. 还原语音模块状态，还原到原有状态
7. 清除语音录制资源 包括读写流等

那么我们一个个的看。

### 开始录制操作
```
/**
     * 开始录制操作
     */
    public void startRecorder() {
        if (this.mHandler == null) return;
        try {
            this.mHandler.post(new Runnable() {
                public void run() {
                    audioFile = FileUtility.createFile(FileUtility.FILE_UTILITY_TYPE_TEMP, null, "amr");
                    if (audioFile == null) {
                        onError("录制语音错误");
                    } else if (!isStart) {
                        try {
                            if (mRecorder == null) {
                                mRecorder = new MediaRecorder();
                            }

                            mRecorder.setAudioSource(MediaRecorder.AudioSource.MIC);
                            mRecorder.setOutputFormat(MediaRecorder.OutputFormat.RAW_AMR);
                            mRecorder.setAudioEncoder(MediaRecorder.AudioEncoder.AMR_NB);
                            mRecorder.setAudioSamplingRate(RECORDER_SIMPLE_RATE);
                            mRecorder.setAudioEncodingBitRate(RECORDER_SIMPLE_BIG_RATE);
                            mRecorder.setOutputFile(audioFile.getAbsolutePath());
                            mRecorder.prepare();
                        } catch (IllegalStateException var6) {
                            if (mRecorder != null) {
                                mRecorder.reset();
                                mRecorder.release();
                                mRecorder = null;
                            }

                            var6.printStackTrace();
                            onError();
                            return;
                        } catch (IOException var7) {
                            if (mRecorder != null) {
                                mRecorder.reset();
                                mRecorder.release();
                                mRecorder = null;
                            }

                            var7.printStackTrace();
                            onError();
                            return;
                        } catch (RuntimeException var8) {
                            try {
                                if (mRecorder != null) {
                                    mRecorder.reset();
                                    mRecorder.release();
                                }
                            } catch (RuntimeException var4) {

                            }

                            mRecorder = null;
                            var8.printStackTrace();
                            onError();
                            return;
                        }

                        try {
                            if (mRecorder != null) {
                                mDuration = System.currentTimeMillis();
                                mRecorder.start();
                            }
                        } catch (RuntimeException var5) {
                            try {
                                if (mRecorder != null) {
                                    mRecorder.reset();
                                    mRecorder.release();
                                }
                            } catch (RuntimeException var3) {

                            }

                            mRecorder = null;
                            onError();
                            return;
                        }

                        isStart = true;
                    }
                }
            });
            this.mHandler.postDelayed(mTimeOut, mMaxRecordTime);
            this.mHandler.post(this.mPeriod);
        } catch (Exception ex) {
            ex.printStackTrace();
        }
    }
```

在这段代码中我们发现，我们是将语音录制的操作放在一个handler中执行的，别问为什么，问就是暴露了。那么在一开始的时候，我们先执行了这么几步操作
```
audioFile = FileUtility.createFile(FileUtility.FILE_UTILITY_TYPE_TEMP, null, "amr");
```
这个就是创建一个缓存文件到本地，并且我将缓存文件的文件类型设置为amr，当然也可以设置成mp3等格式，看个人喜好。
```
mRecorder.setAudioSource(MediaRecorder.AudioSource.MIC); // 设置语音录制的来源，我们规定是麦克风
mRecorder.setOutputFormat(MediaRecorder.OutputFormat.RAW_AMR);// 设置语音录制的输出格式
mRecorder.setAudioEncoder(MediaRecorder.AudioEncoder.AMR_NB); // 设置语音录制的输出编码格式
mRecorder.setAudioSamplingRate(RECORDER_SIMPLE_RATE); // 设置语音的默认码率
mRecorder.setAudioEncodingBitRate(RECORDER_SIMPLE_BIG_RATE); // 设置语音的最大码率
mRecorder.setOutputFile(audioFile.getAbsolutePath()); // 设置语音输出的缓存路径
mRecorder.prepare(); // 录音准备完成
```
这里面其实没什么好说的，只要你是录制语音，都要有这些设置，唯一不同的可能就是要设置的输出格式，这个我们是可以和后台的同事沟通的。
值得注意的地方时，这里我们监听了好几个异常处理，只要是发生了异常，我们都必须mRecorder重置，并且释放资源，如果长时间不释放资源的话，会造成内存泄漏。并且在这里我们需要调用onError方法，为的就是通知页面当前录制语音发生错误。
```
                        try {
                            if (mRecorder != null) {
                                mDuration = System.currentTimeMillis();
                                mRecorder.start();
                            }
                        } catch (RuntimeException var5) {
                            try {
                                if (mRecorder != null) {
                                    mRecorder.reset();
                                    mRecorder.release();
                                }
                            } catch (RuntimeException var3) {

                            }

                            mRecorder = null;
                            onError();
                            return;
                        }

                        isStart = true;
```
这段代码主要是开始录制语音，并且需要记录一下当前开始录制语音的时间，因为如果录制的语音时间太长或者太短我们都要执行相应的操作。
```
this.mHandler.postDelayed(mTimeOut, mMaxRecordTime);
his.mHandler.post(this.mPeriod);
```
这里我们运行了一个postDelayed方法，规定在超出最大时间限制之后，需要执行mTimeOut，为我们关闭语音录制，并且在关闭的时候，需要保存语音录制缓存文件。
这里我们还运行了另外一个post方法，这个post方法主要是发送语音录制音量等操作。

### 停止录音
```
    private void stopRecord() {
        if (!this.isStart) return;
        try {
            this.mHandler.removeCallbacks(this.mTimeOut);
            this.mHandler.removeCallbacks(this.mPeriod);
            if (this.mRecorder != null) {
                try {
                    this.mRecorder.stop();
                } catch (RuntimeException var3) {
                    var3.printStackTrace();
                }

                this.mRecorder.release();
                this.mRecorder = null;
            }

            this.isStart = false;
            long duration = System.currentTimeMillis() - this.mDuration;
            if (duration < this.mMinRecordTime) {
                this.onError("录制语音时间过短");
            } else if (this.audioFile == null) {
                this.onError("语音文件创建失败");
            } else if (this.mCallback != null) {
                this.mCallback.onSuccess(this.audioFile.getAbsolutePath(), (int) (duration / SECOND_PEROID));
            }
        } catch (Exception ex) {
            ex.printStackTrace();
        }

    }

```
在这个方法中我们需要根据不同的语音录制结果，调用不同的回调方法，通知页面执行相应的操作。需要注意的是一定要将两个Runnable子线程从handler中移除，否则也会造成内存泄漏。我们判断语音录制结果的最主要的依据就是语音录制的时间。这里就不再一一的描述了。

### 取消录音
```
    public void cancel() {
        if (!this.isStart) return;
        try {
            this.mHandler.post(new Runnable() {
                public void run() {
                    mHandler.removeCallbacks(mTimeOut);
                    mHandler.removeCallbacks(mPeriod);
                    if (isStart) {
                        if (mRecorder != null) {
                            try {
                                mRecorder.stop();
                            } catch (RuntimeException var2) {
                                var2.printStackTrace();
                            }

                            mRecorder.release();
                            mRecorder = null;
                        }

                        isStart = false;
                        deleteFile();
                    }
                }
            });
        } catch (Exception ex) {
            ex.printStackTrace();
        }
    }
```
这段代码相对而言比较简单，这里就不再一一赘述，唯一要注意的是，取消录音，需要将缓存文件删除。

### 回收资源
直接看代码，没什么好说的。
```
    public void recycle() {
        if (this.mHandler == null) return;
        try {
            this.mHandler.post(new Runnable() {
                public void run() {
                    if (mHandler != null) {
                        mHandler.getLooper().quit();
                        mHandler = null;
                    }

                }
            });
        } catch (Exception ex) {
            ex.printStackTrace();
        }

    }

```

## 设置音量
这里主要还是使用了android为我们提供的语音录制模块中的方法。
```
    /**
     * 设置音量
     *
     * @return
     */
    public int getAmplitude() {
        if (!this.isStart || this.mRecorder == null) return 0;
        try {
            return this.mRecorder.getMaxAmplitude();
        } catch (Exception ex) {
            ex.printStackTrace();
        }
        return 0;
    }

```

这样语音的录制基本操作就完成了，其实我们还是依赖于android提供的语音模块，只不过我们通过一些方法将他们给封装起来了，与其说是语音录制模块，倒不如说是语音录制管理模块。使用的时候也非常简单，有兴趣的可以去看一下代码[例子](https://github.com/xiaoniudadi/IYingLibrary/tree/master/mylibrary/src/main/java/com/niupuyue/mylibrary/widgets/chatkeyboard)

2019年06月29日00:16:13
在开发的过程中，还遇到了另外一个情况，就是IOS和Android发送语音时，由于录制的格式不同导致语音无法播放的问题。具体的就是IOS录制语音默认使用的是CAF的格式，而Android录制语音时使用的PCM格式。遇到这样的问题，最好的解决办法就是两端统一使用PCM进行语音的录制工作，因为Android端无法解析播放CAF的格式，必须通过格式转换才能完成。另外一种解决办法就是由后台统一解码，全部按照PCM格式存储。还有一个方式就是Android端对发送来的语音先进行格式判断，如果编码是CAF，则需要转码。这个转码的功能我还没有实现。三种方法，第一种在播放语音的时候使用AudioTrack的方式播放字节流。第二种通过Android系统的JNI库实现编码格式的转化。最后一个就是利用FFmPeg。这个框架应该说是非常经典的框架了，我感觉如果能够把这个框架吃透，android技术会再上一个新台阶。


