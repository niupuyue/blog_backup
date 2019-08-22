---
title: 重拾android路(十五) broadcastReceiver
date: 2016-09-15 14:26:36
tags:
  - android
---

BroadcastReceiver 广播接受者用于接受系统或者其他应用程序发送的广播
<!--more-->
# 注册广播
在Android中如果我们想要使用广播，就必须自定义广播接收者
需要写一个类继承BroadcastReceiver，并且重写里面的onReceiver()方法，实现接受特定的广播，然后去执行相应的事情

自定义一个广播接受者
```
public class MyBroadCastReceiver extends BroadcastReceiver   
{  
   @Override  
   public void onReceive(Context context, Intent intent)   
   {   
       //在这里可以写相应的逻辑来实现一些功能
       //可以从Intent中获取数据、还可以调用BroadcastReceiver的getResultData()获取数据
   }   
}
```
这样我们就创建好了一个广播接受者，这个很简单，但是接下来我们要把这个广播注册起来，就像Activity，service需要在清单配置文件中注册一样。
## 两种注册广播的方式
### 代码中动态注册
步骤如下:
1. 实例化自定义好的广播接收者
2. 实例化意图过滤器，并设置需要过滤的广播类型(比如我们接受短信系统发送的广播)
3. 使用context中的registReceiver(BroadcastReceiver,IntentFilter,String,Handler)方法注册广播
```
//new出上边定义好的BroadcastReceiver
MyBroadCastReceiver yBroadCastReceiver = new MyBroadCastReceiver();

//实例化过滤器并设置要过滤的广播  
IntentFilter intentFilter = new IntentFilter("android.provider.Telephony.SMS_RECEIVED");

//注册广播   
myContext.registerReceiver(smsBroadCastReceiver,intentFilter, 
             "android.permission.RECEIVE_SMS", null);
```
### 在清单配置文件中注册
直接在Manifest.xml文件的<application>节点中配置广播接收者。
```
<receiver android:name=".MyBroadCastReceiver">  
            <!-- android:priority属性是设置此接收者的优先级（从-1000到1000） -->
            <intent-filter android:priority="20">
            <actionandroid:name="android.provider.Telephony.SMS_RECEIVED"/>  
            </intent-filter>  
</receiver>
```
还要在<code><application></code>同级的位置配置可能使用到的权限
```
<uses-permission android:name="android.permission.RECEIVE_SMS">
</uses-permission>
```
### 两种广播注册的方式
1. 第一种不是常驻广播，会随着生命周期的结束而消失
2. 第二种是常驻广播，就算应用关闭了，只要有相应的广播，都可以唤醒

# 发送广播
当我们需要发送一个自定义广播来通知应用程序中其他组件的状态时，就可以发送一个广播
两种发送广播的方式
1. 通过context.sendBroadcast(Intent)或者context.sendBroadcast(Intent,String)发送无序广播，其中后者是添加了权限
2. 通过mContext.sendOrderedBroadcast(Intent, String, BroadCastReceiver, Handler, int, String, Bundle)发送的是有序广播

> 无需广播:所有的接受者都可以接受事件，不可以被修改，不可以被拦截
> 有序广播:按照优先级，一级一级的往下传递，接受者可以修改广播数据，也可以终止广播

## 无序广播的使用
定义一个按钮，为他添加点击事件，然后发送一个无序广播
```
Intent intent = new Intent();
//设置intent的动作为com.example.broadcast，可以任意定义
intent.setAction("com.example.broadcast");
//发送无序广播
sendBroadcast(intent);
```
定义一个广播接受者，来接受这个广播
```
public class MyReceiver extends BroadcastReceiver {
    public MyReceiver() {
    }
    @Override
    public void onReceive(Context context, Intent intent) {
        Toast.makeText(context,"收到广播", Toast.LENGTH_SHORT).show();
    }
}
```
在清单配置文件中配置该接收者
```
<receiver
            android:name=".MyReceiver" >
            <intent-filter>
                <!-- 动作设置为发送的广播动作 -->
                <action android:name="com.example.broadcast"/>
            </intent-filter>
</receiver>
```
## 有序广播
和无序广播使用不同的是 通过 mContext.sendOrderedBroadcast(Intent, String, BroadCastReceiver, Handler, int, String, Bundle)和每个接收者设置优先级，就可以在小于自己优先级的接收者得到广播前，修改或终止广播。
定义一个按钮，设置其点击事件，发送一个有序广播。
```
        Intent intent = new  Intent();
        //设置intent的动作为com.example.broadcast，可以任意定义
        intent.setAction("com.example.broadcast");
        //发送无序广播
        //第一个参数：intent
        //第二个参数：String类型的接收者权限
        //第三个参数：BroadcastReceiver 指定的接收者
        //第四个参数：Handler scheduler
        //第五个参数：int 此次广播的标记 
        //第六个参数：String 初始数据
        //第七个参数：Bundle 往Intent中添加的额外数据
        sendOrderedBroadcast(intent, null, null, null, "这是初始数据", );
```
定义多个广播接收者，来接收这个广播事件。通过Toast的打印判断是否收到广播
```
public class MyReceiver1 extends BroadcastReceiver {
    public MyReceiver1() {
    }
    @Override
    public void onReceive(Context context, Intent intent) {
        //获取广播中的数据（即得到 "这是初始数据" 字符串）
        String message = getResultData();
        Toast.makeText(context ,message ,Toast.LENGTH_SHORT).show();
        //修改数据
        setResultData("这是修改后的数据");
    }
}
```
```
public class MyReceiver2 extends BroadcastReceiver {
    public MyReceiver2() {
    }
    @Override
    public void onReceive(Context context, Intent intent) {
        String message = getResultData();
        Toast.makeText(context ,message ,Toast.LENGTH_SHORT).show();
        //终止广播
        abortBroadcast();
    }
}
```
```
public class MyReceiver3 extends BroadcastReceiver {
    public MyReceiver3() {
    }
    @Override
    public void onReceive(Context context, Intent intent) {
        String message = getResultData();
        Toast.makeText(context ,message ,Toast.LENGTH_SHORT).show();
    }
}
```
在Manifest.xml中配置该接收者。并设置优先级：MyReceiver1>MyReceiver2>MyReceiver3。
```
<!-- 优先级相等的话，写在前面的receiver的优先级大于后面的 -->
<receiver
            android:name=".MyReceiver1" >
            <!-- 定义广播的优先级 -->
            <intent-filter android:priority="1000">                
                <!-- 动作设置为发送的广播动作 -->
                <action android:name="com.example.broadcast"/>
            </intent-filter>
</receiver>
<receiver 
               android:name=".MyReceiver2" >
                   <!-- 定义广播的优先级 -->
                   <intent-filter  android:priority="0">
                   <!-- 动作设置为发送的广播动作 -->
                   <action android:name="com.example.broadcast"/>
            </intent-filter>
</receiver>
<receiver 
               android:name=".MyReceiver3" >
                   <!-- 定义广播的优先级 -->
                   <intent-filter  android:priority="-1000">
                   <!-- 动作设置为发送的广播动作 -->
                   <action android:name="com.example.broadcast"/>
            </intent-filter>
</receiver>
```
运行结果：MyReceiver1得到广播数据后打印“这是初始数据”，MyReceiver2接收到广播数据打印“这是修改后的数据”，MyReceiver3没有打印。


# 参考资料
[android两种注册，发送广播的方式](https://www.jianshu.com/p/ea5e233d9f43)
