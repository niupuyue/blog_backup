---
title: 源码分析(三) Handler机制
date: 2020-03-04 22:09:13
tags:
 - 源码分析
---

Handler机制
<!--more-->

先来看一张Android消息循环流程图
[Android消息循环流程图](/assets/sourcecode/handler_01.png)
主要涉及的角色如下所示：
- message：消息。
- MessageQueue：消息队列，负责消息的存储与管理，负责管理由 Handler 发送过来的 Message。读取会自动删除消息，单链表维护，插入和删除上有优势。在其next()方法中会无限循环，不断判断是否有消息，有就返回这条消息并移除。
- Looper：消息循环器，负责关联线程以及消息的分发，在该线程下从 MessageQueue获取 Message，分发给Handler，Looper创建的时候会创建一个 - MessageQueue，调用loop()方法的时候消息循环开始，其中会不断调用messageQueue的next()方法，当有消息就处理，否则阻塞在messageQueue的next()方法中。当Looper的quit()被调用的时候会调用messageQueue的quit()，此时next()会返回null，然后loop()方法也就跟着退出。
- Handler：消息处理器，负责发送并处理消息，面向开发者，提供 API，并隐藏背后实现的细节。

整个消息的循环流程还是比较清晰的，具体说来：
1. Handler通过sendMessage()发送消息Message到消息队列MessageQueue。
2. Looper通过loop()不断提取触发条件的Message，并将Message交给对应的target handler来处理。
3. target handler调用自身的handleMessage()方法来处理Message。

OK,废话说话了，下面开始说说实现原理

我们都知道Android应用的入口函数是在ActivityThread中的main方法，那么我们来看一下main方法帮我们实现了什么内容
```
    public static void main(String[] args) {
        Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "ActivityThreadMain");

        // Install selective syscall interception
        AndroidOs.install();

        // CloseGuard defaults to true and can be quite spammy.  We
        // disable it here, but selectively enable it later (via
        // StrictMode) on debug builds, but using DropBox, not logs.
        CloseGuard.setEnabled(false);

        Environment.initForCurrentUser();

        // Make sure TrustedCertificateStore looks in the right place for CA certificates
        final File configDir = Environment.getUserConfigDirectory(UserHandle.myUserId());
        TrustedCertificateStore.setDefaultUserDirectory(configDir);

        Process.setArgV0("<pre-initialized>");

        Looper.prepareMainLooper();

        // Find the value for {@link #PROC_START_SEQ_IDENT} if provided on the command line.
        // It will be in the format "seq=114"
        long startSeq = 0;
        if (args != null) {
            for (int i = args.length - 1; i >= 0; --i) {
                if (args[i] != null && args[i].startsWith(PROC_START_SEQ_IDENT)) {
                    startSeq = Long.parseLong(
                            args[i].substring(PROC_START_SEQ_IDENT.length()));
                }
            }
        }
        ActivityThread thread = new ActivityThread();
        thread.attach(false, startSeq);

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

        throw new RuntimeException("Main thread loop unexpectedly exited");
    }

```
main方法中有两个比较关键的代码，①Looper.prepareMainLooper()和②Looper.loop()。这两个方法的调用，帮我们完成了Handler机制的初始化操作。
首先来看Looper.prepareMainLooper()帮我们作了什么内容
```
    /**
     * Initialize the current thread as a looper, marking it as an
     * application's main looper. The main looper for your application
     * is created by the Android environment, so you should never need
     * to call this function yourself.  See also: {@link #prepare()}
     */
    public static void prepareMainLooper() {
        prepare(false);
        synchronized (Looper.class) {
            if (sMainLooper != null) {
                throw new IllegalStateException("The main Looper has already been prepared.");
            }
            sMainLooper = myLooper();
        }
    }

```
在这个方法中我们首先执行了prepare(false)方法，这个方法主要是帮我们创建一个Looper对象，并且将Looper对象设置到ThreadLocal中，代码如下
```
    /** Initialize the current thread as a looper.
      * This gives you a chance to create handlers that then reference
      * this looper, before actually starting the loop. Be sure to call
      * {@link #loop()} after calling this method, and end it by calling
      * {@link #quit()}.
      */
    public static void prepare() {
        prepare(true);
    }

    private static void prepare(boolean quitAllowed) {
        if (sThreadLocal.get() != null) {
            throw new RuntimeException("Only one Looper may be created per thread");
        }
        sThreadLocal.set(new Looper(quitAllowed));
    }

```
这里有一个ThreadLocal对象，他不是线程而是一个线程内部的数据存储类，通过他可以在指定线程中存储数据，存储之后只有在指定进程中可以获取到存储数据，对于其他线程来说无法获取到该数据。例如Handler创建的时候回采用当前线程的Looper来构造消息循环系统，那么Handler内部获取Looper对象就是通过ThreadLocal。在日常开发中我们可能使用到的并不是非常多，但是在一切特殊情况下，通过ThreadLocal可以实现比较复杂的功能。使用场景一般来说，某些数据以线程为作用域并且不同线程有不同的副本，就可以用ThreadLocal。比如对于Handler来说，它需要获取当前线程的Looper，很显然Looper的作用域就是线程并且不同线程具有不同的Looper，这个时候通过ThreadLocal就可以轻松实现Looper在线程中的存取，如果不采用ThreadLocal，那么系统就必须提供一个全局的哈希表供Handler查找指定线程的Looper，这样一来就必须提供一个类似于LooperManager的类了，但是系统并没有这么做而是选择了ThreadLocal，这就是ThreadLocal的好处。看一个例子，如下所示
```
mBooleanThreadLocal.set(true);
Log.d(TAG, "[Thread#main]mBooleanThreadLocal=" + mBooleanThreadLocal.get());
 
new Thread("Thread#1") {
	@Override
	public void run() {
		mBooleanThreadLocal.set(false);
		Log.d(TAG, "[Thread#1]mBooleanThreadLocal=" + mBooleanThreadLocal.get());
	};
}.start();
 
new Thread("Thread#2") {
	@Override
	public void run() {
		Log.d(TAG, "[Thread#2]mBooleanThreadLocal=" + mBooleanThreadLocal.get());
	};
}.start();

```
代码中，在主线程中设置mBooleanThreadLocal的值为true，在子线程1中设置mBooleanThreadLocal的值为false，在子线程2中不设置mBooleanThreadLocal的值，然后分别在3个线程中通过get方法去mBooleanThreadLocal的值，根据前面对ThreadLocal的描述，这个时候，主线程中应该是true，子线程1中应该是false，而子线程2中由于没有设置值，所以应该是null。

回到我们的prepare方法中，我们在sThreadLocal.set()方法中创建了一个Looper对象。然后在prepareMainLooper()方法中我们将当前线程中的Looper对象赋值给了sMainLooper对象，这个对象是主线程的Looper对象。到目前为止我们的主线程中Looper对象已经初始化完成了。有同学可能会说：“什么？你啥也没说啊，这就结束了？”额，好吧，确实没说完。因为刚才我们说在sThreadLocal.set()方法中创建了一个Looper对象，但是这个Looper对象是如何创建的，我们还没有说呢。我们来看一下，如何创建Looper对象。
```
    private Looper(boolean quitAllowed) {
        mQueue = new MessageQueue(quitAllowed);
        mThread = Thread.currentThread();
    }

```
在Looper的构造方法中，我们就执行了两句话，第一句话是创建了一个MessageQueue对象，这是一个有单链表实现的队列，第二句话是创建了一个mThread对象，而这个mThread对象就是当前对象。我们需要注意的是在创建MessageQueue时，我们完成了一些初始化操作，不过这个对于我们现在来说还用不上，暂时先不考虑
```
    MessageQueue(boolean quitAllowed) {
        mQuitAllowed = quitAllowed;
        mPtr = nativeInit();
    }
// 其中nativeInit()方法是本地方法
```
当目前为止我们的Looper.prepareMainLooper()就算分析完了，我们这里主要做的就是创建一个Looper对象，创建一个MessageQueue对象，这两个对象在当前线程中是唯一的。

在AactivityThread中第二句话Looper.loop()方法是我们的重点，我们一起来看一下，我会在代码中添加注释
```
    /**
     * Return the Looper object associated with the current thread.  Returns
     * null if the calling thread is not associated with a Looper.
     */
    public static @Nullable Looper myLooper() {
        return sThreadLocal.get();
    }

    /**
     * Run the message queue in this thread. Be sure to call
     * {@link #quit()} to end the loop.
     * 在当前线程中运行MessageQueue，当停止当前loop对象时必须调用quit方法
     */
    public static void loop() {
        // 获取当前线程的Looper对象
        final Looper me = myLooper();
        if (me == null) {
            // 如果当前线程Looper对象为空，则表示没有初始化Looper对象，抛出异常
            throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
        }
        // 将当前线程中的Looper对象的MessageQueue对象拿到
        final MessageQueue queue = me.mQueue;

        // Make sure the identity of this thread is that of the local process,
        // and keep track of what that identity token actually is.
        Binder.clearCallingIdentity();
        final long ident = Binder.clearCallingIdentity();

        // Allow overriding a threshold with a system prop. e.g.
        // adb shell 'setprop log.looper.1000.main.slow 1 && stop && start'
        final int thresholdOverride =
                SystemProperties.getInt("log.looper."
                        + Process.myUid() + "."
                        + Thread.currentThread().getName()
                        + ".slow", 0);

        boolean slowDeliveryDetected = false;

        // 开启死循环换检测(其实就是Looper循环从MessageQueue中获取Message对象)
        for (;;) {
            // 获取MessageQueue中队列的第一个数据
            Message msg = queue.next(); // might block
            if (msg == null) {
                // 这里如果获取到的msg==null，代表Looper已经执行了quit方法，所有的操作都必须停止，MessageQueue中没有数据，但是Message不是null,这里为什么不是null，等一会在说到延迟消息的时候会说到 todo
                // No message indicates that the message queue is quitting.
                return;
            }

            // This must be in a local variable, in case a UI event sets the logger
            final Printer logging = me.mLogging;
            if (logging != null) {
                logging.println(">>>>> Dispatching to " + msg.target + " " +
                        msg.callback + ": " + msg.what);
            }
            // Make sure the observer won't change while processing a transaction.
            final Observer observer = sObserver;

            final long traceTag = me.mTraceTag;
            long slowDispatchThresholdMs = me.mSlowDispatchThresholdMs;
            long slowDeliveryThresholdMs = me.mSlowDeliveryThresholdMs;
            if (thresholdOverride > 0) {
                slowDispatchThresholdMs = thresholdOverride;
                slowDeliveryThresholdMs = thresholdOverride;
            }
            final boolean logSlowDelivery = (slowDeliveryThresholdMs > 0) && (msg.when > 0);
            final boolean logSlowDispatch = (slowDispatchThresholdMs > 0);

            final boolean needStartTime = logSlowDelivery || logSlowDispatch;
            final boolean needEndTime = logSlowDispatch;

            if (traceTag != 0 && Trace.isTagEnabled(traceTag)) {
                Trace.traceBegin(traceTag, msg.target.getTraceName(msg));
            }

            final long dispatchStart = needStartTime ? SystemClock.uptimeMillis() : 0;
            final long dispatchEnd;
            Object token = null;
            if (observer != null) {
                token = observer.messageDispatchStarting();
            }
            long origWorkSource = ThreadLocalWorkSource.setUid(msg.workSourceUid);
            try {
                // 通过msg.target调用dispatchMessage方法，实现消息的转发，这里我们可以知道target其实就是handler对象
                msg.target.dispatchMessage(msg);
                if (observer != null) {
                    observer.messageDispatched(token, msg);
                }
                dispatchEnd = needEndTime ? SystemClock.uptimeMillis() : 0;
            } catch (Exception exception) {
                if (observer != null) {
                    observer.dispatchingThrewException(token, msg, exception);
                }
                throw exception;
            } finally {
                ThreadLocalWorkSource.restore(origWorkSource);
                if (traceTag != 0) {
                    Trace.traceEnd(traceTag);
                }
            }
            if (logSlowDelivery) {
                if (slowDeliveryDetected) {
                    if ((dispatchStart - msg.when) <= 10) {
                        Slog.w(TAG, "Drained");
                        slowDeliveryDetected = false;
                    }
                } else {
                    if (showSlowLog(slowDeliveryThresholdMs, msg.when, dispatchStart, "delivery",
                            msg)) {
                        // Once we write a slow delivery log, suppress until the queue drains.
                        slowDeliveryDetected = true;
                    }
                }
            }
            if (logSlowDispatch) {
                showSlowLog(slowDispatchThresholdMs, dispatchStart, dispatchEnd, "dispatch", msg);
            }

            if (logging != null) {
                logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);
            }

            // Make sure that during the course of dispatching the
            // identity of the thread wasn't corrupted.
            final long newIdent = Binder.clearCallingIdentity();
            if (ident != newIdent) {
                Log.wtf(TAG, "Thread identity changed from 0x"
                        + Long.toHexString(ident) + " to 0x"
                        + Long.toHexString(newIdent) + " while dispatching to "
                        + msg.target.getClass().getName() + " "
                        + msg.callback + " what=" + msg.what);
            }

            msg.recycleUnchecked();
        }
    }

```

# 参考资料
