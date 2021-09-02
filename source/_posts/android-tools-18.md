---
title: android奇技淫巧 18 Android多种方法实现倒计时功能
date: 2019-09-19 21:55:10
tags:
  - android
---

最近在写一个自己练习的项目，需要实现广告显示三秒钟然后关闭页面的功能，仔细想了一下如何实现一个倒计时功能

<!--more-->

因为很多地方都要使用到handler对象，所以我这里直接声明一个Handler 的弱引用

```
 private static class LooperHandler extends Handler {
        WeakReference<Activity> weakReference;

        public LooperHandler(Activity activity) {
            weakReference = new WeakReference<>(activity);
        }

        @Override
        public void handleMessage(@NonNull Message message) {
            super.handleMessage(message);
            MainActivity activity = (MainActivity) weakReference.get();
            switch (message.what) {
                case 0:
                    int count = (int) message.obj;
                    count--;
                    if (count >= 0) {
                        activity.msg.setText(count + "s");
                        Message m = Message.obtain();
                        m.what = 0;
                        m.obj = count;
                        activity.handler.sendMessageDelayed(m, 1000);
                    }
                    break;
                case 1:
                    activity.msg.setText(message.obj + "s");
                    break;
            }
        }
    }
```
实现起来还是比较简单的，这里我通过自己和在网上搜到的有五种方法，记录一下。

## 使用handler+postDelayed()

这个方法应该是属于比较传统的方法，直接看代码吧

```
 btn01 = findViewById(R.id.btn01);
        btn01.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                // 当点击按钮时发送Message交给handler处理，handler处理之前先去判断计数
                Message message = Message.obtain();
                message.what = 0;
                message.obj = 5;
                handler.sendMessageDelayed(message, 1000);
            }
        });
```

## Timer和TimerTask

```
btn02 = findViewById(R.id.btn02);
        btn02.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                // 通过声明一个Timer对象实现
                Timer timer = new Timer();
                final TimerTask task = new TimerTask() {
                    @Override
                    public void run() {
                        if (MainActivity.this.count >= 0) {
                            Message message = Message.obtain();
                            message.what = 1;
                            message.obj = MainActivity.this.count;
                            handler.sendMessage(message);
                            MainActivity.this.count--;
                        }
                    }
                };
                if (task != null && timer != null)
                    timer.schedule(task, 0, 1000);
            }
        });
```

## ScheduledExecutorService

```
btn03 = findViewById(R.id.btn03);
        btn03.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                // 初始化一个线程大小为1的线程池
                ScheduledExecutorService schedul = new ScheduledThreadPoolExecutor(1);
                schedul.scheduleAtFixedRate(new Runnable() {
                    @Override
                    public void run() {
                        if (MainActivity.this.count >= 0) {
                            Message m = new Message();
                            m.what = 1;
                            m.obj = MainActivity.this.count;
                            handler.sendMessage(m);
                            MainActivity.this.count--;
                        }
                    }
                }, 0, 1000, TimeUnit.MILLISECONDS);
            }
        });
```

## RxJava

```
 btn04 = findViewById(R.id.btn04);
        btn04.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                // 通过RxJava
        final long count = TOTAL_TIME / 1000;
        Observable.interval(0, 1, TimeUnit.SECONDS)//设置0延迟，每隔一秒发送一条数据
                .take((int) (count + 1)) //设置总共发送的次数
                .map(new Func1<Long, Long>() {//long 值是从小到大，倒计时需要将值倒置
                    @Override
                    public Long call(Long aLong) {
                        return count - aLong; 
                    }
                })
                .subscribeOn(Schedulers.computation())
                // doOnSubscribe 执行线程由下游逻辑最近的 subscribeOn() 控制，下游没有 subscribeOn() 则跟Subscriber 在同一线程执行
                //执行计时任务前先将 button 设置为不可点击
                .doOnSubscribe(new Action0() { 
                    @Override
                    public void call() { 
                        mStart.setEnabled(false);//在发送数据的时候设置为不能点击
                        mStart.setBackgroundColor(Color.GRAY);//背景色设为灰色
                    }
                })
                .observeOn(AndroidSchedulers.mainThread())//操作UI主要在UI线程
                .subscribe(new Subscriber<Long>() {
                    @Override
                    public void onCompleted() {
                        mTvValue.setText(getResources().getString(R.string.done));
                        mStart.setEnabled(true);
                        mStart.setBackgroundColor(Color.parseColor("#f97e7e"));
                    }

                    @Override
                    public void onError(Throwable e) {
                        e.printStackTrace();
                    }

                    @Override
                    public void onNext(Long aLong) { //接收到一条就是会操作一次UI
                        String value = String.valueOf(aLong);
                        mTvValue.setText(value);
                    }
                });
            }
        });
```

## CountDownTimer

```
btn05 = findViewById(R.id.btn05);
        btn05.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                // CountDownTimer
                CountDownTimer timer = new CountDownTimer(5000,1000) {
                    @Override
                    public void onTick(long time) {
                        MainActivity.this.msg.setText((time/1000)+"s");
                    }

                    @Override
                    public void onFinish() {

                    }
                };
                timer.start();
            }
        });
```

[demo地址](https://github.com/niupuyue/blog_demo_android/tree/master/TimeCountDemo)