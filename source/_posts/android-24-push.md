---
title: 重拾android路(二十三) Android推送集成
date: 2019-2-08 11:27:11
tags:
  - android
  - 推送
---

目前所在的公司让我把之前项目中的推送重新整理一下。因为现在的需求是在应用被后台杀死的情况下，依然可以推送消息，那么只是单一的使用一个推送工具是无法实现的，比如友盟。那么就需要同时集成Umeng，华为，小米，Oppo等手机厂商提供的PushSDK。这本来是简简单单的一件事，突然之后，工作量无形之后增大。不过好在经过一段时间的尝试，终于集成成功，今天把这个历程记录下来，方面以后查看。
<!--more-->
# Umeng
这个是官方网址，里面介绍的还算是比较详细，所以还是把官网地址粘出来:[官网地址](https://www.umeng.com/push)
在官网中，集成友盟推送的方式有两种，一种是通过gradle的maven仓库，另外一种是通过jar包引入的方式。这里我直接选择第一种，因为感觉gradle至少不用来回拷贝jar包，在更新的时候，直接更改Gradle依赖的版本既可。这里官方还非常贴心的给出了一个官方的[demo](https://download.umeng.com/online/2018/12/20181228110151948.zip)。结合官方的例子，集成Umeng推送的步骤如下所示

> 关于Appkey的申请等工作，这里就不在叙述，具体内容请参看官方文档

## 接入SDK
接入SDK时需要在gradle的文件中，添加如下代码，然后重新编译既可
```
//PushSDK必须依赖基础组件库，所以需要加入对应依赖
compile 'com.umeng.umsdk:common:1.5.4'
//PushSDK必须依赖utdid库，所以需要加入对应依赖
compile 'com.umeng.umsdk:utdid:1.1.5.3'
//PushSDK
compile 'com.umeng.umsdk:push:5.0.2'
```
这里使用了三个依赖，其中第一个和第二个是公共库，只要接入Umeng的内容，不管是推送，还是数据统计，都需要引入的内容。第三个是Umeng推送需要依赖的内容。这里使用的是当前的最新版5.0.2.
在执行编译之前，需要在项目的build.gradle中添加maven库，具体代码如下：
```
buildscript {
    repositories {
        google()
        jcenter()
        maven { url 'https://dl.bintray.com/umsdk/release' }
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:3.1.4'
        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}
allprojects {
    repositories {
        google()
        jcenter()
        mavenCentral()
        maven { url 'https://dl.bintray.com/umsdk/release' }
    }
}
```
在执行完成之后，就可以开始编译了。

## 基础接口引入
这里的基础接口只是能实现基本的推送接收功能，比如获取一个从控制台推送的推送。
### 初始化
必须在项目中重新自定义Application，并且在自定义的Application中的onCreate方法中添加推送的注册等操作，具体如下所示
```
public class App extends Application {

    @Override
    public void onCreate() {
        super.onCreate();
        initPush();
    }

    protected void initPush() {
        UMConfigure.init(this, UMENG_APPKEY, UMENG_APPCHANNEL, UMConfigure.DEVICE_TYPE_PHONE, UMENG_APPSECRET);
    }
}
```
其中

UMENG_APPKEY，UMENG_APPSECRET是我们在注册账号时Umeng官方为我们的应用分配的，直接使用即可。

UMENG_CHANNEL是渠道名称。

UMConfigure.DEVICE_TYPE_PHONE表示的是设配类型是手机，除此之外还有UMConfigure.DEVICE_TYPE_BOX表示设备类型是盒子。

在执行初始化之后，我们的基本操作完成了，但是为了后面能够更好的使用推送，我们需要执行下面更多的操作。
### 注册
我们需要通过Umeng向Umeng官方注册，其实也就是告诉Umeng控制台，这里我们是有一个手机设备需要接受推送消息的。并且在注册成功之后，获取注册的DeviceToken。
```
public class App extends Application {

    @Override
    public void onCreate() {
        super.onCreate();
        initPush();
    }

    protected void initPush() {
        UMConfigure.init(this, UMENG_APPKEY, UMENG_APPCHANNEL, UMConfigure.DEVICE_TYPE_PHONE, UMENG_APPSECRET);

        //获取推送代理，这个代理可以帮我们去执行诸如点击事件，样式不同的通知栏等操作
        PushAgent pushAgent = PushAgent.getInstance(this);
        pushAgent.register(new IUmengRegisterCallback() {
            @Override
            public void onSuccess(String s) {
                //注册成功
                Log.e("NPL", "deviceToken=" + s);
            }

            @Override
            public void onFailure(String s, String s1) {
                //注册失败
            }
        });
    }
}
```
这里注册成功之后，一般会想我们自己的后台发送一个deviceToken，这样方便以后发送推送消息是由后台控制的。例如给某个特定用的用户推送消息。

之后还有一个操作，就是帮助Umeng后台统计日活情况的主要依据，一定要加上。Activity中所有的都要添加，建议直接在BaseActivity中的oncreate方法中实现。
```
PushAgent.getInstance(context).onAppStart();
```
到这里基本的推送就完成了，下面来尝试推送一个新消息到特定的手机。
这里的操作可以直接参考官方的，官方的截图和各种步骤提示已经非常清楚了，这里就不在一一赘述。

## 高级功能
Umeng推送高级功能实现了自定义通知栏，自定义显示通知的动作等，这里的高级功能只适应于Android4.0以上版本，除此之后，可能由于不同的手机会出现不同的效果，如ROM定制型较强的小米，华为手机，部分效果无法显示。这里官方也提供了一个[demo](https://github.com/umeng/MultiFunctionAndroidDemo.git)。

### 自定义图标和自定义标题栏声音
在drawable目录下放置两张图片，分别命名为umeng_push_notification_default_large_icon和umeng_push_notification_default_small_icon，那么在Umeng推送中，就会使用提供的图标，如果没有，则使用应用默认图标。
> 小米手机暂时无法使用自定义图标

在res/raw目录下添加资源文件，并且命名为umeng_push_notification_default_sound。如果没有，则使用系统的通知声音。
> 若需要在线配置声音，则需先将与配置的声音文件放置在res/raw下，然后自发送后台指定声音的id，即R.raw.[sound]里的sound；

> 自定义通知栏声音仅在Android 8.0以下机型生效，如需适配Android 8.0及以上版本，请参考自定义通知栏样式，重写getNotification方法，设置声音。

### 自定义通知栏是否显示
有时候会有这样的需求，如果当前应用出去前台展示，则不用显示通知栏推送，只有当应用处于后台时，才需要在通知栏显示，那么这是用就可使用下面的代码
```
mPushAgent.setNotificaitonOnForeground(false);
```
> 这个设置只能在执行完注册(regist)方法之后，才能调用

### 自定义通知栏样式
UmengMessageHandler类负责处理消息，包括通知和自定义消息。我们可以通过getNotification函数设置不同的通知栏样式。
```
protected void initPush() {
        UMConfigure.init(this, UMENG_APPKEY, UMENG_APPCHANNEL, UMConfigure.DEVICE_TYPE_PHONE, UMENG_APPSECRET);

        //获取推送代理，这个代理可以帮我们去执行诸如点击事件，样式不同的通知栏等操作
        PushAgent pushAgent = PushAgent.getInstance(this);
        pushAgent.register(new IUmengRegisterCallback() {
            @Override
            public void onSuccess(String s) {
                //注册成功
                Log.e("NPL", "deviceToken=" + s);
            }

            @Override
            public void onFailure(String s, String s1) {
                //注册失败
            }
        });

        pushAgent.setNotificaitonOnForeground(true);

        UmengMessageHandler umengMessageHandler = new UmengMessageHandler() {
            @Override
            public Notification getNotification(Context context, UMessage uMessage) {
                /**
                 * context:上下文
                 * uMessage:表示当前传递过来的消息，在消息中，我们通过变量builder_id判断使用哪种样式
                 */
                switch (uMessage.builder_id) {
                    case UMENG_NOTIFICATION_NORMAL:
                        //创建通知栏对象
                        Notification.Builder builder = new Notification.Builder(context);
                        RemoteViews remoteView = new RemoteViews(context.getPackageName(), R.layout.view_notification_normal);
                        remoteView.setTextViewText(R.id.notification_title, uMessage.title);
                        remoteView.setTextViewText(R.id.notification_text, uMessage.text);
                        remoteView.setImageViewBitmap(R.id.notification_large_icon,
                                getLargeIcon(context, uMessage));
                        remoteView.setImageViewResource(R.id.notification_small_icon,
                                getSmallIconId(context, uMessage));
                        builder.setContent(remoteView)
                                .setSmallIcon(getSmallIconId(context, uMessage))
                                .setTicker(uMessage.ticker)
                                .setAutoCancel(true);
                        return builder.getNotification();
                    case UMENG_NOTIFICATION_LARGE:
                        return null;
                    case UMENG_NOTIFICATION_SMALL:
                        return null;
                    default:
                        return super.getNotification(context, uMessage);
                }

            }
        };
        pushAgent.setMessageHandler(umengMessageHandler);
    }
```
在测试时，发送的消息需要在下面的截图中添加相应的类型：

![自定义通知栏样式](/assets/push/push01.png)

### 自定义通知栏打开动作
我们通过解析UMessage中custome字段的内容，可以实现自定义通知栏打开动作。当我们需要执行自定义动作时，需要重写dealWithCustomAction方法。在自定义动作的时候，一般是通过传递不同的数据，来实现动态判断执行哪种动作的。
在传递数据时，通过如下的内容实现：
![自定义通知栏打开动作](/assets/push/push02.png)
这里我们传递了三个参数，分别是type，url，innerUrl。那么我们就通过type类型来判断跳转不同的页面，然后在通过url，判断跳转的子页面，通过innerUrl来判断跳转到H5页面。

```
protected void initPush() {
        UMConfigure.init(this, UMENG_APPKEY, UMENG_APPCHANNEL, UMConfigure.DEVICE_TYPE_PHONE, UMENG_APPSECRET);
        //获取推送代理，这个代理可以帮我们去执行诸如点击事件，样式不同的通知栏等操作
        PushAgent pushAgent = PushAgent.getInstance(this);
        pushAgent.register(new IUmengRegisterCallback() {
            @Override
            public void onSuccess(String s) {
                //注册成功
                Log.e("NPL", "deviceToken=" + s);
            }
            @Override
            public void onFailure(String s, String s1) {
                //注册失败
            }
        });
        pushAgent.setNotificaitonOnForeground(true);
        UmengMessageHandler umengMessageHandler = new UmengMessageHandler() {
            @Override
            public Notification getNotification(Context context, UMessage uMessage) {
                /**
                 * context:上下文
                 * uMessage:表示当前传递过来的消息，在消息中，我们通过变量builder_id判断使用哪种样式
                 */
                switch (uMessage.builder_id) {
                    case UMENG_NOTIFICATION_NORMAL:
                        //创建通知栏对象
                        Notification.Builder builder = new Notification.Builder(context);
                        RemoteViews remoteView = new RemoteViews(context.getPackageName(), R.layout.view_notification_normal);
                        remoteView.setTextViewText(R.id.notification_title, uMessage.title);
                        remoteView.setTextViewText(R.id.notification_text, uMessage.text);
                        remoteView.setImageViewBitmap(R.id.notification_large_icon,
                                getLargeIcon(context, uMessage));
                        remoteView.setImageViewResource(R.id.notification_small_icon,
                                getSmallIconId(context, uMessage));
                        builder.setContent(remoteView)
                                .setSmallIcon(getSmallIconId(context, uMessage))
                                .setTicker(uMessage.ticker)
                                .setAutoCancel(true);
                        return builder.getNotification();
                    case UMENG_NOTIFICATION_LARGE:
                        return null;
                    case UMENG_NOTIFICATION_SMALL:
                        return null;
                    default:
                        return super.getNotification(context, uMessage);
                }
            }
        };
        pushAgent.setMessageHandler(umengMessageHandler);
        UmengNotificationClickHandler umengNotificationClickHandler = new UmengNotificationClickHandler() {
            @Override
            public void dealWithCustomAction(Context context, UMessage uMessage) {
                //这里需要解析uMessage，然后通过custome属性执行不同的操作
                if (uMessage == null) {
                    return;
                }
                HashMap<String, String> hm = (HashMap<String, String>) uMessage.extra;
                String type = "";
                String url = "";
                String innerUrl = "";
                if (hm != null && hm.size() > 0) {
                    if (hm.containsKey("type")) {
                        type = hm.get("type");
                    }
                    if (hm.containsKey("url")) {
                        url = hm.get("url");
                    }
                    if (hm.containsKey("innerUrl")) {
                        innerUrl = hm.get("innerUrl");
                    }
                }
                if (type.equalsIgnoreCase("login")){
                    //跳转到登录页面
                }else if (type.equalsIgnoreCase("regist")){
                    //跳转到注册页面
                }else if (type.equalsIgnoreCase("im")){
                    //跳转到聊天页面
                }else {
                    //跳转到web页面
                    if(TextUtils.isEmpty(innerUrl)){
                        innerUrl = "http://www.baidu.com";
                    }
                }
            }
        };
        pushAgent.setNotificationClickHandler(umengMessageHandler);
    }
 ```

> 其实在UmengNotificationClickHandler中，除了上面的方法之外，还有别的方法可以完成通知栏的点击操作，如launchApp，openUrl，openActivity，dealWithCustomAction等，这几个方法代表了点击通知栏之后不同的操作，但是都会传递UMessage对象，所以所能执行到的效果类似，就不一一演示了。

### 自定义消息(透传消息)
透传消息不是通知，也就不会在通知栏上显示，友盟会将透传消息传递给SDK，之后透传消息需要展示的样式和执行的操作，完全由代码决定。
![透传消息](/assets/push/push03.png)
自定义消息可以用于应用内部或者特殊的逻辑。如我们需要推送的内容不是在通知栏显示，而是以一个弹窗的样式展示，则可以通过自定义消息。
想要实现对自定义消息的处理，需要在UmengMessageHandler中重写dealWithCustomMessage方法。这个方法就是当发送自定义消息时，由SDK触发的。透传消息的自定义内容是放在UMessage对象中的custome参数中的，返回的数据是一个String类型，通过解析String内容，获取自定义消息。
```
protected void initPush() {
        UMConfigure.init(this, UMENG_APPKEY, UMENG_APPCHANNEL, UMConfigure.DEVICE_TYPE_PHONE, UMENG_APPSECRET);
        //获取推送代理，这个代理可以帮我们去执行诸如点击事件，样式不同的通知栏等操作
        PushAgent pushAgent = PushAgent.getInstance(this);
        pushAgent.register(new IUmengRegisterCallback() {
            @Override
            public void onSuccess(String s) {
                //注册成功
                Log.e("NPL", "deviceToken=" + s);
            }
            @Override
            public void onFailure(String s, String s1) {
                //注册失败
            }
        });
        pushAgent.setNotificaitonOnForeground(true);
        UmengMessageHandler umengMessageHandler = new UmengMessageHandler() {
            @Override
            public Notification getNotification(Context context, UMessage uMessage) {
                /**
                 * context:上下文
                 * uMessage:表示当前传递过来的消息，在消息中，我们通过变量builder_id判断使用哪种样式
                 */
                switch (uMessage.builder_id) {
                    case UMENG_NOTIFICATION_NORMAL:
                        //创建通知栏对象
                        Notification.Builder builder = new Notification.Builder(context);
                        RemoteViews remoteView = new RemoteViews(context.getPackageName(), R.layout.view_notification_normal);
                        remoteView.setTextViewText(R.id.notification_title, uMessage.title);
                        remoteView.setTextViewText(R.id.notification_text, uMessage.text);
                        remoteView.setImageViewBitmap(R.id.notification_large_icon,
                                getLargeIcon(context, uMessage));
                        remoteView.setImageViewResource(R.id.notification_small_icon,
                                getSmallIconId(context, uMessage));
                        builder.setContent(remoteView)
                                .setSmallIcon(getSmallIconId(context, uMessage))
                                .setTicker(uMessage.ticker)
                                .setAutoCancel(true);
                        return builder.getNotification();
                    case UMENG_NOTIFICATION_LARGE:
                        return null;
                    case UMENG_NOTIFICATION_SMALL:
                        return null;
                    default:
                        return super.getNotification(context, uMessage);
                }
            }
            @Override
            public void dealWithCustomMessage(final Context context, final UMessage uMessage) {
                //自定义消息的内容是放在uMessage中的custome参数中的
                if (uMessage == null) {
                    return;
                }
                String custome = uMessage.custom;
                if (TextUtils.isEmpty(custome)) {
                    return;
                }
                mHandler.post(new Runnable() {
                    @Override
                    public void run() {
                        // 对于自定义消息，PushSDK默认只统计送达。若开发者需要统计点击和忽略，则需手动调用统计方法。
                        boolean isClickOrDismissed = true;
                        if (isClickOrDismissed) {
                            //自定义消息的点击统计
                            UTrack.getInstance(getApplicationContext()).trackMsgClick(uMessage);
                        } else {
                            //自定义消息的忽略统计
                            UTrack.getInstance(getApplicationContext()).trackMsgDismissed(uMessage);
                        }
                        Toast.makeText(context, uMessage.custom, Toast.LENGTH_LONG).show();
                        //一般需要将custome转换成Json数据，然后通过Json数据中的内容，判断需要执行的操作
                    }
                });
            }
        };
        pushAgent.setMessageHandler(umengMessageHandler);
        UmengNotificationClickHandler umengNotificationClickHandler = new UmengNotificationClickHandler() {
            @Override
            public void dealWithCustomAction(Context context, UMessage uMessage) {
                //这里需要解析uMessage，然后通过custome属性执行不同的操作
                if (uMessage == null) {
                    return;
                }
                HashMap<String, String> hm = (HashMap<String, String>) uMessage.extra;
                String type = "";
                String url = "";
                String innerUrl = "";
                if (hm != null && hm.size() > 0) {
                    if (hm.containsKey("type")) {
                        type = hm.get("type");
                    }
                    if (hm.containsKey("url")) {
                        url = hm.get("url");
                    }
                    if (hm.containsKey("innerUrl")) {
                        innerUrl = hm.get("innerUrl");
                    }
                }
                if (type.equalsIgnoreCase("login")) {
                    //跳转到登录页面
                } else if (type.equalsIgnoreCase("regist")) {
                    //跳转到注册页面
                } else if (type.equalsIgnoreCase("im")) {
                    //跳转到聊天页面
                } else {
                    //跳转到web页面
                    if (TextUtils.isEmpty(innerUrl)) {
                        innerUrl = "http://www.baidu.com";
                    }
                }
            }
        };
        pushAgent.setNotificationClickHandler(umengMessageHandler);
    }
```
### 标签&别名
我们可以这样理解，如果我们想给一群特定的人推送消息，这一群人可以是会员，或者被系统拉黑的用户等等，而其他的用户是接收不到这个推送消息的。为了简化这样的流程，引入了标签的概念。
别名是我们可一个特定的某一个用户推送消息。例如一个用户的好友将他删除了，我们可以通过发送推送，告诉被删除者，你的好友将你删除了。
在代码中，我们通过addTags和addAlias来实现添加标签和别名。
标签：
```
//添加标签，将tag1,tag2添加到当前设备中，一般情况下，我们会有一些判断，然后再为不同的用户添加不同的tag
        pushAgent.getTagManager().addTags(new TagManager.TCallBack() {
            @Override
            public void onMessage(boolean b, ITagManager.Result result) {

            }
        }, "tag1", "tag2");

        //删除标签，将tag1，tag2从当前设备中删除
        pushAgent.getTagManager().deleteTags(new TagManager.TCallBack() {
            @Override
            public void onMessage(boolean b, ITagManager.Result result) {

            }
        }, "tag1", "tag2");

        //获取服务器端所有的标签
        pushAgent.getTagManager().getTags(new TagManager.TagListCallBack() {
            @Override
            public void onMessage(boolean b, List<String> list) {
                
            }
        });
```
别名：
```
//增加别名，将某一类型的别名绑定至某一设备，老的绑定设备信息依然保留，别名和deviceToken的映射关系是一对多
pushAgent.addAlias("alias1", "type1", new UTrack.ICallBack() {
            @Override
            public void onMessage(boolean b, String s) {

            }
});
//移除别名
pushAgent.deleteAlias("alias1", "type1", new UTrack.ICallBack() {
            @Override
            public void onMessage(boolean b, String s) {
                
            }
});
```
> 设置别名之前，需要先进行注册并且获取到deviceToken。

### 插屏消息
这个类型其实我们在平时也很常见，具体的例子，如图显示(盗图狂魔上线)
![插屏消息](/assets/push/push04.png)
使用demo
```
InAppMessageManager.getInstance(this).showCardMessage(this, this.getClass().getSimpleName(), new IUmengInAppMsgCloseCallback() {
            @Override
            public void onClose() {
                //差评消息关闭时调用该方法
                Log.e("NPL","关闭了插屏消息");
            }
        });
```
> InAppMessageManager.getInstance(Context context).showCardMessage(Activity activity, String label, IUmengInAppMsgCloseCallback callback);
> 注意
> 1. label ：表示当前插屏消息的标识
> 2. 客户端先调用showCardMessage，将label发送给服务端，之后U-Push后台展示位置才会出现可选label
> 3. 插屏消息的图片会执行缓存，但有新消息来时，旧消息的缓存会被删除

同时我们可以自定义插屏消息的样式，需要在添加布局文件umeng_custom_card_message.xml。使用的模板如下，里面除了一个ImageView和两个button不能改变之外，其他的均可以改变

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:gravity="center"
    android:background="#33000000">

    <FrameLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="@color/white"
        android:layout_margin="60dp">

        <RelativeLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_margin="20dp">

            <ImageView
                android:id="@+id/umeng_card_message_image"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:scaleType="centerCrop"/>

            <Button
                android:id="@+id/umeng_card_message_ok"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_below="@id/umeng_card_message_image"
                android:layout_marginTop="20dp"
                android:text="确定"/>
        </RelativeLayout>

        <Button
            android:id="@+id/umeng_card_message_close"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="top|right"
            android:text="关闭"/>

    </FrameLayout>

</RelativeLayout>
```
## 其他
### 检查集成配置文件
为了便于开发者更好的集成配置文件，我们提供了对于AndroidManifest配置文件的检查工具，可以自行检查开发者的配置问题。SDK默认是不检查集成配置文件的
```
mPushAgent.setPushCheck(true);
```
### 关闭推送
```
mPushAgent.disable(new IUmengCallback() {

    @Override
    public void onSuccess() {

    }

    @Override
    public void onFailure(String s, String s1) {

    }

});
```
在调用关闭推送之后，想要再次打开推送，则使用下面的代码
```
mPushAgent.enable(new IUmengCallback() {

    @Override
    public void onSuccess() {

    }

    @Override
    public void onFailure(String s, String s1) {

    }

});
```
## 常见问题汇总
1. 集成推送之后，小米手机会报一个错误，错误截图如下：
![常见错误问题1](/assets/push/push05.png)
解决方法：
在app项目的manifest文件中加入tools:replace=”android:allowBackup”标签
```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    package="com.example.push.umeng.umengomnipushdemo">

    <application
        tools:replace="android:allowBackup"
        android:allowBackup="true"
        android:name=".App"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme"
        >
        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>
</manifest>
```
2. 对于从老版本升级上来的Umeng推送，之前的Appkey等信息是在清单配置文件中设置的 ，如下所示
```
<!--友盟正式AppKey-->
        <meta-data
            android:name="UMENG_APPKEY"
            android:value="xxxxxxxxxxxxxxxxxxxxxx" />
        <!--友盟正式渠道-->
        <meta-data
            android:name="UMENG_CHANNEL"
            android:value="ipush" />
        <!--正式推送SECRET-->
        <meta-data
            android:name="UMENG_MESSAGE_SECRET"
            android:value="xxxxxxxxxxxxxxxxxxxxxx" />
```
在新版本中，就算在Manifest文件中设置了AppKey等信息，同样要执行UMConfigure.init方法。

# 小米

小米推送主要是用来适配小米手机的。所以，在做的时候，一般会判断当前手机是否是小米手机，如果是小米手机，则去使用小米推送，注册小米推送，如果不是，则默认使用Umeng推送。
在集成小米推送之前，需要先注册小米开发者账号，具体的步骤，这里不再叙述，看[注册为开发者](https://dev.mi.com/console/doc/detail?pId=848)

## 小米推送
小米推送同时支持Android和iOS两大移动平台，推送稳定。。。算了，我真的编不下去了，想看的，去[官网](https://dev.mi.com/console/appservice/push.html)看看他们的文档吧。

## 小米推送集成
对于这个官方有文档，写的还是比较清晰的，[官方文档](https://dev.mi.com/console/doc/detail?pId=100)。
### 下载小米推送SDK
小米提供的都是jar的方式，暂时不支持gradle的方式，所以直接去下面的路径中下载相应的jar包。[下载路径](http://dev.xiaomi.com/mipush/downpage/)。当前小米推送的最新版本是3.6.12。下载的SDK中不仅包括jar包，同时还包括demo。将demo导入到AndroidStudio中，更改部分配置既可运行。

### 小米推送注册
小米推送注册需要放在Application中去执行，在执行之前，最好先判断当前手机是否是小米手机，判断当前手机是否是小米手机，代码如下：
```
 if (Build.MANUFACTURER.equalsIgnoreCase("xiaomi")){
            initMiPush();
 }
```
如果是小米手机，则需要执行如下代码：
```
 /**
     * 初始化小米推送
     */
    protected void initMiPush() {
        //在此处注册小米推送之后，需要由MiMessageReceiver去实现注册之后的回调
        MiPushClient.registerPush(this, MIPUSH_APPID, MIPUSH_APPKEY);
    }
```
在这里注册完小米推送之后，需要创建一个新的类来处理注册之后的回调，处理接收消息的回调，点击通知栏消息的回调等。在执行之前，需要先在Manifest文件中添加部分代码，具体如下所示：
1. 添加权限
```
<uses-sdk
        android:minSdkVersion="8"
        android:targetSdkVersion="26" />

    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
    <uses-permission android:name="android.permission.READ_PHONE_STATE" />
    <uses-permission android:name="android.permission.GET_TASKS" />
    <!-- the following 2 com.xiaomi.mipushdemo should be changed to your package name -->
    <permission
        android:name="com.paulniu.mypush.permission.MIPUSH_RECEIVE"
        android:protectionLevel="signature" />

    <uses-permission android:name="com.paulniu.mypush.permission.MIPUSH_RECEIVE" />
    <uses-permission android:name="android.permission.VIBRATE" />
```
> 注意：
> 这里需要做一些修改，需要将
```<permission
        android:name="com.paulniu.mypush.permission.MIPUSH_RECEIVE"
        android:protectionLevel="signature" />
```
中的包名改成自己的包名，需要将uses-permission android:name="com.paulniu.mypush.permission.MIPUSH_RECEIVE" />改成自己的包名
2. 添加小米推送系统广播接收器和服务
```
<!--小米推送开始-->
        <service
            android:name="com.xiaomi.push.service.XMJobService"
            android:enabled="true"
            android:exported="false"
            android:permission="android.permission.BIND_JOB_SERVICE"
            android:process=":pushservice" />

        <service
            android:name="com.xiaomi.push.service.XMPushService"
            android:enabled="true"
            android:process=":pushservice" />

        <service
            android:name="com.xiaomi.mipush.sdk.PushMessageHandler"
            android:enabled="true"
            android:exported="true" />
        <service
            android:name="com.xiaomi.mipush.sdk.MessageHandleService"
            android:enabled="true" />

        <receiver
            android:name="com.xiaomi.push.service.receivers.NetworkStatusReceiver"
            android:exported="true">
            <intent-filter>
                <action android:name="android.net.conn.CONNECTIVITY_CHANGE" />

                <category android:name="android.intent.category.DEFAULT" />
            </intent-filter>
        </receiver>
        <receiver
            android:name="com.xiaomi.push.service.receivers.PingReceiver"
            android:exported="false"
            android:process=":pushservice">
            <intent-filter>
                <action android:name="com.xiaomi.push.PING_TIMER" />
            </intent-filter>
        </receiver>
        <!--小米推送结束-->
```
这部分内容是固定的，直接复制使用即可
3. 创建一个新的类IMiPushMessageReceiver，并且让这个类集成PushMessageReceiver，小米推送注册，推送接收，推送点击都是在这个接收器中实现，接收器在Manifest文件中声明如下：
```
        <!--自定义MessageReceiver-->
        <receiver
            android:name="com.paulniu.mypush.IMiPushMessageReceiver"
            android:exported="true">
            <intent-filter>
                <action android:name="com.xiaomi.mipush.RECEIVE_MESSAGE" />
            </intent-filter>
            <intent-filter>
                <action android:name="com.xiaomi.mipush.MESSAGE_ARRIVED" />
            </intent-filter>
            <intent-filter>
                <action android:name="com.xiaomi.mipush.ERROR" />
            </intent-filter>
        </receiver>
```

### 小米推送实现
这里需要在IMiPushMessageReceiver类中重写多个方法，具体使用看代码:
```
public class IMiPushMessageReceiver extends PushMessageReceiver {

    /**
     * 1.PushMessageReceiver是一个抽象类，通过集成该类，实现小米推送注册之后的回调
     * 2.需要将IMIPushMessageReceiver注册到Manifest文件中
     * 3.通过onReceivePassThroughMessage方法处理服务器向客户端发送的透传消息
     * 4.通过onNotificationMessageClicked方法服务器向客户端发送通知消息，该回调方法会在用户点击通知之后触发
     * 5.通过onNotificationMessageArrived方法服务器向客户端发送通知消息，该回调方法会在通知消息到达客户端之后触发。另外应用在前台时不展示通知也在该方法中调用
     * 6.通过onCommandResult方法来接收客户端向服务器发送命令后的响应结果
     * 7.通过onReceiveRegisterResult方法接收客户端向服务器注册推送后响应的结果
     * 8.当前所有的操作都没有运行在UI线程
     */

    public String regId = "";

    /**
     * 客户端注册推送之后的回调
     *
     * @param context
     * @param miPushCommandMessage
     */
    @Override
    public void onReceiveRegisterResult(Context context, MiPushCommandMessage miPushCommandMessage) {
        List<String> arguments = miPushCommandMessage.getCommandArguments();
        String cmdArg = ((arguments != null && arguments.size() > 0)) ? arguments.get(0) : "";
        //注册成功
        if (MiPushClient.COMMAND_REGISTER.equals(miPushCommandMessage.getCommand()) && ErrorCode.SUCCESS == miPushCommandMessage.getResultCode() && !TextUtils.isEmpty(cmdArg)) {
            Log.e("NPL", "小米推送注册成功，regId=" + regId);
            regId = MiPushClient.getRegId(App.getContext());
            Message msg = Message.obtain();
            msg.obj = regId;
            msg.what = 1;
            App.mHandler.sendMessage(msg);
        } else {
            //注册失败
            Log.e("NPL", "小米推送注册失败");
        }
    }

    /**
     * 客户端接收到推送之后触发，包括如果当前应用处于前台，则可以不用在通知栏展示
     *
     * @param context
     * @param miPushMessage
     */
    @Override
    public void onNotificationMessageArrived(Context context, MiPushMessage miPushMessage) {
        //判断应用是否处于前台，如果处于前台，则可以显示弹窗，如果不在前台，在通知栏展示
        if (true) {

        } else {
            super.onNotificationMessageArrived(context, miPushMessage);
        }
    }

    /**
     * 点击通知栏的推送
     *
     * @param context
     * @param miPushMessage
     */
    @Override
    public void onNotificationMessageClicked(Context context, MiPushMessage miPushMessage) {
        //如果当前推送的消息为null，则不要执行任何操作
        if (miPushMessage == null || miPushMessage.getExtra() == null || miPushMessage.getExtra().size() <= 0 || TextUtils.isEmpty(miPushMessage.getTitle())) {
            return;
        }
        String content = miPushMessage.getContent();
        HashMap<String, String> hm = (HashMap<String, String>) miPushMessage.getExtra();
        //根据传递过来的数据，可以执行不同的操作
    }

    /**
     * 客户端接收到透传消息
     *
     * @param context
     * @param miPushMessage
     */
    @Override
    public void onReceivePassThroughMessage(Context context, MiPushMessage miPushMessage) {
        if (miPushMessage == null || TextUtils.isEmpty(miPushMessage.getContent())) {
            return;
        }
        String content = miPushMessage.getContent();
        //通常需要将content转换成Json数据，然后在根据Json数据执行不同的操作
    }
}
```
这样既完成了小米推送的集成。

Oppo推送服务，也是在Oppo手机中使用的比较多。而且现在Oppo和Vivo两款手机最早提出美颜功能，所以这两款手机在市场上的占有率还是比较高的。
Oppo推送目前已经开发注册，Vivo目前只对部分应用开发了推送服务功能。所以目前部分应用无法使用vivo推送的暂时不要着急，后面都会有的。

# OPPO推送
首先还是先去Oppo开放平台注册自己的账号和添加应用，然后获取AppKey和AppId等信息，这些操作就不再一一叙述，具体查看[官网](https://open.oppomobile.com/)
这里同时将Oppo平台使用指南的官方地址写出来，其实就是如何使用控制台，后面会针对几个特殊的内容，比如传递自定义数据等截图具体分析。[使用指南](https://open.oppomobile.com/wiki/doc#id=10198)
## OPPO推送SDK下载
Oppo为我们提供了[下载地址](https://open.oppomobile.com/wiki/doc#id=10201)，其中还包括一个官方提供的demo，不过这个demo我没有仔细看过。希望以后有时间可以将demo仔细的看一遍
## OPPO推送集成
将下载的SDK中Oppo推送的jar包拷贝出来，直接放在libs文件夹中。
在执行Oppo推送之前，最好先判断当手机是否是Oppo手机，判断使用下面的内容
```
if(android.os.Build.BRAND.toLowerCase().contains("oppo")){
	initOppoPush();
}
```
然后在Application中执行Oppo推送初始化
```
    /**
     * 初始化OPPO推送
     */
    protected void initOppoPush() {
        //在执行Oppo推送注册之前，需要先判断当前平台是否支持Oppo推送
        if (PushManager.isSupportPush(this)) {
            PushManager.getInstance().register(this, OPPO_APPID, OPPO_APPKEY, new PushAdapter() {
                @Override
                public void onRegister(int i, String s) {
                    if (i == ErrorCode.SUCCESS) {
                        //注册成功
                        Log.e("NPL", "注册成功，registerId=" + s);
                    } else {
                        //注册失败
                        Log.e("NPL", "注册失败");
                    }
                }
            });
        }
    }
```
到这里基本的操作已经完成了，当通过Oppo控制台推送一些通知时，可以通过点击启动App。但是如果我们想要传递的内容不同而执行不同的操作，例如可以打开web页面，可以跳转到不同的Activity，那么这是我们需要另外一Activity。这个Activity没有实际的含义，只是起到一个承接的作用，即当点击通知时，获取传递的数据，然后根据不同的数据跳转到不同的页面。
需要在Manifest文件中添加这个Activity的声明，具体声明代码如下：
```
 <activity
            android:name=".OPPOPushMessageActivity"
            android:configChanges="keyboardHidden|orientation"
            android:launchMode="singleTask"
            android:screenOrientation="portrait"
            android:theme="@style/AppTheme">
            <intent-filter>
                <action android:name="com.paulniu.mypush.oppopush" />
                <category android:name="android.intent.category.DEFAULT" />
            </intent-filter>
        </activity>
```
> 其中的action android:name="com.paulniu.mypush.oppopush" />是当我们发送应用内页时的标识

如图所示：
![在这里插入图片描述](/assets/push/push06.png)
 2019年2月15日15:10:13 OPPO官网对这个地方的样式做了调整
 1.如果传递的是Intent Action ，如图 
![在这里插入图片描述](/assets/push/push07.png)
则在使用的时候需要在Manifest文件中设置如图
![在这里插入图片描述](/assets/push/push08.png)
2.如果使用的是Activity，如图
 ![在这里插入图片描述](/assets/push/push09.png)
 这个就不多说了
 3.如果使用的是Scheme,如图
 ![在这里插入图片描述](/assets/push/push10.png)
 那么在Manifest文件中使用的应该是如下所示
 
![在这里插入图片描述](/assets/push/push11.png)


并且当我们选择插入的键值，就需要在客户端获取到并且解析。那么就需要在OPPOPushMessageActivity中解析。
```
public class OPPOPushMessageActivity extends AppCompatActivity {

    public String type = "";
    public String url = "";
    public String innerUrl = "";

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        // 取参数值
        if (getIntent().getExtras() != null) {
            // 取参数值
            Bundle bundle = getIntent().getExtras();
            Set<String> set = bundle.keySet();
            HashMap<String, String> hm = new HashMap<>();
            if (set != null) {
                for (String key : set) {
                    hm.put(key, bundle.getString(key));
                }
            }
            Log.e("NPL", "hm的值是：" + hm.toString());
            if (hm.size() > 0) {
                //解析当前的HashMap对象，可以获取具体的数据
                if (set.contains("type")) {
                    type = hm.get("type");
                }
                if (set.contains("url")) {
                    url = hm.get("url");
                }
                if (set.contains("innerUrl")) {
                    innerUrl = hm.get("innerUrl");
                }
                //根据当前type类型去执行不同的操作
            }
        }
    }
}

```

# vivo推送
vivo推送官网地址如下： https://dev.vivo.com.cn/home
由于vivo推送目前还没有全面开发，所以这里使用的测试版本
对于vivo的推送平台的使用，这里贴出官网地址：[vivo推送平台使用](https://dev.vivo.com.cn/documentCenter/doc/151 )
vivo推送SDK下载：[下载地址](https://swsdl.vivo.com.cn/appstore/developer/uploadfile/20181119/20181119180214292.zip) 这里面包括API文档，demo，SDK等内容
> 该说不说，还是vivo提供的文档最好，各种清晰，方便操作，对于我这样的小白很适用
## vivo推送集成
### SDK
首先将SDK拷贝出来放到libs文件夹中，然后重新编译
### 声明
在Manifest文件中对于Vivo推送需要添加部分权限和声明
权限：
```
<!--vivo推送所需要的权限-->
<uses-permission android:name="android.permission.INTERNET" />
```
声明：
```
 <!--vivo推送开始-->
        <!--vivo推送配置项-->
        <meta-data
            android:name="com.vivo.push.api_key"
            android:value="xxxxxxxxxxxxx" />
        <meta-data
            android:name="com.vivo.push.app_id"
            android:value="xxxxxxx" />
        <!--推送服务需要配置的 service、activity-->
        <service
            android:name="com.vivo.push.sdk.service.CommandClientService"
            android:exported="true" />
        <activity
            android:name="com.vivo.push.sdk.LinkProxyClientActivity"
            android:exported="false"
            android:screenOrientation="portrait"
            android:theme="@android:style/Theme.Translucent.NoTitleBar" />
        <!--vivo推送结束-->
```
之后需要在Application中执行Vivo推送的注册
```
 /**
     * 初始化vivo推送
     */
    protected void initVivoPush() {
        //初始化vivo推送
        PushClient.getInstance(this).initialize();
        //并且打开推送服务
        PushClient.getInstance(this).turnOnPush(new IPushActionListener() {
            @Override
            public void onStateChanged(int i) {
                if (i == 0) {
                    Log.e("NPL", "打开推送服务成功");
                } else {
                    Log.e("NPL", "打开推送服务失败");
                }
            }
        });
    }
```
这样就完成了基本操作。但如果需要相应点击事件，则需要创建一个广播接收器，并在里面实现相应的方法
广播接收器的注册：
```
<receiver android:name=".IVivoPushMessageReceiver">
            <intent-filter>
                <!-- 接收push消息 -->
                <action android:name="com.vivo.pushclient.action.RECEIVE" />
            </intent-filter>
        </receiver>
```
广播接收器的代码实现：
```
public class IVivoPushMessageReceiver extends OpenClientPushMessageReceiver {
    @Override
    public void onNotificationMessageClicked(Context context, UPSNotificationMessage upsNotificationMessage) {
        long msgId;
        String customeContent = "";
        if (upsNotificationMessage != null) {
            msgId = upsNotificationMessage.getMsgId();
            customeContent = upsNotificationMessage.getSkipContent();
            Log.e("NPL", "获取通知内容如下:msgId = " + msgId + ";customeContent=" + customeContent);
        }
    }
    @Override
    public void onReceiveRegId(Context context, String s) {
        if (TextUtils.isEmpty(s)) {
            //获取regId失败
            Log.e("NPL", "获取RegId失败");
        } else {
            Log.e("NPL", "获取RegId成功，regid = " + s);
        }
    }
}
```
这样就实现了vivo的推送。





