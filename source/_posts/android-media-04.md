---
title: android 音视频开发(四)
date: 2019-08-06 20:00:10
tags:
  - android
  - 音视频
---

Camera2 API 采集视频并SurfaceView、TextureView 预览
<!--more-->
从5.0开始（API Level 21），可以完全控制安卓设备相机的新api Camera2(android.hardware.Camera2)被引入了进来。在以前的Camera api(android.hardware.Camera)中，对相机的手动控制需要更改系统才能实现，而且api也不友好。不过老的Camera API在5.0上已经过时，如今Android推荐使用Camera2采集视频，借着写这篇记录的过程，熟悉和理解Camera2流程。

## 一些名词

### YVU
颜色编码的方法，在旧Camera API 常用的是NV21和YV12，可以转成RGB编码
```
// 获取Camera2支持的颜色编码
StreamConfigurationMap map characteristics.get(CameraCharacteristics,SCALER_STREAM_CONFIGURATION_MAP);
map.getOutputFormats();
```

### CameraManager
Camera2中负责管理，查询摄像头信息，打开可用摄像头的管理对象
1. cameraId  通过getCameraidList()枚举获得，代表使用哪个摄像头
2. 设备信息通过CameraCharateristic.getCameraCharacteristics(cameraId)获取到
3. 打开摄像头 openCamera(String cameraId,CameraManager.StateCallback callback,Handler handler),其中StateCallback是接收设备状态的更新回调，例如CameraDevice就是通过StateCallback中的onOpen()回调中拿到，handler表示打开摄像头具体操作在哪个Handler的Looper中，也就是在哪个线程中执行，设置为null，表示在当前线程

### CameraDevice
具体的摄像头，提供一组属性信息，描述硬件设备以及设备的可用设置和参数
1. CamearDevice是在CameraManager打开摄像头后，通过CameraDevice.StateCallback的回调中拿到的，是一个异步过程
2. createCaptureRequest()创建CaptureRequest.Builder,CaptureRequest.Builder负责创建各种捕获图像的请求CaptureRequest
3. createCaptureSession() 负责创建捕获对象会话CameraCaptureSession

### CaptureRequest
一次捕获请求，通过CaptureRequest.Builder的build()方法创建，其实请求参数也是通过Builder来设置：CaptureRequest.Builder常用的方法：
1. addTarget(Surface outputTarget) 将Surface添加到输出列表中，才可以显示在SurfaceView，TextureView或者输出到ImageReader中
2. set(Key<T> key,T value) 设置其他属性

### CameraCaptureSession
捕获的会话Session，预览，拍照，都由该对象进行控制
1. CameraCaptureSession是通过CameraDevice的createCaptureSession(List<Surface>,CameraCaptureSessionStateCallback,handler)创建
2. 拍照 capture(CaptureRequest,CameraCaptureSession.CaptureCallback,hanlder)
3. 预览 setRepeatingRequest(CaptureRequest,CameraCaptureSession.CaptureCallback,handler)

# SurfaceView

完整代码

```
/**
 * Coder: niupuyue
 * Date: 2019/8/6
 * Time: 18:33
 * Desc: 通过SurFaceView预览Camera
 * Version:
 */
public class SurfaceViewActivity extends AppCompatActivity implements View.OnClickListener {

    private static final String TAG = SurfaceViewActivity.class.getSimpleName();

    // SurfaceView预览相机
    private SurfaceView surfaceview;
    // 获取当前相机的id(我们要使用前置相机)
    private String mCameraId;
    private Size mPreviewSize;
    private HandlerThread mCameraThread;
    private Handler mCameraHandler;
    // 摄像头驱动
    private CameraDevice cameraDevice;
    private CaptureRequest.Builder mCaptureRequestBuilder;
    private CaptureRequest mCaptureRequest;
    private CameraCaptureSession session;
    private SurfaceHolder holder;

    private boolean isSurfaceViewCreate = false;
    private int surfaceViewHolderWidth;
    private int surfaceViewHolderHeight;

    private boolean isFontCamera = true;

    private Button btn_surfaceview_font;
    private Button btn_surfaceview_back;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_surfaceview);
        surfaceview = findViewById(R.id.surfaceview);

        btn_surfaceview_font = findViewById(R.id.btn_surfaceview_font);
        btn_surfaceview_back = findViewById(R.id.btn_surfaceview_back);

    }

    @Override
    protected void onResume() {
        super.onResume();
        initCameraThread();

        holder = surfaceview.getHolder();
        surfaceview.setZOrderMediaOverlay(true);
        // 设置半透明
        holder.setFormat(PixelFormat.TRANSLUCENT);
        holder.addCallback(mSurfaceHolderCallback);
    }

    private void initCameraThread() {
        mCameraThread = new HandlerThread("CameraSufaceViewThread");
        mCameraThread.start();
        mCameraHandler = new Handler(mCameraThread.getLooper());
    }

    private SurfaceHolder.Callback mSurfaceHolderCallback = new SurfaceHolder.Callback() {
        @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
        @Override
        public void surfaceCreated(SurfaceHolder surfaceHolder) {
//            surfaceViewHolderWidth = surfaceHolder.getSurfaceFrame().width();
//            surfaceViewHolderHeight = surfaceHolder.getSurfaceFrame().height();
//            isSurfaceViewCreate = true;
            // 设置摄像头的基本配合信息，宽高跟SurfaceView的尺寸相关
            setupCamerar(surfaceHolder.getSurfaceFrame().width(), surfaceHolder.getSurfaceFrame().height());
            openCamera();
        }

        @Override
        public void surfaceChanged(SurfaceHolder surfaceHolder, int i, int i1, int i2) {

        }

        @Override
        public void surfaceDestroyed(SurfaceHolder surfaceHolder) {

        }
    };

    /**
     * 设置Camera的基本配置信息
     *
     * @param width
     * @param height
     */
    @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
    private void setupCamerar(int width, int height) {
        // 获取摄像头管理者CameraManager
        CameraManager cameraManager = (CameraManager) getSystemService(CAMERA_SERVICE);
        try {
            // 遍历所有的摄像头
            for (String cameraId : cameraManager.getCameraIdList()) {
                // CameraCharacteristics 该属性用来描述摄像头驱动信息
                CameraCharacteristics cameraCharacteristics = cameraManager.getCameraCharacteristics(cameraId);
                // 获取摄像头相对于屏幕的方向
                Integer facing = cameraCharacteristics.get(CameraCharacteristics.LENS_FACING);
                // 设置打开后置摄像头
                if (facing != null && facing == CameraCharacteristics.LENS_FACING_FRONT)
                    continue;

                // 获取StreamConfigurationMap，他是管理摄像头支持的所有输出格式和尺寸
                StreamConfigurationMap map = cameraCharacteristics.get(CameraCharacteristics.SCALER_STREAM_CONFIGURATION_MAP);
                assert map != null;
                mPreviewSize = getOptimalSize(map.getOutputSizes(SurfaceTexture.class), width, height);
                mCameraId = cameraId;
            }
        } catch (Exception ex) {
            ex.printStackTrace();
        }
    }

    // 选择sizeMap中大于并最接近width和height的size
    @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
    private Size getOptimalSize(Size[] sizeMap, int width, int height) {
        List<Size> sizeList = new ArrayList<>();
        for (Size option : sizeMap) {
            if (width > height) {
                if (option.getWidth() > width && option.getHeight() > height) {
                    sizeList.add(option);
                }
            } else {
                if (option.getWidth() > height && option.getHeight() > width) {
                    sizeList.add(option);
                }
            }
        }
        if (sizeList.size() > 0) {
            return Collections.min(sizeList, new Comparator<Size>() {
                @Override
                public int compare(Size left, Size right) {
                    return Long.signum(left.getWidth() * left.getHeight() - right.getWidth() * right.getHeight());
                }
            });
        }
        return sizeMap[0];
    }

    /**
     * 打开摄像头
     */
    @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
    private void openCamera() {
        CameraManager cameraManager = (CameraManager) getSystemService(CAMERA_SERVICE);
        if (ActivityCompat.checkSelfPermission(this, Manifest.permission.CAMERA) != PackageManager.PERMISSION_GRANTED) {
            return;
        }
        try {
            Log.e(TAG, "cameraID = " + mCameraId);
            cameraManager.openCamera(mCameraId, mCameraDeviceStateCallback, mCameraHandler);
        } catch (Exception ex) {
            ex.printStackTrace();
        }
    }

    /**
     * 当摄像机的状态发生改变的时候触发该监听
     */
    @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
    private CameraDevice.StateCallback mCameraDeviceStateCallback = new CameraDevice.StateCallback() {
        @Override
        public void onOpened(CameraDevice cameraDevice) {
            SurfaceViewActivity.this.cameraDevice = cameraDevice;
            startPreView();
        }

        @Override
        public void onDisconnected(CameraDevice cameraDevice) {
            if (SurfaceViewActivity.this.cameraDevice != null) {
                SurfaceViewActivity.this.cameraDevice.close();
                cameraDevice.close();
                SurfaceViewActivity.this.cameraDevice = null;
            }
        }

        @Override
        public void onError(CameraDevice cameraDevice, int i) {
            if (SurfaceViewActivity.this.cameraDevice != null) {
                SurfaceViewActivity.this.cameraDevice.close();
                cameraDevice.close();
                SurfaceViewActivity.this.cameraDevice = null;
            }
        }
    };

    @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
    private void startPreView() {
        try {
            Surface surface = holder.getSurface();
            mCaptureRequestBuilder = cameraDevice.createCaptureRequest(CameraDevice.TEMPLATE_PREVIEW);
            if (surface != null) {

                mCaptureRequestBuilder.addTarget(surface);
            } else {
                Log.e(TAG, "surface为空");
            }

            cameraDevice.createCaptureSession(Arrays.asList(surface), new CameraCaptureSession.StateCallback() {
                @Override
                public void onConfigured(CameraCaptureSession cameraCaptureSession) {
                    mCaptureRequest = mCaptureRequestBuilder.build();
                    session = cameraCaptureSession;
                    try {
                        session.setRepeatingRequest(mCaptureRequest, null, mCameraHandler);
                    } catch (Exception ex) {
                        ex.printStackTrace();
                    }
                }

                @Override
                public void onConfigureFailed(CameraCaptureSession cameraCaptureSession) {

                }
            }, mCameraHandler);

        } catch (Exception ex) {
            ex.printStackTrace();
        }
    }

    @Override
    protected void onPause() {
        super.onPause();
        if (session != null) {
            session.close();
            session = null;
        }
        if (cameraDevice != null) {
            cameraDevice.close();
            cameraDevice = null;
        }
    }

    @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
    @Override
    public void onClick(View v) {
        switch (v.getId()) {
            case R.id.btn_surfaceview_font:
                // 打开后置摄像头
                isFontCamera = true;
                // 设置摄像头的基本配合信息，宽高跟SurfaceView的尺寸相关
                setupCamerar(surfaceViewHolderWidth, surfaceViewHolderHeight);
                openCamera();
                break;
            case R.id.btn_surfaceview_back:
                // 打开前置摄像头
                isFontCamera = false;
                // 设置摄像头的基本配合信息，宽高跟SurfaceView的尺寸相关
                setupCamerar(surfaceViewHolderWidth, surfaceViewHolderHeight);
                openCamera();
                break;
        }
    }
}
```

# TextureView

完整代码

```
/**
 * Coder: niupuyue
 * Date: 2019/8/6
 * Time: 18:34
 * Desc: 使用TextureViewActivity预览相机
 * Version:
 */
public class TextureViewActivity extends AppCompatActivity implements View.OnClickListener {

    private String cameraId;
    private Size previewSize;
    private HandlerThread handlerThread;
    private Handler handler;
    private CameraDevice cameraDevice;
    private CaptureRequest request;
    private CaptureRequest.Builder builder;
    private CameraCaptureSession session;

    private Button btn_texture_font, btn_texture_back;
    private TextureView textureView;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_textureview);

        btn_texture_font = findViewById(R.id.btn_texture_font);
        btn_texture_back = findViewById(R.id.btn_texture_back);

        btn_texture_font.setOnClickListener(this);
        btn_texture_back.setOnClickListener(this);

        textureView = findViewById(R.id.textureView);
    }

    @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
    @Override
    protected void onResume() {
        super.onResume();
        startCameraThread();
        if (!textureView.isAvailable()) {
            textureView.setSurfaceTextureListener(textureListener);
        } else {
            startPreView();
        }
    }

    private TextureView.SurfaceTextureListener textureListener = new TextureView.SurfaceTextureListener() {
        @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
        @Override
        public void onSurfaceTextureAvailable(SurfaceTexture surface, int width, int height) {
            // 当特效图review可用的时候，打开相机
            setupCamera(width, height);
            openCamera();
        }

        @Override
        public void onSurfaceTextureSizeChanged(SurfaceTexture surface, int width, int height) {

        }

        @Override
        public boolean onSurfaceTextureDestroyed(SurfaceTexture surface) {
            return false;
        }

        @Override
        public void onSurfaceTextureUpdated(SurfaceTexture surface) {

        }
    };

    @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
    private void setupCamera(int width, int height) {
        // 获取Camera管理对象CameraManager
        CameraManager manager = (CameraManager) getSystemService(CAMERA_SERVICE);
        try {
            // 遍历所有的摄像头
            for (String cameraId : manager.getCameraIdList()) {
                CameraCharacteristics cameraCharacteristics = manager.getCameraCharacteristics(cameraId);
                Integer facing = cameraCharacteristics.get(CameraCharacteristics.LENS_FACING);
                // 在此处默认打开后置摄像头
                if (facing != null && facing == CameraCharacteristics.LENS_FACING_FRONT) {
                    continue;
                }
                // 获取StreamConfigurationMap,他是管理所有摄像头支持的输出格式和尺寸
                StreamConfigurationMap map = cameraCharacteristics.get(CameraCharacteristics.SCALER_STREAM_CONFIGURATION_MAP);
                assert map != null;
                previewSize = getOptimalSize(map.getOutputSizes(SurfaceTexture.class), width, height);
                TextureViewActivity.this.cameraId = cameraId;
            }
        } catch (Exception ex) {
            ex.printStackTrace();
        }
    }

    //选择sizeMap中大于并且最接近width和height的size
    @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
    private Size getOptimalSize(Size[] sizeMap, int width, int height) {
        List<Size> sizeList = new ArrayList<>();
        for (Size option : sizeMap) {
            if (width > height) {
                if (option.getWidth() > width && option.getHeight() > height) {
                    sizeList.add(option);
                }
            } else {
                if (option.getWidth() > height && option.getHeight() > width) {
                    sizeList.add(option);
                }
            }
        }
        if (sizeList.size() > 0) {
            return Collections.min(sizeList, new Comparator<Size>() {
                @Override
                public int compare(Size lhs, Size rhs) {
                    return Long.signum(lhs.getWidth() * lhs.getHeight() - rhs.getWidth() * rhs.getHeight());
                }
            });
        }
        return sizeMap[0];
    }

    @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
    private void openCamera() {
        CameraManager cameraManager = (CameraManager) getSystemService(CAMERA_SERVICE);

        if (ActivityCompat.checkSelfPermission(this, Manifest.permission.CAMERA) != PackageManager.PERMISSION_GRANTED) {
            return;
        }
        try {
            cameraManager.openCamera(cameraId, mStateCallback, handler);
        } catch (CameraAccessException e) {
            e.printStackTrace();
        }
    }

    @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
    private CameraDevice.StateCallback mStateCallback = new CameraDevice.StateCallback() {
        @Override
        public void onOpened(@NonNull CameraDevice camera) {
            cameraDevice = camera;
            startPreView();
        }


        @Override
        public void onDisconnected(@NonNull CameraDevice camera) {
            if (cameraDevice != null) {
                cameraDevice.close();
                camera.close();
                cameraDevice = null;
            }
        }

        @Override
        public void onError(@NonNull CameraDevice camera, int error) {
            if (cameraDevice != null) {
                cameraDevice.close();
                camera.close();
                cameraDevice = null;
            }
        }
    };

    @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
    private void startPreView() {
        SurfaceTexture mSurfaceTexture = textureView.getSurfaceTexture();
        mSurfaceTexture.setDefaultBufferSize(previewSize.getWidth(), previewSize.getHeight());

        Surface previewSurface = new Surface(mSurfaceTexture);
        try {
            builder = cameraDevice.createCaptureRequest(CameraDevice.TEMPLATE_PREVIEW);
            builder.addTarget(previewSurface);
            cameraDevice.createCaptureSession(Arrays.asList(previewSurface), new CameraCaptureSession.StateCallback() {
                @Override
                public void onConfigured(@NonNull CameraCaptureSession session) {
                    request = builder.build();
                    session = session;
                    try {
                        session.setRepeatingRequest(request, null, handler);
                    } catch (CameraAccessException e) {
                        e.printStackTrace();
                    }
                }

                @Override
                public void onConfigureFailed(@NonNull CameraCaptureSession session) {

                }
            }, handler);
        } catch (CameraAccessException e) {
            e.printStackTrace();
        }
    }

    private void startCameraThread() {
        handlerThread = new HandlerThread("CameraHandlerThread");
        handlerThread.start();
        handler = new Handler(handlerThread.getLooper());
    }

    @Override
    protected void onPause() {
        super.onPause();
        if (session != null) {
            session.close();
            session = null;
        }
        if (cameraDevice != null) {
            cameraDevice.close();
            cameraDevice = null;
        }
    }

    @Override
    public void onClick(View v) {
        switch (v.getId()) {
            case R.id.btn_texture_font:

                break;
            case R.id.btn_texture_back:

                break;
        }
    }
}
```

[demo代码](https://github.com/xiaoniudadi/android-media-demo/tree/master/03-android-media-camera-video)

# 参考博客
[Camera2 API 采集视频并SurfaceView、TextureView 预览](https://www.jianshu.com/p/e01c11b96829)
