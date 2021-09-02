---
title: 重拾android路(十六) 进程保活
date: 2016-10-12 15:51:26
tags:
  - android
---

随着现在智能手机的普及，越来越多的手机应用充斥着各种市场，然后并不是每一个应用都会在手机里使用，不是每一个应用都会被经常使用，所以，Android手机有自己的一套内存管理方法，当你的手机在一定时间内没有使用该应用，那么可能会被杀死。但是对于我们公司或者说开发者而言，我们肯定不希望这样的事情发生，因为如果一直存活下去的话，手机再次打开的时候，这个应用的使用会比较快速，同样也能得到用户的青睐。那么我们怎么做到，像QQ一样，不被杀死。
<!--more-->
很多人，很多公司都在考虑这个问题，但是，并不代表他们已经把这个问题解决的很好。
这几日看了一篇文章，关于进程保活的，感觉挺好，反正比我之前的好，(说来惭愧以前的应用感觉都像是流氓软件)颇有感触，记录下来，和大家分享，原文地址我会在下面贴出来

# 保活手段
目前来说，保活手段有三种”黑，白，灰“.
1. 黑色保活：不同的app进程使用广播相互唤醒(在手机进程后端，你会发现类似于软件相互关联的内容，就这个)
2. 白色保活：启动前台service(其实就是像一些酷狗音乐，网易云音乐，基本上都是这么干的，当然是明面上的，背地里，呵呵🙃)
3. 灰色保活：利用系统漏洞启动前台service

## 黑色保活
所谓的黑色保活就是相当于把各个有利益关系的app绑定在一起，只要其中一个被打开了，那么另外几个也会同样被激活，在内存里运行，所以你的内存总是不够.举个栗子🌰
1. 开机，网络切换，拍照，拍视频的时候，利用系统发出的广播唤醒app
2. 接入第三方SDK也会唤醒相应的app进程，如微信sdk会唤醒微信，支付宝sdk会唤醒支付宝。由此发散开去，就会直接触发了下面的第三个例子
3. 如果你的手机里装了去哪网app，淘宝app，天猫app，那么抱歉，基本上所有的app都会被唤醒(又拿马云巴巴举例子，会不会被打死)

是不是很恶心？！对，这是这么样的。这也就是在初期的时候，Android就是干不过iPhone的原因。
不过后来google公司认识到了这一点，所以在Android N中取消了ACTION_NEW_PICTURE（拍照），ACTION_NEW_VIDEO（拍视频），CONNECTIVITY_ACTION（网络切换）等三种广播，无疑给了很多app沉重的打击。
针对场景2和场景3，因为调用SDK唤醒app进程属于正常行为，此处不讨论(不来了，真的不来了，顺丰快递已经在路上了，我要跑路了，尴尬而不失礼貌的微笑)。但是在借助LBE分析app之间的唤醒路径的时候，发现了两个问题：
1. 很多推送SDK也存在唤醒app的功能
2. app之间的唤醒路径真是多，且错综复杂
来吧，给大家看一下(这里跟大家说一下，我的是vivo手机，不能获取root权限，很尴尬，只能偷图)
![process01](/assets/process/process01.png)
![process02](/assets/process/process02.png)
![process03](/assets/process/process03.png)
可以看到以上3条唤醒路径，但是涵盖的唤醒应用总数却达到了23+43+28款，数目真心惊人。请注意，这只是我手机上一款app的唤醒路径而已，到了这里是不是有点细思极恐。

当然，这里依然存在一个疑问，就是LBE分析这些唤醒路径和互相唤醒的应用是基于什么思路，我们不得而知。所以我们也无法确定其分析结果是否准确，如果有LBE的童鞋看到此文章，不知可否告知一下思路呢？但是，手机打开一个app就唤醒一大批，我自己可是亲身体验到这种酸爽的......

## 白色保活
白色保活手段非常简单，就是调用系统api启动一个前台的Service进程，这样会在系统的通知栏生成一个Notification，用来让用户知道有这样一个app在运行着，哪怕当前的app退到了后台.这个就不给图了，自己看手机吧

## 灰色保活
灰色保活，这种保活手段是应用范围最广泛。它是利用系统的漏洞来启动一个前台的Service进程，与普通的启动方式区别在于，它不会在系统通知栏处出现一个Notification，看起来就如同运行着一个后台Service进程一样。这样做带来的好处就是，用户无法察觉到你运行着一个前台进程（因为看不到Notification）,但你的进程优先级又是高于普通后台进程的。那么如何利用系统的漏洞呢，大致的实现思路和代码如下
1. API < 18，启动前台Service时直接传入new Notification()；
2. API >= 18，同时启动两个id相同的前台Service，然后再将后启动的Service做stop处理；
```
public class GrayService extends Service {

    private final static int GRAY_SERVICE_ID = 1001;

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        if (Build.VERSION.SDK_INT < 18) {
            startForeground(GRAY_SERVICE_ID, new Notification());//API < 18 ，此方法能有效隐藏Notification上的图标
        } else {
            Intent innerIntent = new Intent(this, GrayInnerService.class);
            startService(innerIntent);
            startForeground(GRAY_SERVICE_ID, new Notification());
        }

        return super.onStartCommand(intent, flags, startId);
    }

    ...
    ...

    /**
     * 给 API >= 18 的平台上用的灰色保活手段
     */
    public static class GrayInnerService extends Service {

        @Override
        public int onStartCommand(Intent intent, int flags, int startId) {
            startForeground(GRAY_SERVICE_ID, new Notification());
            stopForeground(true);
            stopSelf();
            return super.onStartCommand(intent, flags, startId);
        }

    }
}
```
代码大致就是这样，能让你神不知鬼不觉的启动着一个前台Service。其实市面上很多app都用着这种灰色保活的手段，什么？你不信？好吧，我们来验证一下。流程很简单，打开一个app，看下系统通知栏有没有一个 Notification，如果没有，我们就进入手机的adb shell模式，然后输入下面的shell命令
```
dumpsys activity services PackageName
```
打印出指定包名的所有进程中的Service信息，看下有没有 isForeground=true 的关键信息。如果通知栏没有看到属于app的 Notification 且又看到 isForeground=true 则说明了，此app利用了这种灰色保活的手段。

其实Google察觉到了此漏洞的存在，并逐步进行封堵。这就是为什么这种保活方式分 API >= 18 和 API < 18 两种情况，从Android5.0的ServiceRecord类的postNotification函数源代码中可以看到这样的一行注释
![process04](/assets/process/process04.png)
当某一天 API >= 18 的方案也失效的时候，我们就又要另谋出路了。需要注意的是，使用灰色保活并不代表着你的Service就永生不死了，只能说是提高了进程的优先级。如果你的app进程占用了大量的内存，按照回收进程的策略，同样会干掉你的app。

其实真正的做到进程保活是可以的，因为我们的Android手机本身在你关闭软件的时候并没有把进程杀死，而是放入了缓存，而当我们的内容不够的时候才会去吧这一些优先级比较低的kill掉。想要做到永远存活是不可能的，不管是用户还是google官方。所以还是老老实实的做优化吧。

# 补充
## 进程
每一个Android应用启动后至少对应一个进程，有的是多个进程，而且主流应用中多个进程的应用比例较大
![process05](/assets/process/process05.png)
### 如何查看进程的基本信息
对于任何一个进程，我们都可以通过adb shell ps|grep 的方式来查看它的基本信息

| 值 |	解释|
|----|----|
|u0_a16	|USER 进程当前用户|
|3881|	进程ID|
|1223|	进程的父进程ID|
|873024|	进程的虚拟内存大小|
|37108|	实际驻留”在内存中”的内存大小|
|com.wangjing.processlive  |	进程名|

### 进程划分
Android中的进程跟封建社会一样，分了三流九等，Android系统把进程的划为了如下几种（重要性从高到低），网上多位大神都详细总结过（备注：严格来说是划分了6种）。
1. 前台进程
场景： 
- 某个进程持有一个正在与用户交互的Activity并且该Activity正处于resume的状态。 
- 某个进程持有一个Service，并且该Service与用户正在交互的Activity绑定。 
- 某个进程持有一个Service，并且该Service调用startForeground()方法使之位于前台运行。 
- 某个进程持有一个Service，并且该Service正在执行它的某个生命周期回调方法，比如onCreate()、 onStart()或onDestroy()。 
- 某个进程持有一个BroadcastReceiver，并且该BroadcastReceiver正在执行其onReceive()方法。
用户正在使用的程序，一般系统是不会杀死前台进程的，除非用户强制停止应用或者系统内存不足等极端情况会杀死
2.  可见进程
场景： 
- 拥有不在前台、但仍对用户可见的 Activity（已调用 onPause()）。 
- 拥有绑定到可见（或前台）Activity 的 Service

用户正在使用，看得到，但是摸不着，没有覆盖到整个屏幕,只有屏幕的一部分可见进程不包含任何前台组件，一般系统也是不会杀死可见进程的，除非要在资源吃紧的情况下，要保持某个或多个前台进程存活
3. 服务进程
场景 
- 某个进程中运行着一个Service且该Service是通过startService()启动的，与用户看见的界面没有直接关联。

在内存不足以维持所有前台进程和可见进程同时运行的情况下，服务进程会被杀死
4. 后台进程
场景： 
- 在用户按了”back”或者”home”后,程序本身看不到了,但是其实还在运行的程序，比如Activity调用了onPause方法

系统可能随时终止它们，回收内存
5. 空进程
场景： 
- 某个进程不包含任何活跃的组件时该进程就会被置为空进程，完全没用,杀了它只有好处没坏处,第一个干它!


# 参考资料
[android进程保活，你所需要知道的一切](https://www.jianshu.com/p/63aafe3c12af)
[android进程保活](http://blog.csdn.net/qq_17007915/article/details/77963570)