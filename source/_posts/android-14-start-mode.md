---
title: 重拾android路(十四) 启动模式
date: 2016-08-28 11:15:07
tags:
  - android
---
启动模式
<!--more-->
# 任务栈

1. 程序打开之后会创建一个任务栈，会存储应用程序的activity，所有的activity属于一个任务栈
2. 一个任务栈包含多个activity，是一个activity的集合，用于有序的选择哪个activity和用户进行交互，只有栈顶的activity才可以和用户进行交互
3. 任务栈可以移动到后台，并且保留每个activity的状态，有序的给用户列出他们的作用，不会丢失他们的信息
4. 退出应用程序时，所有的任务栈中所有的activity都会被清除出栈，并且销毁任务栈，退出应用

> 缺点：
> 每次打开activity，都会向任务栈中加入一个新的对象，只有当所有的activity都退出的时候才可以销毁任务栈，退出应用，这样会让用户多次点击，体验较差
> 每次开启一个页面都会在任务栈中添加一个activity，会造成数据冗余，重复数据太多会造成内存溢出

为了解决任务栈的问题，引入了启动模式。
# 启动模式
启动模式在多个Activity跳转的时候扮演了非常重要的角色，他决定了是否生成新的activity对象，是否可以重用已经存在的activity对象，是否与其他activity公用同一个task。这里的task，是一个具有栈结构的对象，一个task可以管理多个activity，启动一个应用程序时也就创建了一个与之对应的task

Activity中有四种启动模式
1. standard
2. singleTop
3. singleTask
4. singleInstance

在配置启动模式的时候，我们可以直接在清单配置文件中设置即可，同时也可以通过startactivity flag设置

## standard
默认启动模式
activity_main.xml
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:paddingBottom="@dimen/activity_vertical_margin"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    tools:context="com.geniusvjr.standard_demo.MainActivity">

    <TextView
        android:id="@+id/tv"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>

    <Button
        android:id="@+id/btn_skip"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="跳转到MainActivity"/>
</LinearLayout>
```
MainAtivity.class
```
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        TextView textView = (TextView) findViewById(R.id.tv);
        textView.setText(this.toString());
        Button button = (Button) findViewById(R.id.btn_skip);
        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Intent intent = new Intent(MainActivity.this, MainActivity.class);
                startActivity(intent);
            }
        });
    }
}
```
MainActivity界面中的TextView用于显示当前Activity实例的序列号，Button用于跳转到下一个FirstActivity界面
当我们多次点击button时
![图片1](/assets/startmode/startmode01.png)
![图片2](/assets/startmode/startmode02.png)
![图片3](/assets/startmode/startmode03.png)
他们都是Activity实例，但是序列号是不同的
![图片4](/assets/startmode/startmode04.png)
如上图所示，每次点击button，系统都会在task中生成一个新的Mainactivity实例，并且放在栈顶的位置，只有当我们按下返回键时才会返回到原来的MainAcitivity中
> 所以standard模式，不管在task中是否存在了当前activity实例，都会创建一个新的对象

## singleTop
我们在上面的基础上为指定属性android:launchMode=”singleTop”，系统就会按照singleTop启动模式处理跳转行为。我们重复上面几个动作，将会出现下面的现象
![图片5](/assets/startmode/startmode05.png)
![图片6](/assets/startmode/startmode06.png)
![图片7](/assets/startmode/startmode07.png)
我们看到这个结果跟standard有所不同，两个序列号是相同的，也就是说使用的都是同一个MainActivity实例；如果按一下后退键，程序立即退出，说明当前栈结构中只有一个Activity实例。singleTop模式的原理如下图所示：
![图片8](/assets/startmode/startmode08.png)
如上图所示，当页面执行跳转的时候，会先查看当前task栈中栈顶是否是该activity实例对象，如果没有，创建一个新的activity实例，如果有则把这个已经存在的实例对象移到栈顶的位置
下面是一个例子，如果当前的Activity不在栈顶，我们需要执行的操作
创建一个新的secondAcitivity
```
public class SecondActivity extends Activity{

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_second);
        TextView textView = (TextView) findViewById(R.id.tv);
        textView.setText(this.toString());
        Button button = (Button) findViewById(R.id.btn_second);
        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Intent intent = new Intent(SecondActivity.this, MainActivity.class);
                startActivity(intent);
            }
        });
    }
}
```
把之前MainAcitivity中的button代码改成下面的内容
```
Intent intent = new Intent(MainActivity.this, SecondActivity.class);
                startActivity(intent);
```
这时候，FirstActivity会跳转到SecondActivity，SecondActivity又会跳转到FirstActivity
![图片9](/assets/startmode/startmode09.png)
![图片10](/assets/startmode/startmode10.png)
![图片11](/assets/startmode/startmode11.png)
从上面的例子我们可以看出，两次MainAcitivy的序列号是不同的
![图片12](/assets/startmode/startmode12.png)
总结上面的例子，我们会发现当页面从SecondActivity跳转到MainActivity中的时候，栈顶不是MainActivity，但是在task中存在MainActivity，这时候我们是创建了一个新的MainActivity实例。
这就是singleTop启动模式，如果发现有对应的Activity实例正位于栈顶，则重复利用，不再生成新的实例。
> 当然这种模式是很不合理的，但是也有自己的用武之地，例如QQ接受到消息后弹出Activity，如果一次来10条消息，总不能一次弹10个Activity

## singleTask
通过在清单配置文件中修改属性<pre>android:launchMode=”singleTask”</pre>,可以得到以下的结果
![图片13](/assets/startmode/startmode13.png)
![图片14](/assets/startmode/startmode14.png)
![图片15](/assets/startmode/startmode15.png)
![图片16](/assets/startmode/startmode16.png)
我们注意到，在上面的过程中，MainActivity的序列号是不变的，SecondActivity的序列号却不是唯一的，说明从SecondActivity跳转到MainActivity时，没有生成新的实例，但是从MainActivity跳转到SecondActivity时生成了新的实例。
![图片17](/assets/startmode/startmode17.png)
SecondActivity跳转到MainActivity后的栈结构变化的结果，我们注意到，SecondActivity消失了，没错，在这个跳转过程中系统发现有存在的FirstActivity实例，于是不再生成新的实例，而是将MainActivity之上的Activity实例统统出栈，将MainActivity变为栈顶对象，显示到幕前。也许朋友们有疑问，如果将SecondActivity也设置为singleTask模式，那么SecondActivity实例是不是可以唯一呢？在我们这个示例中是不可能的，因为每次从SecondActivity跳转到MainActivity时，SecondActivity实例都被迫出栈，下次等MainActivity跳转到SecondActivity时，找不到存在的SecondActivity实例，于是必须生成新的实例。但是如果我们有ThirdActivity，让SecondActivity和ThirdActivity互相跳转，那么SecondActivity实例就可以保证唯一。 
这就是singleTask模式，如果发现有对应的Activity实例，则使此Activity实例之上的其他Activity实例统统出栈，使此Activity实例成为栈顶对象，显示到幕前。

> singleTask适合作为程序入口点。例如浏览器的主界面。不管从多少个应用启动浏览器，只会启动主界面一次，其余情况都会走onNewIntent，并且会清空主界面上面的其他页面。之前打开过的页面，打开之前的页面就ok，不再新建。

## singleInstance
这个模式比较特殊，因为他会启动一个新的task结构，将Activity放在另一个新的栈结构中，并且保障不再有其他的activity实例进入
我们修改MainActivity的launchMode=”standard”，SecondActivity的launchMode=”singleInstance”，由于涉及到了多个栈结构，我们需要在每个Activity中显示当前栈结构的id，所以我们为每个Activity添加如下代码
```
TextView textView = (TextView) findViewById(R.id.tv);
        textView.setText("current task id:" + this.getTaskId());
```
![图片18](/assets/startmode/startmode18.png)
![图片19](/assets/startmode/startmode19.png)
我们会发现两个Activity实例被放在不同的栈结构中，原理如下
![图片20](/assets/startmode/startmode20.png)
我们看到从MainActivity跳转到SecondActivity时，重新启用了一个新的栈结构，来放置SecondActivity实例，然后按下后退键，再次回到原始栈结构；图中下半部分显示的在SecondActivity中再次跳转到MainActivity，这个时候系统会在原始栈结构中生成一个MainActivity实例，然后回退两次，注意，并没有退出，而是回到了SecondActivity，为什么呢？是因为从SecondActivity跳转到MainActivity的时候，我们的起点变成了SecondActivity实例所在的栈结构，这样一来，我们需要“回归”到这个栈结构。

> singleInstance适合需要与程序分离开的页面。例如闹铃提醒，将闹铃提醒与闹铃设置分离。singleInstance不要用于中间页面，如果用于中间页面，跳转会有问题，比如：A -> B (singleInstance) -> C，完全退出后，在此启动，首先打开的是B。

## java代码实现设置启动模式
```
Intent intent = new Intent（this， B.class）;   
intent.setFlags（Intent.FLAG_ACTIVITY_CLEAR_TOP）;  
startActivity（intent）;  
```
```
Intent intent = new Intent（this， MainActivity.class）;  
intent.addFlags（Intent.FLAG_ACTIVITY_REORDER_TO_FRONT）;   
startActivity（intent）;
```
# 总结
1. standard，标准模式，每次启动都会创建新的实例，不管实例是否存在 
![图片21](/assets/startmode/startmode21.png)
2. singleTop，栈顶复用，如果Activity处于栈顶，则不会创建新的Activity
![图片22](/assets/startmode/startmode22.png)
3. singleTask，栈内复用，只要Activity在栈内存在，那么多次启动该Activity都不会创建新的Activity实例。默认具有clear top的效果。
![图片23](/assets/startmode/startmode23.png)
4. singleInstance，在singleTask的基础上，限制了此模式的Activity只能单独地位于一个任务栈中
![图片24](/assets/startmode/startmode24.png)

# 参考资料
[activity四中启动模式](http://blog.csdn.net/CodeEmperor/article/details/50481726)
[深入理解启动模式](http://blog.csdn.net/zhuzp_blog/article/details/51367477)