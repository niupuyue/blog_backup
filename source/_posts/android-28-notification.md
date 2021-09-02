---
title: 重拾android路(二十七) Notification
date: 2019-06-16 19:44:50
tags:
  - android
---

最近公司要求把IM这一模块重写一遍，然后使用MQTT的方式实现消息的收发功能。这个内容我后面在总结，今天先看一下另外一个模块。这个内容的需求是这样的，要求在MQTT收到消息之后，在通知栏显示消息内容，其实就是需要有一个通知栏可以显示IM消息。所以趁着这个时间，再把Notification的知识点总结一下。
这里我觉得代码上的实现总是比较简单，我会在最后把代码的思路交代一下，而前面的篇幅主要用来描述，Notification的机制
<!--more-->
# 什么是Notification机制
Notification，中文名翻译为通知，每个 app 可以自定义通知的样式和内容等，它会显示在系统的通知栏等区域。用户可以打开抽屉式通知栏查看通知的详细信息。在实际生活中，Android Notification 机制有很广泛的应用，例如 IM app 的新消息通知，资讯 app 的新闻推送等等

> 源码分析的时候主要是以Android7.0为主

# Notification的发送逻辑

一般来说，如果我们自己的 app 想发送一条新的 Notification，可以参照以下代码

```
NotificationCompat.Builder mBuilder =
       new NotificationCompat.Builder(this)
       .setSmallIcon(R.drawable.notification_icon)
       .setWhen(System.currentTimeMillis())
       .setContentTitle("Test Notification Title")
       .setContentText("Test Notification Content!");
Intent resultIntent = new Intent(this, ResultActivity.class);

PendingIntent contentIntent =
       PendingIntent.getActivity(
           this, 
           0, 
           resultIntent, 
           PendingIntent.FLAG_UPDATE_CURRENT
       );
mBuilder.setContentIntent(resultPendingIntent);
NotificationManager mNotificationManager =
   (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
// mId allows you to update the notification later on.
mNotificationManager.notify(mId, mBuilder.build());
```

可以看到，我们通过 NotificationCompat.Builder 新建了一个 Notification 对象，最后通过 NotificationManager#notify() 方法将 Notification 发送出去

# NotificationManager#notify()

```

public void notify(int id, Notification notification)
{
   notify(null, id, notification);
}

// 省略部分注释
public void notify(String tag, int id, Notification notification)
{
   notifyAsUser(tag, id, notification, new UserHandle(UserHandle.myUserId()));
}

/**
* @hide
*/
public void notifyAsUser(String tag, int id, Notification notification, UserHandle user)
{
   int[] idOut = new int[1];
   INotificationManager service = getService();
   String pkg = mContext.getPackageName();
   // Fix the notification as best we can.
   Notification.addFieldsFromContext(mContext, notification);
   if (notification.sound != null) {
       notification.sound = notification.sound.getCanonicalUri();
       if (StrictMode.vmFileUriExposureEnabled()) {
           notification.sound.checkFileUriExposed("Notification.sound");
       }
   }
   fixLegacySmallIcon(notification, pkg);
   if (mContext.getApplicationInfo().targetSdkVersion > Build.VERSION_CODES.LOLLIPOP_MR1) {
       if (notification.getSmallIcon() == null) {
           throw new IllegalArgumentException("Invalid notification (no valid small icon): "
                   + notification);
       }
   }
   if (localLOGV) Log.v(TAG, pkg + ": notify(" + id + ", " + notification + ")");
   final Notification copy = Builder.maybeCloneStrippedForDelivery(notification);
   try {
       // !!!
       service.enqueueNotificationWithTag(pkg, mContext.getOpPackageName(), tag, id,
               copy, idOut, user.getIdentifier());
       if (localLOGV && id != idOut[0]) {
           Log.v(TAG, "notify: id corrupted: sent " + id + ", got back " + idOut[0]);
       }
   } catch (RemoteException e) {
       throw e.rethrowFromSystemServer();
   }
}
```
我们可以看到，到最后会调用 service.enqueueNotificationWithTag() 方法，这里的是 service 是 INotificationManager 接口。如果熟悉 AIDL 等系统相关运行机制的话，就可以看出这里是代理类调用了代理接口的方法，实际方法实现是在 NotificationManagerService 当中

# NotificationManagerService#enqueueNotificationWithTag()
```
@Override
public void enqueueNotificationWithTag(String pkg, String opPkg, String tag, int id,
        Notification notification, int[] idOut, int userId) throws RemoteException {
   enqueueNotificationInternal(pkg, opPkg, Binder.getCallingUid(),
           Binder.getCallingPid(), tag, id, notification, idOut, userId);
}

void enqueueNotificationInternal(final String pkg, final String opPkg, final int callingUid,
       final int callingPid, final String tag, final int id, final Notification notification,
       int[] idOut, int incomingUserId) {
   if (DBG) {
       Slog.v(TAG, "enqueueNotificationInternal: pkg=" + pkg + " id=" + id
            + " notification=" + notification);
   }
   checkCallerIsSystemOrSameApp(pkg);
   final boolean isSystemNotification = isUidSystem(callingUid) || ("android".equals(pkg));
   final boolean isNotificationFromListener = mListeners.isListenerPackage(pkg);

   final int userId = ActivityManager.handleIncomingUser(callingPid,
           callingUid, incomingUserId, true, false, "enqueueNotification", pkg);
   final UserHandle user = new UserHandle(userId);

   // Fix the notification as best we can.
   try {
       final ApplicationInfo ai = getContext().getPackageManager().getApplicationInfoAsUser(
               pkg, PackageManager.MATCH_DEBUG_TRIAGED_MISSING,
               (userId == UserHandle.USER_ALL) ? UserHandle.USER_SYSTEM : userId);
       Notification.addFieldsFromContext(ai, userId, notification);
   } catch (NameNotFoundException e) {
       Slog.e(TAG, "Cannot create a context for sending app", e);
       return;
   }

   mUsageStats.registerEnqueuedByApp(pkg);

   if (pkg == null || notification == null) {
       throw new IllegalArgumentException("null not allowed: pkg=" + pkg
            + " id=" + id + " notification=" + notification);
   }
   final StatusBarNotification n = new StatusBarNotification(
           pkg, opPkg, id, tag, callingUid, callingPid, 0, notification,
           user);

   // Limit the number of notifications that any given package except the android
   // package or a registered listener can enqueue.  Prevents DOS attacks and deals with leaks.
   if (!isSystemNotification && !isNotificationFromListener) {
       synchronized (mNotificationList) {
           if(mNotificationsByKey.get(n.getKey()) != null) {
               // this is an update, rate limit updates only
               final float appEnqueueRate = mUsageStats.getAppEnqueueRate(pkg);
               if (appEnqueueRate > mMaxPackageEnqueueRate) {
                   mUsageStats.registerOverRateQuota(pkg);
                   final long now = SystemClock.elapsedRealtime();
                   if ((now - mLastOverRateLogTime) > MIN_PACKAGE_OVERRATE_LOG_INTERVAL) {
                       Slog.e(TAG, "Package enqueue rate is " + appEnqueueRate
                               + ". Shedding events. package=" + pkg);
                           mLastOverRateLogTime = now;
                   }
                   return;
               }
           }

           int count = 0;
           final int N = mNotificationList.size();
           for (int i=0; i<N; i++) {
               final NotificationRecord r = mNotificationList.get(i);
               if (r.sbn.getPackageName().equals(pkg) && r.sbn.getUserId() == userId) {
                   if (r.sbn.getId() == id && TextUtils.equals(r.sbn.getTag(), tag)) {
                       break;  // Allow updating existing notification
                   }
                   count++;
                   if (count >= MAX_PACKAGE_NOTIFICATIONS) {
                       mUsageStats.registerOverCountQuota(pkg);
                       Slog.e(TAG, "Package has already posted " + count
                               + " notifications.  Not showing more.  package=" + pkg);
                       return;
                   }
               }
           }
       }
   }

   // Whitelist pending intents.
   if (notification.allPendingIntents != null) {
       final int intentCount = notification.allPendingIntents.size();
       if (intentCount > 0) {
           final ActivityManagerInternal am = LocalServices
                   .getService(ActivityManagerInternal.class);
           final long duration = LocalServices.getService(
                   DeviceIdleController.LocalService.class).getNotificationWhitelistDuration();
           for (int i = 0; i < intentCount; i++) {
               PendingIntent pendingIntent = notification.allPendingIntents.valueAt(i);
               if (pendingIntent != null) {
                   am.setPendingIntentWhitelistDuration(pendingIntent.getTarget(), duration);
               }
           }
       }
   }

   // Sanitize inputs
   notification.priority = clamp(notification.priority, Notification.PRIORITY_MIN,
           Notification.PRIORITY_MAX);

   // setup local book-keeping
   final NotificationRecord r = new NotificationRecord(getContext(), n);
   mHandler.post(new EnqueueNotificationRunnable(userId, r));

   idOut[0] = id;
}

```
这里代码比较多，但通过注释可以清晰地理清整个逻辑：

1. 首先检查通知发起者是系统进程或者是查看发起者发送的是否是同个 app 的通知信息，否则抛出异常；
2. 除了系统的通知和已注册的监听器允许入队列外，其他 app 的通知都会限制通知数上限和通知频率上限；
3. 将 notification 的 PendingIntent 加入到白名单；
4. 将之前的 notification 进一步封装为 StatusBarNotification 和 NotificationRecord，最后封装到一个异步线程 EnqueueNotificationRunnable 中

这里有一个点，就是 mHandler，涉及到切换线程，我们先跟踪一下 mHandler 是在哪个线程被创建。

mHandler 是 WorkerHandler 类的一个实例，在 NotificationManagerService#onStart() 方法中被创建，而 NotificationManagerService 是系统 Service，所以 EnqueueNotificationRunnable 的 run 方法会运行在 system_server 的主线程

# NotificationManagerService.EnqueueNotificationRunnable#run()
```
@Override
public void run() {
   synchronized(mNotificationList) {
       // 省略代码
       if (notification.getSmallIcon() != null) {
           StatusBarNotification oldSbn = (old != null) ? old.sbn : null;
           mListeners.notifyPostedLocked(n, oldSbn);
       } else {
           Slog.e(TAG, "Not posting notification without small icon: " + notification);
           if (old != null && !old.isCanceled) {
               mListeners.notifyRemovedLocked(n);
           }
           // ATTENTION: in a future release we will bail out here
           // so that we do not play sounds, show lights, etc. for invalid
           // notifications
           Slog.e(TAG, "WARNING: In a future release this will crash the app: " + n.getPackageName());
       }
       buzzBeepBlinkLocked(r);
   }
}
```
1. 省略的代码主要的工作是提取 notification 相关的属性，同时通知 notification ranking service，有新的 notification 进来，然后对所有 notification 进行重新排序；
2. 然后到最后会调用 mListeners.notifyPostedLocked() 方法。这里 mListeners 是 NotificationListeners 类的一个实例

# NotificationManagerService.NotificationListeners#notifyPostedLocked()-> NotificationManagerService.NotificationListeners#notifyPosted()

```
public void notifyPostedLocked(StatusBarNotification sbn, StatusBarNotification oldSbn) {
   // Lazily initialized snapshots of the notification.
   TrimCache trimCache = new TrimCache(sbn);
   for (final ManagedServiceInfo info: mServices) {
       boolean sbnVisible = isVisibleToListener(sbn, info);
       boolean oldSbnVisible = oldSbn != null ? isVisibleToListener(oldSbn, info) : false;
       // This notification hasn't been and still isn't visible -> ignore.
       if (!oldSbnVisible && !sbnVisible) {
           continue;
       }
       final NotificationRankingUpdate update = makeRankingUpdateLocked(info);
       // This notification became invisible -> remove the old one.
       if (oldSbnVisible && !sbnVisible) {
           final StatusBarNotification oldSbnLightClone = oldSbn.cloneLight();
           mHandler.post(new Runnable() {
               @Override
               public void run() {
                   notifyRemoved(info, oldSbnLightClone, update);
               }
           });
           continue;
       }
       final StatusBarNotification sbnToPost = trimCache.ForListener(info);
       mHandler.post(new Runnable() {
           @Override
           public void run() {
               notifyPosted(info, sbnToPost, update);
           }
       });
   }
}

private void notifyPosted(final ManagedServiceInfo info, final StatusBarNotification sbn, NotificationRankingUpdate rankingUpdate) {
   final INotificationListener listener = (INotificationListener) info.service;
   StatusBarNotificationHolder sbnHolder = new StatusBarNotificationHolder(sbn);
   try {
       listener.onNotificationPosted(sbnHolder, rankingUpdate);
   } catch (RemoteException ex) {
       Log.e(TAG, "unable to notify listener (posted): " + listener, ex);
   }
}

```

调用到最后会执行 listener.onNotificationPosted() 方法。通过全局搜索得知，listener 类型是 NotificationListenerService.NotificationListenerWrapper 的代理对象

# NotificationListenerService.NotificationListenerWrapper#onNotificationPosted()
```
public void onNotificationPosted(IStatusBarNotificationHolder sbnHolder, NotificationRankingUpdate update) {
   StatusBarNotification sbn;
   try {
       sbn = sbnHolder.get();
   } catch (RemoteException e) {
       Log.w(TAG, "onNotificationPosted: Error receiving StatusBarNotification", e);
       return;
   }
   try {
       // convert icon metadata to legacy format for older clients
       createLegacyIconExtras(sbn.getNotification());
       maybePopulateRemoteViews(sbn.getNotification());
   } catch (IllegalArgumentException e) {
       // warn and drop corrupt notification
       Log.w(TAG, "onNotificationPosted: can't rebuild notification from " + sbn.getPackageName());
       sbn = null;
   }
   // protect subclass from concurrent modifications of (@link mNotificationKeys}.
   synchronized(mLock) {
       applyUpdateLocked(update);
       if (sbn != null) {
           SomeArgs args = SomeArgs.obtain();
           args.arg1 = sbn;
           args.arg2 = mRankingMap;
           mHandler.obtainMessage(MyHandler.MSG_ON_NOTIFICATION_POSTED, args).sendToTarget();
       } else {
           // still pass along the ranking map, it may contain other information
           mHandler.obtainMessage(MyHandler.MSG_ON_NOTIFICATION_RANKING_UPDATE, mRankingMap).sendToTarget();
       }
   }
}
```
这里在一开始会从 sbnHolder 中获取到 sbn 对象，sbn 隶属于 StatusBarNotificationHolder 类，继承于 IStatusBarNotificationHolder.Stub 对象。注意到这里捕获了一个 RemoteException，猜测涉及到跨进程调用，但我们不知道这段代码是在哪个进程中执行的，所以在这里暂停跟踪代码。

笔者之前是通过向系统发送通知的方式跟踪源码，发现走不通。故个人尝试从另一个角度入手，即系统接收我们发过来的通知并显示到通知栏这个方式入手跟踪代码

# 系统如何显示 Notification，即对于系统端来说，Notification 的接收逻辑
系统显示 Notification 的过程，猜测是在 PhoneStatusBar.java 中，因为系统启动的过程中，会启动 SystemUI 进程，初始化整个 Android 显示的界面，包括系统通知栏。

## PhoneStatusBar#start() -> BaseStatusBar#start()

```
public void start() {
   // 省略代码
   // Set up the initial notification state.
   try {
       mNotificationListener.registerAsSystemService(mContext,
               new ComponentName(mContext.getPackageName(), getClass().getCanonicalName()),
               UserHandle.USER_ALL);
   } catch (RemoteException e) {
       Log.e(TAG, "Unable to register notification listener", e);
   }
   // 省略代码
}
```
这段代码中，会调用 NotificationListenerService#registerAsSystemService() 方法，涉及到我们之前跟踪代码的类。我们继续跟进去看一下。

## NotificationListenerService#registerAsSystemService()
```
public void registerAsSystemService(Context context, ComponentName componentName,
       int currentUser) throws RemoteException {
   if (mWrapper == null) {
       mWrapper = new NotificationListenerWrapper();
   }
   mSystemContext = context;
   INotificationManager noMan = getNotificationInterface();
   mHandler = new MyHandler(context.getMainLooper());
   mCurrentUser = currentUser;
   noMan.registerListener(mWrapper, componentName, currentUser);
}
```
这里会初始化一个 NotificationListenerWrapper 和 mHandler。由于这是在 SystemUI 进程中去调用此方法将 NotificationListenerService 注册为系统服务，所以在前面分析的那里：

NotificationListenerService.NotificationListenerWrapper#onNotificationPosted()，这段代码是运行在 SystemUI 进程，而 mHandler 则是运行在 SystemUI 主线程上的 Handler。所以，onNotificationPosted() 是运行在 SystemUI 进程中，它通过 sbn 从 system_server 进程中获取到 sbn 对象。下一步是通过 mHandler 处理消息，查看 NotificationListenerService.MyHandler#handleMessage() 方法，得知当 message.what 为 MSG_ON_NOTIFICATION_POSTED 时，调用的是 onNotificationPosted() 方法

但是，NotificationListenerService 是一个抽象类，onNotificationPosted() 为空方法，真正的实现是它的实例类。
观察到之前 BaseStatusBar#start() 中，是调用了 mNotificationListener.registerAsSystemService() 方法。那么，mNotificationListener 是在哪里进行初始化呢？

## BaseStatusBar.mNotificationListener#onNotificationPosted

```
private final NotificationListenerService mNotificationListener = new NotificationListenerService() {
   // 省略代码
   
   @Override
   public void onNotificationPosted(final StatusBarNotification sbn, final RankingMap rankingMap) {
       if (DEBUG) Log.d(TAG, "onNotificationPosted: " + sbn);
       if (sbn != null) {
           mHandler.post(new Runnable() {
               @Override
               public void run() {
                   processForRemoteInput(sbn.getNotification());
                   String key = sbn.getKey();
                   mKeysKeptForRemoteInput.remove(key);
                   boolean isUpdate = mNotificationData.get(key) != null;
                   // In case we don't allow child notifications, we ignore children of
                   // notifications that have a summary, since we're not going to show them
                   // anyway. This is true also when the summary is canceled,
                   // because children are automatically canceled by NoMan in that case.
                   if (!ENABLE_CHILD_NOTIFICATIONS && mGroupManager.isChildInGroupWithSummary(sbn)) {
                       if (DEBUG) {
                           Log.d(TAG, "Ignoring group child due to existing summary: " + sbn);
                       }
                       // Remove existing notification to avoid stale data.
                       if (isUpdate) {
                           removeNotification(key, rankingMap);
                       } else {
                           mNotificationData.updateRanking(rankingMap);
                       }
                       return;
                   }
                   if (isUpdate) {
                       updateNotification(sbn, rankingMap);
                   } else {
                       addNotification(sbn, rankingMap, null /* oldEntry */ );
                   }
               }
           });
       }
   }
   // 省略代码
}
```
1. 通过上述代码，我们知道了在 BaseStatusBar.java 中，创建了 NotificationListenerService 的实例对象，实现了 onNotificationPost() 这个抽象方法；
2. 在 onNotificationPost() 中，通过 handler 进行消息处理，最终调用 addNotification() 方法

## PhoneStatusBar#addNotification()
```
@Override
public void addNotification(StatusBarNotification notification, RankingMap ranking, Entry oldEntry) {
   if (DEBUG) Log.d(TAG, "addNotification key=" + notification.getKey());
   mNotificationData.updateRanking(ranking);
   Entry shadeEntry = createNotificationViews(notification);
   if (shadeEntry == null) {
       return;
   }
   boolean isHeadsUped = shouldPeek(shadeEntry);
   if (isHeadsUped) {
       mHeadsUpManager.showNotification(shadeEntry);
       // Mark as seen immediately
       setNotificationShown(notification);
   }
   if (!isHeadsUped && notification.getNotification().fullScreenIntent != null) {
       if (shouldSuppressFullScreenIntent(notification.getKey())) {
           if (DEBUG) {
               Log.d(TAG, "No Fullscreen intent: suppressed by DND: " + notification.getKey());
           }
       } else if (mNotificationData.getImportance(notification.getKey()) < NotificationListenerService.Ranking.IMPORTANCE_MAX) {
           if (DEBUG) {
               Log.d(TAG, "No Fullscreen intent: not important enough: " + notification.getKey());
           }
       } else {
           // Stop screensaver if the notification has a full-screen intent.
           // (like an incoming phone call)
           awakenDreams();
           // not immersive & a full-screen alert should be shown
           if (DEBUG) Log.d(TAG, "Notification has fullScreenIntent; sending fullScreenIntent");
           try {
               EventLog.writeEvent(EventLogTags.SYSUI_FULLSCREEN_NOTIFICATION, notification.getKey());
               notification.getNotification().fullScreenIntent.send();
               shadeEntry.notifyFullScreenIntentLaunched();
               MetricsLogger.count(mContext, "note_fullscreen", 1);
           } catch (PendingIntent.CanceledException e) {}
       }
   }
   // !!!
   addNotificationViews(shadeEntry, ranking);
   // Recalculate the position of the sliding windows and the titles.
   setAreThereNotifications();
}
```
在这个方法中，最关键的方法是最后的 addNotificationViews() 方法。调用这个方法之后，你创建的 Notification 才会被添加到系统通知栏上

# 代码实现
这里我创建了一个Notification管理类，通过单例构建一个管理类对象，当我们需要显示或者清除通知时，直接调用里面的方法即可
主要代码：
```
public class PushNotificationManager extends ContextWrapper {

    private static Context mContext;
    private NotificationManager notificationManager;
    private Notification.Builder notification;

    public static String NOTIFICATION_STATIC_ID = "1001";
    private static String NOTIFICATION_STATIC_NAME = "新消息通知";
    private static int NOTIFICATION_CHANNEL_ID = 0x100;

    private PushNotificationManager(Context context) {
        super(context);
        // 在构造方法中，声明NotificationManager对象
        if (null == notificationManager && null != context) {
            notificationManager = (NotificationManager) context.getSystemService(Context.NOTIFICATION_SERVICE);
        }
    }

    /**
     * 判断当前SDK是否大于26
     *
     * @return
     */
    private boolean checkSdkVersionMoreO() {
        return Build.VERSION.SDK_INT >= Build.VERSION_CODES.O;
    }

    /**
     * 创建Notification对象，针对Android8.0以下
     */
    private void createPushNotificationAfter3(String content, ImDownMessageModel model) {
        if (model == null) return;
        try {
            if (null == notificationManager) {
                notificationManager = (NotificationManager) mContext.getSystemService(Context.NOTIFICATION_SERVICE);
            }
            notification = new Notification.Builder(mContext)
                    .setSmallIcon(R.mipmap.ic_launcher)
                    .setContentTitle(mContext.getString(R.string.app_name))
                    .setContentText(content)
                    .setPriority(Notification.PRIORITY_HIGH)
                    .setWhen(System.currentTimeMillis())
                    .setContentIntent(initJumpAction(model.from))
                    .setOngoing(false)
                    .setAutoCancel(true);
            int default_setting = 0;
            // 根据设置决定是否给出提示音和震动效果
            if (EditSharedPreferences.getNotifySound() && EditSharedPreferences.getNotifyVibrator()) {
                default_setting = Notification.DEFAULT_SOUND | Notification.DEFAULT_VIBRATE;
            } else if (EditSharedPreferences.getNotifySound() && !EditSharedPreferences.getNotifyVibrator()) {
                default_setting = Notification.DEFAULT_SOUND;
            } else if (EditSharedPreferences.getNotifyVibrator() && !EditSharedPreferences.getNotifySound()) {
                default_setting = Notification.DEFAULT_VIBRATE;
            } else if (!EditSharedPreferences.getNotifySound() && !EditSharedPreferences.getNotifyVibrator()) {
                default_setting = 0;
            }
            notification.setDefaults(default_setting);
            notificationManager.notify(NOTIFICATION_CHANNEL_ID, notification.build());
        } catch (Exception ex) {
            UtilityException.catchException(ex);
        }
    }

    /**
     * 创建Notification对象，针对Android8.0以上
     */
    @RequiresApi(api = Build.VERSION_CODES.O)
    private void createPushNotificationAfter8(String content, ImDownMessageModel model) {
        if (model == null) return;
        try {
            if (null == notificationManager) {
                notificationManager = (NotificationManager) mContext.getSystemService(Context.NOTIFICATION_SERVICE);
            }
            NotificationChannel mChannel = new NotificationChannel(NOTIFICATION_STATIC_ID, NOTIFICATION_STATIC_NAME, NotificationManager.IMPORTANCE_DEFAULT);
            notificationManager.createNotificationChannel(mChannel);
            notification = new Notification.Builder(mContext, NOTIFICATION_STATIC_ID)
                    .setContentTitle(mContext.getString(R.string.app_name))
                    .setContentText(content)
                    .setSmallIcon(R.mipmap.ic_launcher_small)
                    .setColor(mContext.getColor(R.color._e64448))
                    .setWhen(System.currentTimeMillis())
                    .setAutoCancel(true)
                    .setPriority(Notification.PRIORITY_HIGH)
                    .setOngoing(false)
                    .setContentIntent(initJumpAction(model.from))
                    .setDefaults(Notification.DEFAULT_ALL);
            //发送
            notificationManager.notify(NOTIFICATION_CHANNEL_ID, notification.build());
        } catch (Exception ex) {
            UtilityException.catchException(ex);
        }
    }

    /**
     * 显示Notification对象
     *
     * @param content 通知栏中的content
     * @param model
     */
    private void showPushNotification(String content, ImDownMessageModel model) {
        if (checkSdkVersionMoreO()) {
            createPushNotificationAfter8(content, model);
        } else {
            createPushNotificationAfter3(content, model);
        }
    }

    /**
     * 清除Notification对象
     */
    public void clearPushNotification() {
        if (notificationManager != null) {
            notificationManager.cancel(NOTIFICATION_CHANNEL_ID);
        }
    }

    /**
     * 初始化跳转PendingIntent
     */
    private PendingIntent initJumpAction(String fromUid) {
        try {
            ChatActivityExtraModel model = new ChatActivityExtraModel();
            model.uid = fromUid;
            Intent intent = ChatActivity.getIntent(mContext, model);
            intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);           //添加为栈顶Activity
            PendingIntent resultPendingIntent = PendingIntent.getActivity(mContext, -1, intent, PendingIntent.FLAG_UPDATE_CURRENT);
            return resultPendingIntent;
        } catch (Exception ex) {
            UtilityException.catchException(ex);
        }
        return null;
    }

    /**
     * 添加通知栏消息
     *
     * @param unreadInfo 未读消息信息
     * @param model      最后一条消息
     */
    public void addPushNotification(final UnreadInfo unreadInfo, final ImDownMessageModel model) {
        if (unreadInfo == null || unreadInfo.unReadUsers <= 0)
            return;

        try {
            final String messageText = IMContentConverter.getShortText(model);
            if (!UtilitySecurity.isEmpty(messageText)) {
                ImHelper.getImInfoByData(model.from, new ILoadImInfo() {
                    @Override
                    public void LoadSuccess(ChatTitleModel chatTitleModel) {
                        if (chatTitleModel == null || UtilitySecurity.isEmpty(chatTitleModel.name))
                            return;
                        showPushNotification(chatTitleModel.name + RRApplication.getAppContext().getString(R.string.im_showText_maohao) + messageText, model);
                    }

                    @Override
                    public void LoadFail() {

                    }
                });
            }
        } catch (Exception ex) {
            UtilityException.catchException(ex);
        }
    }

    /**
     * ///////////////////////////////////////////////////////匿名内部类构造单例模式 开始//////////////////////////////////////////////////////////////////
     */
    public static PushNotificationManager getInstance(Context context) {
        mContext = context;
        return PushNotificationManagerHolder.instance;
    }

    private static class PushNotificationManagerHolder {
        private static final PushNotificationManager instance = new PushNotificationManager(mContext);
    }
    /**
     * ///////////////////////////////////////////////////////匿名内部类构造单例模式 结束//////////////////////////////////////////////////////////////////
     */
}
```



