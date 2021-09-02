---
title: 重拾android路(八) UI
date: 2016-05-15 23:24:36
tags:
  - android
---

Android中的UI是非常重要的方面，直接关系到用户体验，所以我打算把之前用到的，比较好的UI拿出来复习一下。
<!--more-->
# 通知 Notification
通知是一个可以在应用程序正常的用户界面之外显示给用户的消息。
通知发出时，它首先出现在状态栏的通知区域中，用户打开通知抽屉可查看通知详情。通知区域和通知抽屉都是用户可以随时查看的系统控制区域

简单的说一下，我们这里使用的是Android4.1以上的版本，比较低的版本暂时不考虑，因为现在市面上很难再见到4.0以下的版本了
## 创建notification
使用创建者模式创建notification，并且设置三个属性，这三个属性是必须的

```
Notification notification = new Notification.Builder(NotificationActivity.this)
                        .setSmallIcon(R.mipmap.ic_launcher) //设置小图标
                        .setContentText("这是内容")  //设置内容
                        .setContentTitle("这是标题") // 设置标题
                        .build();
```

## 创建notification管理器

```
//获取notification管理器
                NotificationManager manager = (NotificationManager) getSystemService(NOTIFICATION_SERVICE);
```

## 使用notification管理器将notification弹出显示

```
//将notification显示
                manager.notify(0,notification);
```

> 其中0表示当前id，用来唯一表示notification

## pendingIntent

到目前为止，我们的notification还不能显示，因为我们还没有为这个notification设置一个显示的操作，也就是说，如果当前的notification没有具体的操作的话，是无意义的。一般情况下，我们会为这个notification设置一个需要跳转的页面或者需要打开浏览器的网址

```
PendingIntent intent = PendingIntent.getActivity(NotificationActivity.this,0,new Intent(NotificationActivity.this,TestActivity.class),PendingIntent.FLAG_UPDATE_CURRENT);
```

PendingIntent字面意思有延迟的意思，用于某个事件结束之后执行特定的Action。
PendingIntent 是 Android 系统管理并持有的用于描述和获取原始数据的对象的标志(引用)。也就是说，即便创建该PendingIntent对象的进程被杀死了，这个PendingItent对象在其他进程中还是可用的。
日常使用中的短信、闹钟等都用到了 PendingIntent
PendingIntent有下面的三种方式获取：

- //获取一个用于启动 Activity 的 PendingIntent 对象
public static PendingIntent getActivity(Context context, int requestCode, Intent intent, int flags);

- //获取一个用于启动 Service 的 PendingIntent 对象
public static PendingIntent getService(Context context, int requestCode, Intent intent, int flags);

- //获取一个用于向 BroadcastReceiver 广播的 PendingIntent 对象
public static PendingIntent getBroadcast(Context context, int requestCode, Intent intent, int flags)

除此之外，PendingIntent有下面几个flag
- FLAG_CANCEL_CURRENT:如果当前系统中已经存在一个相同的 PendingIntent 对象，那么就将先将已有的 PendingIntent 取消，然后重新生成一个 PendingIntent 对象。

- FLAG_NO_CREATE:如果当前系统中不存在相同的 PendingIntent 对象，系统将不会创建该 PendingIntent 对象而是直接返回 null 。

- FLAG_ONE_SHOT:该 PendingIntent 只作用一次。

- FLAG_UPDATE_CURRENT:如果系统中已存在该 PendingIntent 对象，那么系统将保留该 PendingIntent 对象，但是会使用新的 Intent 来更新之前 PendingIntent 中的 Intent 对象数据，例如更新 Intent 中的 Extras 。

## 更新Notification

更新通知很简单，只需要再次发送相同 ID 的通知即可，如果之前的通知还未被取消，则会直接更新该通知相关的属性；如果之前的通知已经被取消，则会重新创建一个新通知。更新通知跟发送通知使用相同的方式

```
//设置当前可以点击之后消失
notification.flags = Notification.FLAG_AUTO_CANCEL;
```

### 取消Notification

取消Notification有五种方式：

1. 点击通知栏的清除按钮，会清除所有可清除的通知
2. 设置了 setAutoCancel() 或 FLAG_AUTO_CANCEL 的通知，点击该通知时会清除它
3. 通过 NotificationManager 调用 cancel(int id) 方法清除指定 ID 的通知
4. 通过 NotificationManager 调用 cancel(String tag, int id) 方法清除指定 TAG 和 ID 的通知
5. 通过 NotificationManager 调用 cancelAll() 方法清除所有该应用之前发送的通知

如果你是通过 NotificationManager.notify(String tag, int id, Notification notify) 方法创建的通知，那么只能通过 NotificationManager.cancel(String tag, int id) 方法才能清除对应的通知，调用NotificationManager.cancel(int id) 无效

下面是一个完整的例子：
XML布局文件：

```
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_height="match_parent"
    android:layout_width="match_parent"
    android:orientation="vertical">


    <ScrollView xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical">

            <Button
                android:id="@+id/btn_remove_all_notification"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="清除所有 Notification" />

            <Button
                android:id="@+id/btn_send_notification"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="发送 id = 1 的通知" />

            <Button
                android:id="@+id/btn_remove_notification"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="移除 id = 1 的通知" />

            <Button
                android:id="@+id/btn_send_notification_with_tag"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="发送 ID = 1,TAG = 'littlejie' 的通知" />

            <Button
                android:id="@+id/btn_remove_notification_with_tag"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="移除 ID = 1,TAG = 'littlejie' 的通知" />

            <Button
                android:id="@+id/btn_send_ten_notification"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="我要一口气发十个通知" />

            <Button
                android:id="@+id/btn_send_flag_no_clear_notification"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="发送一个 FLAG_NO_CLEAR 的通知,不能被移除" />

            <Button
                android:id="@+id/btn_send_flag_ongoing_event_notification"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="发送一个 FLAG_ONGONG_EVENT 的通知,不能被移除" />

            <Button
                android:id="@+id/btn_send_flag_auto_cancecl_notification"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="发送一个 FLAG_AUTO_CANCEL 的通知,能移除" />

        </LinearLayout>

    </ScrollView>
  <LinearLayout>
```

java文件

```
public class SimpleNotificationActivity extends Activity implements View.OnClickListener {

    //Notification.FLAG_FOREGROUND_SERVICE    //表示正在运行的服务
    public static final String NOTIFICATION_TAG = "littlejie";
    public static final int DEFAULT_NOTIFICATION_ID = 1;

    private NotificationManager mNotificationManager;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_simple_notification);

        findViewById(R.id.btn_remove_all_notification).setOnClickListener(this);
        findViewById(R.id.btn_send_notification).setOnClickListener(this);
        findViewById(R.id.btn_remove_notification).setOnClickListener(this);
        findViewById(R.id.btn_send_notification_with_tag).setOnClickListener(this);
        findViewById(R.id.btn_remove_notification_with_tag).setOnClickListener(this);
        findViewById(R.id.btn_send_ten_notification).setOnClickListener(this);
        findViewById(R.id.btn_send_flag_no_clear_notification).setOnClickListener(this);
        findViewById(R.id.btn_send_flag_ongoing_event_notification).setOnClickListener(this);
        findViewById(R.id.btn_send_flag_auto_cancecl_notification).setOnClickListener(this);

        mNotificationManager = (NotificationManager) this.getSystemService(Context.NOTIFICATION_SERVICE);
    }

    @Override
    public void onClick(View v) {
        switch (v.getId()) {
            case R.id.btn_remove_all_notification:
                //移除当前 Context 下所有 Notification,包括 FLAG_NO_CLEAR 和 FLAG_ONGOING_EVENT
                mNotificationManager.cancelAll();
                break;
            case R.id.btn_send_notification:
                //发送一个 Notification,此处 ID = 1
                sendNotification();
                break;
            case R.id.btn_remove_notification:
                //移除 ID = 1 的 Notification,注意:该方法只针对当前 Context。
                mNotificationManager.cancel(DEFAULT_NOTIFICATION_ID);
                break;
            case R.id.btn_send_notification_with_tag:
                //发送一个 ID = 1 并且 TAG = littlejie 的 Notification
                //注意:此处发送的通知与 sendNotification() 发送的通知并不冲突
                //因为此处的 Notification 带有 TAG
                sendNotificationWithTag();
                break;
            case R.id.btn_remove_notification_with_tag:
                //移除一个 ID = 1 并且 TAG = littlejie 的 Notification
                //注意:此处移除的通知与 NotificationManager.cancel(int id) 移除通知并不冲突
                //因为此处的 Notification 带有 TAG
                mNotificationManager.cancel(NOTIFICATION_TAG, DEFAULT_NOTIFICATION_ID);
                break;
            case R.id.btn_send_ten_notification:
                //连续发十条 Notification
                sendTenNotifications();
                break;
            case R.id.btn_send_flag_no_clear_notification:
                //发送 ID = 1, flag = FLAG_NO_CLEAR 的 Notification
                //下面两个 Notification 的 ID 都为 1,会发现 ID 相等的 Notification 会被最新的替换掉
                sendFlagNoClearNotification();
                break;
            case R.id.btn_send_flag_auto_cancecl_notification:
                sendFlagOngoingEventNotification();
                break;
            case R.id.btn_send_flag_ongoing_event_notification:
                sendFlagAutoCancelNotification();
                break;
        }
    }

    /**
     * 发送最简单的通知,该通知的ID = 1
     */
    private void sendNotification() {
        //这里使用 NotificationCompat 而不是 Notification ,因为 Notification 需要 API 16 才能使用
        //NotificationCompat 存在于 V4 Support Library
        NotificationCompat.Builder builder = new NotificationCompat.Builder(this)
                .setSmallIcon(R.mipmap.ic_launcher)
                .setContentTitle("Send Notification")
                .setContentText("Hi,My id is 1");
        mNotificationManager.notify(DEFAULT_NOTIFICATION_ID, builder.build());
    }

    /**
     * 使用notify(String tag, int id, Notification notification)方法发送通知
     * 移除对应通知需使用 cancel(String tag, int id)
     */
    private void sendNotificationWithTag() {
        NotificationCompat.Builder builder = new NotificationCompat.Builder(this)
                .setSmallIcon(R.mipmap.ic_launcher)
                .setContentTitle("Send Notification With Tag")
                .setContentText("Hi,My id is 1,tag is " + NOTIFICATION_TAG);
        mNotificationManager.notify(NOTIFICATION_TAG, DEFAULT_NOTIFICATION_ID, builder.build());
    }

    /**
     * 循环发送十个通知
     */
    private void sendTenNotifications() {
        for (int i = 0; i < 10; i++) {
            NotificationCompat.Builder builder = new NotificationCompat.Builder(this)
                    .setSmallIcon(R.mipmap.ic_launcher)
                    .setContentTitle("Send Notification Batch")
                    .setContentText("Hi,My id is " + i);
            mNotificationManager.notify(i, builder.build());
        }
    }

    /**
     * 设置FLAG_NO_CLEAR
     * 该 flag 表示该通知不能被状态栏的清除按钮给清除掉,也不能被手动清除,但能通过 cancel() 方法清除
     * Notification.flags属性可以通过 |= 运算叠加效果
     */
    private void sendFlagNoClearNotification() {
        NotificationCompat.Builder builder = new NotificationCompat.Builder(this)
                .setSmallIcon(R.mipmap.ic_launcher)
                .setContentTitle("Send Notification Use FLAG_NO_CLEAR")
                .setContentText("Hi,My id is 1,i can't be clear.");
        Notification notification = builder.build();
        //设置 Notification 的 flags = FLAG_NO_CLEAR
        //FLAG_NO_CLEAR 表示该通知不能被状态栏的清除按钮给清除掉,也不能被手动清除,但能通过 cancel() 方法清除
        //flags 可以通过 |= 运算叠加效果
        notification.flags |= Notification.FLAG_NO_CLEAR;
        mNotificationManager.notify(DEFAULT_NOTIFICATION_ID, notification);
    }

    /**
     * 设置FLAG_AUTO_CANCEL
     * 该 flag 表示用户单击通知后自动消失
     */
    private void sendFlagAutoCancelNotification() {
        //设置一个Intent,不然点击通知不会自动消失
        Intent resultIntent = new Intent(this, MainActivity.class);
        PendingIntent resultPendingIntent = PendingIntent.getActivity(
                this, 0, resultIntent, PendingIntent.FLAG_UPDATE_CURRENT);
        NotificationCompat.Builder builder = new NotificationCompat.Builder(this)
                .setSmallIcon(R.mipmap.ic_launcher)
                .setContentTitle("Send Notification Use FLAG_AUTO_CLEAR")
                .setContentText("Hi,My id is 1,i can be clear.")
                .setContentIntent(resultPendingIntent);
        Notification notification = builder.build();
        //设置 Notification 的 flags = FLAG_NO_CLEAR
        //FLAG_AUTO_CANCEL 表示该通知能被状态栏的清除按钮给清除掉
        //等价于 builder.setAutoCancel(true);
        notification.flags |= Notification.FLAG_AUTO_CANCEL;
        mNotificationManager.notify(DEFAULT_NOTIFICATION_ID, notification);
    }

    /**
     * 设置FLAG_ONGOING_EVENT
     * 该 flag 表示发起正在运行事件（活动中）
     */
    private void sendFlagOngoingEventNotification() {
        NotificationCompat.Builder builder = new NotificationCompat.Builder(this)
                .setSmallIcon(R.mipmap.ic_launcher)
                .setContentTitle("Send Notification Use FLAG_ONGOING_EVENT")
                .setContentText("Hi,My id is 1,i can't be clear.");
        Notification notification = builder.build();
        //设置 Notification 的 flags = FLAG_NO_CLEAR
        //FLAG_ONGOING_EVENT 表示该通知通知放置在正在运行,不能被手动清除,但能通过 cancel() 方法清除
        //等价于 builder.setOngoing(true);
        notification.flags |= Notification.FLAG_ONGOING_EVENT;
        mNotificationManager.notify(DEFAULT_NOTIFICATION_ID, notification);
    }

}
```

## 其他内容

上面的例子都是关于一些基本操作的，但是有时候我们的通知Notification可能需要有一些特殊的样式，比如当有新的通知的时候，应该有一些提示声音，震动等操作。
Notification 有震动、响铃、呼吸灯三种响铃效果，可以通过 setDefaults(int defualts) 方法来设置。 Default 属性有以下四种，一旦设置了 Default 效果，自定义的效果就会失效

```
//设置系统默认提醒效果，一旦设置默认提醒效果，则自定义的提醒效果会全部失效。具体可看源码
//添加默认震动效果,需要申请震动权限
//<uses-permission android:name="android.permission.VIBRATE" />
Notification.DEFAULT_VIBRATE

//添加系统默认声音效果，设置此值后，调用setSound()设置自定义声音无效
Notification.DEFAULT_SOUND

//添加默认呼吸灯效果，使用时须与 Notification.FLAG_SHOW_LIGHTS 结合使用，否则无效
Notification.DEFAULT_LIGHTS

//添加上述三种默认提醒效果
Notification.DEFAULT_ALL
```

除了以上几种设置 Notification 默认通知效果，还可以通过以下几种 FLAG 设置通知效果。

```
//提醒效果常用 Flag
//三色灯提醒，在使用三色灯提醒时候必须加该标志符
Notification.FLAG_SHOW_LIGHTS

//发起正在运行事件（活动中）
Notification.FLAG_ONGOING_EVENT

//让声音、振动无限循环，直到用户响应 （取消或者打开）
Notification.FLAG_INSISTENT

//发起Notification后，铃声和震动均只执行一次
Notification.FLAG_ONLY_ALERT_ONCE

//用户单击通知后自动消失
Notification.FLAG_AUTO_CANCEL

//只有调用NotificationManager.cancel()时才会清除
Notification.FLAG_NO_CLEAR

//表示正在运行的服务
Notification.FLAG_FOREGROUND_SERVICE
```

我们可以写一个工具类，帮助我们实现各种效果

```
public class NotificationUtils {

    private static final int NOTIFICATION_ID = 1;

    Context context;
    NotificationManager manager;

    public NotificationUtils(Context context){
        this.context = context;
        manager = (NotificationManager) context.getSystemService(Context.NOTIFICATION_SERVICE);
    }

    /**
     * 普通通知效果
     */
    public void showSimpleNotification(){
        NotificationCompat.Builder builder = new NotificationCompat.Builder(context)
                .setSmallIcon(R.mipmap.ic_launcher)
                .setContentText("我是文本信息")
                .setContentTitle("我是标题");

        manager.notify(NOTIFICATION_ID,builder.build());

    }

    /**
     * 展示有自定义铃声效果的通知
     * 使用系统自带的铃声效果:Uri.withAppendedPath(Audio.Media.INTERNAL_CONTENT_URI, "6");
     */
    public void showNotificationWithRing(){
        NotificationCompat.Builder builder = new NotificationCompat.Builder(context)
                .setSmallIcon(R.mipmap.ic_launcher)
                .setContentTitle("我是标题")
                .setContentText("我是内容")
                .setSound(Uri.withAppendedPath(MediaStore.Audio.Media.INTERNAL_CONTENT_URI,"6"));
        manager.notify(2,builder.build());
    }

    /**
     * 震动效果
     * <uses-permission android:name="android.permission.VIBRATE" />
     */
    public void showNotifyWithVibrate(){
        long [] vibrate = new long[]{0,500,1000,1500};
        NotificationCompat.Builder builder = new NotificationCompat.Builder(context)
                .setSmallIcon(R.mipmap.ic_launcher)
                .setContentText("我是内容")
                .setContentTitle("我是标题")
                .setVibrate(vibrate);

        manager.notify(3,builder.build());
    }

    public void showNotifyWithLights(){
        final NotificationCompat.Builder builder = new NotificationCompat.Builder(context)
                .setSmallIcon(R.mipmap.ic_launcher)
                .setContentTitle("我是带有呼吸灯效果的通知")
                .setContentText("一闪一闪亮晶晶~")
                //ledARGB 表示灯光颜色、 ledOnMS 亮持续时间、ledOffMS 暗的时间
                .setLights(0xFF0000, 3000, 3000);
        Notification notify = builder.build();
        //只有在设置了标志符Flags为Notification.FLAG_SHOW_LIGHTS的时候，才支持呼吸灯提醒。
        notify.flags = Notification.FLAG_SHOW_LIGHTS;
        //设置lights参数的另一种方式
        //notify.ledARGB = 0xFF0000;
        //notify.ledOnMS = 500;
        //notify.ledOffMS = 5000;
        //使用handler延迟发送通知,因为连接usb时,呼吸灯一直会亮着
        Handler handler = new Handler();
        handler.postDelayed(new Runnable() {
            @Override
            public void run() {
                manager.notify(4, builder.build());
            }
        }, 10000);
    }

    /**
     * 显示带有默认铃声、震动、呼吸灯效果的通知
     * 如需实现自定义效果,请参考前面三个例子
     */
    private void showNotifyWithMixed() {
        NotificationCompat.Builder builder = new NotificationCompat.Builder(context)
                .setSmallIcon(R.mipmap.ic_launcher)
                .setContentTitle("我是有铃声+震动+呼吸灯效果的通知")
                .setContentText("我是最棒的~")
                //等价于setDefaults(Notification.DEFAULT_SOUND | Notification.DEFAULT_LIGHTS | Notification.DEFAULT_VIBRATE);
                .setDefaults(Notification.DEFAULT_ALL);
        manager.notify(5, builder.build());
    }

    /**
     * 清除所有通知
     */
    public void clearAllNotification(){
        manager.cancelAll();
    }

}
```

## Android8.0的通知

在Android O中，使用了新的方式完成通知。
Android O 引入了 通知渠道（Notification Channels），以提供统一的系统来帮助用户管理通知，如果是针对 android O 为目标平台时，必须实现一个或者多个通知渠道，以向用户显示通知。若并不以 Android O 为目标平台，当应用运行在 android O 设备上时，其行为将与运行在 Android 7.0 上时相同。
开发者可以为需要发送的每个不同的通知类型创建一个通知渠道。还可以创建通知渠道来反映应用的用户做出的选择。例如，可以为聊天应用的用户创建的每个聊天组建立单独的通知渠道。
Android O 的用户可以使用一致的系统 UI 管理大多数与通知有关的设置。所有发布至通知渠道的通知都具有相同的行为。当用户修改任何下列特性的行为时，修改将作用于通知渠道：

- 重要性
- 光
- 声音
- 震动
- 在锁屏上显示
- 替换免打扰模式

用户可以访问 Settings（右划一段距离通知可以看到Settings），或长按通知来更改这些行为，甚至可以随时屏蔽通知渠道。一旦在创建完某个通知渠道并将其提交到通知管理器后，便无法通过编程方式修改通知渠道的行为，这些设置由用户掌控

### 通知的优先级和重要性
Android O 弃用了为单个通知设置优先级的功能。创建通知渠道时可以设置建议重要性级别。为通知渠道指定的重要性级别适用于发布至该渠道的所有通知消息。可以配置五个级别中的一个，这些级别代表着通知渠道可以打断用户的程度，范围是 IMPORTANCE_NONE(0)至 IMPORTANCE_HIGH(4)。默认重要性级别为 3：在所有位置显示，发出提示音，但不会对用户产生视觉干扰。创建通知渠道后，只有系统可以修改其重要性。用户可以在设置中找到

### 创建渠道

创建渠道需要执行以下操作

1. 构建一个在软件包内具有唯一 ID 的通知渠道对象。
2. 为该通知渠道对象配置所需的任何初始设置（例如提示音以及对用户可见的可选说明）。
3. 将通知渠道对象提交到通知管理器

```
/**
         * Android8.0通知渠道
         */
        NotificationManager manager = (NotificationManager) getSystemService(NOTIFICATION_SERVICE);
        //渠道ID
        String channelId = "channelid";
        // 用户可以看到的通知渠道的名字.
        CharSequence name = "我是标题";
        // 用户可以看到的通知渠道的描述
        String description = "我是描述内容";
        //优先级
        int importment = NotificationManager.IMPORTANCE_HIGH;
        NotificationChannel channel = new NotificationChannel(channelId,name,importment);
        channel.setDescription(description);
        // 设置通知出现时的闪灯（如果 android 设备支持的话）
        channel.enableLights(true);
        channel.setLightColor(Color.RED);
        // 设置通知出现时的震动（如果 android 设备支持的话）
        channel.enableVibration(true);
        channel.setVibrationPattern(new long[]{100, 200, 300, 400, 500, 400, 300, 200, 400});
        //最后在notificationmanager中创建该通知渠道
        manager.createNotificationChannel(channel);
```

可以通过调用 createNotificationChannels(List < NotificationChannel > channels)
一次性创建多个通知渠道。

# android support v4包解析

## Android supportv4如何支持低版本

Android support v4这个包是为Android 1.6(API版本为4)及以上的版本设计的（从android-support-v4-24.2.0开始，V4包支持的最低版本是Android 2.3即API Level 9），该支持库可以让在旧版本 Android 平台上运行的应用，支持新版本平台推出的功能。

举个Fragment的例子说明一下，假设我们某个应用支持的最低版本是：minSdkVersion=8，但是应用中用到了android.app.Fragment类，而Fragment类是在Android 11的时候才开始加入的，那么当我们的应用运行在Android 11以下的手机就会出现问题，那么如何能让Fragment在低于11的手机上也能正常使用呢？我们需要引入android.support.v4包中android.support.v4.app.Fragment来替换掉原来用到的android.app.Fragment类，android.support.v4.app.Fragment和android.app.Fragment有一样的效果，但是它能在低于11的手机上正常使用，这就是support支持库提供的功能，能兼容低版本的Android平台。

android.support.v4包支持的最低版本是Android 4，v4的意思是就是支持最低版本是4，如果你要使用Fragment，最低版本只兼容到4了。

拿上面的例子来说：应用中的minSdkVersion=8，为了兼容低版本的手机，引入了android.support.v4包中android.support.v4.app.Fragment

1. 当运行在Android版本是4-10手机上，手机Android框架没有提供Fragmeng提供的功能：则android.support.v4支持库会调用自身android.support.v4.app.Fragment；

2. 当运行在Android版本是11及以上的手机上，手机Android框架提供了Fragmeng提供的功能：则android.support.v4支持库会调用手机Android框架android.app.Fragment。

也就是说，如果应用调用其中一个支持类的方法，则支持库的行为将取决于运行应用的手机的Android 版本。如果手机Android框架提供必要的功能，则支持库将通过调用手机Android框架执行任务。如果应用在旧版本的 Android 上运行，且手机Android框架未提供所需的功能，则支持库自身可能会尝试提供相应的功能或什么都不做。无论是哪一种情形，应用通常都不需要检查其在哪一版本的 Android 上运行，而是通过支持库执行检查并选择适当的行为。

还有一些android.support.v4中类，比如ViewPager等，不管在Android那个版本，都没有这个类，所以要用到ViewPager，就必须引用android.support.v4包了。

## android v4包版本介绍
> 随着系统的迭代Android 1.6的设备已经很少了，官方在Support Library 24.2.0版本的时候移除了对Android 2.2(API Level 8)及以下版本的支持，所以从Android Support v4 24.2.0开始，V4包支持的最低版本是Android 2.3即API Level 9

我们可以发现android-support-v4后面都跟着版本号：比如android-support-v4-23.0.0 (对应Android Api Level 23)，如果不清楚这个版本号，在开发中也会带来很多问题。

最常见的问题就是已经引入了android-support-v4包，但是某个类或者某个方法却找不到，这个原因应该就是版本号不对了。

比如我们在targetSdkVersion < 23的时，用到android.support.v4.content.PermissionChecker这个类来检查权限，但是引入了android-support-v4-22.2.1.jar后，却找不到PermissionChecker类，原因就是PermissionChecker是23.0.0版本才加入的，所以引入android-support-v4-23.0.0.jar就行了。

遇到这种问题，可以去Android官方中文网站找到对应的类或方法，看看它们加入的版本：added in version，然后在引入对应的support包

> 在android-support-v4-24.2.0及之后的版本中，为了增强效率和减小APK的大小起见，Android将android-support-v4包从一个独立的依赖包拆分成v4 compat library、v4 core-utils library、v4 core-ui library、v4 media-compat library和v4 fragment library这5个包，考虑到V4的向后兼容，你在工程中依赖V4这个依赖包时默认是包含拆分后的5个包的，但为了节省APK大小，建议在开发过程中根据实际情况依赖对应的V4包，移除不必要的V4包

## Android v4包关键API介绍
v4 compat library 

兼容一些 Framework API，如 Context.getDrawable() 和 View.performAccessibilityAction()等，在AS中的依赖方式如下：

```
compile 'com.android.support:support-compat:24.2.1'
```

v4 core-utils library

提供一系列核心的工具类，如 AsyncTaskLoader 和 PermissionChecker，在AS中的依赖方式如下，按自己需求选择合适版本：

```
compile 'com.android.support:support-core-utils:24.2.1'
```

core-ui library

提供一系列核心的 UI，如 ViewPager、 NestedScrollView，在AS中的依赖方式如下：

```
compile 'com.android.support:support-core-ui:24.2.1'
```

v4 media-compat library

android.media 兼容库，包括 MediaBrowser 和 MediaSession，在AS中的依赖方式如下：

```
compile 'com.android.support:support-media-compat:24.2.1'
```

v4 fragment library

跟fragment相关部分，v4 fragment library这个子库依赖了其他4个子库，所以我们一旦依赖这个库就会自动导入其他4个子库，这跟直接依赖整个support-v4效果类似，在AS中的依赖方式如下：

```
compile 'com.android.support:support-fragment:24.2.1'
```

拆包并不一定代表能够真的解决效率和减小APK的大小问题，V4包拆分后的5个子包有依赖关系。即拆包之后，要用到某个子包的API时，可能还得依赖其它的子包，这也是有坑的地方。当我们编译没有问题，运行出现Do not find class之类的错误时，一定要看看是不是子包之间的依赖关系造成的，如果是引入相应的子包。出现这个依赖问题，再加上版本可能出现问题，对于新手来说，比较棘手，建议新手全部导入

# android support v7包解析

android support v7包和v4包一样，都是为了支持比较低版本的Android系统而出现的内容。
例：应用在android框架 5.0（API 级别 21）版本以下的 手机系统上运行时，将无法显示 Material Design 元素，因为5.0版本以下的 Android 框架不支持 Material Design。但是，如果此应用引入了android support V7库，则可以访问 5.0（API 级别 21）中具有的许多功能，其中包括对 Material Design 的支持。

android support V7，同样包含多个依赖包，但和V4不同的是，V7下的多个子包并不是后面拆分开来的，而是最初发布时就以各个独立库的形式发布的。它是针对Android 2.3(API Level 9)及以上的版本谷歌提供了一系列的support包（和V4包的命名一样，V7最初支持的最低版本是Android 2.1即API Level 7，所以称其为V7，同样在android-support-v7-24.2.0将V7支持的最低版本改为Android 2.3即API Level 9了），这些support包各自对应着特定的功能，每一个都可以单独地被引用。

v7 appcompat library

这个包支持对Action Bar接口的设计模式、Material Design接口的实现等，核心类有ActionBar、AppCompatActivity、AppCompatDialog、ShareActionProvider等，在AS中的依赖方式如下：

```
compile'com.android.support:appcompat-v7:24.2.1'
```

注意：这个包需要依赖android-support-v4，版本要对应。

v7 cardview library

支持cardview控件，使用Material Design语言设计，卡片式的信息展示，在电视App中有广泛的使用，在AS中的依赖方式如下，按自己需求选择合适版本：

```
compile'com.android.support:cardview-v7:24.2.1'
```

v7 gridlayout library

支持GridLayout布局的support包，在AS中的依赖方式如下：

```
com.android.support:gridlayout-v7:24.2.1
```

v7 mediarouter library

用于设备间音频、视频交换显示的support包，在AS中的依赖方式如下：

```
com.android.support:mediarouter-v7:24.2.1
```

v7 palette library

该库提供了palette类，使用这个类可以很方便提取出图片中主题色。比如在音乐App中，从音乐专辑封面图片中提取出专辑封面图片的主题色，然后将播放界面的背景色设置为封面的主题色，随着播放音乐的改变，播放界面的背景色也会巧妙的跟着改变，从而提供更好的用户体验。，在AS中的依赖方式如下：

```
com.android.support:palette-v7:24.2.1
```

v7 recyclerview library

核心类是RecyclerView，用于替换ListView、GridView，具体可以查阅RecyclerView方面的资料，在AS中的依赖方式如下：

```
com.android.support:recyclerview-v7:24.2.1
```

v7 Preference Support Library

用于支持各种控件存储配置数据的support包，比如CheckBoxPreference和ListPreference，在AS中的依赖方式如下：

```
com.android.support:preference-v7:24.2.1
```

> 关于v4，v7包中的新组件，新样式，到后面说道material design的时候再说

# tablayout+viewpager

一般情况下，我们的viewpager是一种预加载的模式，也就是说，在页面还没有显示的时候，已经请求的网络数据。虽然现在流量的使用逐渐增加，但是给用户的体验却没有那么友好，所以，我们一般采用懒加载的方式实现。
不过，我们先实现最普通的样式效果
代码如下：
主页面的XML文件:

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <android.support.v4.view.ViewPager
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:id='@+id/tabview_viewpager'
        android:layout_alignParentTop="true"
        android:layout_above="+id/tabview_bottom_tab"></android.support.v4.view.ViewPager>

    <android.support.design.widget.TabLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:id='@+id/tabview_bottom_tab'
        android:background="#EF8D11"
        app:tabIndicatorColor="#f00"
        app:tabIndicatorHeight="4dp"
        app:tabMode="fixed"
        app:tabSelectedTextColor="#FFFFFF"
        app:tabTextColor="#FFFFFF"
        android:layout_alignParentBottom="true"></android.support.design.widget.TabLayout>

</RelativeLayout>
```

主页面的Activity：

```
public class TabViewpagerActivity extends AppCompatActivity {

    private TabLayout tabLayout;
    private ViewPager viewPager;
    private TabLayout.Tab one,two,three;
    private MyFragmentPagerAdapter adapter;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_tabviewpager);
        tabLayout = findViewById(R.id.tabview_bottom_tab);
        viewPager = findViewById(R.id.tabview_viewpager);
        adapter = new MyFragmentPagerAdapter(getSupportFragmentManager());
        viewPager.setAdapter(adapter);
        tabLayout.setupWithViewPager(viewPager);

        one = tabLayout.getTabAt(0);
        two = tabLayout.getTabAt(1);
        three = tabLayout.getTabAt(2);

        one.setIcon(R.mipmap.ic_launcher);
        two.setIcon(R.mipmap.ic_launcher);
        three.setIcon(R.mipmap.ic_launcher);

    }
}
```

适配器MyFragmentAdapter:

```
package com.paulniu.ui_demo.ui.tablayouts;

import android.support.v4.app.Fragment;
import android.support.v4.app.FragmentManager;
import android.support.v4.app.FragmentPagerAdapter;

/**
 * Created by niupule
 * E-mail: niupuyue@aliyun.com
 * Time: 2018/1/12  12:30
 * PurPose:
 */

public class MyFragmentPagerAdapter extends FragmentPagerAdapter {
    private String [] titles = new String[]{
            "首页",
            "发现",
            "我的"
    };

    public MyFragmentPagerAdapter(FragmentManager fm) {
        super(fm);
    }

    @Override
    public Fragment getItem(int position) {
        switch (position)
        {
            case 1:
                return new Fragment2();
            case 2:
                return new Fragment3();
        }
        return new Fragment1();
    }

    @Override
    public int getCount() {
        return titles.length;
    }

    @Override
    public CharSequence getPageTitle(int position) {
        return titles[position];
    }
}
```

Fragment页面，三个页面的样式是一样的：

```
public class Fragment1 extends Fragment {


    @Nullable
    @Override
    public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
        return inflater.inflate(R.layout.fragment_viewpager,container,false);
    }

}
```

Fragment的XML页面:

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <TextView
        android:id="@+id/textView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentTop="true"
        android:layout_centerHorizontal="true"
        android:layout_marginTop="218dp"
        android:text="Page:"
        android:textSize="20sp"
        android:textStyle="bold"/>

    <TextView
        android:id="@+id/titile"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_below="@+id/textView"
        android:layout_centerHorizontal="true"
        android:textSize="50sp"
        android:text="1"/>
</RelativeLayout>
```

最终实现的效果如图所示
![效果](/assets/android/service-life.png)
## setUserVisibleHint(boolean isVisibleToUser)
setUserVisibleHint(boolean isVisibleToUser)是Fragment中的一个回调函数。当前Fragment可见时，setUserVisibleHint()回调，其中isVisibleToUser=true。当前Fragment由可见到不可见或实例化时，setUserVisibleHint()回调，其中isVisibleToUser=false。
### setUserVisibleHint(boolean isVIsibleToUser)调用的时机
1. 在Fragment实例化，即在ViewPager中，由于ViewPager默认会预加载左右两个页面。此时预加载页面回调的生命周期流程：setUserVisibleHint() -->onAttach() --> onCreate()-->onCreateView()--> onActivityCreate() --> onStart() --> onResume()
> 此时参数为false，不可见
2. 在Fragmetn可见时，即ViewPager中滑动到当前页面时，因为已经预加载过了，之前生命周期已经走到onResume() 
> 此时参数为true，可见
3. 在Fragment由可见变为不可见，即ViewPager由当前页面滑动到另一个页面，因为还要保持当前页面的预加载过程，所以只会回调：setUserVisibleHint()。
> 此时参数为false，不可见
4. 由TabLayout直接跳转到一个未预加载的页面，此时生命周期的回调过程：setUserVisibleHint() -->setUserVisibleHint() -->onAttach() --> onCreate()-->onCreateView()--> onActivityCreate() --> onStart()--> onResume()
> 回调了两次setUserVisibleHint() ，一次代表初始化时，传入参数是false，一次代表可见时，传入参数是true。这种情况比较特殊

> 总结
> 无论何时，setUserVisibleHint()都是先于其他生命周期调用的，并且有三种情况，初始化时调用，可见时时调用，由可见转换成不可见时调用。

### 通过封装实现
#### 进行网络加载的情况
- setUserVisibleHint()中的参数是true，即fragment可见
- onCreateView()方法调用完毕，返回一个rootview，防止发生空指针异常
- 第一次调用网络加载，之前没有执行过.
满足以上三个条件，可以进行网络加载
#### 不进行网络加载
- setUserVisibleHint()中的参数有true变为false，即fragment由可见变成不可见
- 不是第一次加载数据，之前加载过
满足以上两个条件(其实满足第一个即可)
### 分装BaseFragment
isFragment表示当前fragment是否可见
isFirst表示是否是已经加载过
onFragmentVisibleChange()是一个可继承的空方法，子类实现，传入true，代表加载网络，传入false,代表取消网络加载。
```
@Override
    public void setUserVisibleHint(boolean isVisibleToUser) {
        super.setUserVisibleHint(isVisibleToUser);
        if (isVisibleToUser) {
            isFragmentVisible = true;
        }
        if (rootView == null) {
            return;
        }
        //可见，并且没有加载过
        if (!isFirst&&isFragmentVisible) {
            onFragmentVisibleChange(true);
            return;
        }
        //由可见——>不可见 已经加载过
        if (isFragmentVisible) {
            onFragmentVisibleChange(false);
            isFragmentVisible = false;
        }
    }
```
```
protected void onFragmentVisibleChange(boolean isVisible) {

    }
```
> 特殊情况：
> 当TabLayout跳转到一个没有预加载过的Fragment，连续调用两次setUserVisibleHint方法，但此时rootView为空，不能进行加载。需要在onCreateView()方法最后判断是否进行网络加载，然后调用onFragmentVisibleChange(true)方法
```
public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
     .................
        //可见，但是并没有加载过
        if (isFragmentVisible && !isFirst) {
            onFragmentVisibleChange(true);
        }
        return rootView;
    }
```
子类中的具体实现
```
@Override
    protected void onFragmentVisibleChange(boolean isVisible) {
        if(isVisible){
            //可见，并且是第一次加载
            mPresenter.requestPhotoList(SIZE,mStartPage);
            LoadingDialog.showDialogForLoading(getActivity());
        }else{
            //取消加载
            RxDisposeManager.get().cancel("photoList");
            LoadingDialog.cancelDialogForLoading();
        }
    }
```
暂时先写这么多，后面的内容，以后补充···

# 参考资料
[android O 通知渠道](https://www.jianshu.com/p/92afa56aee05)
[android support v4支持包](http://www.jianshu.com/u/d82bd37b1d29)
[Android Google sample](https://developer.android.com/samples/index.html?language=java)