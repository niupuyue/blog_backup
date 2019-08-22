---
title: 重拾Android之路(二十二) eventbus
date: 2018-04-12 20:24:39
tags:
  - android
---

在进行项目开发的过程中，我们往往需要应用程序的各个组件，线程之间进行通信，比如子线程中进行请求数据，当数据请求完毕之后，通过Handler或者广播的方式通知UI更新，或者一个Activity中包含两个Fragment，当Activity进行一些操作的饿时候可以通知Fragment进行更新数据。当我们的项目越来越复杂的时候，我们所使用的Intent，Handler，广播就会非常多，然后我们管理起来非常不方便，这样就催生了EventBust的产生。目前使用的EventBus版本是3.0(官方已经更新到了3.1.1).
<!--more-->
# 定义
EventBus就是能够简化组件之间通信，让我们的代码书写变得更加简单，能够有效的分离事件发送方和接收方，更够避免复杂和容易出错的依赖性和生命周期问题的框架

# 关于EventBus
## 要素
1. Event 事件，可以是任意类型
2. Subscribe() 事件订阅者，或者说是事件接受者。在EventBus3之前，使用的时候，我们必须定义以onEvent开头的那几个方法，onEvent,onEventMainThread,onEventBackgroundThread和onEventAsync。但是在EventBus3之后我们处理事件的方法名可以随意取，只要加上响应的注解@subscribe()即可，并且指定了线程模型是POSTING
3. Publisher 事件发布者，就是事件是由他发布的，他通知别人去做什么事情。我们可以在任意地方发布事件，一般情况下，使用EventBus.getDefault()就可以得到一个EventBus对象，然后调用post(object)方法即可
## 四种线程模型
1. POSTING（默认） 表示事件处理函数的线程跟发布事件的线程是同一个线程(不管之前发布的消息的事主线程还是子线程，都使用发布事件对象所处的线程)
2. MAIN 表示事件处理函数的线程是主线程，也就是UI线程，如果我们需要执行UI更新，则需要适用这样的方式，但是注意不能执行耗时操作
3. BACKGOUND 表示事件处理函数的线程在后台线程，因此不能执行UI操作，如果发布时间的线程就是主线程，那么事件处理函数将会开启一个子线程，如果发布事件的线程就是子线程，那么事件处理的线程就直接使用该线程
4. ASYNC 表示无论时间发布的线程是哪一个，都会创建一个新的线程，这样我们是不能更新UI的

# 基本用法
这里我们举个例子，我们需要在一个Activity钟注册EventBus时间，然后定义接收方法，这有点像广播。所以我们的步骤如下
1. 定义一个事件类 表示我们当前需要传递的数据类型
2. 注册事件 表示当前的Activity需要监听某一个事件，如果事件发生了，则需要执行相依的操作，所以一般我们会在onCreate方法中注册事件
3. 解除事件 一般会在onDestroy方法中
4. 发送事件
5. 处理事件

# 一个例子
引入依赖
```
 compile 'org.greenrobot:eventbus:3.1.1'
```
定义一个事件类，比如这里我们需要发送一个学生对象
```
package c.niupule.eventbus_demo;

/**
 * Created: niupule
 * Date: 2018/4/12  下午8:49
 * E-mail:niupuyue@aliyun.com
 * des:
 */

public class Student {
    
    private int stu_id;
    private String stu_name;
    private String stu_school;
    private int stu_age;

    public Student() {
    }

    public Student(int stu_id, String stu_name, String stu_school, int stu_age) {
    
        this.stu_id = stu_id;
        this.stu_name = stu_name;
        this.stu_school = stu_school;
        this.stu_age = stu_age;
    }

    public int getStu_id() {
        return stu_id;
    }

    public void setStu_id(int stu_id) {
        this.stu_id = stu_id;
    }

    public String getStu_name() {
        return stu_name;
    }

    public void setStu_name(String stu_name) {
        this.stu_name = stu_name;
    }

    public String getStu_school() {
        return stu_school;
    }

    public void setStu_school(String stu_school) {
        this.stu_school = stu_school;
    }

    public int getStu_age() {
        return stu_age;
    }

    public void setStu_age(int stu_age) {
        this.stu_age = stu_age;
    }

    @Override
    public String toString() {
        return "Student{" +
                "stu_id=" + stu_id +
                ", stu_name='" + stu_name + '\'' +
                ", stu_school='" + stu_school + '\'' +
                ", stu_age=" + stu_age +
                '}';
    }
}
```
接着在onDestroy方法和onCreate方法中分别注册和解除事件对象
```
package c.niupule.eventbus_demo;

import android.content.Intent;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.view.View;
import android.widget.Button;
import android.widget.TextView;

import org.greenrobot.eventbus.EventBus;
import org.greenrobot.eventbus.Subscribe;
import org.greenrobot.eventbus.ThreadMode;

public class MainActivity extends AppCompatActivity {

    private TextView textView;
    private Button btn;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        EventBus.getDefault().register(this);

        textView = findViewById(R.id.text);

        btn = findViewById(R.id.btn);

        btn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                startActivity(new Intent(MainActivity.this,SecondActivity.class));
            }
        });

    }

    @Subscribe(threadMode = ThreadMode.MAIN)
    public void getText(Student ss){
        textView.setText(ss.toString());
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        if(EventBus.getDefault().isRegistered(this)){
            EventBus.getDefault().unregister(this);
        }
    }
}
```
事件处理，这里由于我们处理的是主线程中的数据，所以直接发送消息就可以了,在第二个Activity中发送消息
```
package c.niupule.eventbus_demo;

import android.os.Bundle;
import android.support.annotation.Nullable;
import android.support.v7.app.AppCompatActivity;
import android.view.View;

import org.greenrobot.eventbus.EventBus;

/**
 * Created: niupule
 * Date: 2018/4/12  下午8:58
 * E-mail:niupuyue@aliyun.com
 * des:
 */

public class SecondActivity extends AppCompatActivity{

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_second);

        findViewById(R.id.btn2).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                EventBus.getDefault().post(new Student(100,"张三","北京大学",22));
                finish();
            }
        });

    }
}
```
之后就可以实现了。
# 总结
这里只是简单的实现EventBus的简单实现，所以暂时先说到这里