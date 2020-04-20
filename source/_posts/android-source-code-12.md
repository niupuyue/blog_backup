---
title: 源码分析(十二) AsyncTask源码分析
date: 2020-03-13 22:09:13
tags:
 - 源码分析
---
AsyncTask源码分析
<!--more-->

AsyncTask是一种轻量级的异步任务类，在线程池中执行异步任务，将执行进度和结果传递给主线程，并让主线程更新UI。

我们使用AsyncTask的方式如下所示
```
 private class DownloadFilesTask extends AsyncTask<URL, Integer, Long> {
 
     @Override
     protected void onPreExecute() {
        //准备工作，在主线程。
     }
 
     protected Long doInBackground(URL... urls) {
         //耗时操作，在子线程。
         int count = urls.length;
         long totalSize = 0;
         for (int i = 0; i < count; i++) {
             totalSize += Downloader.downloadFile(urls[i]);
             publishProgress((int) ((i / (float) count) * 100));
             // Escape early if cancel() is called
             if (isCancelled()) break;
         }
         return totalSize;
     }

     @Override
     protected void onProgressUpdate(Integer... progress) {
         //显示进度，在主线程
         setProgressPercent(progress[0]);
     }
     
     @Override
     protected void onCancelled(Float result) {
         //任务被取消会调用这个方法，不会调用 onPostExecute ，在主线程。
         showDialog("Cancelled " + result + " bytes");
     }

     @Override
     protected void onPostExecute(Long result) {
         //任务完成，没有被取消，调用这个方法，在主线程。
         showDialog("Downloaded " + result + " bytes");
     }
 }
 
 //使用
 new DownloadFilesTask().execute(url1, url2, url3);
```

在使用的时候，我们首先创建了一个AsyncTask的对象，所以我们需要先看一下构造方法
```
public AsyncTask() {
        mHandler = callbackLooper == null || callbackLooper == Looper.getMainLooper()
            ? getMainHandler()
            : new Handler(callbackLooper);

        mWorker = new WorkerRunnable<Params, Result>() {
            public Result call() throws Exception {
                mTaskInvoked.set(true);
                Result result = null;
                try {
                    Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);
                    //noinspection unchecked
                    result = doInBackground(mParams);
                    Binder.flushPendingCommands();
                } catch (Throwable tr) {
                    mCancelled.set(true);
                    throw tr;
                } finally {
                    postResult(result);
                }
                return result;
            }
        };

        mFuture = new FutureTask<Result>(mWorker) {
            @Override
            protected void done() {
                try {
                    postResultIfNotInvoked(get());
                } catch (InterruptedException e) {
                    android.util.Log.w(LOG_TAG, e);
                } catch (ExecutionException e) {
                    throw new RuntimeException("An error occurred while executing doInBackground()",
                            e.getCause());
                } catch (CancellationException e) {
                    postResultIfNotInvoked(null);
                }
            }
        };

}
```
在构造方法中我们我们创建了一个mWorker对象，这个对象是一个WorkerRunnable对象，并且WorkerRunnable类实现了Callable接口，这个接口与Runnable接口类似，不过Callable接口中的call方法(类似于run方法)有返回值，并且需要添加异常捕获。同时我们还创建了一个mFuture对象，这个对象将mWorker作为参数传递，我们Thread类。想一下我们通过Runnable创建子线程时，启动子线程是不是也是类似的方式执行的？
在创建完成对象之后，调用execute方法开始执行，而这个方法执行的操作如下所示
```
    @MainThread
    public final AsyncTask<Params, Progress, Result> execute(Params... params) {
        return executeOnExecutor(sDefaultExecutor, params);
    }

```
在execute方法中调用了AsyncTask的executeOnExecutor方法，并且传递了两个对象sDefaultExecutor和params，params是我们需要传递的参数，而sDefaultExecutor的实现如下所示
```
    /**
     * An {@link Executor} that executes tasks one at a time in serial
     * order.  This serialization is global to a particular process.
     */
    public static final Executor SERIAL_EXECUTOR = new SerialExecutor();

    @UnsupportedAppUsage
    private static volatile Executor sDefaultExecutor = SERIAL_EXECUTOR;

    private static class SerialExecutor implements Executor {
        final ArrayDeque<Runnable> mTasks = new ArrayDeque<Runnable>();
        Runnable mActive;

        public synchronized void execute(final Runnable r) {
            mTasks.offer(new Runnable() {
                public void run() {
                    try {
                        r.run();
                    } finally {
                        scheduleNext();
                    }
                }
            });
            if (mActive == null) {
                scheduleNext();
            }
        }

        protected synchronized void scheduleNext() {
            if ((mActive = mTasks.poll()) != null) {
                THREAD_POOL_EXECUTOR.execute(mActive);
            }
        }
    }
```
如果是第一次看，可能觉得代码很多，无法看懂，但是我们需要抓到关键点，就是synchhronized关键字，有了这个关键字，不管当前在AsyncTask中有多少个子线程，最终执行的时候，只能有一个在执行。其实他就是在模拟单线程。我们会发现通过SerialExector的创建，将SERIAL_EXECTUOR传递给sDefaultExecutor，而SerialExecutor是实现了Executor接口，这个接口中我们需要重写execute方法。并且在这个类中，我们创建了两个对象一个是ArrayDeque的task，另一个是Runnable的active。所有的任务都放在ArrayDeque这个队列中，SERIAL_EXECUTOR执行execute方法就会创建一个任务到队列中，当active==null时说明队列中没有任务，直接执行scheduleNext方法，从队列中poll出一个任务并且执行该任务，执行任务是由TREAD_POOL_EXECUTOR这个线程池执行的。run() 方法中又会执行传进来的那个任务 final Runnable r，执行完后，同样调用 scheduleNext(); 再去取下一个任务，如此循环，直到队列中没有任务为止。
关于THREAD_POOL_EXECUTOR的声明如下所示
```
private static final int CPU_COUNT = Runtime.getRuntime().availableProcessors();
private static final int CORE_POOL_SIZE = CPU_COUNT + 1;
private static final int MAXIMUM_POOL_SIZE = CPU_COUNT * 2 + 1;
private static final int KEEP_ALIVE = 1;
private static final BlockingQueue<Runnable> sPoolWorkQueue =
        new LinkedBlockingQueue<Runnable>(128); 
public static final Executor THREAD_POOL_EXECUTOR = new ThreadPoolExecutor(CORE_POOL_SIZE, MAXIMUM_POOL_SIZE, KEEP_ALIVE,
        TimeUnit.SECONDS, sPoolWorkQueue, sThreadFactory);
```

既然我们调用AsyncTask是通过execute方法执行的，就像我们上面所说的，最终都会调用到executeOnExecutor方法，主要代码如下
```
    @MainThread
    public final AsyncTask<Params, Progress, Result> executeOnExecutor(Executor exec,
            Params... params) {
        if (mStatus != Status.PENDING) {
            switch (mStatus) {
                case RUNNING:
                    throw new IllegalStateException("Cannot execute task:"
                            + " the task is already running.");
                case FINISHED:
                    throw new IllegalStateException("Cannot execute task:"
                            + " the task has already been executed "
                            + "(a task can be executed only once)");
            }
        }

        mStatus = Status.RUNNING;

        onPreExecute();

        mWorker.mParams = params;
        exec.execute(mFuture);

        return this;
    }

```
在这个方法中我们首先判断了一下当前的状态mStatus，如果状态是RUNNING或者FINISHED都会抛出异常，然后将状态改成RUNNING的状态，然后在执行onPreExecute方法，这个方法在当前类中是空实现，如果我们重写了该方法，则可以执行一些初始化操作，接着讲params传递给mWorker中的mParams对象，然后执行sDefaultExecutor的execute方法，将mFuture包装成任务放在队列ArrayDeque中，然后执行mFuture中的run方法。
既然最后执行的是Future中的run方法，我们看一下Future是如何实现的，他的实现类是FetureTask
```
public class FutureTask<V> implements RunnableFuture<V> {
...
    public void run() {
        if (state != NEW ||
            !U.compareAndSwapObject(this, RUNNER, null, Thread.currentThread()))
            return;
        try {
            Callable<V> c = callable;
            if (c != null && state == NEW) {
                V result;
                boolean ran;
                try {
                    result = c.call();
                    ran = true;
                } catch (Throwable ex) {
                    result = null;
                    ran = false;
                    setException(ex);
                }
                if (ran)
                    set(result);
            }
        } finally {
            // runner must be non-null until state is settled to
            // prevent concurrent calls to run()
            runner = null;
            // state must be re-read after nulling runner to prevent
            // leaked interrupts
            int s = state;
            if (s >= INTERRUPTING)
                handlePossibleCancellationInterrupt(s);
        }
    }
    
    protected void set(V v) {
        if (U.compareAndSwapInt(this, STATE, NEW, COMPLETING)) {
            outcome = v;
            U.putOrderedInt(this, STATE, NORMAL); // final state
            finishCompletion();
        }
    }
    
    private void finishCompletion() {
        // assert state > COMPLETING;
        for (WaitNode q; (q = waiters) != null;) {
            if (U.compareAndSwapObject(this, WAITERS, q, null)) {
                for (;;) {
                    Thread t = q.thread;
                    if (t != null) {
                        q.thread = null;
                        LockSupport.unpark(t);
                    }
                    WaitNode next = q.next;
                    if (next == null)
                        break;
                    q.next = null; // unlink to help gc
                    q = next;
                }
                break;
            }
        }
    
        done();
    
        callable = null;        // to reduce footprint
    }
...
}
```
构造方法中我们传递了一个Callable对象，也就是我们的AsyncTask的构造方法中传递的mWorker对象，我们通过<code>Callable<V> c = callable; result = c.call();</code>的方式调用了mWorker，然后调用set方法将结果赋值给outcome，并且执行finishComplete方法，最后调用done方法。done方法中调用postResultIfNotInvoked方法保证任务能够被执行，并保证执行postResult方法把结果返回给主线程。
```
private void postResultIfNotInvoked(Result result) {
    final boolean wasTaskInvoked = mTaskInvoked.get();
    if (!wasTaskInvoked) {
        postResult(result);
    }
}
```
而在mWorker的创建时，我们的call方法里的代码
```
        mWorker = new WorkerRunnable<Params, Result>() {
            public Result call() throws Exception {
                mTaskInvoked.set(true);
                Result result = null;
                try {
                    Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);
                    //noinspection unchecked
                    result = doInBackground(mParams);
                    Binder.flushPendingCommands();
                } catch (Throwable tr) {
                    mCancelled.set(true);
                    throw tr;
                } finally {
                    postResult(result);
                }
                return result;
            }
        };
// 这里就是AsyncTask的构造方法
```
首先设置任务被调度的标识为true，设置进程优先级为THREAD_PRIORITY_BACKGROUND，紧接着在调用doInBackGround方法，斌能够且将结果赋值给result，通过postResult方法，将结果返回。
```
    private Result postResult(Result result) {
        @SuppressWarnings("unchecked")
        Message message = getHandler().obtainMessage(MESSAGE_POST_RESULT,
                new AsyncTaskResult<Result>(this, result));
        message.sendToTarget();
        return result;
    }

```
在这个方法中我们创建了一个Message，并且将MESSAAGE_POST_RESULT作为标志设置到Message中，并且交个handler处理。
```
    private static Handler getMainHandler() {
        synchronized (AsyncTask.class) {
            if (sHandler == null) {
                sHandler = new InternalHandler(Looper.getMainLooper());
            }
            return sHandler;
        }
    }

    private Handler getHandler() {
        return mHandler;
    }

```
这里的代码调用其实是在AsyncTask的构造方法中已经创建完了，最后就是InternalHandler对象
```
    private void finish(Result result) {
        if (isCancelled()) {
            onCancelled(result);
        } else {
            onPostExecute(result);
        }
        mStatus = Status.FINISHED;
    }

    private static class InternalHandler extends Handler {
        public InternalHandler(Looper looper) {
            super(looper);
        }

        @SuppressWarnings({"unchecked", "RawUseOfParameterizedType"})
        @Override
        public void handleMessage(Message msg) {
            AsyncTaskResult<?> result = (AsyncTaskResult<?>) msg.obj;
            switch (msg.what) {
                case MESSAGE_POST_RESULT:
                    // There is only one result
                    result.mTask.finish(result.mData[0]);
                    break;
                case MESSAGE_POST_PROGRESS:
                    result.mTask.onProgressUpdate(result.mData);
                    break;
            }
        }
    }

```
handleMessage 方法中处理收到的消息，如果消息是 MESSAGE_POST_RESULT，调用 finish 方法。 finish 方法中根据任务是否被取消，来执行不同的方法，如果取消则执行 onCancelled(result);，没有取消执行 onPostExecute(result);，此时 AsyncTask 的第四步骤就走完了。整个任务执行完毕。可以看到 onCancelled 和 onPostExecute 只会执行一个。
如果是 MESSAGE_POST_PROGRESS 的消息，则执行 onProgressUpdate(result.mData) 方法,这个步骤的消息是如下
```
@WorkerThread
protected final void publishProgress(Progress... values) {
    if (!isCancelled()) {
        getHandler().obtainMessage(MESSAGE_POST_PROGRESS,
                new AsyncTaskResult<Progress>(this, values)).sendToTarget();
    }
}
```
我们可以在工作线程中调用这个方法，也就是在 doInBackground 中使用。如果任务没有取消，就会创建一个 what 是 MESSAGE_POST_PROGRESS，将进度值封装成一个 AsyncTaskResult 对象做为 obj 的 Message 给 Handler 处理.
最后就是cancel方法，这个方法中调用了mFuture中的cancel方法，如果有正在执行的任务则会调用interrupt并且调用finishCompletion方法
```
public final boolean cancel(boolean mayInterruptIfRunning) {
    mCancelled.set(true);
    return mFuture.cancel(mayInterruptIfRunning);
}
```

# 参考资料
[AsyncTask源码解析](https://www.jianshu.com/p/ff8d9d1ba12b)