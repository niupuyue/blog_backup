---
title: 重拾android路(一) 服务(Service)
date: 2015-12-07 20:31:58
tags:
  - android
---

从毕业到现在，做android开发已经半年多了，光凭自己在大学时期学到的内容，感觉应付工作已经没有什么问题了，但是，如果想要再次提升自己，我觉得要从基础开始，回过头来，再次学习。所以我决定，重走android开发之路，此文为据。
<!--more-->
此次从新学习不会像以前那样，按部就班的走，如果还是那样，基本收效甚微，所以我决定以自己的实际经验，在自己需要着重学习的地方再下功夫。

# Service介绍

Service是android四大组件之一，和Activity最为相似，他们都代表可执行程序，两者的区别在于：service只能在后台运行，他没有用户界面，决不允许到前台来，同时他也有自己的声明周期。

# Service使用

## 创建，配置Service

开发需要两个步骤:

1. 定义一个继承Service的子类
2. 在android配置文件中配置该Service

Service和Activity还有一个共同之处，他们都是从Context中派生出来的，所以都可以调佣Context中的方法，例如getResources(),getContextResolver()

在Service中也定义了一些生命周期方法:

- IBinder onBind(Intent intent): 子类必须实现，返回一个IBinder对象，可以通过该对象在Service组件中通信
- void onCreate(): 该方法在Service第一次被创建的时候立即回调该方法
- void onDestory(): 在该Service被关闭之前将会调用该方法
- void onStartCommand(Intent intent,int flags,int startId): 该方法的早期版本是onStart()方法。在用户调用startService(Intent)方法启动Service时都会回调该方法
- boolean onUnbind(Intent intent): 当该Service上绑定的所有客户端都断开连接是将会回调该方法

##### Service类源码
```
public abstract class Service extends ContextWrapper implements ComponentCallbacks2 {
    public static final int START_CONTINUATION_MASK = 15;
    public static final int START_FLAG_REDELIVERY = 1;
    public static final int START_FLAG_RETRY = 2;
    public static final int START_NOT_STICKY = 2;
    public static final int START_REDELIVER_INTENT = 3;
    public static final int START_STICKY = 1;
    public static final int START_STICKY_COMPATIBILITY = 0;
    public static final int STOP_FOREGROUND_DETACH = 2;
    public static final int STOP_FOREGROUND_REMOVE = 1;

    public Service() {
        super((Context)null);
        throw new RuntimeException("Stub!");
    }

    public final Application getApplication() {
        throw new RuntimeException("Stub!");
    }

    public void onCreate() {
        throw new RuntimeException("Stub!");
    }

    /** @deprecated */
    @Deprecated
    public void onStart(Intent intent, int startId) {
        throw new RuntimeException("Stub!");
    }

    public int onStartCommand(Intent intent, int flags, int startId) {
        throw new RuntimeException("Stub!");
    }

    public void onDestroy() {
        throw new RuntimeException("Stub!");
    }

    public void onConfigurationChanged(Configuration newConfig) {
        throw new RuntimeException("Stub!");
    }

    public void onLowMemory() {
        throw new RuntimeException("Stub!");
    }

    public void onTrimMemory(int level) {
        throw new RuntimeException("Stub!");
    }

    public abstract IBinder onBind(Intent var1);

    public boolean onUnbind(Intent intent) {
        throw new RuntimeException("Stub!");
    }

    public void onRebind(Intent intent) {
        throw new RuntimeException("Stub!");
    }

    public void onTaskRemoved(Intent rootIntent) {
        throw new RuntimeException("Stub!");
    }

    public final void stopSelf() {
        throw new RuntimeException("Stub!");
    }

    public final void stopSelf(int startId) {
        throw new RuntimeException("Stub!");
    }

    public final boolean stopSelfResult(int startId) {
        throw new RuntimeException("Stub!");
    }

    public final void startForeground(int id, Notification notification) {
        throw new RuntimeException("Stub!");
    }

    public final void stopForeground(boolean removeNotification) {
        throw new RuntimeException("Stub!");
    }

    public final void stopForeground(int flags) {
        throw new RuntimeException("Stub!");
    }

    protected void dump(FileDescriptor fd, PrintWriter writer, String[] args) {
        throw new RuntimeException("Stub!");
    }
}
```
#### 例子
```
public class MyService extends Service {
    @Override
    public IBinder onBind(Intent intent) {
        return null;
    }

    /**
     * service被创建时回调该方法
     */
    @Override
    public void onCreate() {
        super.onCreate();
    }

    /**
     * 通过startService()方法启动Service时回调该方法
     * @param intent
     * @param flags
     * @param startId
     * @return
     */
    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        return super.onStartCommand(intent, flags, startId);
    }

    /**
     * Service被关闭时回调该方法
     */
    @Override
    public void onDestroy() {
        super.onDestroy();
    }
}
```
还需要在清单配置文件中书写如下代码
```
   <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
        <service android:name=".MyService"></service>
    </application>
```

## 启动和停止Service

两种方法：

1. 通过Context的startService(Intent)方法，访问者和Service没有关联，即使访问者退出了，Service依然可以运行
2. 通过Context的bindService()方法，访问者和Service绑定在一起，访问者退出，Service也退出

### 第一种情况

```
public class MyActivity extends Activity {

    Button start,stop;

    @Override
    public void onCreate(@Nullable Bundle savedInstanceState, @Nullable PersistableBundle persistentState) {
        super.onCreate(savedInstanceState, persistentState);

        setContentView(R.layout.activity_main);

        start = findViewById(R.id.start);
        stop = findViewById(R.id.stop);

        //启动Service的Intent
        //google 5.0以后必须显示启动Service组件
        final Intent intent = new Intent(this,MyService.class);

        start.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                startService(intent);
            }
        });

        stop.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                stopService(intent);
            }
        });


    }
}
```
由于通过startService和stopService来操作Service，这时候，Service和访问者的关系并不大，所以无法进行通信，如果想要进行通信，就必须使用第二种方式bindService()

### 第二种情况

Context中的bindService()的完整写法是:
```
public abstract boolean bindService(Intent intent, ServiceConnection conn, int flags);
public abstract void unbindService(ServiceConnection conn);    
```
参数分析：
1. intent：改参数指定需要启动的Service
2. conn:是一个ServiceConnection连接对象，该对象用于监听访问者和Service之间的连接情况，当访问者和Service连接成功的时候，就会回调onServiceConnected(ComponentName name,IBinder service)方法，当访问者和Service之间断开连接的时候，就会回调onServiceDisconnected(ComponentName name)方法
>注意：如果访问者调用unBindService()方法断开连接的时候，并不会调用onServiceDisconnected(ComponentName name)方法，只有Service宿主所在的进程由于异常终止，才会调用
```
public interface ServiceConnection {
    void onServiceConnected(ComponentName var1, IBinder var2);

    void onServiceDisconnected(ComponentName var1);

    default void onBindingDied(ComponentName name) {
        throw new RuntimeException("Stub!");
    }
}
```
3. flags:指定绑定时是否自动创建Service(如果Service还没有被创建),0 不自定创建；BIND_AUTO_CREATE 自动创建

其中在onServiceConnected(ComponentName name,IBinder service)中有一个IBinder对象service，该对象可以实现和Service之间的通信。这时候Service类中的onBind方法就会获取到这个值，并且完成通信。

## Service的生命周期

![Service生命周期图](/assets/android/service-life.png)

在Service的生命周期中，有一个特殊的情况:
> 如果Service已经有某个客户端通过startService的方法启动了，接下来其他客户端调用bindService方法绑定后，在调用unBindService方法解除绑定，最后在调用bindService()方法再次绑定到Service，这个过程的生命周期，应该是:onCreate()>>onStartCommand()>>onBind()>>onUnbind()>>onRebind()

## IntentService
IntentService是Service的子类，因此他并不是普通的Service，他比普通的Service增加了额外的功能，首先我们应该知道Service的两个缺点：
1. Service不会专门启动一个单独的进程，Service和他所在的应用程序在同一个进程中
2. Service不是一个新的线程，因此不能在Service中处理耗时操作

除非我们特别为某个操作指定特定的线程，否则大部分在前台UI界面上的操作任务都执行在一个叫做UI Thread的特殊线程中。这可能存在某些隐患，因为部分在UI界面上的耗时操作可能会影响界面的响应性能。UI界面的性能问题会容易惹恼用户，甚至可能导致系统ANR错误。为了避免这样的问题，Android Framework提供了几个类，用来帮助你把那些耗时操作移动到后台线程中执行。那些类中最常用的就是IntentService.

### 创建IntentService
IntentService为在单一后台线程中执行任务提供了一种直接的实现方式。它可以处理一个耗时的任务并确保不影响到UI的响应性。另外IntentService的执行还不受UI生命周期的影响，以此来确保AsyncTask能够顺利运行。

但是IntentService依然存在着一下几个问题
1. 不可以直接和UI做交互。为了把他执行的结果体现在UI上，需要把结果返回给Activity
2. 工作任务队列是顺序执行的，如果一个任务正在IntentService中执行，此时你再发送一个新的任务请求，这个新的任务会一直等待直到前面一个任务执行完毕才开始执行
3. 正在执行的任务无法打断

虽然有上面那些限制，然而在在大多数情况下，IntentService都是执行简单后台任务操作的理想选择

创建一个IntentService组件，需要自定义一个新的类，它继承自IntentService，并重写onHandleIntent()方法
```
public class MyIntentService extends IntentService {
    public MyIntentService(String name) {
        super(name);
    }

    @Override
    protected void onHandleIntent(@Nullable Intent intent) {

    }
}
```
注意一个普通Service组件的其他回调，例如onStartCommand()会被IntentService自动调用。在IntentService中，要避免重写那些回调

IntentService需要在manifest文件添加相应的条目，将此条目<service>作为<application>元素的子元素下进行定义
```
 <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
        <service android:name=".MyService"></service>
        <service android:name=".MyIntentService"
            android:exported="false"></service>
    </application>
```
android:name属性指明了IntentService的名字。

注意<service>标签并没有包含任何intent filter。因为发送任务给IntentService的Activity需要使用显式Intent，所以不需要filter。这也意味着只有在同一个app或者其他使用同一个UserID的组件才能够访问到这个Service。

至此，你已经有了一个基本的IntentService类，你可以通过构造Intent对象向它发送操作请求。构造这些对象以及发送它们到你的IntentService的方式，将在接下来的课程中描述

### 向后台服务发送任务请求

为了创建一个任务请求并发送到IntentService。需要先创建一个显式Intent，并将请求数据添加到intent中，然后通过调用 startService() 方法把任务请求数据发送到IntentService
```
/*
 * Creates a new Intent to start the RSSPullService
 * IntentService. Passes a URI in the
 * Intent's "data" field.
 */
mServiceIntent = new Intent(getActivity(), RSSPullService.class);
mServiceIntent.setData(Uri.parse(dataUrl));
```
执行startService()方法
```
// Starts the IntentService
getActivity().startService(mServiceIntent);
```

注意可以在Activity或者Fragment的任何位置发送任务请求。例如，如果你先获取用户输入，您可以从响应按钮单击或类似手势的回调方法里面发送任务请求。

一旦执行了startService()，IntentService在自己本身的onHandleIntent()方法里面开始执行这个任务，任务结束之后，会自动停止这个Service。

下一步是如何把工作任务的执行结果返回给发送任务的Activity或者Fragment。下节课会演示如何使用BroadcastReceiver来完成这个任务

### 返回任务执行状态

为了在IntentService中向其他组件发送任务状态，首先创建一个Intent并在data字段中包含需要传递的信息。作为一个可选项，还可以给这个Intent添加一个action与data URI。

下一步，通过执行LocalBroadcastManager.sendBroadcast() 来发送Intent。Intent被发送到任何有注册接受它的组件中。为了获取到LocalBroadcastManager的实例，可以执行getInstance()
```
public final class Constants {
    ...
    // Defines a custom Intent action
    public static final String BROADCAST_ACTION =
        "com.example.android.threadsample.BROADCAST";
    ...
    // Defines the key for the status "extra" in an Intent
    public static final String EXTENDED_DATA_STATUS =
        "com.example.android.threadsample.STATUS";
    ...
}
public class RSSPullService extends IntentService {
...
    /*
     * Creates a new Intent containing a Uri object
     * BROADCAST_ACTION is a custom Intent action
     */
    Intent localIntent =
            new Intent(Constants.BROADCAST_ACTION)
            // Puts the status into the Intent
            .putExtra(Constants.EXTENDED_DATA_STATUS, status);
    // Broadcasts the Intent to receivers in this app.
    LocalBroadcastManager.getInstance(this).sendBroadcast(localIntent);
...
}
```
下一步是在发送任务的组件中接收发送出来的broadcast数据

为了接受广播的数据对象，需要使用BroadcastReceiver的子类并实现BroadcastReceiver.onReceive() 的方法，这里可以接收LocalBroadcastManager发出的广播数据

```
// Broadcast receiver for receiving status updates from the IntentService
private class ResponseReceiver extends BroadcastReceiver
{
    // Prevents instantiation
    private DownloadStateReceiver() {
    }
    // Called when the BroadcastReceiver gets an Intent it's registered to receive
    @
    public void onReceive(Context context, Intent intent) {
...
        /*
         * Handle Intents here.
         */
...
    }
}
```

一旦定义了BroadcastReceiver，也应该定义actions，categories与data用过滤广播。为了实现这些，需要使用IntentFilter
```
// Class that displays photos
public class DisplayActivity extends FragmentActivity {
    ...
    public void onCreate(Bundle stateBundle) {
        ...
        super.onCreate(stateBundle);
        ...
        // The filter's action is BROADCAST_ACTION
        IntentFilter mStatusIntentFilter = new IntentFilter(
                Constants.BROADCAST_ACTION);

        // Adds a data filter for the HTTP scheme
        mStatusIntentFilter.addDataScheme("http");
        ...
```

为了给系统注册这个BroadcastReceiver和IntentFilter，需要通过LocalBroadcastManager执行registerReceiver()的方法

```
// Instantiates a new DownloadStateReceiver
DownloadStateReceiver mDownloadStateReceiver =
        new DownloadStateReceiver();
// Registers the DownloadStateReceiver and its intent filters
LocalBroadcastManager.getInstance(this).registerReceiver(
        mDownloadStateReceiver,
        mStatusIntentFilter);

```

一个BroadcastReceiver可以处理多种类型的广播数据。每个广播数据都有自己的ACTION。这个功能使得不用定义多个不同的BroadcastReceiver来分别处理不同的ACTION数据。为BroadcastReceiver定义另外一个IntentFilter，只需要创建一个新的IntentFilter并重复执行registerReceiver()即可

```
/*
 * Instantiates a new action filter.
 * No data filter is needed.
 */
statusIntentFilter = new IntentFilter(Constants.ACTION_ZOOM_IMAGE);
...
// Registers the receiver with the new filter
LocalBroadcastManager.getInstance(getActivity()).registerReceiver(
        mDownloadStateReceiver,
        mIntentFilter);
```

发送一个广播Intent并不会启动或重启一个Activity。即使是你的app在后台运行，Activity的BroadcastReceiver也可以接收、处理Intent对象。但是这不会迫使你的app进入前台。当你的app不可见时，如果想通知用户一个发生在后台的事件，建议使用Notification。永远不要为了响应一个广播Intent而去启动Activity

## 几个简单的小例子

### 电话管理器

TelephonyManager是一个管理手机通话状态，电话网络信息的服务类，通过该类，我们可以获取电话网络的相关信息

```
package WangLi.Service.TelephonyStatus;  
  
import java.util.ArrayList;  
import java.util.HashMap;  
import java.util.Map;  
  
import android.app.Activity;  
import android.content.Context;  
import android.os.Bundle;  
import android.telephony.TelephonyManager;  
import android.widget.ListView;  
import android.widget.SimpleAdapter;  
  
public class TelephonyStatus extends Activity {  
    ListView showView;  
    // 声明代表状态名的数组  
    String[] statusNames;  
    // 声明代表手机状态的集合  
    ArrayList<String> statusValues = new ArrayList<String>();  
  
    /** Called when the activity is first created. */  
    @Override  
    public void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.main);  
        // 获到系统的TelephonyManager对象  
        TelephonyManager tManager = (TelephonyManager) getSystemService(Context.TELEPHONY_SERVICE);  
        // 获取各种状态名称的数组  
        statusNames = getResources().getStringArray(R.array.statusNames);  
        // 获取代表SIM卡状态的数组  
        String[] simState = getResources().getStringArray(R.array.simState);  
        // 获取代表电话网络类型的数组  
        String[] phoneType = getResources().getStringArray(R.array.phoneType);  
        // 获取设备编号  
        statusValues.add(tManager.getDeviceId());  
        // 获取系统平台的版本  
        statusValues.add(tManager.getDeviceSoftwareVersion() != null ? tManager  
                .getDeviceSoftwareVersion() : "未知");  
        // 获取网络运营商代号  
        statusValues.add(tManager.getNetworkOperator());  
        // 获取网络运营商名称  
        statusValues.add(tManager.getNetworkOperatorName());  
        // 获取手机网络类型  
        statusValues.add(phoneType[tManager.getPhoneType()]);  
        // 获取设备所在位置  
        statusValues.add(tManager.getCellLocation() != null ? tManager  
                .getCellLocation().toString() : "未知位置");  
        // 获取SIM卡的国别  
        statusValues.add(tManager.getSimCountryIso());  
        // 获取SIM卡的序列号  
        statusValues.add(tManager.getSimSerialNumber());  
        // 获取SIM卡的状态  
        statusValues.add(simState[tManager.getSimState()]);  
        // 获取ListView对象  
        showView = (ListView) findViewById(R.id.show);  
        ArrayList<Map<String, String>> status = new ArrayList<Map<String, String>>();  
        // 遍历statusValues集合,将statusNames,statusValues  
        // 的数据封装到List<Map<String,String>>集合中  
        for (int i = 0; i < statusValues.size(); i++) {  
            HashMap<String, String> map = new HashMap<String, String>();  
            map.put("name", statusNames[i]);  
            map.put("value", statusValues.get(i));  
            status.add(map);  
        }  
        // 使用SimpleAdapter封装List数据  
        SimpleAdapter adapter = new SimpleAdapter(this, status, R.layout.line,  
                new String[] { "name", "value" }, new int[] { R.id.name,  
                        R.id.value });  
        showView.setAdapter(adapter);  
    }  
} 
```
主页面布局文件
```
<?xml version="1.0" encoding="utf-8"?>  
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"  
    android:orientation="vertical"  
    android:layout_width="fill_parent"  
    android:layout_height="fill_parent"  
    >  
<ListView   
    android:id="@+id/show"  
    android:layout_width="fill_parent"   
    android:layout_height="fill_parent"   
    android:entries="@array/statusNames"  
    />  
</LinearLayout> 
```
创建数组文件
```
<?xml version="1.0" encoding="utf-8"?>  
<resources>  
    <!-- 声明一个名为statusNames的字符串数组 -->  
    <string-array name="statusNames">  
        <item>设备编号</item>  
        <item>软件版本</item>  
        <item>网络运营商代号</item>  
        <item>网络运营商名称</item>  
        <item>手机制式</item>  
        <item>设备当前位置</item>  
        <item>SIM卡的国别</item>  
        <item>SIM卡序列号</item>  
        <item>SIM卡状态</item>       
    </string-array>  
    <!-- 声明一个名为simState的字符串数组 -->  
    <string-array name="simState">  
        <item>状态未知</item>  
        <item>无SIM卡</item>  
        <item>被PIN加锁</item>  
        <item>被PUK加锁</item>  
        <item>被NetWork PIN加锁</item>  
        <item>已准备好</item>  
    </string-array>  
    <!-- 声明一个名为phoneType的字符串数组 -->  
    <string-array name="phoneType">     
        <item>未知</item>  
        <item>GSM</item>  
        <item>CDMA</item>  
    </string-array>     
</resources>  
```
别忘了声明权限
```
<!--  添加访问手机位置的权限  -->   
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />   
<!--  添加访问手机状态的权限  -->   
<uses-permission android:name="android.permission.READ_PHONE_STATE" />   
```

>android 6.0 为了保护用户权限，做出了一件保护用户信息的事情，一些权限需要动态申请，关于动态权限声明的问题，后面再解决

### 监听手机来电

```
/** 
 * Describe:</br> 
 * 监视手机来电 
 * 本实例实现了监视当前手机的来电状态， 
 * 并将来电与接听纪录写入log_phone文件中 
 * @author paulniu 
 * Date:2015.12.07 
 * */  
public class MonitorPhone extends Activity {  
    TelephonyManager tManager;  
    private String incomingNumber;  
    @Override  
    protected void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.main);  
        //获取系统的TelephonyManager对象  
        tManager=(TelephonyManager) getSystemService(Context.TELEPHONY_SERVICE);  
        //创建一个通话状态监听器  
        PhoneStateListener pListener=new PhoneStateListener(){  
            @Override  
            public void onCallStateChanged(int state, String number) {  
                // TODO Auto-generated method stub  
                switch (state) {  
                case TelephonyManager.CALL_STATE_IDLE://无任何状态                     
                    break;  
                case TelephonyManager.CALL_STATE_OFFHOOK://接听来电   
                    writeFile(state,number);  
                    break;  
                case TelephonyManager.CALL_STATE_RINGING://来电     
                    incomingNumber=number;  
                    writeFile(state,number);  
                    break;  
                default:  
                    break;  
                }                 
                super.onCallStateChanged(state, incomingNumber);  
            }             
        };  
        //为tManager添加监听器  
        tManager.listen(pListener, PhoneStateListener.LISTEN_CALL_STATE);  
    }  
   //将接听电话，与来电信息写入到文件  
    protected void writeFile(int state, String number) {  
        // TODO Auto-generated method stub  
        StringBuffer sb=new StringBuffer();  
        SimpleDateFormat sdf=new SimpleDateFormat("yyyy-MM-dd hh.mm.ss");  
        sb.append("时间："+sdf.format(new Date())+"\n");  
        switch (state) {          
        case TelephonyManager.CALL_STATE_OFFHOOK://接听来电   
            sb.append("接听了电话号为："+incomingNumber+"的电话");  
            break;  
        case TelephonyManager.CALL_STATE_RINGING://来电     
            sb.append(number+"来电");  
            break;        
        }  
        sb.append("\n-----------------------\n");  
        FileOutputStream fos=null;  
        try {  
            //以追加的方式打开输出流  
            fos=openFileOutput("log_phone", MODE_APPEND);  
              
        } catch (FileNotFoundException e) {  
            // TODO Auto-generated catch block  
            e.printStackTrace();  
        }         
        //将输出流封装成PrintStream对象  
        PrintStream ps=new PrintStream(fos);  
        //输入文件内容  
        ps.println(sb.toString());  
        //关闭输出流  
        ps.close();  
    }  
}  
```
权限声明
```
<!-- 授予应用读取通话状态的权限 -->  
<uses-permission android:name="android.permission.READ_PHONE_STATE"/>  
```

### 短信管理器

```
/**
 * 发送短信实例
 */
public class SendSmsActivity extends AppCompatActivity {

EditText phone,content;
    Button send;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_send_sms);

        //获取 SMSManager 管理器
        final SmsManager smsManager = SmsManager.getDefault();

        //初始化控件
        phone = (EditText) findViewById(R.id.et_phone);
        content = (EditText) findViewById(R.id.et_content);
        send = (Button) findViewById(R.id.btn_send);

        send.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {

                //创建一个 android.app.PendingIntent 对象
                PendingIntent pi = PendingIntent.getActivity(SendSmsActivity.this,0,new Intent(),0);

                //发送短信
                smsManager.sendTextMessage(phone.getText().toString(),null,content.getText().toString(),
                        pi,null);

                //提示短信发送完成
                Toast.makeText(SendSmsActivity.this, "短信发送完成", Toast.LENGTH_SHORT).show();
            }
        });
    }
}
```
布局文件
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context="com.test.smsmanagerdemo.SendSmsActivity">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal">

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="收件人"/>

        <EditText
            android:id="@+id/et_phone"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginLeft="10dp"/>
    </LinearLayout>


    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="10dp"
        android:orientation="horizontal">

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="发送内容"/>

        <EditText
            android:id="@+id/et_content"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginLeft="10dp"
            android:gravity="top"
            android:lines="5"
            android:text="你好"/>
    </LinearLayout>

<Button
    android:layout_gravity="center_horizontal"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="发送"
    android:id="@+id/btn_send"
    />
</LinearLayout>
```
权限声明
```
<uses-permission android:name="android.permission.SEND_SMS"/>
```

未完，待续

# 参考资料

[android官方培训课-胡凯大神翻译整理](http://hukai.me/android-training-course-in-chinese/index.html)


