---
title: android 音视频开发(二)
date: 2019-08-02 21:51:10
tags:
  - android
  - 音视频
---

绘制一张图片，使用至少三种不同的方式
<!--more-->
### ImageView

这个比较简单，就是在布局文件中设置一个ImageView的标签，然后通过属性src或者background来设置具体的图片资源，然后这样子就可以在view中显示该图片了。
只不过这里有两种方式去实现我们想要的操作，一种是在xml文件中设置ImageView标签，另外一种是在java代码中动态创建一个ImageView。这两种方式相比较而言都比较简单。具体实现就不一一介绍了。

### 自定义View
自定义View中有三个重要的生命周期：
1. onLayout()布局
2. onMeasure()测量
3. onDraw()绘制

因为我们只要显示图片，所以我们不需要执行太多的操作，所以我们只需要onDraw方法即可

```
private void initView(){
    mPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
    mBitmap = BitmapFactory.decodeResourcw(getContext().getResource(),R.mipmap.images);
}

@Override
protected void onDraw(Canvas canvas){
    super.onDraw(canvas);
    Rect srcRect = new Rect(0,0,mBitmap.getHeight(),mBitmap.getWidth());
    Rect destRect = getBitmapRect(mBitmap);// 获取调整后bitmap的显示位置
    canvas.drawBitmap(mBitmap,srcRect,destRect,mPaint);
}
```

### SurfaceView 显示图片

SurfaceView和普通view的区别是：
- surfaceview不需要在UI线程中绘制，可以在子线程中绘制
- surfaceview提供了双缓冲机制，绘制效率高
- surfaceview是创建一个位于应用窗口之上的窗口，所以无法应用动画，变换，缩放，无法折叠

SurfaceView需要实现SurfaceHolder.Callback接口，这里包括了三个生命周期

```
surfaceCreated(SurfaceHolder holder);
surfaceChanged(SurfaceHolder holder,int format,int height,int width);
surfaceDestoryed(SurfaceHolder holder);
```
具体实现代码：
```
public class ImageSurfaceActivity extends AppCompatActivity implements SurfaceHolder.Callback {
    private SurfaceView mSurfaceView;
    private ExecutorService mThread;
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        mSurfaceView = new SurfaceView(this);
        setContentView(mSurfaceView);
        initSurfaceView();
    }

    private void initSurfaceView() {
        // 创建一个只有一个线程的线程池，其实用Thread也可以
        mThread = new ThreadPoolExecutor(1, 1, 2000L, TimeUnit.MILLISECONDS, 
                  new LinkedBlockingDeque<Runnable>());
        // 添加SurfaceHolder.callback，在回调中可以绘制
        mSurfaceView.getHolder().addCallback(this);
    }

    @Override
    public void surfaceCreated(SurfaceHolder surfaceHolder) {
        // 执行绘制
        mThread.execute(new DrawRunnable());
    }

    @Override
    public void surfaceChanged(SurfaceHolder surfaceHolder, int i, int i1, int i2) {

    }

    @Override
    public void surfaceDestroyed(SurfaceHolder surfaceHolder) {
        if (!mThread.isShutdown()) {
            mThread.shutdown();
        }
    }

    private class DrawRunnable implements Runnable {
        @Override
        public void run() {
            Bitmap bimap = BitmapFactory.decodeResource(ImageSurfaceActivity.this.getResources(), 
                           R.mipmap.images);
            SurfaceHolder surfaceHolder = mSurfaceView.getHolder();
            Canvas canvas = surfaceHolder.lockCanvas(); // 获取画布
            Paint paint = new Paint();
            Rect srcRect = new Rect(0, 0, bimap.getHeight(), bimap.getWidth());
            Rect destRect = getBitmapRect(bimap);
            canvas.drawBitmap(bimap, srcRect, destRect, paint);
            surfaceHolder.unlockCanvasAndPost(canvas);
        }
    }

    /**
     * 图片的尺寸和屏幕的尺寸不一样，需要把图片调整居中
     **/
    private Rect getBitmapRect(Bitmap bimap) {
        int bimapHeight = bimap.getHeight();
        int bimapWidth = bimap.getWidth();
        int viewWidth = mSurfaceView.getWidth();
        int viewHeight = mSurfaceView.getHeight();
        float bimapRatio = (float) bimapWidth / (float) bimapHeight; // 宽高比
        float screenRatio = (float) viewWidth / (float) viewHeight;
        int factWidth;
        int factHeight;
        int x1, y1, x2, y2;
        if (bimapRatio > screenRatio) {
            factWidth = viewWidth;
            factHeight = (int)(factWidth / bimapRatio);
            x1 = 0;
            y1 = (viewHeight - factHeight) / 2;
        } else if (bimapRatio < screenRatio) {
            factHeight = viewHeight;
            factWidth = (int)(factHeight * bimapRatio);
            x1 = (viewWidth - factWidth) / 2;
            y1 = 0;
        } else {
            factWidth = bimapWidth;
            factHeight = bimapHeight;
            x1 = 0;
            y1 = 0;
        }
        x2 = x1 + factWidth;
        y2 = y1 + factHeight;
        return new Rect(x1, y1, x2, y2);
    }
}
```

[demo代码](https://github.com/xiaoniudadi/android-media-demo/tree/master/01-android-media-create-image)

### 参考资料

[Android ImageView、TextureView、自定义View显示图片](https://blog.csdn.net/qq_15893929/article/details/81411584)