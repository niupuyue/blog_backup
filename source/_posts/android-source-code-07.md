---
title: 源码分析(七) Android系统服务机制
date: 2020-03-08 22:09:13
tags:
 - 源码分析
---

Android系统服务机制
<!--more-->

直接分析：
通常情况下，我们如果想要获取系统服务，都是通过Context.getSystemService(serviceName)的方式获取的。
其实也就是说，我们调用了Context这个类中的getSystemService方法。Context是一个抽象方法，他有两个子类，分别是ContextImpl和ContextWrapper,其中ContextWrapper类中有三个子类，分别是Application，Service和ContextThemeWrapper对象，其中的ContextThemeWrapper对象里面有一个子类就是Activity，所以如果我们想要实现换肤功能，这部分的内容不能不知道。
回到刚才所说的内容，ContextImpl是Context的实现类，其中那个大部分Context的方法都是在这个类中具体实现的，我们直接看这个类中的getSystemService方法

```
    @Override
    public Object getSystemService(String name) {
        return SystemServiceRegistry.getSystemService(this, name);
    }

```
这里我们发现该方法直接返回一个SystemServiceRegistry类中的静态方法getSystemService，并且需要传递两个对象，分别是上下文Context和服务名name，那么我们再来看你一下这个SystemServiceRegistry类，在这个类中，我们做的事情就是将所有的系统服务按照我们的需要返回给调用者。并且我们的方法中有一个name属性，所以我们很容易想到，我们在可能是会通过一个Map集合存储系统服务信息。

```
    /**
     * Gets a system service from a given context.
     */
    public static Object getSystemService(ContextImpl ctx, String name) {
        ServiceFetcher<?> fetcher = SYSTEM_SERVICE_FETCHERS.get(name);
        return fetcher != null ? fetcher.getService(ctx) : null;
    }

```
在这个方法中，我们通过SYSTEM_SERVICE_FETCHERS获取到一个ServiceFetcher对象，其中的SYSTEM_SERVICE_FETCHERS就是一个Map集合，声明如下：
```
    private static final Map<String, ServiceFetcher<?>> SYSTEM_SERVICE_FETCHERS =
            new ArrayMap<String, ServiceFetcher<?>>();
```
这里我们会有两个疑问，第一个，既然我们是通过get方法拿到的这个系统服务对象，那么肯定有通过put方法填充系统服务对象；第二个，SetviceFetcher是一个什么样的对象？
先来看第一个问题，我们调用SYSTEM_SERVICE_FETCER的put方法是在一个registService方法中调用的，而且这个方法是一个私有静态方法，那该方法只会在当前类中调用
```
    /**
     * Statically registers a system service with the context.
     * This method must be called during static initialization only.
     */
    private static <T> void registerService(String serviceName, Class<T> serviceClass,
            ServiceFetcher<T> serviceFetcher) {
        SYSTEM_SERVICE_NAMES.put(serviceClass, serviceName);
        SYSTEM_SERVICE_FETCHERS.put(serviceName, serviceFetcher);
    }

```
这个方法的使用我们会发现是在一个静态代码块中调用的，这个静态代码块就是注册系统服务的地方法
```
static {
        // 内容比较多，我只选择平时开发使用比较多的
        registerService(Context.ALARM_SERVICE, AlarmManager.class,
                new CachedServiceFetcher<AlarmManager>() {
            @Override
            public AlarmManager createService(ContextImpl ctx) throws ServiceNotFoundException {
                IBinder b = ServiceManager.getServiceOrThrow(Context.ALARM_SERVICE);
                IAlarmManager service = IAlarmManager.Stub.asInterface(b);
                return new AlarmManager(service, ctx);
            }});
        // AudioManager
        registerService(Context.AUDIO_SERVICE, AudioManager.class,
                new CachedServiceFetcher<AudioManager>() {
            @Override
            public AudioManager createService(ContextImpl ctx) {
                return new AudioManager(ctx);
            }});
        // BluetoothManager
        registerService(Context.BLUETOOTH_SERVICE, BluetoothManager.class,
                new CachedServiceFetcher<BluetoothManager>() {
            @Override
            public BluetoothManager createService(ContextImpl ctx) {
                return new BluetoothManager(ctx);
            }});
        // ClipboardManager
        registerService(Context.CLIPBOARD_SERVICE, ClipboardManager.class,
            new CachedServiceFetcher<ClipboardManager>() {
            @Override
            public ClipboardManager createService(ContextImpl ctx) throws ServiceNotFoundException {
                return new ClipboardManager(ctx.getOuterContext(),
                        ctx.mMainThread.getHandler());
            }});

        // The clipboard service moved to a new package.  If someone asks for the old
        // interface by class then we want to redirect over to the new interface instead
        // (which extends it).
        SYSTEM_SERVICE_NAMES.put(android.text.ClipboardManager.class, Context.CLIPBOARD_SERVICE);
        // ConnectivityManager
        registerService(Context.CONNECTIVITY_SERVICE, ConnectivityManager.class,
                new StaticApplicationContextServiceFetcher<ConnectivityManager>() {
            @Override
            public ConnectivityManager createService(Context context) throws ServiceNotFoundException {
                IBinder b = ServiceManager.getServiceOrThrow(Context.CONNECTIVITY_SERVICE);
                IConnectivityManager service = IConnectivityManager.Stub.asInterface(b);
                return new ConnectivityManager(context, service);
            }});
        // LocationManager
        registerService(Context.LOCATION_SERVICE, LocationManager.class,
                new CachedServiceFetcher<LocationManager>() {
            @Override
            public LocationManager createService(ContextImpl ctx) throws ServiceNotFoundException {
                IBinder b = ServiceManager.getServiceOrThrow(Context.LOCATION_SERVICE);
                return new LocationManager(ctx, ILocationManager.Stub.asInterface(b));
            }});
        // PowerManager
        registerService(Context.POWER_SERVICE, PowerManager.class,
                new CachedServiceFetcher<PowerManager>() {
            @Override
            public PowerManager createService(ContextImpl ctx) throws ServiceNotFoundException {
                IBinder b = ServiceManager.getServiceOrThrow(Context.POWER_SERVICE);
                IPowerManager service = IPowerManager.Stub.asInterface(b);
                return new PowerManager(ctx.getOuterContext(),
                        service, ctx.mMainThread.getHandler());
            }});
        // SensorManager    
        registerService(Context.SENSOR_SERVICE, SensorManager.class,
                new CachedServiceFetcher<SensorManager>() {
            @Override
            public SensorManager createService(ContextImpl ctx) {
                return new SystemSensorManager(ctx.getOuterContext(),
                  ctx.mMainThread.getHandler().getLooper());
            }});
        
        //CHECKSTYLE:ON IndentationCheck
    }
```
在这些方法中我们会发现最终都会通过CaceServiceFetcer或者staticServiceFetcer或StaticApplicationContextServiceFetcher的匿名内部类，通过实现createService方法返回一个xxxManager对象。其中这三个类都是继承了ServiceFetcher接口。这也就是刚才所说的第二个问题。

这里我们以Vibrator为例，我们的注册方法是如下所示
```
registerService(Context.VIBRATOR_SERVICE, Vibrator.class,
                new CachedServiceFetcher<Vibrator>() {
            @Override
            public Vibrator createService(ContextImpl ctx) {
                return new SystemVibrator(ctx);
            }});

```
最终返回的是一个SystemVibrator对象，这个对象就是系统提供的震动管理器对象。在这个类中，我们继承Vibrator抽象类，在Vibrator抽象类中，有我们会使用到的一些方法，比如vibrate()开启震动方法，cancel()取消震动方法，hasVibrator()方法。那么这些方法都是最终在SystemVibrator类中实现的。我们一最简单的hasVibrator()方法为例，看一下，在SystemVibrator中它是如何实现的
```
    @Override
    public boolean hasVibrator() {
        if (mService == null) {
            Log.w(TAG, "Failed to vibrate; no vibrator service.");
            return false;
        }
        try {
            return mService.hasVibrator();
        } catch (RemoteException e) {
        }
        return false;
    }

```
我们会发现这里我们通过调用mService中的hasVibrator方法，将数据返回。先来看一下mService是如何声明的
```
    private final IVibratorService mService;

    @UnsupportedAppUsage
    public SystemVibrator() {
        mService = IVibratorService.Stub.asInterface(ServiceManager.getService("vibrator"));
    }

```
这里我们发现最终使用的是IVibratorService这个接口对象，相信大家看到这个之后都会焕然大悟，其实就是AIDL，通过AIDL实现跨进程通信
那么我们很容易就自导IVibratorService是一个接口，我们需要找到实现了该接口的对象，下面就是找IVibratorService的实现类了。这里如何确定，是要根据经验积累的，所以这里我们直接看到他的实现类，也就是VibratorService类。在这个类中我们发现，他继承了IVibratorService.Stub抽象类，所以，IVibratorService中的方法，在VibratorService中是有具体实现的。
```
    @Override // Binder call
    public boolean hasVibrator() {
        return doVibratorExists();
    }

    private boolean doVibratorExists() {
        // For now, we choose to ignore the presence of input devices that have vibrators
        // when reporting whether the device has a vibrator.  Applications often use this
        // information to decide whether to enable certain features so they expect the
        // result of hasVibrator() to be constant.  For now, just report whether
        // the device has a built-in vibrator.
        //synchronized (mInputDeviceVibrators) {
        //    return !mInputDeviceVibrators.isEmpty() || vibratorExists();
        //}
        return vibratorExists();
    }

    static native boolean vibratorExists();
```

上面的三个方法调用是依次执行的，所以最终我们调用的native库中的vibraatorExists方法，并且将返回的数据传递给调用者。

# 参考资料