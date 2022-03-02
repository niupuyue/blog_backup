---
title: 重拾android路(三十九) Camera
date: 2020-06-22 21:35:51
tags:
  - android
  - camera
---

Camera拍照预览分为两个部分，分别是Camera1和Camera2

<!--more-->

不管是Camera1还是Camera2，都需要申明权限
```
<uses-feature
        android:name="android.hardware.camera"
        android:required="true" />

<uses-permission android:name="android.permission.CAMERA" />
```
在Android6.0以上要动态声明权限，这里只做比较简单的demo
```
ActivityCompat.requestPermissions(this, new String[]{Manifest.permission.CAMERA},
                    REQUEST_CAMERA_PERMISSION)
```

## Camera1

在进行Camera的具体操作之前，我们需要先获取设备中Camera的信息，因为市面上的Android设备千差万别，而对于摄像头，几乎每个设备都会有自己的一套规则
```
// 获取当前设备支持的摄像头数量
val phoneNumbers = Camera.getNumberOfCameras()
```
获取数量信息之后，我们需要根据CameraID获取每一个Camera的基本信息，这些基本信息在我们开启相机预览和保存拍摄图片中是十分重要的，CameraInfo包含下面几个我们需要重点关注的信息

- facing:摄像头的方向，一般我们的摄像头分为前置摄像头和后置摄像头，分别对应的数值是Camera.CameraInfo.CAMERA_FACING_FRONT和Camera.CameraInfo.CAMERA_FACING_BACK
- orientation:表示摄像头按照顺时针旋转多少度后是正常画面
- canDisableShutterSound:表示是否支持静音拍摄，也就是拍照时是否会出现“咔嚓”一声。小日子国因为一些特殊原因，是不支持静音拍摄的

我们可以通过遍历的方式获取手机中摄像头的具体信息，如下所示
```
    private var mFrontCameraInfo: Camera.CameraInfo? = null
    private var mFrontCameraID: Int? = 0
    private var mBackCameraInfo: Camera.CameraInfo? = null
    private var mBackCameraID: Int? = 0

    private fun initCameraInfo() {
        val phoneNumbers = Camera.getNumberOfCameras()
        for (cameraId in 0 until phoneNumbers) {
            val cameraInfo = Camera.CameraInfo()
            Camera.getCameraInfo(cameraId, cameraInfo)
            Log.e("Camera1Helper", cameraInfo.toString())
            if (cameraInfo.facing == Camera.CameraInfo.CAMERA_FACING_FRONT) {
                // 前置摄像头
                mFrontCameraID = cameraId
                mFrontCameraInfo = cameraInfo
            } else if (cameraInfo.facing == Camera.CameraInfo.CAMERA_FACING_BACK) {
                // 后置摄像头
                mBackCameraID = cameraId
                mBackCameraInfo = cameraInfo
            }
        }
    }
```
我们通过Camera.getCameraInfo(cameraId,cameraInfo)这个方法获取相机信息，相机的id实际是从0增加到numberOfCameras,一般情况下手机的后置摄像头为0，前置摄像头为1
打开相机的方法是使用**open**方法打开摄像头
```
if(mCamera != null){
  throw new RuntimeException("相机已经被开启，无法同时开启多个相机实例！")
}
try{
  mCamera.open(mFrontCameraID)
}catch(e:Exception){
  // 打开摄像头失败，释放相机资源
  releaseCamera()
}
```
在上面的代码中我们提到在打开摄像头失败之后，需要释放相机资源。其实相机属于比较耗资源的硬件设备，如果在不使用之后，没有及时释放资源，会导致内存泄漏等一系列的问题。除了在异常捕获时释放相机资源，在Activity的生命周期中也要及时的释放资源
```
    /**
     * 释放相机资源
     */
    fun releaseCamera() {
        if (mCamera != null) {
            mCamera?.stopPreview()
            mCamera?.setPreviewCallback(null)
            mCamera?.release()
            mCamera = null
        }
    }
```

上面我们介绍的都是一些比较常用的操作，但我们可能需要的是另外一些操作，比如预览和捕获数据。我们需要认识一下相机中的参数。
相机功能的强大取决于手机厂商是否在底层实现了相应的功能，我们在基于手机进行开发之前需要先判断手机相机是否支持相对应的功能。我们通过**Camera.Parameters**来判断是否存在对应的功能。
**Camera.Parameters**提供了大量类似于**getSupportedXXX**的方法方便我们判断相机是否支持某一个功能。例如通过**getSupportedPreviewSizes()**来判断相机支持的预览尺寸，这些预览尺寸可以帮助我们更好的适配手机的预览尺寸。
同样我们可以通过**Camera.Parameters**来获取相机的绝大部分参数，或者通过**Camera.setParameters()**方法重新设置相机参数。

1. 通过**Camera.getParameters()** 获取**Camera.Parameters**实体对象
2. 通过**Camera.Parameters.getSupportedXXX**获取某个参数的支持情况
3. 通过**Camera.Parameters.set()**方法设置参数
4. 通过**Camera.setParameters()**将参数应用到底层

> Camera.getParameters() 是一个比较耗时的操作，实测 20ms 到 100ms不等，所以尽可能地一次性设置所有必要的参数，然后通过 Camera.setParameters() 一次性应用到底层

设置相机的预览尺寸，指的是把相机画面输出到手机屏幕上的尺寸，通常情况下我们希望在不超过手机分辨率的情况下，愈大愈好，当然也有可能会根据业务需要修改尺寸，比如在自定义相机时，我们需要设置特定尺寸的预览以保证预览时不会变形。在设置手机的预览尺寸之前，我们需要先获取尺寸列表,
通过**Camera.Parameters.getSupportedPreviewSizes()**方法

> 预览尺寸的宽是长边，高是短边，例如 1920x1080，而不是 1080x1920

```
    private fun getFitPreviewOutputSize(camera:Camera): Size {
        val parameters = camera.parameters
        val supportPreviewSize = parameters.supportedPreviewSizes
        ...
    }
```
获取预览尺寸之后，根据实际情况取一个最合适的尺寸作为预览尺寸
```
    private fun getFitPreviewOutputSize(camera:Camera): Size {
        val parameters = camera.parameters
        val supportPreviewSize = parameters.supportedPreviewSizes

    }
```
![手机支持相机尺寸列表](/assets/camera/camera_01.png)

添加预览Surface，相机预览的画面最终是要绘制到Surface上的，Surface 可以来自 SurfaceHolder 或者 SurfaceTexture。一般实现SurfaceHolde的两种方式如下

1. 通过 Camera.setPreviewDisplay() 方法设置 SurfaceHolder 给相机，通常是在你使用 SurfaceView 作为预览控件时会使用该方法。
2. 通过 Camera.setPreviewTexture() 方法设置 SurfaceTexture 给相机，通常是在你使用 TextureView 作为预览控件或者自己创建 SurfaceTexture 时使用该方法

开始相机预览，如下所示
```
    /**
     * 开始相机预览
     */
    private fun startPreview() {
        mCamera?.let {
            it.setPreviewDisplay(mSurfaceHolder)
            // 设置相机旋转角度的问题
            setCameraDisplayOrientation()
            // 开始预览
            it.startPreview()
        }
    }
```

代码里我们会重新设置一下相机的旋转角度问题，之前我们在说道CameraInfo的时候，说道获取的CameraInfo中就包含了旋转信息。如果我们使用相机进行预览，不作任何处理，则会出现画面是横向展示。这是因为我们手机中的传感器方法不一定是垂直的。这里我们需要了解几个概念

- 自然方向：当我们谈论方向的时候，实际上都是相对于某一个 0° 方向的角度，这个 0° 方向被称作自然方向，例如人站立的时候就是自然方向，你总不会认为一个人要倒立的时候才是自然方向吧，而接下来我们要谈论的设备方向就有的自然方向的定义
- 设备方向：设备方向指的是硬件设备在空间中的方向与其自然方向的顺时针夹角。这里提到的自然方向指的就是我们手持一个设备的时候最习惯的方向，比如手机我们习惯竖着拿，而平板我们则习惯横着拿，所以通常情况下手机的自然方向就是竖着的时候，平板的自然方向就是横着的时候

当我们把手机垂直放置且屏幕朝向我们的时候，设备方向为 0°，即设备自然方向
当我们把手机向右横放且屏幕朝向我们的时候，设备方向为 90°
当我们把手机倒着放置且屏幕朝向我们的时候，设备方向为 180°
当我们把手机向左横放且屏幕朝向我们的时候，设备方向为 270°
我们可以通过 OrientationEventListener 监听设备的方向，进而判断设备当前是否处于自然方向，当设备的方向发生变化的时候会回调 OrientationEventListener.onOrientationChanged(int) 方法，传给我们一个 0° 到 359° 的方向值，其中 0° 就代表设备处于自然方向

- 局部坐标系：所谓的局部坐标系指的是当设备处于自然方向时，相对于设备屏幕的坐标系，该坐标系是固定不变的，不会因为设备方向的变化而改变，下图是基于手机的局部坐标系示意图

![局部坐标系指示图](/assets/camera/camera_02.png)

x 轴是当手机处于自然方向时，和手机屏幕平行且指向右边的坐标轴。
y 轴是当手机处于自然方向时，和手机屏幕平行且指向上方的坐标轴。
z 轴是当手机处于自然方向时，和手机屏幕垂直且指向屏幕外面的坐标轴。

为了进一步解释【坐标系是固定不变的，不会因为设备方向的变化而改变】的概念，这里举个例子，当我们把手机向右横放且屏幕朝向我们的时候，此时设备方向为 90°，局部坐标系相对于手机屏幕是保持不变的，所以 y 轴正方向指向右边，x 轴正方向指向下方，z 轴正方向还是指向屏幕外面，如下图所示

![设备方向90°](/asssets/camera/camera_03.png)

- 屏幕方向：屏幕方向指的是屏幕上显示画面与局部坐标系 y 轴的顺时针夹角，注意这里实际上指的是显示的画面，而不是物理硬件上的屏幕，只是我们习惯上称作屏幕方向而已
- 摄像头传感器方向：摄像头传感器方向指的是传感器采集到的画面方向经过顺时针旋转多少度之后才能和局部坐标系的 y 轴正方向一致，其实就是Camera.CameraInfo.orientation属性。
例如 orientation 为 90° 时，意味我们将摄像头采集到的画面顺时针旋转 90° 之后，画面的方向就和局部坐标系的 y 轴正方向一致，换个说法就是原始画面的方向和 y 轴的夹角是逆时针 90°

> 考虑一个特殊情况，就是前置摄像头的画面是做了镜像处理的，也就是所谓的前置镜像操作，这个情况下， orientation 的值并不是实际我们要旋转的角度，我们需要取它的镜像值才是我们真正要旋转的角度，例如 orientation 为 270°，实际我们要旋转的角度是 90°

> 摄像头传感器方向在不同的手机上可能不一样，大部分手机都是 90°，也有小部分是 0° 的，所以我们要通过 Camera.CameraInfo.orientation 去判断方向，而不是假设所有设备的摄像头传感器方向都是 90°

矫正的代码如下所示
```
    /**
     * 相机旋转角度问题
     */
    private fun setCameraDisplayOrientation() {
        val cameraInfo = Camera.CameraInfo()
        Camera.getCameraInfo(mCameraFacing, cameraInfo)
        // 获取当前相机默认旋转的角度
        val rotation = mActivity.windowManager.defaultDisplay.rotation
        var screenAngle = 0
        when (rotation) {
            Surface.ROTATION_0 -> screenAngle = 0
            Surface.ROTATION_90 -> screenAngle = 90
            Surface.ROTATION_180 -> screenAngle = 180
            Surface.ROTATION_270 -> screenAngle = 270
        }

        if (cameraInfo.facing == Camera.CameraInfo.CAMERA_FACING_FRONT) {
            // 如果是前置摄像头
            mDisplayOrientation = (cameraInfo.orientation + screenAngle) % 360
            mDisplayOrientation =
                (360 - mDisplayOrientation) % 360          // compensate the mirror
        } else {
            mDisplayOrientation = (cameraInfo.orientation - screenAngle + 360) % 360
        }

        mCamera?.setDisplayOrientation(mDisplayOrientation)
    }
```

预览已经可以了，我们接下来看一下如何保存数据
在开启相机预览时，我们通过回调的方式获取相机预览的数据，并且可以配置预览数据的数据格式。
如果想要设置预览数据的格式，需要先去判断预览的格式是否支持
```
/**
 * 判断指定的预览格式是否支持。
 */
private boolean isPreviewFormatSupported(Camera.Parameters parameters, int format) {
    List<Integer> supportedPreviewFormats = parameters.getSupportedPreviewFormats();
    return supportedPreviewFormats != null && supportedPreviewFormats.contains(format);
}
```
在确定预览的格式可以被支持之后，我们在通过**setPreviewFormat()**方法来设置预览数据的格式
```
private static final int PREVIEW_FORMAT = ImageFormat.NV21;

if (isPreviewFormatSupported(parameters, PREVIEW_FORMAT)) {
    parameters.setPreviewFormat(PREVIEW_FORMAT);
}
```
至于数据的获取，我们通过**Camera.PreviewCallback**实现该接口，并且绑定给相机即可。注册回调的方式有两种

- setPreviewCallback()：注册预览回调
- setPreviewCallbackWithBuffer()：注册预览回调，并且使用已经配置好的缓冲池

使用 setPreviewCallback() 注册预览回调获取预览数据是最简单的，因为你不需要其他配置流程，直接注册即可，但是出于性能考虑，官方推荐我们使用 setPreviewCallbackWithBuffer()，因为它会使用我们配置好的缓冲对象回调预览数据，避免重复创建内存占用很大的对象。所以接下来我们重点介绍如何根据预览尺寸配置对象池并注册回调，整个步骤如下

1. 根据需求确定预览尺寸
2. 根据需求确定预览数据格式
3. 根据预览尺寸和数据格式计算出每一帧画面要占用的内存大小
4. 通过 addCallbackBuffer() 方法提前添加若干个创建好的 byte 数组对象作为缓冲对象供回调预览数据使用
5. 通过 setPreviewCallbackWithBuffer() 注册预览回调
6. 使用完缓冲对象之后，通过 addCallbackBuffer() 方法回收缓冲对象

这里我们需要重新设置预览尺寸的大小，在配置预览尺寸的同时根据预览尺寸和数据格式 设置缓存区大小
```
@WorkerThread
private void setPreviewSize(int shortSide, int longSide) {
    Camera camera = mCamera;
    if (camera != null && shortSide != 0 && longSide != 0) {
        float aspectRatio = (float) longSide / shortSide;
        Camera.Parameters parameters = camera.getParameters();
        List<Camera.Size> supportedPreviewSizes = parameters.getSupportedPreviewSizes();
        for (Camera.Size previewSize : supportedPreviewSizes) {
            if ((float) previewSize.width / previewSize.height == aspectRatio && previewSize.height <= shortSide && previewSize.width <= longSide) {
                parameters.setPreviewSize(previewSize.width, previewSize.height);
                Log.d(TAG, "setPreviewSize() called with: width = " + previewSize.width + "; height = " + previewSize.height);

                if (isPreviewFormatSupported(parameters, PREVIEW_FORMAT)) {
                    parameters.setPreviewFormat(PREVIEW_FORMAT);
                    int frameWidth = previewSize.width;
                    int frameHeight = previewSize.height;
                    int previewFormat = parameters.getPreviewFormat();
                    PixelFormat pixelFormat = new PixelFormat();
                    PixelFormat.getPixelFormatInfo(previewFormat, pixelFormat);
                    int bufferSize = (frameWidth * frameHeight * pixelFormat.bitsPerPixel) / 8;
                    camera.addCallbackBuffer(new byte[bufferSize]);
                    camera.addCallbackBuffer(new byte[bufferSize]);
                    camera.addCallbackBuffer(new byte[bufferSize]);
                    Log.d(TAG, "Add three callback buffers with size: " + bufferSize);
                }

                camera.setParameters(parameters);
                break;
            }
        }
    }
}
```

> 在预览回调方法里使用完 Buffer 之后，记得一定要调用 addCallbackBuffer() 将 Buffer 重新添加到缓冲池里供相机使用




[demo地址](https://github.com/niupuyue/blog_demo_android/tree/master/CameraDemo)

# 参考资料
[Android Camera1 教程](https://www.jianshu.com/p/705d4792e836)
