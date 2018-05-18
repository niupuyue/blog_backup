---
title: android-interview-2017
date: 2017-12-24 19:16:29
tags:
  - 面试
---

2017年结尾，我打算离开广州，前往北京发展，所以又一次面临找工作。
好久都没有去考过面试方面的东西了，所以，这一次我要把之前面试的东西总结一下，同时结合别人的面试经验，为过完年之后的面试做好准备。这次总结会添加一些自己之前面试的经验，也会添加一些别人面试题的总结。
<!--more-->
首先看一张图片，Android知识图谱
![Android知识图谱](/assets/interview/interview01.png)
面试其实也就是上面的这些内容，其实还是挺多知识点的。一般中高级Android开发会问的比较深，所以要有心理准备。下面是一些重点知识，必须掌握的。如果不想在面试的时候被别人鄙视，那就一定要好好的准备

1. 基础知识：四大组件，生命周期，使用场景，如何启动
2. java基础：线程，数据结构，MVC模式，设计模式
3. 通信：网络通信(httpclient,HttpUrlConnection),Socket
4. 数据持久化：SQLite，Sharepreference，ContentProvider
5. 性能优化：布局优化，内存优化，电池优化
6. 安全：数据加密，代码混淆，WebView/JS调用，https
7. UI：动画
8. 其他：JNI，AIDl，Handler，Intent
9. 开源框架：Volley，Gilde，Rxjava
10. 拓展：Android6 7 8 新特性，kotlin语言，I/O大会

回答问题的时候要有自己的理解，不要死记硬背，因为面试官一天面试了很多人，所以，要有自己的思维
# android基础知识汇总
## Activity
### Activity的生命周期
#### Activity的定义：
Android是与用户进行交互的设备，Android提供的一个可以让用户看到，点击，滑动等操作的页面就是Activity
#### 四种状态
> running / pause / stopped / killed  
- running表示当前页面处于activity栈顶，用户可以看到，并且可以进行交互，
- pause表示activity失去焦点，或者被一个非全屏的Activity占据，或者一个透明的Activity覆盖，没有被销毁，所有的成员变量和信息都还在，但是如果内容紧张，会被回收
- stopped表示一个Activity被另外外一个Activity完全覆盖，成员变量和信息可能还在，除非内存紧张
- killed表示activity被销毁，成员变量和信息也已经不存在了

#### Activity生命周期分析
> Activity启动->onCreate()->onStart()->onResume()
- onCeate()表示Activity被创建的时候回调，生命周期第一个被调用的方法，设置一些资源，数据的预加载
- onStart()表示Activity正在被启动，已经处于可见的状态，但是还没有前台显示，用户不可以和他进行交互(可以看见，无法触摸)
- onResume()表示activity可见，并且用户可以和Activity进行交互(和onStart方法一样可以在里面执行一些初始化操作)
> 点击Home建回到主页面(Activity不可见)->onPause()->onStop()
- onPause()表示当前activity可见，但是失去焦点，不能和用户进行交互
- onStop()表示当前Activity已经被停止，不可见。一般情况下成员变量和信息没有被回收，只有当内存紧张的时候才会被回收
> 当我们再次回到原Activity时 ->onRestart()->onStart()->onResume()
- onRestart()表示一个activity由不可见状态切换到可见状态的时候才会使用的方法（例如支付宝换气）
> 退出当前Activity时 ->onPause()->onStop() ->onDestroy()
-onDestroy()表示Activity已经被销毁，数据全部丢失
> Android 数据需要保存本地在哪一个activity的生命周期中保存，这个地方其实是考察生命周期对于成员变量和信息的操作的，在onStop方法之后，数据可能会由于内容不足而被销毁掉，所以数据存储应该放在onPause方法中进行存储

横竖屏切换

[博客](http://blog.csdn.net/wenzhi20102321/article/details/68941263)

### Android任务栈

一个App中可能不止一个任务栈，某些特殊情况下，单独一个Actvity可以独享一个任务栈。还有一点就是一个Task中的Actvity可以来自不同的App，同一个App的Activity也可能不在一个Task中。

### 启动模式

1. standard：默认启动模式，在这样模式下，每启动一个Activity都会重新创建一个Activity的新实例，并且将其加入任务栈中，而且完全不会去考虑这个实例是否已存在

2. singletop：栈顶复用模式，如果有新的Activity已经存在任务栈的栈顶，那么此Activity就不会被重新创建新实例，而是复用已存在任务栈栈顶的Activity。这里重点是位于栈顶，才会被复用，如果新的Activity的实例已存在但没有位于栈顶，那么新的Activity仍然会被重建

3. singletask：站内复用模式，是一种单例模式，与singTop点类似，只不过singTop是检测栈顶元素是否有需要启动的Activity，而singTask则是检测整个栈中是否存在当前需要启动的Activity，如果存在就直接将该Activity置于栈顶，并将该Activity以上的Activity都从任务栈中移出销毁，同时也会回调onNewIntent方法

4. singleinstance： 该Activity在整个android系统内存中有且只有一个实例，而且该实例单独尊享一个Task

### scheme跳转协议
Android中的scheme是一种页面内跳转协议，是一种非常好的实现机制，通过定义自己的scheme协议，可以非常方面的跳转到app中的各个页面，通过scheme协议，服务器可以定制化的告诉app跳转那个页面，可以通过通知栏消息定制化跳转页面，也可以通过H5页面跳转页面。

## Fragment

### fragment加载到Activity中的两种方式
1. 静态加载
2. 动态加载
> 通过fragmentManager创建一个FragmentTransaction实例->调用FragmentTransaction中的add方法将fragment添加到FragmentTransaction的事务中，并且通过commit方法提交事务

### FragmentPagerAdapter和FragmentStatePagerAdapter的区别
FragmentPagerAdapter适用于页面较少的fragment 在destroyItem方法中使用的是Detach方法让fragment和activity取消绑定
FragmentStatePagerAdapter适用于页面较多的fragment  在destroyItem方法中使用的是remove方法直接移除出内存

### Fragment生命周期

> 开始创建Fragment对象 -> onAttach() -> onCreate() -> onCreateView() -> onViewCreated() -> activity中的onCreate() -> onActivityCreated() -> Activity中的onStart() -> onStart() -> Activity中的onResume() -> onResume() -> onPause() -> Activity中额onPause() -> onStop() -> Activity中的onStop() -> onDestroyView() -> onDestroy() -> onDetach() -> Activity中的onDestroy()方法

- onAttach()表示Fragment和Activity之间建立联系
- onCreate()表示Fragment在初次创建的时候调用的方法，只是用来创建Fragment，此时的Activity还没有创建完成
- onCreateView()表示Fragment首次初始化页面的时候调用的方法
- onViewCreated()表示当前面的初始化已经完成，可以在这初始化页面中的控件资源
- Activity中onCreate()表示当前Activity被渲染成功之后的方法
- onActivityCreated()表示当前activity的渲染已经完成
- Activity中onStart()表示当前activity已经可见
- onStart()表示当前Fragment已经可见
- Activity中的onResume表示当前用户可以和手机进行交互
- onResume()表示fragment可以和用户进行交互

> 到此Fragment完成初始化完毕
 
- onDestroyView()表示当前Fragment即将结束，数据将会被保存
- onDetach()表示当前Fragment和Activity解绑，Fragment不会再被使用

### Fragment通信
1. 在Fragment中调用Activity中的方法  getActivity()
2. 在Activity中调用Fragment中的方法  接口回调
3. 在Fragment中调用Fragment中的方法  findFragmentById

Activity向Fragment中传递数据：
    Fragment f = new Fragment();
    f.setArguments(new Bundle());
     之后再Fragment中接收数据Bundle b = getArguments();
Fragment向Activity传递数据：
   1. 通过getActivity()方法
   2. 通过接口回调的方式 
[博客](http://blog.csdn.net/w18756901575/article/details/51957226)

## Service

### Service是什么
作为四大组件之一，可以在后台执行一些逻辑代码，或者做一些后台操作，并且用户是不可见的。
Service是一种可以在后台执行长时间运行操作而没有用户界面的应用组件
运行在主线程中，和广播一样，不能做耗时操作

### Service和Thread的区别
- Thread是程序执行的最小单元，线程，执行一些异步操作，相对而言比较独立
- Service是一种Android运行的机制，必须依托于主线程，没有那么独立
> 如果Activity被销毁了，而这个时候依然有子线程在后台运行，该如何操作？
> 因为如果子线程实在Activity中声明的，那么当Activity被销毁的时候，子线程也会被迫停止，但是如果将子线程放在Service中，就可以很好的避免这个问题，因为Service是后台运行，并不会跟随Activity的停止而被销毁
- 实际开发中两者不同，Thread是子线程，一般会做一些耗时操作，而Service不能执行耗时操作
- 应用场景，当执行一些较长时间的操作，会使用Thread，而当需要一些比如听音乐，获取天气信息等则需要Service在后台执行

### 两种启动方式
1. startService(关闭Service两种方式stopService(Intent)或者stopSelf())
一旦开启，在没有stop的前提下，会一直在后台运行，不受Activity的生命周期的影响
> onCreate()表示首次创建服务时，系统将调用该方法执行一次性设置程序，如果Service已经运行了，则不会调用，该方法只会在创建的时候执行一次，可以认为是初始化的方法
> onStartCommand()表示每次通过startService启动一个Service，都会调用该方法
> 在onStartCommand()方法中的返回值会返回一个int类型
> onDestroy()表示Service被销毁时调用的方法，在这个方法中需要回收资源
2. bindService()
创建BindService服务器，继承自Service并在类中创建一个实现IBinder接口的实例对象并提供公共方法给客户端调用
从onBind()回调方法返回此Binder实例
在客户端中，从onServiceConnection()回调方法接受Binder，并使用提供的方法调用绑定服务
```
public class BindServiceDemo extends Service {
    private LocalBinder binder = new LocalBinder();
    private int count;
    private Thread thread;
    private boolean quite = true;
    /**
     * 创建一个binder对象，返回给Activity使用，完成数据的交互
     */
    public class LocalBinder extends Binder {
        BindServiceDemo getService(){
            return BindServiceDemo.this;
        }
    }
    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        return binder;
    }
    @Override
    public void onCreate() {
        super.onCreate();
        thread = new Thread(new Runnable() {
            @Override
            public void run() {
                while (quite){
                    try{
                        Thread.sleep(1000);
                    }catch (Exception e){
                        e.printStackTrace();
                    }
                    count++;
                }

            }
        });
        thread.start();
    }
    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        return super.onStartCommand(intent, flags, startId);
    }
    @Override
    public void onDestroy() {
        super.onDestroy();
    }
    @Override
    public boolean onUnbind(Intent intent) {
        return super.onUnbind(intent);
    }
    public int getCount(){
        return count;
    }
    public void setQuite(boolean quite){
        this.quite = quite;
    }
    public boolean getQuite(){
        return quite;
    }
}
```
Activity中代码：
```
conn = new ServiceConnection() {
            @Override
            public void onServiceConnected(ComponentName name, IBinder service) {
                BindServiceDemo.LocalBinder binder = (BindServiceDemo.LocalBinder) service;
                mService = binder.getService();
            }
            @Override
            public void onServiceDisconnected(ComponentName name) {
                mService = null;
            }
        };
        btn03 = findViewById(R.id.btn03);
        btn03.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Intent intent = new Intent(MainActivity.this,BindServiceDemo.class);
                bindService(intent,conn, Service.BIND_AUTO_CREATE);
            }
        });
        btn04 = findViewById(R.id.btn04);
        btn04.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
//                mService.setQuite(true);
            }
        });
        btn05 = findViewById(R.id.btn05);
        btn05.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Log.i("计算数字",mService.getCount()+"");
            }
        });
```

## Broadcast Receiver

[博客](https://www.jianshu.com/p/ca3d87a4cdf3)
[博客](http://blog.csdn.net/white_idiot/article/details/54862939)

### 广播
1. 广播的定义
应用程序之间传递信息的机制，类似于观察者模式
在Android中，Broadcast是一种广泛运用在应用程序之间传递信息的机制，Android中我们要发送的广播内容是一个Intent，这个Intent中可以携带我们要传送的数据。实现了不同数据之间的传输和共享，通知的作用
2. 广播的使用场景
- 同一个app具有多个进程的不同组件之间的消息通信机制
- 不同app之间的组件之间消息通信
3. 广播的种类
- 普通广播：通过context.sendBroadcast()
- 系统广播:通过context.sendOrderBroadcast()
- 本地广播:只在自身App内传播

### 实现广播-receiver
1. 静态注册：注册完成之后一直运行
2. 动态广播：跟随Activity的生命周期，在onDestroy方法中一定要清除广播，不然会造成内存泄漏

### 广播的内部实现机制
1. 自定义广播接收者BroadcastReceiver,并且复写onRecvice()方法
2. 通过Binder机制想AMS(Activity Manager Service )进行注册
3. 广播发送者通过Binder机制向AMS发送广播
4. AMS查找符合相应条件(IntentFilter/permission等)的BroadcasReceiver，将广播发送到BroadcastReceiver(一般是Activity)响应的消息循环队列中
5. 消息循环执行拿到此广播，回调BroadcastReceiver中的onReceiver()方法

### LocalBroadcastManager(本地广播的实现)
1. 使用它发送的广播只能自身app内传递，因此不用担心泄露隐私数据
2. 其他app无法对你的app发送该广播，因为我们的app是根本无法接收到非自身应用发送的广播的，因此不存在安全泄漏问题
3. 比系统全局广播更加高效
### 详解
1. LocalBroadcastManager高效的原因主要是因为他内部通过handler实现，他的sendBroadcast()犯法含义并非和我们平时所用的一样，他的sendBroadcast()方法通过handler中的发送消息实现
2. 由于它内部是通过handler实现广播发送，相比较系统广播通过Binder实现肯定更加高效，同时使用handler实现，别的应用无法向我们的应用发送该广播，而我们应用内发送的广播也不会离开我们的应用
3. LocalBroadcastManager内部协作主要是靠两个Map集合：mReceivers和mActions，当然还有一个list集合mPendingBroadcasts，这个主要是存储待接收的广播对象的

## WebView

### webview常见的一些问题
1. 在Android API level16以及之前的版本存在远程代码执行安全漏洞，该漏洞源于程序没有真正限制使用webview.addJavascripteInterface()方法，远程攻击者通过使用Java Reflection API利用该漏洞执行任意的java对象方法
2. webview在布局文件中的使用，有些时候，我们再添加webview的时候是通过动态添加的方式去实现的。例如，我们可以通过LinearLayout的addView方法将webview添加到容器中，而当退出Activity时，需要在onDestroy方法中先将当前的webView从容器中删除，然后再调用webview的removesAllViews()方法和destroy()方法，才能真正的完成销毁，并且不会出现内存泄漏问题。
3. webview的JSBridge：一端是web段，另外是app段，主要的作用是让两者可以进行交互
4. webviewClientonPageFinished表示页面加载完成之后会回调的方法，但是他会判断页面是否加载完成，而当前的页面如果发生跳转的话，这个方法就会调用多次，所以说当webview需要加载各种各样的网页，并且需要执行一些操作，应该调用WebChromeClient.onProgressChanged方法
5. 后台耗电，性能优化，当程序通过webview加载网页的时候，会开启线程，如果没有将webview完全销毁，这些线程就会在后台运行，导致应用耗电量增加。比较粗暴的解决办法在Activity的onDestroy方法中调用System.exit()方法，直接把虚拟机关闭。
6. webview硬件加速导致页面渲染问题，Android3.0，容易出现页面白块，并且出现闪烁，解决办法暂时关闭webview硬件加速

### webview内存泄漏问题
1. 独立进程，简单暴力，不过可能涉及进程间通信，webview需要依赖于Activity，webview执行的操作实在内部一个线程中的，时间Activity无法确定，Activity的生命周期和这个新线程的生命周期是不一样的，导致webview一直持有Activity引用，不能回收，(内部类持有外部类的引用，外部类无法回收)解决办法：独立进程，单独给webview进行操作，但是会有进程间通信的问题，
2. 动态添加webview，对传入webview中所使用的context使用弱引用，这里动态添加webview的方式是指在布局文件中添加一个viewgroup来放置webview，当Activity创建的时候加载进来，在Activity停止的时候remove掉
> Java的四种引用方式
> Java提供引入方式的目的是为了让程序员可以自己掌握对象的生命周期，也可以方便jvm垃圾回收
> 1. 强引用：创建一个对象并且把这个对象赋值给一个引用变量 
> 	 Object object =new Object();
>    String str ="hello";
> 强引用有引用变量指向的时候永远不会被回收，如果想中断这样的操作，可以显示的将引用赋值为null
> 2. 软引用：如果一个对象是软引用，那么只要内存足够，那么他就不会被回收。如果内存不足，则会回收这些对象的内存，只要垃圾回收机制没有回收他，他就可以被程序使用。
> 软引用可以应用到一些内存敏感的高速缓存，比如图片缓存。可以有效防止内存泄漏
>   MyObject obj = new MyObject()
>	SoftReference soft = new SoftReference(obj);
> 此时的obj对象还是强引用
> 通过给obj复制为null，变成软引用
>   obj = null;
> 3. 弱引用：弱引用对象是用来描述一些非必须对象，当jvm进行垃圾回收的时候，不管内存是否足够，都会回收弱引用对象的内存
> WeakReference<People>reference=new WeakReference<People>(new People("zhouqian",20));  
> 这里我们使用的弱引用，如果调用System.gc()，则会将对象删除，而如果我们中间引入的是一个强引用，那么就不会被回收
> 4. 虚引用： 如果一个对象与虚引用关联，则跟没有引用与之关联一样，在任何时候都可能被垃圾回收器回收。
> 要注意的是，虚引用必须和引用队列关联使用，当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会把这个虚引用加入到与之 关联的引用队列中。程序可以通过判断引用队列中是否已经加入了虚引用，来了解被引用的对象是否将要被垃圾回收。如果程序发现某个虚引用已经被加入到引用队列，那么就可以在所引用的对象的内存被回收之前采取必要的行动。

## Binder

### Linux内核基础知识
1. 进程隔离 / 虚拟地址空间
2. 系统调用
3. Binder驱动

### Binder通信机制的介绍
1. 为什么使用Binder
Android使用的是Linux内核，而Linux内核中又包含了很多进程间的通信机制(管道，socket)
- 性能问题，导致了Android使用了新的Binder这种方式完成进程间通信，如果在移动设备上广泛的使用线程间通信机制，肯定会对通信机制有较高的要求，而binder相当于传统的socket方式更加高效
- 安全问题，传统的进程间通信对通信双方的什么没有进行校验，例如socket的ip地址是由客户端手动填写的，而Binder机制在协议中就允许通信双方进行身份校验
2. Binder通信模型
- 建立一张ServiceManager的表
- 将Service注册到ServiceManger表中
- 当需要进行通信的时候，需要通过查询ServiceManager表，并且把查询到的结果返回
- 客户端拿到返回信息之后和想要通信的对象进行通信
3. Binder实例aidl


## Handler

### 什么是handler(handler发送消息接收消息都是本地线程)
Android消息机制的上层接口
通过发送和处理Message和Runnable对象来管理按相应的线程的MessageQueue
1. 可以让对应的Message和Runnable在未来的某个时间点进行相应的处理
2. 让自己想要处理的耗时操作放在子线程中，让更新UI的操作放在主线程中

### handler的使用方法
1. 通过post(runnable)  post方法的最终形式和sendMessage的方法是一样的
- 需要在成员变量中创建一个Handler，这个handler和主线程相关，自动绑定到了主线程中
- 创建一个Thread，然后再Thread中执行耗时操作
- Thread执行完耗时操作之后，创建一个Runnable对象，然后让这个Runnable对象去更新UI页面
- 将这个Runnable对象通过handler.post(runnable)的方式，将这个更新真正的运用到主线程中
2. sendMessage(message)
- 在成员变量中创建一个Handler对象，不过这里我们需要实现里面的HandMessage方法
- 创建一个子线程，然后在子线程中执行耗时操作
- 耗时操作执行完成之后，创建一个Message对象(可以通过new Message的方式，可以通过Message.obtain方式创建),通过Message对象中的what属性赋值一个标识(还有arg1，arg2，obj)，再通过handler的sendMessage(message)方法，将数据传递个Handler中，让HandMessage方法去处理
### handler机制的原理

### 由于handler引起的内存泄漏以及解决办法
由于handler对象持有Activity对象，所以当handler对象在后台执行一些耗时操作，而如果这时候关闭到Activity，那么这时候Activity是没有办法被回收的，所以为了解决这个问题，我们一般有两种方法解决这个问题，第一种是把当前的handler对象编程static变量，第二种就是在Activity的onDestroy方法中调用handler的removecallback方法(静态内部类持有外部类的匿名引用，导致外部Activity无法被释放)，第三种方法是handler内部持有外部Activity的弱引用

## AsyncTask
### 什么是AsyncTask
是Android提供的轻量级异步类。它本质上就是一个封装了线程池和handler的异步框架。尽量做一些耗时操作比较少的操作

### AsyncTask的使用方法
1. 三个参数 AsyncTask<params1,params2,params3>
- params1表示当前需要传递过来的数据类型
- params2表示当前进行请求的时候的进度
- params3表示当前请求返回的结果类型
2. 五个方法
- onPreExecute() 表示耗时操作还没有执行的时候进行一些初始化操作，在UI线程中调用，一般会显示一些进度条
- onInBackground() 表示当前需要进行的耗时操作
- publishProgress() 进度条的显示
- onPostExecute() 表示当前数据的返回
- onProgressUpdate() 表示可以动态显示进度条
    
### AsyncTask的机制原理
1. AsyncTask的本质是一个线程池，AsyncTask派生出的子类可以实现不同的异步任务，这些任务都是提交到静态的线程池中的任务
2. 线程池中的工作线程执行onInBackground(mParams)方法执行异步任务
3. 当任务状态改变之后，工作线程会向UI线程发送消息，AsyncTask内部的InternalHandler响应这些消息，并且调用相关的回调函数

### AsyncTask的注意事项
1. 内存泄漏
2. 生命周期：AsyncTask并不一定会随着Activity的生命周期的终结而结束，可能会造成崩溃
3. 结果丢失：屏幕旋转或者由于内存不足导致Activity重新创建，会导致当前数据丢失
4. 串行or并行：

## HandlerThread
### HandlerThread是什么
一般会开启一个子线程进行耗时操作，方便但是有消耗资源，多次创建和销毁线程是很消耗资源的。
handler + thread + looper

> 子线程为什么不能直接开启handler？
> handler的sendMessage和handMessage都需要一个MessageQueue来保存当前的消息，但是在子线程中，如果直接调用handler是没有办法开启Looper运行器的，所以我们需要自己去调用Looper的prepare方法和looper方法。

### HandlerThread的特点
1. HandlerThread本质上是一个线程类，他继承了Thread
2. HandlerThread有自己的内部Looper对象，可以进行Looper循环
3. 通过获取HandlerThread的looper对象传递给Handler对象，可以再HandleMessage方法中执行异步任务
4. 优点是不会造成堵塞，减少了对性能的消耗，缺点是不能同时进行多任务的处理，需要等待进行处理，但是处理效率会比较低
5. 与线程池注重并发不同。HandlerThread是一个串行队列，HandlerThread背后只有一个线程

### HandlerThread的源码分析

## IntentService

### IntentService是什么
优先级高于Service，并且继承Service
IntentService是继承和处理异步请求的类，在IntentService内有一个工作线程来处理耗时操作，启动IntentService的方式和启动传统的Service是一样的，同事当任务执行完成之后，IntentService会自动停止，而不需要我们手动控制。另外，可以启动IntentService多次，而每一个耗时操作会以工作队列的方式在IntentService的onHandleIntent回调方法中执行，并且每次只会执行一个工作线程，执行完成之后在执行第二个
本质上就是一个Service，继承自Service并且本身就是一个抽象类
他内部通过HandlerThread和Handler实现异步操作

### 如何使用IntentService
创建IntentService时，只需要实现onHandleIntent和构造方法，onHandleIntent为异步方法，可移植性耗时操作

## View的绘制机制
### view树的绘制流程
重要方法：
1. 构造方法：构造函数是View的入口，可以用于初始化一些内容，获取自定义的属性
2. onMeasure():测量View大小。View的大小不仅受自身影响，同时也会受到父控件的影响，为了我们的空间能够更好的适应各种情况，一般会自己进行测量。测量View大小使用的事onMeasure方法，我们可以从onMeasure的两个参数中取出宽高等方法。
```
@Override
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    //这里是获取值，如果是修改至，则需要使用setMeasuredDimension(widthSize,heightSize)
    int widthSize = MeasureSpec.getSize(widthMeasureSpec);//取出宽度的测量值 
    int widthMode = MeasureSpec.getMode(widthMeasureSpec);//取出宽度的测量模式
    
    int heightSize = MeasureSpec.getSize(heightMeasureSpec);
    int heightMode = MeasureSpec.getMode(heightMeasureSpec);
}
```
android的测量模式一共有三种，分别是unspecified(默认值，父控件没有给子view任何限制，子View可以设置为任意值),exactly(表示父控件已经确定了子View的带下),at_most(表示子view具体大小没有尺寸限制，但存在上线，一般是父view的大小)
3. layout:根据测量出的尺寸来放置子view的位置，相对于父view而言
4. onSizeChanged()：确定view大小，如果视图的大小发生变化，则会调用这个方法
5. draw：绘制视图
- invalidate():该方法会引起view的重绘，一般会是在内部调用，比如setVisillity()或者需要刷新页面，由UI线程调用该方法。总结来说，当子view调用了invalidate()方法，会为当前view添加一个标记，同时不断向父容器请求刷新，父容器通过计算，得出自己需要重绘的区域，直到传递到ViewRootImpl，最终触发performTraversals，进行开始view树的重绘(只需要绘制需要重绘的视图)
- requestLayout():当布局发生变化的时候，如方向或者大小动调用方法。那么这时候view树会进行重新一次的测量，定位，绘制的工作。总结来说：子view调用了requestLayout()方法，会标记当前view和父容器，同时逐层向上传递，知道ViewRootImpl处理该事件，viewRootImpl会调动三大流程，也就从measure开始依次对有标记的View和子view进行测量，定位，绘制

## 事件分发机制
### 为什么会有事件分发机制
Android中view是树形结构，view可以重叠在一起，当我们点击的地方有多个view都可以响应的时候，这个点击事件应该传递给那个view处理，而引入事件分发机制
> Window：Android所有视图的超类，所有的操作，布局等都是要在这里实现的，但是由于他是一个抽象类，所以会交个他唯一的一个实现类PhoneWindow实现
> PhoneWidow：Window的唯一实现类，但是他也不会进行具体的操作，他把所有的操作权限都交给了他的内部类DecorView。

### 三个重要的事件分发的方法
1. dispathTouchEvent:决定了TouchEvent是由自己处理还是分发个子View处理，
2. onInterceptTouchEvent：拦截事件，父控件会分发事件给子控件，如果子控件需要自己处理，那么就会使用拦截，不让当前的操作在向子控件传递
3. onTouchEvent：处理传递过来的手势事件

### 事件分发机制流程
Activity -> PhoneWindow -> decorView -> viewGroup -> ... -> view
如果view没有处理事件，则会再回传给Activity
> Activity的dispatchTouchEvent开始 -> PhoneWindow的superDispatchTouchEvent -> DecorView的dispatchTouchEvent -> rootView的dispatchTouchEvent -> rootView的onInterceptTouchEvent -> ViewGroup的dispatchTouchEvent -> viewGroup的onInterceptTouchEvent -> view1的dispatchTouchEvent -> view1的onTouchEvent

如果view没有处理，回传

> view1的onTouchEvent -> View1的dispatchTouchEvent -> viewGroup的dispatchTouchEvent -> rootView的dispatchTouchEvent -> decorView的dispatchTouchEvent -> PhoneWindow的superDispatchTouchEvent -> activity的dispatchTouchEvent

## ListView
### listview的定义
ListView是一个能用数据集合以动态滚动的方式展示到用户界面的view

### ListView的适配器模式
adapter通过根据数据绘制每一个数据的view视图，交给listview管理。

### ListView的RecycleBin机制

### listview的性能优化
1. 利用convertview重用  和 使用viewHolder
2. 三级缓存 / 监听滑动事件 (图片加载的时候需要利用缓存，设置监听事件，当滑动停止的时候再去进行数据加载等耗时操作)
3. item的布局当中，尽量避免半透明的元素，因为半透明的绘制要比不透明更耗时
4. 开启硬件加速

## RecycleView
[recycleView](http://blog.csdn.net/qq_37293612/article/details/54915250)


## ANR问题
### 什么是ANR
Application not Responding
就是应用程序无响应的对话框

### ANR产生的主要原因
应用程序的响应主要是由ActivityManager和WindowManager等系统服务来监视的，如果监视到Activity5秒或者广播10秒没有响应，则会产生ANR异常
1. 主线程被IO操作(Android4.0之后不允许在主线程中执行网络IO操作)造成阻塞
2. 主线程中存在耗时计算等操作
> Android哪些操作是放在主线程中的？
> 1. Activity所有生命周期的回调是放在主线程中的
> 2. Service默认执行也是在主线程中的
> 3. BroadcastReceiver的onReceive回调也是在主线程中执行的
> 4. 没有子线程looper的Handler的sendHandMessage，post(Runnable)是在主线程运行
> 5. AsyncTask的回调方法中除了onInBackground方法都会在主线程中执行

### 如何解决ANR
- 使用AsyncTask执行耗时IO操作
- 使用Thread或者HandlerThread提高优先级
- 使用Handler来处理工作线程的耗时任务
- 在Activity的onCreate和onResume回调方法中尽量避免执行耗时操作

## OOM问题
### 什么是OOM
当前占用的内存加上我们申请的内存资源超过了Dalvik虚拟机最大的内存限制，就会抛出out of memory异常
Android手机会为每一个app申请一个Dalvik虚拟机，也会为他分配一个内存空间，但是每一个内存空间都是有限的，一般会出现bitmap加载问题(大图)

### 容易出现混淆的问题
1. 内存溢出：OOM
2. 内存抖动：在短时间内大量的对象被创建，然后又被马上释放， 这样瞬间产生的对象会严重占用内存区域
3. 内存泄漏：进程中的某些对象，没有被引用或使用，但是却可以直接或者间接的引用其他还没有被回收的对象，导致GC无法起作用

### 如何解决OOM
1. 有关bitmap的优化
- 图片显示：在需要显示缩略图的时候，不要通过网络请求加载大图，只有在需要的时候再去加载大图
- 及时释放内存：C区域的内存
- 图片压缩：在将图片加载到内存之前，需要通过计算得出一个比较合适的尺寸比率，避免不需要的大图加载
- inBitmap属性：
- 捕获异常：Error属性
2. 其他方法
- ListView：convertview  /  lru机制缓存bitmap(最近最少使用)
- 避免在onDraw方法中执行对象的创建
- 谨慎使用多进程

## Bitmap
### recycle()
bitmap是存储在native内存和java内存中的，当他被对象回收的时候，需要分为两部分进行回收

## UI卡顿
### UI卡顿的原理
- 每隔16毫秒刷新UI页面，所以在16毫秒内完成渲染工作
- overdraw：重复绘制，UI不居中出现了大量重叠的部分或者是一些非必要重叠背景
- 动画执行次数过多

### UI卡顿的原因分析
1. 人为的在UI线程中做了一些轻微耗时操作，导致UI线程卡顿
2. 布局layout过于复杂，无法在16毫秒内完成渲染
3. 同一时间执行动画次数过多，导致CPU或者GPU负载过重
4. View的过度绘制，导致某些像素在同一帧的时间内被绘制了多次，从而使CPU或者GPU负载过重
5. View频繁触发measure，layout，导致measure，layout累计耗时过多及整个View的频繁的重复渲染
6. 内存频繁GC过多，导致暂时阻塞渲染操作
7. 冗余资源或者逻辑等导致加载或者执行缓慢
8. ANR

### UI卡顿的总结
1. 布局优化：可以使用include等，尽量不存在冗余嵌套或者过于复杂的布局，尽量使用gone来实现隐藏
2. 列表和Adapter的优化：
3. 背景和图片等的内存分配优化
4. 避免ANR

## 内存泄漏

### java内存泄漏基础知识

简单来说就是该被回收的对象被另外一个对象所持有，应该回收却没有被回收

1. java内存的分配策略
- 静态存储区:也叫方法区，在这里面放置一些静态数据，全局变量的该数据。在这块内存当中在程序编译的时候已经分配好了，并且在静态存储区中存储的变量会在整个程勋运行期间一直存在
- 栈区：方法体中的局部变量会在栈区中创建内存空间，并在方法执行结束之后，这些变量所持有的内存会被自动释放，因为栈内存分配运算内置于处理器当中，所以处理效率很高，但是存储空间有限
- 堆区：动态内存分配，通常是new对象出来的内存或者数组，这部分内存在不使用的时候由java的垃圾回收机制进行管理

2. java是如何管理内存的
本质上是资源的分配和释放问题

3. java中的内存泄漏
内存泄漏指的是无用对象(不再使用的对象)持续占有内存或者无用对象内存得不到及时的释放，从而造成内存空间的浪费

2. Android内存泄漏
- 单例：
- 匿名内部类
- handler
- 避免使用static变量
- 未关闭资源而造成的内存泄漏
- AsyncTask造成内存泄漏(非静态内部类持有静态内部类的引用造成内存泄漏),在onDestroy方法中取消AsyncTask任务，调用cancle的回调方法
> 补充Bitmap需要执行recycel()才能回收C那部分的资源

## 内存管理
### 内存管理机制
从操作系统的角度来说，内存其实就是一系列的存储空间，可以被系统调用的
操作系统会为每一个进程分配内存空间
1. 分配机制：操作系统会为每一个进程合理的分配一个内存空间，让进程可以正常的运行
2. 回收机制：当系统资源不足的时候，有一个可以回收资源并且再分配的机制，就是回收机制

### Android中的内存管理机制

1. 分配机制：Android系统在为进程分配内存的时候，采用的是弹性的分配机制(一开始系统不会为app分配过多的资源，而是给每一个app提供一个较小的资源，这个大小是根据应用的实际尺寸决定的，随着app的运行，Android又会为每一个app分配的新的内存)，从而让Android手机可以运行更多的进程，从而使app再次启动的时候就不会再启动一个新的线程，而且也会让手机运行速度更快
2. 回收机制：特点就是最大限度的使用资源，在内存中保存尽量多的数据

### 内存管理机制的特点
1. 更少的占用内存
2. 在合适的时候，合理的释放系统资源
3. 在系统内存紧张的情况下，能释放大部分不重要的资源，来为Android系统提供可用的资源
4. 能够在合理的特殊生命周期中，保存或者还原数据，以至于系统能够正确的恢复该应用

### Android内存优化的方法
1. 当Service使用完成之后，尽量停止他(可以使用IntentService替换Service，可以执行耗时操作，并且在执行完成之后自动关闭)
2. 在UI不可见的时候，释放掉一些只有UI才会使用的资源(会根据android的onTrimMemory通知App回收UI资源)
3. 在系统资源紧张的时候，尽可能多的释放掉一些 非重要资源
4. 避免乱用bitmap导致的内存浪费(我们一般会根据屏幕分辨率来压缩图片)，在使用完成bitmap之后要调用recycle方法释放掉C区域的内存，也可以通过使用软引用来引用一个bitmap，那么这时候就会在系统资源紧张的时候自动释放掉这些资源
5. 使用对内存优化过的数据容器，使用google推荐使用的数据集合比如SparseArray等，而且要避免使用枚举，因为枚举会占用大量的内存资源。
6. 避免使用注入依赖框架
7. 使用ZIP对其的APK(ZIP是一个工具，会压缩资源)
8. 使用多进程，把消耗内存过度的模块或者需要长期在后台运行的模块，放在其他进程中(例如后台定位，通知,webview也是要开启多进程打开URL)

## 冷启动优化

### 什么是冷启动
1. 定义：冷启动就是在启动应用之前，系统中没有该应用的任何进程信息(包括Activity，Service等)
2. 冷启动和热启动的区别
- 定义：用户使用返回键退出该应用，然后又立刻启动该应用
- 启动特点：因为冷启动系统回创建一个新的进程给到应用，所以会先初始化Application类，在创建和初始化MainActivity类，然后再通过UI的绘制显示到屏幕上；而热启动会从已有的进程来启动，不会执行Application，而是直接走MainActivity，进行测量绘制
- 冷启动时间的计算
这个时间值从应用启动(创建进程)开始计算，到完成视图第一次绘制(Activity内容对用户可见)为止

### 冷启动的流程
当点击图标的时候，系统会从Zygote进程fork一个新的进程，创建和初始化Application类，创建MainActivity类，Inflate布局，当onCreate方法，onStart方法，onResume方法都走完之后，contentView的measure/layout/draw显示在界面上
> 总结：Application的构造方法->attachBaseContext()->onCreate()->Activity中的构造方法->activity的onCreate方法->配置主题中的背景属性->activity的onStart方法->Activity中的onResume方法->测量布局绘制显示在屏幕上

### 对冷启动时间的优化(懒加载)
1. 减少onCreate方法中的工作量
2. 不要让Application执行业务操作
3. 不要在Application中执行耗时操作
4. 不要以静态变量的方式在Application中保存数据
5. 尽量减少布局的深度

## MVC  MVP  MVVM模式

[编程模式](http://blog.csdn.net/mynameishuangshuai/article/details/52808032)

### MVC模式
1. 定义
- 视图层（View）：一般采用XML文件进行界面的描述，使用的时候可以非常方便的引入。 
- 控制层（Controller）：Android的控制层通常是在Acitvity中实现。 
- 模型层（Model）：对数据库的操作、对网络等的操作都应该在Model里面处理，当然对业务计算等操作也是必须放在的该层的
2. 特点
- 耦合性低： 这个模块代码之间的关联程度不是非常高，可以拆检各种业务模块，利用该框架可以很好的将View层和Model层分离，减少模块之间的代码影响
- 可扩展性好
- 模块职责划分明确

3. 总结
- 利用MVC的设计模式，使项目有了很好的扩展性和维护性
- Controller控制器是一个中间桥梁
- 应用场景：使用于一些比较大型的项目，项目结构比较复杂，页面结构比较多

### MVP模式
1. 定义
和之前的MVC模式不同之处在于将Activity视为View层，而Presenter负责处理View和Model的交互
- M:依然是业务逻辑和实体模型
- V:对应的是Activity，负责的是View的绘制和用户的交互
- P:负责完成View和Model层的交互


## android插件化

## Android各个版本新特性
### Android 5.0
- 通知
- 声音和震动
- 媒体播放
- 浮动显示
- 锁定屏幕的可见性
- Material Desgin
- 支持Android NDK 64位
- 只能显示绑定到服务，不能隐式绑定到服务

### Android 6.0
- 运行时请求权限
- 低电耗模式和应用待机模式
- 取消支持Apache HTTP客户端
- Boring SSL
- 硬件标识符访问权
- 通知
- 音频管理器变更
- 支持文本选择
- Android 密匙库不在支持DSA
- WLAN和网络连接变更
- 相机服务变更
- APK验证
- USB连接

### Android 7.0
- 电池和内存
- 低耗电模式
- Project Svelte：后台优化
- 权限变更
- 系统权限变更
- 在应用间文件共享权限控制
- 多窗口控制
- 支持通知栏快捷回复
- 支持VR
- 引入JIT编译器
- 画中画
- App快捷菜单

### Android 7.1
- 加入重启按钮
- App圆形图标
- 添加新的Emoji

### Android 8.0
- 优化通知
- 通知渠道
- 通知标志
- 休眠
- 通知超时
- 通知设置
- 通知清除
- 自动填充框架
- 画中画模式：清单中Activity设置android:supportsPictureInPicture
- 可下载字体：FontRequest
- XML 中的字体
- 自动调整 TextView 的大小
- 自适应图标
- 颜色管理
- WebView API
- 多显示器支持
- 统一的布局外边距和-
- 指针捕获
- 应用类别
- Android TV 启动器
- AnimatorSet
- 新的 StrictMode 检测程序

### DVM和JVM的区别
1. java虚拟机运行的是Java字节码，Dalvik虚拟机运行的是Dalvik字节码；传统的Java程序经过编译，生成Java字节码保存在class文件中，java虚拟机通过解码class文件中的内容来运行程序。而Dalvik虚拟机运行的是Dalvik字节码，所有的Dalvik字节码由Java字节码转换而来，并被打包到一个DEX(Dalvik Executable)可执行文件中Dalvik虚拟机通过解释Dex文件来执行这些字节码。
2. Dalvik可执行文件体积更小。SDK中有一个叫dx的工具负责将java字节码转换为Dalvik字节码。
3. java虚拟机与Dalvik虚拟机架构不同。java虚拟机基于栈架构。程序在运行时虚拟机需要频繁的从栈上读取或写入数据。这过程需要更多的指令分派与内存访问次数，会耗费不少CPU时间，对于像手机设备资源有限的设备来说，这是相当大的一笔开销。Dalvik虚拟机基于寄存器架构，数据的访问通过寄存器间直接传递，这样的访问方式比基于栈方式快的多.

### Android打包流程
详细的build流程
![官方](/assets/interview/interview03.png)
具体打包步骤

1. 打包资源文件，生成R.java文件
打包资源的工具是aapt，在这个过程中Android清单配置文件和布局文件都会进行编译，生成响应的R.java文件，但是清单配置文件会被打包成二进制文件。存放在res下的资源，在App打包之前大多数会被编译，会变成二进制文件，并会为每个该类文件赋予一个resource id。对于该类资源的访问，应用层代码则是通过resource id进行访问的。Android应用在编译过程中aapt工具会对资源文件进行编译，并生成一个resource.arsc文件，resource.arsc文件相当于一个文件索引表，记录了很多跟资源相关的信息。

2. 处理aidl文件，生成响应的Java文件
这一过程中使用到的工具是aidl（Android Interface Definition Language），即Android接口描述语言
aidl工具解析接口定义文件然后生成相应的Java代码接口供程序调用。如果在项目没有使用到aidl文件，则可以跳过这一步。

3. 编译项目源文件，生成响应的.class文件
项目中所有的Java代码，包括R.java和.aidl文件，都会变Java编译器（javac）编译成.class文件，生成的class文件位于工程中的bin/classes目录下。

4. 转换所有的class文件，生成classes.dex文件
dx工具生成可供Android系统Dalvik虚拟机执行的classes.dex文件，
任何第三方的libraries和.class文件都会被转换成.dex文件。dx工具的主要工作是将Java字节码转成成Dalvik字节码、压缩常量池、消除冗余信息等

5. 打包成APK文件
所有没有编译的资源，如images、assets目录下资源（该类文件是一些原始文件，APP打包时并不会对其进行编译，而是直接打包到APP中，对于这一类资源文件的访问，应用层代码需要通过文件名对其进行访问）；编译过的资源和.dex文件都会被apkbuilder工具打包到最终的.apk文件中。
打包的工具apkbuilder位于 android-sdk/tools目录下。apkbuilder为一个脚本文件，实际调用的是（E:\Documents\Android\sdk\tools\lib）文件中的com.android.sdklib.build.ApkbuilderMain类

6. 对APK进行签名
一旦APK文件生成，它必须被签名才能被安装在设备上。
在开发过程中，主要用到的就是两种签名的keystore。一种是用于调试的debug.keystore，它主要用于调试，在Eclipse或者Android Studio中直接run以后跑在手机上的就是使用的debug.keystore。
另一种就是用于发布正式版本的keystore。

7. 对签名后的APK进行对其处理
如果你发布的apk是正式版的话，就必须对APK进行对齐处理，用到的工具是zipalign（E:\Documents\Android\sdk\build-tools\25.0.0\zipalign.exe）
对齐的主要过程是将APK包中所有的资源文件距离文件起始偏移为4字节整数倍，这样通过内存映射访问apk文件时的速度会更快。对齐的作用就是减少运行时内存的使用。


#​ 另一份Android面试题

[android面试](http://blog.csdn.net/huangqili1314/article/details/72792682)
[java面试](http://blog.csdn.net/huangqili1314/article/details/79448187)


# 面试题
## Java方面
1. 河南郑州逸休联盟
> 题目：
> 字符串
> 1.  A1，B1，C1，C2，B2，A2，D1，D2，D3
> 2.  E1，E2，E3，F1，F2，F3 
> 字符串类型特特点：字符串里面的每个TOKEN都是 字母+数字，需要对字符串做做一个函数处理得到下面结果
> 题中只给了两个符合上面规则的字符串，符合上面规则的字符串是无限的。
> 字符串1结果：
> A -->{A1，A2}
> B -->{B1,B2}
> C -->{C1,C2}
> D-->{D1,D2,D3}
> 字符串2结果：
> E -->{ E1,E2,E3}
> F -->{F1,F2,F3}
> 请写一个函数， 输入是字符串，输出打印结果如上图所示
> 请完成后发送代码截图

主要是考察关于字符串函数的灵活应用，代码如下
```
package com.paul.demo;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.Map.Entry;
import java.util.Scanner;

public class Test {

	public static void main(String[] args) {
		Map<String,List<String>> res = new HashMap();
		Scanner str = new Scanner(System.in);
		String ss = str.next();
		String [] arr = ss.split(",");
		for(int i=0;i<arr.length;i++) {
			List<String> value = new ArrayList();
			String s = arr[i].split("\\d")[0];
			for(int y=0;y<arr.length;y++){
				if(arr[y].contains(s)) {
					value.add(arr[y]);
				}
			}
			res.put(s, value);
		}

		for(Entry<String,List<String>> hh:res.entrySet()) {
			System.out.println(hh.getKey()+"--->"+hh.getValue().toString());
		}
		
	}

}
```
运行结果：
![逸休联盟](/assets/interview/interview02.png)
## android方面
### 北京狐狸金服公司(搜狐旗下)
1. Android需要在页面跳转的时候吧数据进行存储，那么放在哪个生命周期中比较好

2. fragment和service的生命周期(附activity)
> Fragment的生命周期onAttach(),onCreate(),onCreateView(),onActivityCreated(),onStart(),onResume(),onPause(),onStop(),onDestroyView(),onDestroy(),onDetach()
> 每个生命周期的具体含义
> onAttach():一旦和他所植入的Activity结合的时候回调
> onCreate():创建初始化操作的时候回调
> onCreateView():创建并返回与当前fragment有关的视图层次结构
> onActivityCreated():告诉引入fragment的activity当前fragment初始化完成，可以完成activity的onCreate()操作
> onViewStateRestored(bundle):告诉fragment所有的视图层次已经恢复完毕
> onStart():使这个fragment显示出来，要基于他所植入的activity已经启动
> onResume():是这个fragment可以与用户进行交互，要基于他所植入的activity已经在运行状态
> onPause():这个fragment不再与用户进行交互，可能是引入植入的activity被pause或者被植入的activity中一个操作正在修改他
> onDestroyView():允许这个fragment清理他所有占有的视图资源
> onDestroy():执行fragment最终清理阶段
> onDetach():立刻与所植入的activity断开连接
> Services的生命周期
> Services的生命周期分为两种情况，第一种是通过onstartcommand的方法创建Services，另外一种是通过绑定的形式实现
> 第一种(startService()): onCreate(),onStartCommand(),onDestroy()
> 第二种(bindService()):onCreate(),onBind(),onUnbind(),onDestroy()
> 其实说的就是这两种Service的启动方式不同，一个是通过startService()方法实现，另外一种是通过bindService()方法实现
> 两种方法的区别：第一种开启任务之后，然后通过其他的client或者自己内部结束服务，而绑定的Service是通过绑定来执行的，当Client调用解绑函数的时候停止服务

3. IntentService和Service的区别
> Android在运行的过程中，有的时候需要在后台进行一些操作，那么这些操作是用户无法看到的，那么为了解决这样的问题，就出现了Service，这里我们需要知道的是Service不是独立的线程，也不是独立额进程，他依赖于应用程序的主线程，也就是说，在Service中不要执行耗时操作，比如网络请求等，不然的话会引起ANR。
> 而当前我们执行一些耗时操作，但又必须用Service进行管理的时候，我们就需要使用IntentService。IntentService继承于Service，包含了所有关于Service的特性，不同之处就是在IntentService执行onCreate操作的时候会创建一个线程，那么就可以执行耗时操作，

4. 四种启动模式
- standard：默认启动模式，不管当前activity栈中是否存在需要展示的activity，都会创建一个新的实例对象，放在对应的栈顶位置，例如栈中的顺序是A-B-C,当我们再次打开C页面的时候，顺序就变成了A-B-C-C
- singleTop：当打开一个新的activity的时候，需要判断当前activity栈顶是否是需要打开的新activity，如果是，则直接复用，如果不是，则需要重新创建，例如栈中的顺序是A-B-C,如果新打开的是C，则顺序是A-B-C,如果需要打开是A，那么顺序则会是A-B-C-A
- singleTask：当当前打开的页面是activity栈顶对象，则直接复用，如果不是，并且在activity栈中不存在，则创建一个新的对象，放在栈顶，如果栈顶不是需要打开的对象，并且activity栈中存在，那么需要把该对象之前的所有activity对象全部清空，以使栈顶是需要显示的对象，例如当前栈的顺序是A-B-C,如果当前打开的C，那么顺序是A-B-C，如果打开的对象是D，那么顺序是A-B-C-D，如果打开的是A，那么顺序是A(因为会把B，C对象清除出栈)
- SingleInstance：是一种全局单例模式，一种加强的singleTask模式，除了有SingleTask的特点之外，还有一个就是具有此模式的Activity仅仅能够单独位于一个activity栈中，一般不常用，只有在launch和锁屏键会使用。

5. Android种如何监听数据库数据变化

6. 当APP运行过程中发生了错误，如何吧错误信息保存在本地

7. handler，looper，message，messagequeue之间的关系

8. 如何实现自定义View

9. 在开发中我们会声明一个全局静态的handler，这是为什么
静态的handler可以在当前类没有开始加载的时候就完成

10. rxjava的特点是什么

11. 创建一个对象的方法
- 通过new一个对象
- 通过类的反射，这种方式适用于那些有无参构造的类 Class user = User.class;user.newinstance();
- 利用java对象的可序列化 http://blog.csdn.net/cadi2011/article/details/51672940
- 通过Object中的clone()方法实现，但是必须实现cloneable接口 User user = uu.clone();

### 北京知旅宝面试
1. String，StringBuffer，StringBuilder

2. java的引用类型

3. 支付宝微信等第三方支付接口的使用流程

4. Android原生和Js交互(通过H5页面保存图片到手机)
```
webView.getSettings().setJavaScriptEnabled(true);
webView.loadUrl("file:///android_asset/web.html");
webView.addJavascriptInterface(MainActivity.this,"android”);
//Button按钮 无参调用HTML js方法
findViewById(R.id.button).setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        // 无参数调用 JS的方法
        webView.loadUrl("javascript:javacalljs()");
    }
});
//Button按钮 有参调用HTML js方法
findViewById(R.id.button2).setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        // 传递参数调用JS的方法
        webView.loadUrl("javascript:javacalljswith(" + "'http://blog.csdn.net/Leejizhou'" + ")");
    }
});
@JavascriptInterface
public void startFunction(){
    runOnUiThread(new Runnable() {
        @Override
        public void run() {
            Toast.makeText(MainActivity.this,"show",3000).show();
        }
    });
}
@JavascriptInterface
public void startFunction(final String text){
    runOnUiThread(new Runnable() {
        @Override
        public void run() {
            new AlertDialog.Builder(MainActivity.this).setMessage(text).show();
        }
    });
}
```
5. 第三方聊天(环信，融云)

6. Android各个版本之间的差异变化

# 参考资料
[面试](http://blog.csdn.net/huangqili1314/article/details/72792682)
