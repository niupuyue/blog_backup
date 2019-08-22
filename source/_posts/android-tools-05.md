---
title: android奇技淫巧 05 Android targetSdkVersion 升级到 26 总结
date: 2019-07-01 22:44:10
tags:
  - android
---

由于各个应用市场要求，需要在 2019年5月1日 之前把 target 升级到 26。所以对本公司全网的 App 和可能影响到的相关 SDK 做一个升级。本文主要记录此次升级的过程和解决的一些问题。
<!--more-->
其实升级 target 技术含量不是很高，但是因为涉及到库（100 多个 SDK）和人员，依赖有点多，涉及到公司所有业务的 App, 而且改动的地方和细节也有多，很容易出现考虑不全，导致线上问题。
主要过程为以下几步：
查看官方文档
找下「轮子」，选择一个合适自己的「轮子」， 优化「轮子」
修改对应的模块和 App，并接入到相关的业务方
遇到的问题，解决问题
查看官方文档
因为公司的 App、组件、模块都是基于 target 22 和 support 24 进行开发的，所以要看下官方文档 releases/platforms 和 libraries/support-library 相关的文档，从中找到影响点。我们受影响主要有 2 个方面：
一、运行时权限申请，(这个是大头)
二、其他问题
找轮子
因为第一个权限问题是比较普遍的，所以应该有相关的开源项目支持，为了效率，我们就不重复制造轮子。参考各个比较流行的开源方案，做了一下对比：
相关库
需要修改 Activity 或者 Fragment
设置界面跳转
![动态申请权限对比](/assets/tools/tools-sdk-01.png)
通过以上对比，我们决定使用 AndPermission 的方案，因为这个对于我们现有 App 的侵入是最少的，改动点比较少，而且支持 Appliction 传入（其实当使用 Application 传入时候，会有问题，后面再说）。
说下 AndPermission
当时的考虑点是我们公司很多 SDK 设计的时候是没有 Activity 的引用。但是我们的 SDK, 基本有个 Application 这个引用的，所以选择了一个能够支持传入 Applicaiton 就能够判断权限回调的库。为什么 AndPermission 能够支持呢？因为 AndPermission 在权限校验的时候会启动 PermissionActivity，并且加上了 intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);，所有的权限请求都是在这个 PermissionActivity 处理，而且改动也比较少。
AndPermission 这个库的动态权限请求具体过程大伙可以看下 AndPermission 这个库的源码。使用方式如下：
![](/assets/tools/tools-sdk-02.png)
从 API 调用来看是十分简单好用的。然后我又仔细看了一下它的实现过程，发现了有些不符合我们产品需求的地方：
业务方接入需要简单，需要有一套默认的权限申请失败提示框
弹框提醒权限必须是 Activity 的。但是 AndPermission 是支持 Application 传入的，那么就会有问题
比如我用户点击某个按钮导致后面的方法链中有可能连续调用多次的权限请求，那么就可能一次会弹出很多个框
这样我们就需要在 AndPermission 基础上进行改造，做出适合自己的动态权限库。我们自定义了默认的 onDenied (申请权限失败后的回调)。这样业务方就不需要关心失败后怎么提示了，只要关心成功后把之前的业务逻辑放到成功后这里执行就好了。权限申请失败提示框，用来启动一个 Activity 来 show 这个 Dialog，这样即使在前面传入的是 Application 的 context 也是没有问题了。对于多次请求权限导致多次弹框的问题，我们在 AndPermission 的基础上添加的请求队列，只有上一个权限请求处理完成后，才进行下一次权限请求，这样的话，即使用户一次行为的方法链过程中有很多次请求也不会多次弹框。然后当我用队列管理请求权限的时候，很怕某个权限阻塞了，或者出了未知异常，所以我对每次权限加入过程做了超时处理。以上几点是对于自己业务场景的几点考虑，进行的改造。
修改各个 SDK 和 App
U51AndPermission SDK 已经生成了，那接下来就是怎么集成到各个 SDK 和 App 中了。首先我们要知道，我们的 App 哪些地方有可能调用到了需要权限请求的 API。如果要人工去看效率实在是太低了。我们有 100 多个库，不可能把库看完且不出错。所以我们使用了之前同事的一个插件去扫描相关的 API，用来定位到可能出现权限的类在哪里，用的是哪个库。这样就提高了准确率和效率。
这个是我们插件需要搜索的 API
![](/assets/tools/tools-sdk-03.png)
以下是部分搜索结果
![](/assets/tools/tools-sdk-04.png)
按照这种方法，我们本需要处理 100 多个类库的，现在只有 20 个不到。一下子少了 4 倍的工作量，而且相对准确。
App 「必选权限」方案选择和问题处理
很多 App 启动的时候需要一些必选权限的，我们的 App 也是一样的，需要在 App 启动的时候验证一个必选权限。如果有必选权限，那么提示用户授权，如果不授权，那么就不能够继续使用我们的产品了。所以进入 App 主要功能前，需要一个前置的拦截。考虑过 2 种方案：
方案一、在 Applicaiton 的 onCreate 方法中去申请必选权限，让启动 Activity 等待 Application 中权限申请好了，再用消息(EventBus) 通知 启动 Activity 继续走下去的流程
方案二、新建一个新的 启动 Activity，在这个 Activity 中做申请权限，申请完后再去启动之前老的启动 Activity
和对接的业务方讨论，他们选择了第一种方案， 这么做的原因也是因为我们很多 SDK 的初始化代码在 Appliction 中，我们必须要在初始化 SDK 之前就应该把必要权限拿到。如果选择第二种，那么我们的初始化代码需要移动到新的启动 Activity 中，这样改动风险有点高。
接入代码如下：
![](/assets/tools/tools-sdk-05.png)
对接完后有几个我们遇到的问题需要提下：
问题一、 因为在 App 启动的时候，如果没有必要权限，那么就会有弹框让用户设置权限，这时候用户点击 "设置"，就会跳转到 App 设置权限页面，当用户授权回来的时候，有部分手机会出现黑屏。导致黑屏的原因是 U51Permission 传入进去的 context 是 Application。如果是 Activity 就不会黑屏。所以解决方法是使用 Activity 去请求权限，回调方式是使用 Application.registerActivityLifecycleCallbacks，如下
![](/assets/tools/tools-sdk-06.png)
问题二、 因为我们有些逻辑是放到前后台切换的代码里面的，前后切换的主要用主要用到 Application.registerActivityLifecycleCallbacks 这个回调，根据 Activity 的生命周期统计来切换前后台(前后台的切换逻辑是使用统计 activity 的个数来实现的)；所以 Application.registerActivityLifecycleCallbacks 这个操作应该是在 启动 Activity 之前就应该被注册。但是在申请必要权限的时候，我先启动 Activity 后再去注册这个 callback 的，所以导致启动 Activity 不在计算范围内，导致前后台的调用逻辑不准确，业务逻辑处理时机不对的问题。后来我们使用一个自己的 registerActivityLifecycleCallbacks，命名 MyActivityLifecycleCallbacks，在 Application 一开始启动的时候就注册了，然后把后面其他需要注册的地方放到 MyActivityLifecycleCallbacks 中，由它统一通知其他需要前后台的回调。
替换之前
![](/assets/tools/tools-sdk-07.png)
替换之后
![](/assets/tools/tools-sdk-08.png)
可以看出来替换后改动代码很少，而且所有的前后台切换都会统一到自己的 MyActivityLifecycleCallbacks 里面进行集中管理。
其他问题
还有其他相关问题，网上都能够找到，我就列举一下，大家自己注意一下就好了：
Android 7.0 相机相关问题
需要显示的注册广播
黑白名单限制， debug 包会有弹框提示，release包是没有的；但是这个需要注意，以后可能会有问题；也可以用扫描工具扫描一下这份名单
悬浮框实现的 LayoutParams.TYPE变动
android.os.FileUriExposedException 的异常，文件共享的限制和第 1 条一样
Sevices.startService 有些手机会奔溃，这个我们的 App 之前就处理过了
vivo 手机相机权限问题
NotificationManager 应该使用 builder 去创建，需要有个 channel， Android 8.0 的修改
荣耀 8 手机，scrollView 嵌套 recycleView 显示不全的问题
Toast 相关的问题
总结
以上就是本次升级需要的修改点了，剩下的还有测试、灰度和正式上线。从整个适配过程来看，这个需求其实不是很难，但是从改动的点来说，沟通协调能力要求还是很高的，会涉及到大部分客户端开发、测试和产品。
对于改造升级大范围的基础库，每个环节尽量多思考和团队成员多讨论，切记不要一个人闷头就是干。但也不要过多的担心，需要胆大心细，这样才能把事情推进下去。好了，target 28 的适配也马上要来了。
作者介绍
Mr.Jie，51信用卡客户端基础组 Android 开发工程师，目前主要负责客户端创新项目相关技术演进工作


