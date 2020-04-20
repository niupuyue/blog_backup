---
title: 源码分析(四) Activity启动流程
date: 2020-03-05 22:09:13
tags:
 - 源码分析
---

Activity启动流程
<!--more-->

# 关键类介绍
在分析Activity启动流程之前需要有几个关键类是我们需要提前了解的，可能对立面的代码不是非常细致的了解，但是至少知道这几个类是干什么用的

1. ActivityManagerService：AMS是Android核心服务之一，主要完成Android四大组件启动，切换，调度以及应用进程管理和调度等工作，其职责与操作系统中的进程管理和调度模块相类似，本身是一个Binder实现类，应用进程通过Binder机制调度系统服务
2. ActivityThread：应用程序入口，系统通过调用main函数，开启消息循环队列，ActivityThread所在的线程叫主线程(UI线程)
3. Instrumentation：工具类，用来监控应用程序与系统之间的交互，包装了ActivityManagerService的调用，一些插件化方案通过hook该类实现
4. ActivityStarter：Activity启动的工具类，处理Activity启动的各种flag
5. ActivityStackSupervisor：管理所有应用的Activity栈，其中mFocusedStack就是当前应用的Activity栈

先来看一张基本流程图，在这张图中我们只是将启动的流程简化，告诉我们基本的启动步骤

[启动基本流程图(简化)](/assets/sourcecode/activity_start_01.png)

大致启动步骤可以分为七步，可能在一开始看到这么多启动步骤，会觉的难以接受，但是如果我们能够分开来看，其实也不是特别复杂
1. 在Launcher中，我们点击应用图标，会通知AMS我们需要启动一个Activity，通知最终会执行到System Service进程中
2. 在System Service进程中，会告诉Launcher需要进入pause状态
3. Launcher收到pause通知后挂起，并通知System Service进程，可以开启新的Activity了
4. System Service进程在得知Launcher挂起后，会通过zygote的初试进程fork出一个新的进程(App进程)，这就是我们启动Activity需要使用到的进程，紧接着执行一些初始化操作
5. App进程完成初始化操作之后返回告诉System Service进程
6. System Service在得知App进程初始化完成之后完成App进程的注册并且启动Activity
7. 启动App进程后Activity处于运行状态，并且告诉System Service

其中我们可以把着七个步骤分为两部分，第一部分包括步骤1，2，3，这个三个步骤主要是Launcher和System Service完成的交互，第二部分包括步骤4，5，6，7，这四个步骤主要是完成应用进程的创建，初始化，注册等工作。那么这样分析完成之后我们就很明了了。

关于Activity启动详细的流程图，在网上有很多，有兴趣的同学可以去网上搜一下，这里我贴出一些别人的图片，供大家参考

[启动详细流程图](/assets/sourcecode/activity_start_02.png)

这里我们将启动流程总的来说一遍(如果面试的时候别人问你启动流程，直接按照这个回答应该就是问题不大，不过后面别人肯定也会进一步的问清楚细节)

当点击桌面的应用图标时，会发起启动远程进程的操作，利用Binder机制发送消息给system_service进程，在system_service进程中会调用ActivityManagerService#startProcessLocked()方法，这个方法内部会调用process.start(android.app.ActivityThread)方法，然后通过socket通信通知zygote进程fork出一个子进程，也就是我们的App进程，在App进程创建之后将ActivityThread加入到App进程中，执行ActivityThread#main()方法；在App进程中，main()方法会初始化ActivityThread，同时创建ApplicationThread，Looper，MessageQueue等对象，调用ActivityThread#attach(false)方法进行Binder通信，在这个方法中调用ActivityManagerService#attachApplication(mAppThread)方法，将thread信息告诉ActivityManagerService，接着开启Looper循环；而在system_service中，ActivityManagerService#attachApplication(mAppThread)方法调用了thread#bindApplication()和mStackSupervisor#attachApplicationLocked()方法。其中thread#bindApplication()方法调用了ActivityThread#sendMessage(H.BIND_APPLICATION,data)方法，最终走到了ActivityThread#handleBindApplication()方法，进而创建Application对象，并调用Application#attach(context)方法，并且绑定context，在完成Application的创建之后，调用mInstrumentation#callApplicationOnce()方法，执行Application#onCreate()方法生命周期；而对于mStackSupervisor#attachApplicationLocked()方法调用了app#thread#scheduleLaunchActivity()方法(其实就是ActivityThread#ApplicationThread#scheduleLaunchActivity()方法)，进而通过ActivityThread#sendMessage(H.LAUNCH_ACTIVITY,r)方法，最终执行到ActivityThread#handleLaunchActivity()方法，进而创建Activity对象，调用Activity#attach()方法，在调用mInstrumentation#callActivityOnCreate()方法，执行Activity#onCreate()的声明周期方法。

内容比较多，大部分的都是方方的嵌套调用，下面我们依次来分析。

> 注意：Android不同版本在启动流程中所调用的方法可能有差别，但是基本顺序是不会改变的，我写这篇博客时采用的编译方式是25.0.2，也就是buildToolsVersion "25.0.2"。其他的版本可能略有不同，比如使用buildToolsversion是28以上时，我们不是调用ActivityManagerNative类中的方法而是嗲用ActivityTaskManager中的方法。其他不同的地方，同学们可以自己去查找和对比

### step 1
当用户点击Launcher中的应用图标时，我们执行的是Activity#startActivity()方法。这里可能有同学会问，为什么和Activity跳转的方式是一样的？这里其实Launcher也是一个应用，我们通过Launcher启动别的Activity其实就是传递Intent对象，让AMS帮我们启动目标应用。

Activity.java文件中代码
```
    @Override
    public void startActivity(Intent intent) {
        this.startActivity(intent, null);
    }

    // 紧接着调用带有bundle对象的startActivity方法
    @Override
    public void startActivity(Intent intent, @Nullable Bundle options) {
        if (options != null) {
            startActivityForResult(intent, -1, options);
        } else {
            // Note we want to go through this call for compatibility with
            // applications that may have overridden the method.
            startActivityForResult(intent, -1);
        }
    }
    // 紧接着调用带有返回值的startActivityForResult方法
    public void startActivityForResult(@RequiresPermission Intent intent, int requestCode,
            @Nullable Bundle options) {
        if (mParent == null) {
            options = transferSpringboardActivityOptions(options);
            Instrumentation.ActivityResult ar =
                mInstrumentation.execStartActivity(
                    this, mMainThread.getApplicationThread(), mToken, this,
                    intent, requestCode, options);
            if (ar != null) {
                mMainThread.sendActivityResult(
                    mToken, mEmbeddedID, requestCode, ar.getResultCode(),
                    ar.getResultData());
            }
            ...
            cancelInputsAndStartExitTransition(options);
            // TODO Consider clearing/flushing other event sources and events for child windows.
        } else {
            ...
        }
    }
    
```
在startActivityForResult方法中我们删除了一些我们暂时不研究的代码，这里面关键的内容就是
```
Instrumentation.ActivityResult ar = mInstrumentation.execStart();
```
在这个方法中，传递的有多个参数，分别是上下文对象(context),实现了IBinder机制的ApplicationThread对象，token值，当前Activity对象，Intent意图，int类型的requestCode，bundle传递的数据。
而在Instrumentation这个类里面，execStartActivity方法如下所示
```
    public ActivityResult execStartActivity(
            Context who, IBinder contextThread, IBinder token, Activity target,
            Intent intent, int requestCode, Bundle options) {
        ... // 这里主要是对访问Activity的Uri的封装
        try {
            intent.migrateExtraStreamToClipData();
            intent.prepareToLeaveProcess(who);
            int result = ActivityManagerNative.getDefault()
                .startActivity(whoThread, who.getBasePackageName(), intent,
                        intent.resolveTypeIfNeeded(who.getContentResolver()),
                        token, target != null ? target.mEmbeddedID : null,
                        requestCode, 0, null, options);
            checkStartActivityResult(result, intent);
        } catch (RemoteException e) {
            throw new RuntimeException("Failure from system", e);
        }
        return null;
    }

```
在execStartActivity()方法中，我们调用了ActivityManagerNative.getDefault().startActivity()方法，我们进入到ActivityManagerNative这个类里面看一下，我们通过代码注释可以了解到，主要是帮我们完成Binder机制的通信功能的，我们暂时先不管这个类里面具体的代码，只看里面的getDefault()方法
```
    /**
     * Retrieve the system's default/global activity manager.
     */
    static public IActivityManager getDefault() {
        return gDefault.get();
    }
    
    // 这里我们发现其实我们就是通过IgDefault这个对象通过get方法获取到的一个IActivitykManager对象
    private static final Singleton<IActivityManager> gDefault = new Singleton<IActivityManager>() {
        protected IActivityManager create() {
            IBinder b = ServiceManager.getService("activity");
            if (false) {
                Log.v("ActivityManager", "default service binder = " + b);
            }
            IActivityManager am = asInterface(b);
            if (false) {
                Log.v("ActivityManager", "default service = " + am);
            }
            return am;
        }
    };    
    
```
这里我们可以将Singleton当做是一个集合，通过get方法可以从集合中获取一个IActivityManager对象。其中create方法就是我们通过Binder机制从ServiceManager中得到一个IActivityManager对象。刚才在代码中我们是通过ActivityManager.getDefault()的这个对象调用startActivity()方法，其实就是通过ServiceManager这个对象中的startActivity方法。而ServiceManager的实现类就是ActivityManagerService。
在ActivityManagerService这个类里面我们首先会发现，他继承自IActivityManager.Stub这个接口，这个接口是AIDL的接口，暂时不用管。在ActivityManagerService这个类中的startActivity方法如下所示
```
    @Override
    public final int startActivity(IApplicationThread caller, String callingPackage,
            Intent intent, String resolvedType, IBinder resultTo, String resultWho, int requestCode,
            int startFlags, ProfilerInfo profilerInfo, Bundle bOptions) {
        return startActivityAsUser(caller, callingPackage, intent, resolvedType, resultTo,
                resultWho, requestCode, startFlags, profilerInfo, bOptions,
                UserHandle.getCallingUserId());
    }

    final int startActivity(Intent intent, ActivityStackSupervisor.ActivityContainer container) {
        enforceNotIsolatedCaller("ActivityContainer.startActivity");
        final int userId = mUserController.handleIncomingUser(Binder.getCallingPid(),
                Binder.getCallingUid(), mStackSupervisor.mCurrentUser, false,
                ActivityManagerService.ALLOW_FULL_ONLY, "ActivityContainer", null);

        // TODO: Switch to user app stacks here.
        String mimeType = intent.getType();
        final Uri data = intent.getData();
        if (mimeType == null && data != null && "content".equals(data.getScheme())) {
            mimeType = getProviderMimeType(data, userId);
        }
        container.checkEmbeddedAllowedInner(userId, intent, mimeType);

        intent.addFlags(FORCE_NEW_TASK_FLAGS);
        return mActivityStarter.startActivityMayWait(null, -1, null, intent, mimeType, null, null, null,
                null, 0, 0, null, null, null, null, false, userId, container, null);
    }

```

在这两个方法中，我们发现最终调用的是mActivityStarter对象中的startActivityMayWait方法，而ActivityStarter这个类使用解释如何启动Activity的。找到我们对应的startActivityMayWait()方法，这个方法的代码表较多，我们只看比较关键的部分
```
    final int startActivityMayWait(IApplicationThread caller, int callingUid,
            String callingPackage, Intent intent, String resolvedType,
            IVoiceInteractionSession voiceSession, IVoiceInteractor voiceInteractor,
            IBinder resultTo, String resultWho, int requestCode, int startFlags,
            ProfilerInfo profilerInfo, WaitResult outResult,
            Configuration globalConfig, Bundle bOptions, boolean ignoreTargetSecurity, int userId,
            IActivityContainer iContainer, TaskRecord inTask, String reason) {
        ···
            int res = startActivityLocked(caller, intent, ephemeralIntent, resolvedType,
                    aInfo, rInfo, voiceSession, voiceInteractor,
                    resultTo, resultWho, requestCode, callingPid,
                    callingUid, callingPackage, realCallingPid, realCallingUid, startFlags,
                    options, ignoreTargetSecurity, componentSpecified, outRecord, container,
                    inTask, reason);

        ···
    }

    int startActivityLocked(IApplicationThread caller, Intent intent, Intent ephemeralIntent,
            String resolvedType, ActivityInfo aInfo, ResolveInfo rInfo,
            IVoiceInteractionSession voiceSession, IVoiceInteractor voiceInteractor,
            IBinder resultTo, String resultWho, int requestCode, int callingPid, int callingUid,
            String callingPackage, int realCallingPid, int realCallingUid, int startFlags,
            ActivityOptions options, boolean ignoreTargetSecurity, boolean componentSpecified,
            ActivityRecord[] outActivity, ActivityStackSupervisor.ActivityContainer container,
            TaskRecord inTask, String reason) {

        ···

        mLastStartActivityResult = startActivity(caller, intent, ephemeralIntent, resolvedType,
                aInfo, rInfo, voiceSession, voiceInteractor, resultTo, resultWho, requestCode,
                callingPid, callingUid, callingPackage, realCallingPid, realCallingUid, startFlags,
                options, ignoreTargetSecurity, componentSpecified, mLastStartActivityRecord,
                container, inTask);

       ···
        return mLastStartActivityResult;
    }

    private int startActivity(IApplicationThread caller, Intent intent, Intent ephemeralIntent,
            String resolvedType, ActivityInfo aInfo, ResolveInfo rInfo,
            IVoiceInteractionSession voiceSession, IVoiceInteractor voiceInteractor,
            IBinder resultTo, String resultWho, int requestCode, int callingPid, int callingUid,
            String callingPackage, int realCallingPid, int realCallingUid, int startFlags,
            ActivityOptions options, boolean ignoreTargetSecurity, boolean componentSpecified,
            ActivityRecord[] outActivity, ActivityStackSupervisor.ActivityContainer container,
            TaskRecord inTask) {
        ···

        return startActivity(r, sourceRecord, voiceSession, voiceInteractor, startFlags, true,
                options, inTask, outActivity);
    }

    private int startActivity(final ActivityRecord r, ActivityRecord sourceRecord,
            IVoiceInteractionSession voiceSession, IVoiceInteractor voiceInteractor,
            int startFlags, boolean doResume, ActivityOptions options, TaskRecord inTask,
            ActivityRecord[] outActivity) {
        ···
        try {
            mService.mWindowManager.deferSurfaceLayout();
            result = startActivityUnchecked(r, sourceRecord, voiceSession, voiceInteractor,
                    startFlags, doResume, options, inTask, outActivity);
        } finally {
           ···
        }
        ···
    }

    private int startActivityUnchecked(final ActivityRecord r, ActivityRecord sourceRecord,
            IVoiceInteractionSession voiceSession, IVoiceInteractor voiceInteractor,
            int startFlags, boolean doResume, ActivityOptions options, TaskRecord inTask,
            ActivityRecord[] outActivity) {

        ···
        if (mDoResume) {
            final ActivityRecord topTaskActivity =
                    mStartActivity.getTask().topRunningActivityLocked();
            if (!mTargetStack.isFocusable()
                    || (topTaskActivity != null && topTaskActivity.mTaskOverlay
                    && mStartActivity != topTaskActivity)) {
                ···
            } else {
               ···
                mSupervisor.resumeFocusedStackTopActivityLocked(mTargetStack, mStartActivity,
                        mOptions);
            }
        } else {
            ···
        }
        ···
    }

```
在这个类中我们发现依次调用了startActivityMayWait(),startActivityLocked(),startActivity(),startActivityUnchecked()等方法，最终调用了mSupervisor中的resumenFocusedStackTopActivityLocked()方法。其中mSupervisor这个变量其实是ActivityStackSupervisor对象。在这个类中我们有三个ActivityStack对象，分别是mHomeStack，mFocusedStack，mLastFocusedStack，这三个Activity栈代表不同的含义，其中mFocusedStack表示的是当前应用的任务栈。mHomeStack是包含Launcher的任务栈，mLastFocusedStack表示上一个应用的任务栈(这个我还没有找到具体的操作，所以展示先这么写，后面可能会修改)。我们找到resumeFocusedStackTopActivityLocked()方法，
```
    boolean resumeFocusedStackTopActivityLocked() {
        return resumeFocusedStackTopActivityLocked(null, null, null);
    }

    boolean resumeFocusedStackTopActivityLocked(
            ActivityStack targetStack, ActivityRecord target, ActivityOptions targetOptions) {
        if (targetStack != null && isFocusedStack(targetStack)) {
            return targetStack.resumeTopActivityUncheckedLocked(target, targetOptions);
        }
        final ActivityRecord r = mFocusedStack.topRunningActivityLocked();
        if (r == null || r.state != RESUMED) {
            mFocusedStack.resumeTopActivityUncheckedLocked(null, null);
        }
        return false;
    }

```
这里我们会发现他又回调了ActivityStack中的resumeTopActivityUncheckedLocked方法，
```
    boolean resumeTopActivityUncheckedLocked(ActivityRecord prev, ActivityOptions options) {
       ···
        try {
            ···
            result = resumeTopActivityInnerLocked(prev, options);
        } finally {
            ···
        }
        ···
        return result;
    }

    private boolean resumeTopActivityInnerLocked(ActivityRecord prev, ActivityOptions options) {
     
        ···
        if (next.app != null && next.app.thread != null) {
           ···
        } else {
            ···
            mStackSupervisor.startSpecificActivityLocked(next, true, true);
        }
        ···
    }

```
在resumeTopActivityInnerLocked方法中我们又执行了StackSupervisor中的startSpecificActivityLocked方法。
在ActivityStackSupervisor类中的startSpecificActivityLocked方法中，会判断要启动的App进程是否已经存在，如果存在则通知进程启动，如果没有则先将进程创建出来
```
    void startSpecificActivityLocked(ActivityRecord r,
            boolean andResume, boolean checkConfig) {
       ···
        if (app != null && app.thread != null) {
            try {
               ···
                // 如果进程已存在，则通知进程启动组件
                realStartActivityLocked(r, app, andResume, checkConfig);
                return;
            } catch (RemoteException e) {
                ···
            }
        }
        // 否则先将进程创建出来
        mService.startProcessLocked(r.processName, r.info.applicationInfo, true, 0,
                "activity", r.intent.getComponent(), false, false, true);
    }

```
这里我们直接看进程没有创建的时候的代码，因为在创建完成之后也是会通知进程启动组件的。创建进程我们调用的是mService.startprocessLocked()方法，我们先看一下这个方法的实现，在ActivityManagerService类中，找到startProcessLocked方法
```
    final ProcessRecord startProcessLocked(String processName,
            ApplicationInfo info, boolean knownToBeDead, int intentFlags,
            String hostingType, ComponentName hostingName, boolean allowWhileBooting,
            boolean isolated, boolean keepIfLarge) {
        return startProcessLocked(processName, info, knownToBeDead, intentFlags, hostingType,
                hostingName, allowWhileBooting, isolated, 0 /* isolatedUid */, keepIfLarge,
                null /* ABI override */, null /* entryPoint */, null /* entryPointArgs */,
                null /* crashHandler */);
    }

    final ProcessRecord startProcessLocked(String processName, ApplicationInfo info,
            boolean knownToBeDead, int intentFlags, String hostingType, ComponentName hostingName,
            boolean allowWhileBooting, boolean isolated, int isolatedUid, boolean keepIfLarge,
            String abiOverride, String entryPoint, String[] entryPointArgs, Runnable crashHandler) {
        ···
        startProcessLocked(
                app, hostingType, hostingNameStr, abiOverride, entryPoint, entryPointArgs);
        ···
    }

    private final void startProcessLocked(ProcessRecord app, String hostingType,
            String hostingNameStr, String abiOverride, String entryPoint, String[] entryPointArgs) {
        ···
            if (entryPoint == null) entryPoint = "android.app.ActivityThread";
            Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "Start proc: " +
                    app.processName);
            checkTime(startTime, "startProcess: asking zygote to start proc");
            ProcessStartResult startResult;
            if (hostingType.equals("webview_service")) {
                ···
            } else {
                checkTime(startTime, "startProcess: asking zygote to start proc");
                startResult = Process.start(entryPoint,
                        app.processName, uid, uid, gids, debugFlags, mountExternal,
                        app.info.targetSdkVersion, seInfo, requiredAbi, instructionSet,
                        app.info.dataDir, invokeWith, entryPointArgs);
            }
            ···
    }

```
我们发现最终启动进程是通过Process.start()方法帮我创建一个进程。
进程创建后将 ActivityThread 加载进去，执行  ActivityThread#main() 方法，实例化 ActivityThread，同时创建 ApplicationThread，Looper，Hander 对象，调用 ActivityThread#attach(false) 方法进行 Binder 通信， 接着 Looper 启动循环；
```
    public static void main(String[] args) {
        ...
        Looper.prepareMainLooper();

        ActivityThread thread = new ActivityThread();
        thread.attach(false);

        if (sMainThreadHandler == null) {
            sMainThreadHandler = thread.getHandler();
        }

        if (false) {
            Looper.myLooper().setMessageLogging(new
                    LogPrinter(Log.DEBUG, "ActivityThread"));
        }

        // End of event ActivityThreadMain.
        Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
        Looper.loop();
        ...
    }

    private void attach(boolean system) {
        ···
        if (!system) {
            ···
            final IActivityManager mgr = ActivityManager.getService();
            try {
                mgr.attachApplication(mAppThread);
            } catch (RemoteException ex) {
                throw ex.rethrowFromSystemServer();
            }
            ···
        } else {
           ···
        }
        ···
    }

```
回到 system_server 中，ActivityManagerService#attachApplication(mAppThread) 方法内部调用了 thread#bindApplication() 和 mStackSupervisor#attachApplicationLocked()  这两个方法。具体代码如下：
```
    @Override
    public final void attachApplication(IApplicationThread thread) {
        synchronized (this) {
            int callingPid = Binder.getCallingPid();
            final long origId = Binder.clearCallingIdentity();
            attachApplicationLocked(thread, callingPid);
            Binder.restoreCallingIdentity(origId);
        }
    }

private final boolean attachApplicationLocked(IApplicationThread thread,
            int pid) {
        ···
        try {
            ···
            if (app.instr != null) {
                thread.bindApplication(processName, appInfo, providers,
                        app.instr.mClass,
                        profilerInfo, app.instr.mArguments,
                        app.instr.mWatcher,
                        app.instr.mUiAutomationConnection, testMode,
                        mBinderTransactionTrackingEnabled, enableTrackAllocation,
                        isRestrictedBackupMode || !normalMode, app.persistent,
                        new Configuration(getGlobalConfiguration()), app.compat,
                        getCommonServicesLocked(app.isolated),
                        mCoreSettingsObserver.getCoreSettingsLocked(),
                        buildSerial);
            } else {
                thread.bindApplication(processName, appInfo, providers, null, profilerInfo,
                        null, null, null, testMode,
                        mBinderTransactionTrackingEnabled, enableTrackAllocation,
                        isRestrictedBackupMode || !normalMode, app.persistent,
                        new Configuration(getGlobalConfiguration()), app.compat,
                        getCommonServicesLocked(app.isolated),
                        mCoreSettingsObserver.getCoreSettingsLocked(),
                        buildSerial);
            }
            ···
        } catch (Exception e) {
            ···
        }
        ···
        // See if the top visible activity is waiting to run in this process...
        if (normalMode) {
            try {
                if (mStackSupervisor.attachApplicationLocked(app)) {
                    didSomething = true;
                }
            } catch (Exception e) {
                ···
            }
        }
        ···
    }

```
在ActivityThread中ApplicationThread的bindApplication方法，如下所示
```
    public final void bindApplication(String processName, ApplicationInfo appInfo,
        ···
            sendMessage(H.BIND_APPLICATION, data);
    }

```
紧接着是sendMessage方法
```
    private void sendMessage(int what, Object obj) {
        sendMessage(what, obj, 0, 0, false);
    }

    private void sendMessage(int what, Object obj, int arg1, int arg2, boolean async) {
        ···
        mH.sendMessage(msg);
    }

```
其中在第二个方法中我们通过mH执行了sendMessage()方法，mH其实就当做是Handler对象即可，
```
        public void handleMessage(Message msg) {
            ···
            switch (msg.what) {
                ···
                // 此处的种类很多包括LAUNCH_ACTIVITY等，但暂时先考虑一个
                case BIND_APPLICATION:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "bindApplication");
                    AppBindData data = (AppBindData)msg.obj;
                    handleBindApplication(data);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                ···
            }
            ···
        }
```
在BIND_APPLOICATION中我们通过Trace跟踪器开启跟踪，并且在handleBindApplication方法中创建Application对象
```
    private void handleBindApplication(AppBindData data) {
        ···
        try {
            // 创建 Application 实例
            Application app = data.info.makeApplication(data.restrictedBackupMode, null);
            mInitialApplication = app;
            ···
            try {
                mInstrumentation.callApplicationOnCreate(app);
            } catch (Exception e) {
                ···
            }
        } finally {
            ···
        }
        ···
    }

```
其中makeApplication方法中是完成创建Application的具体操作
```
    public Application makeApplication(boolean forceDefaultAppClass,
            Instrumentation instrumentation) {
        if (mApplication != null) {
            return mApplication;
        }
        ···
        Application app = null;
        ···
        try {
            ···
            app = mActivityThread.mInstrumentation.newApplication(
                    cl, appClass, appContext);
            ···
        } catch (Exception e) {
            ···
        }
        ···
        if (instrumentation != null) {
            try {
                //执行 Application#onCreate() 生命周期
                instrumentation.callApplicationOnCreate(app);
            } catch (Exception e) {
                ···
            }
        }
        ···
        return app;
    }

```
通过makApplication方法可见，其实也是通过ActivityThread.Instrumentation.newApplication()方法创建的。并且在创建完成之后执行callApplicationOnCreate方法，开始执行我们的生命周期。

mStackSupervisor#attachApplicationLocked() 方法中调用 app#thread#scheduleLaunchActivity() 即 ActivityThread#ApplicationThread#scheduleLaunchActivity() 方法，进而通过 ActivityThread#sendMessage(H.LAUNCH_ACTIVITY, r) 方法，最终走到了 ActivityThread#handleLaunchActivity() ，进而创建 Activity 对象，然后调用 activity.attach() 方法，再调用 mInstrumentation#callActivityOnCreate() 执行 Activity#onCreate() 生命周期；
```
    boolean attachApplicationLocked(ProcessRecord app) throws RemoteException {
      
        for (int displayNdx = mActivityDisplays.size() - 1; displayNdx >= 0; --displayNdx) {
            ···
            for (int stackNdx = stacks.size() - 1; stackNdx >= 0; --stackNdx) {
                ···
                if (hr != null) {
                    if (hr.app == null && app.uid == hr.info.applicationInfo.uid
                            && processName.equals(hr.processName)) {
                        try {
                            if (realStartActivityLocked(hr, app, true, true)) {
                                didSomething = true;
                            }
                        } catch (RemoteException e) {
                            ···
                        }
                    }
                }
            }
        }
        ···
    }

    final boolean realStartActivityLocked(ActivityRecord r, ProcessRecord app,
            boolean andResume, boolean checkConfig) throws RemoteException {
        ···
        try {
            ···
            app.thread.scheduleLaunchActivity(new Intent(r.intent), r.appToken,
                    System.identityHashCode(r), r.info,
                    // TODO: Have this take the merged configuration instead of separate global and
                    // override configs.
                    mergedConfiguration.getGlobalConfiguration(),
                    mergedConfiguration.getOverrideConfiguration(), r.compat,
                    r.launchedFromPackage, task.voiceInteractor, app.repProcState, r.icicle,
                    r.persistentState, results, newIntents, !andResume,
                    mService.isNextTransitionForward(), profilerInfo);
             ···
        } catch (RemoteException e) {
            ···
        }
        ···
    }

```
在最后一个方法中我们调用了scheduleLaunchActivity方法，这个方法是在ActivityThread的ApplicationThread中
```
        @Override
        public final void scheduleLaunchActivity(Intent intent, IBinder token, int ident,
                ActivityInfo info, Configuration curConfig, Configuration overrideConfig,
                CompatibilityInfo compatInfo, String referrer, IVoiceInteractor voiceInteractor,
                int procState, Bundle state, PersistableBundle persistentState,
                List<ResultInfo> pendingResults, List<ReferrerIntent> pendingNewIntents,
                boolean notResumed, boolean isForward, ProfilerInfo profilerInfo) {
            ···
            sendMessage(H.LAUNCH_ACTIVITY, r);
        }

```
这里我们发现他也是发送了一个sendMessage方法，其中的变量是LAUNCH_ACTIVITY，剩下的内容其实跟Application基本类似，就不再做叙述了。

至此，Activity的启动流程完全走完，剩下的就是Activity的生命周期方法了。

# 参考资料
[Activity启动流程](https://my.oschina.net/u/920274/blog/3064455)