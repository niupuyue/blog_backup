---
title: 重拾Android路(五) RxJava和RxAndroid
date: 2016-03-23 16:57:14
tags:
  - android
---

现在RxJava和RxAndroid越来越火爆，自己在业余时间也学习了一下，感觉确实很好用，之前 为了完成页面刷新，数据请求，组件信息传递的时候，要使用handler，真的是逻辑思路很强，稍微不注意，就各种错误一大堆。这下有了RxJava和RxAndroid，真的爽。
<!--more-->
# RxJava
网上有很多给RxJava做定义的，很多人说的比较官方，而我是比较笨的那种人，所以看了很久也没有看懂，不过最后在一个博客中，看到

> RxJava就是异步

我决定这句话概括的还是非常好的!!!
![](/assets/android/rxjava1.png)
不过我们还是要来先看一下官方给出的定义
## 定义
> RxJava：a library for composing asynchronous and event-based programs using observable sequences for the Java VM
> // 翻译：RxJava 是一个在 Java VM 上使用可观测的序列来组成异步的、基于事件的程序的库

所以说，RxJava 是一个 基于事件流、实现异步操作的库
### 特点:
RxJava的使用方式是：基于事件流的链式调用
1. 逻辑简洁
2. 实现优雅
3. 使用简单
更重要的是，随着程序逻辑的复杂性提高，它依然能够保持简洁 & 优雅
### 原理
这个地方使用的是别人的例子，我觉得很好
顾客到餐厅吃饭
![示意图](/assets/android/rxjava2.png)
![流程图](/assets/android/rxjava3.png)
RxJava是一种基于扩展的观察者模式
在RxJava中有四种角色：
|角色 |	作用 |	类比 |
|--------------------|:--------:|
|被观察者（Observable）|	产生事件 |	顾客|
|观察者（Observer）|	接收事件，并给出响应动作 |	厨房|
|订阅（Subscribe）|	连接 被观察者 & 观察者	| 服务员|
|事件（Event）|	被观察者 & 观察者 沟通的载体 |	菜式 |

![示意图](/assets/android/rxjava4.png)
![流程图](/assets/android/rxjava5.png)

即RxJava原理可总结为：被观察者 （Observable） 通过 订阅（Subscribe） 按顺序发送事件(Event) 给观察者 （Observer）， 观察者（Observer） 按顺序接收事件 & 作出对应的响应动作。具体如下图
![](/assets/android:rxjava6.png)
### 配置
```
compile 'io.reactivex.rxjava2:rxjava:2.0.1'
compile 'io.reactivex.rxjava2:rxandroid:2.0.1'
```

## RxJava的使用
使用方式：
1. 分步骤实现：该方法主要为了深入说明Rxjava的原理 & 使用，主要用于演示说明
2. 基于事件流的链式调用：主要用于实际使用

### 分步实现的步骤
![](/assets/android/rxjava7.png)

1. 创建被观察者(Obserable)和生产事件
- 即：顾客-坐下餐桌-点菜
具体实现：
```
// 1. 创建被观察者 Observable 对象
        Observable<Integer> observable = Observable.create(new ObservableOnSubscribe<Integer>() {
          // create() 是 RxJava 最基本的创造事件序列的方法
          // 此处传入了一个 OnSubscribe 对象参数
          // 当 Observable 被订阅时，OnSubscribe 的 call() 方法会自动被调用，即事件序列就会依照设定依次被触发
          // 即观察者会依次调用对应事件的复写方法从而响应事件
          // 从而实现被观察者调用了观察者的回调方法 & 由被观察者向观察者的事件传递，即观察者模式

        // 2. 在复写的subscribe（）里定义需要发送的事件
            @Override
            public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {
                // 通过 ObservableEmitter类对象产生事件并通知观察者
                // ObservableEmitter类介绍
                    // a. 定义：事件发射器
                    // b. 作用：定义需要发送的事件 & 向观察者发送事件
                emitter.onNext(1);
                emitter.onNext(2);
                emitter.onNext(3);
                emitter.onComplete();
            }
        });


        <--扩展：RxJava 提供了其他方法用于 创建被观察者对象Observable -->
// 方法1：just(T...)：直接将传入的参数依次发送出来
  Observable observable = Observable.just("A", "B", "C");
  // 将会依次调用：
  // onNext("A");
  // onNext("B");
  // onNext("C");
  // onCompleted();

// 方法2：from(T[]) / from(Iterable<? extends T>) : 将传入的数组 / Iterable 拆分成具体对象后，依次发送出来
  String[] words = {"A", "B", "C"};
  Observable observable = Observable.from(words);
  // 将会依次调用：
  // onNext("A");
  // onNext("B");
  // onNext("C");
  // onCompleted();

```
> 关于onComplete和onError唯一并且互斥这一点, 是需要自行在代码中进行控制, 如果你的代码逻辑中违背了这个规则, **并不一定会导致程序崩溃. ** 比如发送多个onComplete是可以正常运行的, 依然是收到第一个onComplete就不再接收了, 但若是发送多个onError, 则收到第二个onError事件会导致程序会崩溃.

2. 创建观察者 （Observer ）并 定义响应事件的行为
- 即 开厨房 - 确定对应菜式
发生的事件类型包括：Next事件、Complete事件 & Error事件
![](/assets/android/rxjava8.png)
具体实现:
```
//方式1：采用Observer 接口 -- >
        // 1. 创建观察者 （Observer ）对象
        Observer<Integer> observer = new Observer<Integer>() {
            // 2. 创建对象时通过对应复写对应事件方法 从而 响应对应事件

            // 观察者接收事件前，默认最先调用复写 onSubscribe（）
            @Override
            public void onSubscribe(Disposable d) {
                Log.d(TAG, "开始采用subscribe连接");
            }

            // 当被观察者生产Next事件 & 观察者接收到时，会调用该复写方法 进行响应
            @Override
            public void onNext(Integer value) {
                Log.d(TAG, "对Next事件作出响应" + value);
            }

            // 当被观察者生产Error事件& 观察者接收到时，会调用该复写方法 进行响应
            @Override
            public void onError(Throwable e) {
                Log.d(TAG, "对Error事件作出响应");
            }

            // 当被观察者生产Complete事件& 观察者接收到时，会调用该复写方法 进行响应
            @Override
            public void onComplete() {
                Log.d(TAG, "对Complete事件作出响应");
            }
        };

        //方式2：采用Subscriber 抽象类 -- >
        // 说明：Subscriber类 = RxJava 内置的一个实现了 Observer 的抽象类，对 Observer 接口进行了扩展

        // 1. 创建观察者 （Observer ）对象
        Subscriber<String> subscriber = new Subscriber<Integer>() {

            // 2. 创建对象时通过对应复写对应事件方法 从而 响应对应事件
            // 观察者接收事件前，默认最先调用复写 onSubscribe（）
            @Override
            public void onSubscribe(Subscription s) {
                Log.d(TAG, "开始采用subscribe连接");
            }

            // 当被观察者生产Next事件 & 观察者接收到时，会调用该复写方法 进行响应
            @Override
            public void onNext(Integer value) {
                Log.d(TAG, "对Next事件作出响应" + value);
            }

            // 当被观察者生产Error事件& 观察者接收到时，会调用该复写方法 进行响应
            @Override
            public void onError(Throwable e) {
                Log.d(TAG, "对Error事件作出响应");
            }

            // 当被观察者生产Complete事件& 观察者接收到时，会调用该复写方法 进行响应
            @Override
            public void onComplete() {
                Log.d(TAG, "对Complete事件作出响应");
            }
        };

        //特别注意：2 种方法的区别，即Subscriber 抽象类与Observer 接口的区别-- >
        // 相同点：二者基本使用方式完全一致（实质上，在RxJava的 subscribe 过程中，Observer总是会先被转换成Subscriber再使用）
        // 不同点：Subscriber抽象类对 Observer 接口进行了扩展，新增了两个方法：
        // 1. onStart()：在还未响应事件前调用，用于做一些初始化工作
        // 2. unsubscribe()：用于取消订阅。在该方法被调用后，观察者将不再接收 & 响应事件
        // 调用该方法前，先使用 isUnsubscribed() 判断状态，确定被观察者Observable是否还持有观察者Subscriber的引用，如果引用不能及时释放，就会出现内存泄露
```
3. 通过订阅（Subscribe）连接观察者和被观察者
- 顾客找到服务员 - 点菜 - 服务员下单到厨房 - 厨房烹调
具体实现:
```
observable.subscribe(observer);
 // 或者 observable.subscribe(subscriber)；
 ```
 扩展说明：
 ```
 <-- Observable.subscribe(Subscriber) 的内部实现 -->

public Subscription subscribe(Subscriber subscriber) {
    subscriber.onStart();
    // 步骤1中 观察者  subscriber抽象类复写的方法，用于初始化工作
    onSubscribe.call(subscriber);
    // 通过该调用，从而回调观察者中的对应方法从而响应被观察者生产的事件
    // 从而实现被观察者调用了观察者的回调方法 & 由被观察者向观察者的事件传递，即观察者模式
    // 同时也看出：Observable只是生产事件，真正的发送事件是在它被订阅的时候，即当 subscribe() 方法执行时
}
```
### 基于事件流的链式调用
在实际应用中，会将上述步骤&代码连在一起，从而更加简洁、更加优雅，即所谓的 RxJava基于事件流的链式调用
```
// RxJava的链式操作
        Observable.create(new ObservableOnSubscribe<Integer>() {
        // 1. 创建被观察者 & 生产事件
            @Override
            public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {
                emitter.onNext(1);
                emitter.onNext(2);
                emitter.onNext(3);
                emitter.onComplete();
            }
        }).subscribe(new Observer<Integer>() {
            // 2. 通过通过订阅（subscribe）连接观察者和被观察者
            // 3. 创建观察者 & 定义响应事件的行为
            @Override
            public void onSubscribe(Disposable d) {
                Log.d(TAG, "开始采用subscribe连接");
            }
            // 默认最先调用复写的 onSubscribe（）

            @Override
            public void onNext(Integer value) {
                Log.d(TAG, "对Next事件"+ value +"作出响应"  );
            }

            @Override
            public void onError(Throwable e) {
                Log.d(TAG, "对Error事件作出响应");
            }

            @Override
            public void onComplete() {
                Log.d(TAG, "对Complete事件作出响应");
            }

        });
    }
}
```
> 注意:整体方法调用顺序：观察者.onSubscribe（）> 被观察者.subscribe（）> 观察者.onNext（）>观察者.onComplete() 

随着程序逻辑的复杂性提高，它依然能够保持简洁 & 优雅。所以，一般建议使用这种基于事件流的链式调用方式实现RxJava
RxJava 2.x 提供了多个函数式接口 ，用于实现简便式的观察者模式
![](/assets/android/rxjava9.png)
以consumer为例，实现观察者模式
```
Observable.just("hello").subscribe(new Consumer<String>() {
            // 每次接收到Observable的事件都会调用Consumer.accept（）
            @Override
            public void accept(String s) throws Exception {
                System.out.println(s);
            }
        });
```
### 实际使用
#### 使用分步实现
```
package com.paulniu.rxjavademo;

import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.util.Log;

import org.reactivestreams.Subscriber;
import org.reactivestreams.Subscription;

import io.reactivex.Observable;
import io.reactivex.ObservableEmitter;
import io.reactivex.ObservableOnSubscribe;
import io.reactivex.Observer;
import io.reactivex.disposables.Disposable;

public class MainActivity extends AppCompatActivity {

    public static final String TAG = "MainActivity";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        //1.创建观察者Observable对象
        Observable<Integer>observable = Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(ObservableEmitter<Integer> e) throws Exception {
                // 在复写的subscribe（）里定义需要发送的事件
                // 通过 ObservableEmitter类对象产生事件并通知观察者
                // ObservableEmitter类介绍
                // a. 定义：事件发射器
                // b. 作用：定义需要发送的事件 & 向观察者发送事件

                e.onNext(1);
                e.onNext(2);
                e.onNext(3);
                //完成发送
                e.onComplete();
            }
        });
        //2：创建观察者 Observer 并 定义响应事件行为
        Observer<Integer>observer = new Observer<Integer>() {
            // 通过复写对应方法来 响应 被观察者

            @Override
            public void onSubscribe(Disposable d) {
                // 默认最先调用复写的 onSubscribe（）
                Log.d(TAG, "开始采用subscribe连接");
            }

            @Override
            public void onNext(Integer value) {
                Log.d(TAG, "对Next事件"+ value +"作出响应"  );
            }

            @Override
            public void onError(Throwable e) {
                Log.d(TAG, "对Error事件作出响应");
            }

            @Override
            public void onComplete() {
                Log.d(TAG, "对Complete事件作出响应");
            }
        };

        //3：通过订阅（subscribe）连接观察者和被观察者
        observable.subscribe(observer);
    }
}
```
实验结果：
![](/assets/android/rxjava10.png)
#### 基于事件流的链式调用方式
```
// RxJava的流式操作
        Observable.create(new ObservableOnSubscribe<Integer>() {
        // 1. 创建被观察者 & 生产事件
            @Override
            public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {
                emitter.onNext(1);
                emitter.onNext(2);
                emitter.onNext(3);
                emitter.onComplete();
            }
        }).subscribe(new Observer<Integer>() {
            // 2. 通过通过订阅（subscribe）连接观察者和被观察者
            // 3. 创建观察者 & 定义响应事件的行为
            @Override
            public void onSubscribe(Disposable d) {
                Log.d(TAG, "开始采用subscribe连接");
            }
            // 默认最先调用复写的 onSubscribe（）

            @Override
            public void onNext(Integer value) {
                Log.d(TAG, "对Next事件"+ value +"作出响应"  );
            }

            @Override
            public void onError(Throwable e) {
                Log.d(TAG, "对Error事件作出响应");
            }

            @Override
            public void onComplete() {
                Log.d(TAG, "对Complete事件作出响应");
            }

        });
```
实现效果和上图一样

> 除了此之外，观察者Observer的subscribe()具有重载方法

```
public final Disposable subscribe() {}
    // 表示观察者不对被观察者发送的事件作出任何响应（但被观察者还是可以继续发送事件）

    public final Disposable subscribe(Consumer<? super T> onNext) {}
    // 表示观察者只对被观察者发送的Next事件作出响应
    public final Disposable subscribe(Consumer<? super T> onNext, Consumer<? super Throwable> onError) {} 
    // 表示观察者只对被观察者发送的Next事件 & Error事件作出响应

    public final Disposable subscribe(Consumer<? super T> onNext, Consumer<? super Throwable> onError, Action onComplete) {}
    // 表示观察者只对被观察者发送的Next事件、Error事件 & Complete事件作出响应

    public final Disposable subscribe(Consumer<? super T> onNext, Consumer<? super Throwable> onError, Action onComplete, Consumer<? super Disposable> onSubscribe) {}
    // 表示观察者只对被观察者发送的Next事件、Error事件 、Complete事件 & onSubscribe事件作出响应

    public final void subscribe(Observer<? super T> observer) {}
    // 表示观察者对被观察者发送的任何事件都作出响应
```

> 可采用 Disposable.dispose() 切断观察者 与 被观察者 之间的连接
> 即观察者 无法继续 接收 被观察者的事件，但被观察者还是可以继续发送事件

```
// 主要在观察者 Observer中 实现
        Observer<Integer> observer = new Observer<Integer>() {
            // 1. 定义Disposable类变量
            private Disposable mDisposable;

            @Override
            public void onSubscribe(Disposable d) {
                Log.d(TAG, "开始采用subscribe连接");
                // 2. 对Disposable类变量赋值
                mDisposable = d;
            }

            @Override
            public void onNext(Integer value) {
                Log.d(TAG, "对Next事件"+ value +"作出响应"  );
                if (value == 2) {
                    // 设置在接收到第二个事件后切断观察者和被观察者的连接
                    mDisposable.dispose();
                    Log.d(TAG, "已经切断了连接：" + mDisposable.isDisposed());
                }
            }

            @Override
            public void onError(Throwable e) {
                Log.d(TAG, "对Error事件作出响应");
            }

            @Override
            public void onComplete() {
                Log.d(TAG, "对Complete事件作出响应");
            }
        };
```
运行结果
![](/assets/android/rxjava11.png)

## RxJava的操作符
作用：创建被观察者对象(Observable)和发送事件
![](/assets/android/rxjava12.png)
### 基本创建
完整创建一个被观察者,同时也是创建被观察者对象最基本的操作符
#### create()
具体使用
```
/ **
   * 1. 通过creat（）创建被观察者 Observable 对象
   */ 
        Observable<Integer> observable = Observable.create(new ObservableOnSubscribe<Integer>() {
          // 传入参数： OnSubscribe 对象
          // 当 Observable 被订阅时，OnSubscribe 的 call() 方法会自动被调用，即事件序列就会依照设定依次被触发
          // 即观察者会依次调用对应事件的复写方法从而响应事件
          // 从而实现由被观察者向观察者的事件传递 & 被观察者调用了观察者的回调方法 ，即观察者模式
/ **
   * 2. 在复写的subscribe（）里定义需要发送的事件
   */ 
            @Override
            public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {
                // 通过 ObservableEmitter类对象 产生 & 发送事件
                // ObservableEmitter类介绍
                    // a. 定义：事件发射器
                    // b. 作用：定义需要发送的事件 & 向观察者发送事件
                   // 注：建议发送事件前检查观察者的isUnsubscribed状态，以便在没有观察者时，让Observable停止发射数据
                    if (!observer.isUnsubscribed()) {
                           emitter.onNext(1);
                           emitter.onNext(2);
                           emitter.onNext(3);
                }
                emitter.onComplete();
            }
        });

// 至此，一个完整的被观察者对象（Observable）就创建完毕了。
```
但是在使用的时候一般采用链式方式
```
// 1. 通过creat（）创建被观察者对象
        Observable.create(new ObservableOnSubscribe<Integer>() {

            // 2. 在复写的subscribe（）里定义需要发送的事件
            @Override
            public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {

                    emitter.onNext(1);
                    emitter.onNext(2);
                    emitter.onNext(3);

                emitter.onComplete();
            }  // 至此，一个被观察者对象（Observable）就创建完毕
        }).subscribe(new Observer<Integer>() {
            // 以下步骤仅为展示一个完整demo，可以忽略
            // 3. 通过通过订阅（subscribe）连接观察者和被观察者
            // 4. 创建观察者 & 定义响应事件的行为
            @Override
            public void onSubscribe(Disposable d) {
                Log.d(TAG, "开始采用subscribe连接");
            }
            // 默认最先调用复写的 onSubscribe（）

            @Override
            public void onNext(Integer value) {
                Log.d(TAG, "接收到了事件"+ value  );
            }

            @Override
            public void onError(Throwable e) {
                Log.d(TAG, "对Error事件作出响应");
            }

            @Override
            public void onComplete() {
                Log.d(TAG, "对Complete事件作出响应");
            }

        });
    }
```
结果
![](/assets/android/rxjava13.png)
#### just()
快速创建1个被观察者对象（Observable），直接发送 传入的事件

> 最多只能传10个参数

具体使用:
```
// 1. 创建时传入整型1、2、3、4
        // 在创建后就会发送这些对象，相当于执行了onNext(1)、onNext(2)、onNext(3)、onNext(4)
        Observable.just(1, 2, 3,4)   
            // 至此，一个Observable对象创建完毕，以下步骤仅为展示一个完整demo，可以忽略
            // 2. 通过通过订阅（subscribe）连接观察者和被观察者
            // 3. 创建观察者 & 定义响应事件的行为
         .subscribe(new Observer<Integer>() {
            
            @Override
            public void onSubscribe(Disposable d) {
                Log.d(TAG, "开始采用subscribe连接");
            }
            // 默认最先调用复写的 onSubscribe（）

            @Override
            public void onNext(Integer value) {
                Log.d(TAG, "接收到了事件"+ value  );
            }

            @Override
            public void onError(Throwable e) {
                Log.d(TAG, "对Error事件作出响应");
            }

            @Override
            public void onComplete() {
                Log.d(TAG, "对Complete事件作出响应");
            }

        });
    }
```
结果:
![](/assets/android/rxjava14.png)
#### fromArray()
快速创建1个被观察者对象（Observable）,直接发送 传入的数组数据

> 会将数组中的数据转换为Observable对象

具体使用：
```
// 1. 设置需要传入的数组
     Integer[] items = { 0, 1, 2, 3, 4 };

        // 2. 创建被观察者对象（Observable）时传入数组
        // 在创建后就会将该数组转换成Observable & 发送该对象中的所有数据
        Observable.fromArray(items) 
        .subscribe(new Observer<Integer>() {
            @Override
            public void onSubscribe(Disposable d) {
                Log.d(TAG, "开始采用subscribe连接");
            }

            @Override
            public void onNext(Integer value) {
                Log.d(TAG, "接收到了事件"+ value  );
            }

            @Override
            public void onError(Throwable e) {
                Log.d(TAG, "对Error事件作出响应");
            }

            @Override
            public void onComplete() {
                Log.d(TAG, "对Complete事件作出响应");
            }

        });
    }

// 注：
// 可发送10个以上参数
// 若直接传递一个list集合进去，否则会直接把list当做一个数据元素发送

/*
  * 数组遍历
  **/
        // 1. 设置需要传入的数组
        Integer[] items = { 0, 1, 2, 3, 4 };

        // 2. 创建被观察者对象（Observable）时传入数组
        // 在创建后就会将该数组转换成Observable & 发送该对象中的所有数据
        Observable.fromArray(items)
                .subscribe(new Observer<Integer>() {
                    @Override
                    public void onSubscribe(Disposable d) {
                        Log.d(TAG, "数组遍历");
                    }

                    @Override
                    public void onNext(Integer value) {
                        Log.d(TAG, "数组中的元素 = "+ value  );
                    }

                    @Override
                    public void onError(Throwable e) {
                        Log.d(TAG, "对Error事件作出响应");
                    }

                    @Override
                    public void onComplete() {
                        Log.d(TAG, "遍历结束");
                    }

                });
```
结果：
![发送事件](/assets/android/rxjava15.png)
![数组遍历](/assets/android/rxjava16.png)
#### fromIterable()
快速创建1个被观察者对象（Observable）,直接发送 传入的集合List数据

> 会将数组中的数据转换为Observable对象

具体实现
```
/*
 * 快速发送集合
 **/
// 1. 设置一个集合
        List<Integer> list = new ArrayList<>();
        list.add(1);
        list.add(2);
        list.add(3);

// 2. 通过fromIterable()将集合中的对象 / 数据发送出去
        Observable.fromIterable(list)
                .subscribe(new Observer<Integer>() {
                    @Override
                    public void onSubscribe(Disposable d) {
                        Log.d(TAG, "开始采用subscribe连接");
                    }

                    @Override
                    public void onNext(Integer value) {
                        Log.d(TAG, "接收到了事件"+ value  );
                    }

                    @Override
                    public void onError(Throwable e) {
                        Log.d(TAG, "对Error事件作出响应");
                    }

                    @Override
                    public void onComplete() {
                        Log.d(TAG, "对Complete事件作出响应");
                    }
                });


/*
 * 集合遍历
 **/
        // 1. 设置一个集合
        List<Integer> list = new ArrayList<>();
        list.add(1);
        list.add(2);
        list.add(3);

        // 2. 通过fromIterable()将集合中的对象 / 数据发送出去
        Observable.fromIterable(list)
                .subscribe(new Observer<Integer>() {
                    @Override
                    public void onSubscribe(Disposable d) {
                        Log.d(TAG, "集合遍历");
                    }

                    @Override
                    public void onNext(Integer value) {
                        Log.d(TAG, "集合中的数据元素 = "+ value  );
                    }

                    @Override
                    public void onError(Throwable e) {
                        Log.d(TAG, "对Error事件作出响应");
                    }

                    @Override
                    public void onComplete() {
                        Log.d(TAG, "遍历结束");
                    }
                });
```
结果：
![发送集合](/assets/android/rxjava17.png)
![集合遍历](/assets/android/rxjava18.png)

> 其他

```
// 下列方法一般用于测试使用

<-- empty()  -->
// 该方法创建的被观察者对象发送事件的特点：仅发送Complete事件，直接通知完成
Observable observable1=Observable.empty(); 
// 即观察者接收后会直接调用onCompleted（）

<-- error()  -->
// 该方法创建的被观察者对象发送事件的特点：仅发送Error事件，直接通知异常
// 可自定义异常
Observable observable2=Observable.error(new RuntimeException())
// 即观察者接收后会直接调用onError（）

<-- never()  -->
// 该方法创建的被观察者对象发送事件的特点：不发送任何事件
Observable observable3=Observable.never();
// 即观察者接收后什么都不调用
```

### 延迟创建
使用场景:
1. 定时操作：在经过了x秒后，需要自动执行y操作
2. 周期性操作：每隔x秒后，需要自动执行y操作
#### defer()

> 作用： 直到有观察者（Observer ）订阅时，才动态创建被观察者对象（Observable） & 发送事件

- 通过 Observable工厂方法创建被观察者对象（Observable）
- 每次订阅后，都会得到一个刚创建的最新的Observable对象，这可以确保Observable对象里的数据是最新的
具体使用：
```
<-- 1. 第1次对i赋值 ->>
        Integer i = 10;

        // 2. 通过defer 定义被观察者对象
        // 注：此时被观察者对象还没创建
        Observable<Integer> observable = Observable.defer(new Callable<ObservableSource<? extends Integer>>() {
            @Override
            public ObservableSource<? extends Integer> call() throws Exception {
                return Observable.just(i);
            }
        });

        <-- 2. 第2次对i赋值 ->>
        i = 15;

        <-- 3. 观察者开始订阅 ->>
        // 注：此时，才会调用defer（）创建被观察者对象（Observable）
        observable.subscribe(new Observer<Integer>() {

            @Override
            public void onSubscribe(Disposable d) {
                Log.d(TAG, "开始采用subscribe连接");
            }

            @Override
            public void onNext(Integer value) {
                Log.d(TAG, "接收到的整数是"+ value  );
            }

            @Override
            public void onError(Throwable e) {
                Log.d(TAG, "对Error事件作出响应");
            }

            @Override
            public void onComplete() {
                Log.d(TAG, "对Complete事件作出响应");
            }
        });
```
结果：
因为是在订阅时才创建，所以i值会取第2次的赋值
![](/assets/android/rxjava19.png)

#### timer()
- 快速创建1个被观察者对象（Observable）
- 发送事件的特点：延迟指定时间后，发送1个数值0（Long类型）

> 本质 = 延迟指定时间后，调用一次 onNext(0)

具体使用:
```
// 该例子 = 延迟2s后，发送一个long类型数值
        Observable.timer(2, TimeUnit.SECONDS) 
                  .subscribe(new Observer<Long>() {
            @Override
            public void onSubscribe(Disposable d) {
                Log.d(TAG, "开始采用subscribe连接");
            }

            @Override
            public void onNext(Long value) {
                Log.d(TAG, "接收到了事件"+ value  );
            }

            @Override
            public void onError(Throwable e) {
                Log.d(TAG, "对Error事件作出响应");
            }

            @Override
            public void onComplete() {
                Log.d(TAG, "对Complete事件作出响应");
            }

        });

// 注：timer操作符默认运行在一个新线程上
// 也可自定义线程调度器（第3个参数）：timer(long,TimeUnit,Scheduler)
```
结果
![](/assets/android/rxjava20.png)
#### interval()
- 快速创建1个被观察者对象（Observable）
- 发送事件的特点：每隔指定时间 就发送 事件

> 发送的事件序列 = 从0开始、无限递增1的的整数序列

具体实现
```
// 参数说明：
        // 参数1 = 第1次延迟时间；
        // 参数2 = 间隔时间数字；
        // 参数3 = 时间单位；
        Observable.interval(3,1,TimeUnit.SECONDS)
                // 该例子发送的事件序列特点：延迟3s后发送事件，每隔1秒产生1个数字（从0开始递增1，无限个）
                .subscribe(new Observer<Long>() {
                    @Override
                    public void onSubscribe(Disposable d) {
                        Log.d(TAG, "开始采用subscribe连接");
                    }
                    // 默认最先调用复写的 onSubscribe（）

                    @Override
                    public void onNext(Long value) {
                        Log.d(TAG, "接收到了事件"+ value  );
                    }

                    @Override
                    public void onError(Throwable e) {
                        Log.d(TAG, "对Error事件作出响应");
                    }

                    @Override
                    public void onComplete() {
                        Log.d(TAG, "对Complete事件作出响应");
                    }

                });

// 注：interval默认在computation调度器上执行
// 也可自定义指定线程调度器（第3个参数）：interval(long,TimeUnit,Scheduler)
```
结果：
![](/assets/android/rxjava21.gif)
#### intervalRange()
- 快速创建1个被观察者对象（Observable）
- 发送事件的特点：每隔指定时间 就发送 事件，可指定发送的数据的数量

> a. 发送的事件序列 = 从0开始、无限递增1的的整数序列
> b. 作用类似于interval（），但可指定发送的数据的数量

具体实现
```
// 参数说明：
        // 参数1 = 事件序列起始点；
        // 参数2 = 事件数量；
        // 参数3 = 第1次事件延迟发送时间；
        // 参数4 = 间隔时间数字；
        // 参数5 = 时间单位
        Observable.intervalRange(3,10,2, 1, TimeUnit.SECONDS)
                // 该例子发送的事件序列特点：
                // 1. 从3开始，一共发送10个事件；
                // 2. 第1次延迟2s发送，之后每隔2秒产生1个数字（从0开始递增1，无限个）
                .subscribe(new Observer<Long>() {
                    @Override
                    public void onSubscribe(Disposable d) {
                        Log.d(TAG, "开始采用subscribe连接");
                    }
                    // 默认最先调用复写的 onSubscribe（）

                    @Override
                    public void onNext(Long value) {
                        Log.d(TAG, "接收到了事件"+ value  );
                    }

                    @Override
                    public void onError(Throwable e) {
                        Log.d(TAG, "对Error事件作出响应");
                    }

                    @Override
                    public void onComplete() {
                        Log.d(TAG, "对Complete事件作出响应");
                    }

                });

```
结果：
![](/assets/android/rxjava22.png)
#### range()
- 快速创建1个被观察者对象（Observable）
- 发送事件的特点：连续发送 1个事件序列，可指定范围

> a. 发送的事件序列 = 从0开始、无限递增1的的整数序列
> b. 作用类似于intervalRange（），但区别在于：无延迟发送事件

具体实现：
```
// 参数说明：
        // 参数1 = 事件序列起始点；
        // 参数2 = 事件数量；
        // 注：若设置为负数，则会抛出异常
        Observable.range(3,10)
                // 该例子发送的事件序列特点：从3开始发送，每次发送事件递增1，一共发送10个事件
                .subscribe(new Observer<Integer>() {
                    @Override
                    public void onSubscribe(Disposable d) {
                        Log.d(TAG, "开始采用subscribe连接");
                    }
                    // 默认最先调用复写的 onSubscribe（）

                    @Override
                    public void onNext(Integer value) {
                        Log.d(TAG, "接收到了事件"+ value  );
                    }

                    @Override
                    public void onError(Throwable e) {
                        Log.d(TAG, "对Error事件作出响应");
                    }

                    @Override
                    public void onComplete() {
                        Log.d(TAG, "对Complete事件作出响应");
                    }

                });

```
结果：
![](/assets/android/rxjava23.png)
#### rangeLong()
- 作用：类似于range（），区别在于该方法支持数据类型 = Long
- 具体使用与range（）类似，此处不作过多描述


最后的总结：
![](/assets/android/rxjava24.png)


一个例子：
1. 添加依赖
```
// Android 支持 Rxjava
    // 此处一定要注意使用RxJava2的版本
    compile 'io.reactivex.rxjava2:rxjava:2.0.1'
    compile 'io.reactivex.rxjava2:rxandroid:2.0.1'

    // Android 支持 Retrofit
    compile 'com.squareup.retrofit2:retrofit:2.1.0'

    // 衔接 Retrofit & RxJava
    // 此处一定要注意使用RxJava2的版本
    compile 'com.jakewharton.retrofit:retrofit2-rxjava2-adapter:1.0.0'

    // 支持Gson解析
    compile 'com.squareup.retrofit2:converter-gson:2.1.0'

    compile 'com.orhanobut:logger:2.1.1'
```
2. 创建实体类
```
package com.paulniu.rxjavademo;

import java.util.List;

/**
 * Created by niupule on 2017/12/30.
 */

public class MovieBean {


    /**
     * count : 20
     * start : 0
     * total : 25
     * subjects : [{"rating":{"max":10,"average":4.9,"stars":"25","min":0},"genres":["喜剧"],"title":"妖铃铃","casts":[{"alt":"https://movie.douban.com/celebrity/1127819/","avatars":{"small":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p45539.webp","large":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p45539.webp","medium":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p45539.webp"},"name":"吴君如","id":"1127819"},{"alt":"https://movie.douban.com/celebrity/1325700/","avatars":{"small":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1356510694.28.webp","large":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1356510694.28.webp","medium":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1356510694.28.webp"},"name":"沈腾","id":"1325700"},{"alt":"https://movie.douban.com/celebrity/1317663/","avatars":{"small":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p40756.webp","large":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p40756.webp","medium":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p40756.webp"},"name":"岳云鹏","id":"1317663"}],"collect_count":10595,"original_title":"妖铃铃","subtype":"movie","directors":[{"alt":"https://movie.douban.com/celebrity/1127819/","avatars":{"small":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p45539.webp","large":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p45539.webp","medium":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p45539.webp"},"name":"吴君如","id":"1127819"}],"year":"2017","images":{"small":"https://img3.doubanio.com/view/photo/s_ratio_poster/public/p2506825662.webp","large":"https://img3.doubanio.com/view/photo/s_ratio_poster/public/p2506825662.webp","medium":"https://img3.doubanio.com/view/photo/s_ratio_poster/public/p2506825662.webp"},"alt":"https://movie.douban.com/subject/26966580/","id":"26966580"},{"rating":{"max":10,"average":0,"stars":"00","min":0},"genres":["儿童","动画","奇幻"],"title":"小猫巴克里","casts":[{"alt":"https://movie.douban.com/celebrity/1386018/","avatars":{"small":"https://img1.doubanio.com/f/movie/ca527386eb8c4e325611e22dfcb04cc116d6b423/pics/movie/celebrity-default-small.png","large":"https://img3.doubanio.com/f/movie/63acc16ca6309ef191f0378faf793d1096a3e606/pics/movie/celebrity-default-large.png","medium":"https://img1.doubanio.com/f/movie/8dd0c794499fe925ae2ae89ee30cd225750457b4/pics/movie/celebrity-default-medium.png"},"name":"林佑俽","id":"1386018"},{"alt":"https://movie.douban.com/celebrity/1386019/","avatars":{"small":"https://img1.doubanio.com/f/movie/ca527386eb8c4e325611e22dfcb04cc116d6b423/pics/movie/celebrity-default-small.png","large":"https://img3.doubanio.com/f/movie/63acc16ca6309ef191f0378faf793d1096a3e606/pics/movie/celebrity-default-large.png","medium":"https://img1.doubanio.com/f/movie/8dd0c794499fe925ae2ae89ee30cd225750457b4/pics/movie/celebrity-default-medium.png"},"name":"郭馨雅","id":"1386019"},{"alt":"https://movie.douban.com/celebrity/1275593/","avatars":{"small":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p35136.webp","large":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p35136.webp","medium":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p35136.webp"},"name":"屈中恒","id":"1275593"}],"collect_count":84,"original_title":"小貓巴克里","subtype":"movie","directors":[{"alt":"https://movie.douban.com/celebrity/1372804/","avatars":{"small":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1512959486.04.webp","large":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1512959486.04.webp","medium":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1512959486.04.webp"},"name":"邱立伟","id":"1372804"}],"year":"2017","images":{"small":"https://img3.doubanio.com/view/photo/s_ratio_poster/public/p2508948724.webp","large":"https://img3.doubanio.com/view/photo/s_ratio_poster/public/p2508948724.webp","medium":"https://img3.doubanio.com/view/photo/s_ratio_poster/public/p2508948724.webp"},"alt":"https://movie.douban.com/subject/26887161/","id":"26887161"},{"rating":{"max":10,"average":7.8,"stars":"40","min":0},"genres":["剧情","历史","战争"],"title":"芳华","casts":[{"alt":"https://movie.douban.com/celebrity/1276105/","avatars":{"small":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1403053084.22.webp","large":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1403053084.22.webp","medium":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1403053084.22.webp"},"name":"黄轩","id":"1276105"},{"alt":"https://movie.douban.com/celebrity/1366978/","avatars":{"small":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1512871367.97.webp","large":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1512871367.97.webp","medium":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1512871367.97.webp"},"name":"苗苗","id":"1366978"},{"alt":"https://movie.douban.com/celebrity/1357009/","avatars":{"small":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1504775118.88.webp","large":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1504775118.88.webp","medium":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1504775118.88.webp"},"name":"钟楚曦","id":"1357009"}],"collect_count":226595,"original_title":"芳华","subtype":"movie","directors":[{"alt":"https://movie.douban.com/celebrity/1274255/","avatars":{"small":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p45667.webp","large":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p45667.webp","medium":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p45667.webp"},"name":"冯小刚","id":"1274255"}],"year":"2017","images":{"small":"https://img3.doubanio.com/view/photo/s_ratio_poster/public/p2507227732.webp","large":"https://img3.doubanio.com/view/photo/s_ratio_poster/public/p2507227732.webp","medium":"https://img3.doubanio.com/view/photo/s_ratio_poster/public/p2507227732.webp"},"alt":"https://movie.douban.com/subject/26862829/","id":"26862829"},{"rating":{"max":10,"average":6.2,"stars":"35","min":0},"genres":["喜剧","爱情","奇幻"],"title":"二代妖精之今生有幸","casts":[{"alt":"https://movie.douban.com/celebrity/1275721/","avatars":{"small":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p36925.webp","large":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p36925.webp","medium":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p36925.webp"},"name":"冯绍峰","id":"1275721"},{"alt":"https://movie.douban.com/celebrity/1049732/","avatars":{"small":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1513067217.13.webp","large":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1513067217.13.webp","medium":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1513067217.13.webp"},"name":"刘亦菲","id":"1049732"},{"alt":"https://movie.douban.com/celebrity/1275178/","avatars":{"small":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p37722.webp","large":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p37722.webp","medium":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p37722.webp"},"name":"李光洁","id":"1275178"}],"collect_count":10818,"original_title":"二代妖精之今生有幸","subtype":"movie","directors":[{"alt":"https://movie.douban.com/celebrity/1331182/","avatars":{"small":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1426816047.48.webp","large":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1426816047.48.webp","medium":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1426816047.48.webp"},"name":"肖洋","id":"1331182"}],"year":"2017","images":{"small":"https://img1.doubanio.com/view/photo/s_ratio_poster/public/p2507022339.webp","large":"https://img1.doubanio.com/view/photo/s_ratio_poster/public/p2507022339.webp","medium":"https://img1.doubanio.com/view/photo/s_ratio_poster/public/p2507022339.webp"},"alt":"https://movie.douban.com/subject/26797419/","id":"26797419"},{"rating":{"max":10,"average":6.1,"stars":"30","min":0},"genres":["喜剧","爱情"],"title":"前任3：再见前任","casts":[{"alt":"https://movie.douban.com/celebrity/1275667/","avatars":{"small":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p14025.webp","large":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p14025.webp","medium":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p14025.webp"},"name":"韩庚","id":"1275667"},{"alt":"https://movie.douban.com/celebrity/1275564/","avatars":{"small":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1366015827.84.webp","large":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1366015827.84.webp","medium":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1366015827.84.webp"},"name":"郑恺","id":"1275564"},{"alt":"https://movie.douban.com/celebrity/1342252/","avatars":{"small":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1408089064.98.webp","large":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1408089064.98.webp","medium":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1408089064.98.webp"},"name":"于文文","id":"1342252"}],"collect_count":5464,"original_title":"前任3：再见前任","subtype":"movie","directors":[{"alt":"https://movie.douban.com/celebrity/1332171/","avatars":{"small":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1391439365.41.webp","large":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1391439365.41.webp","medium":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1391439365.41.webp"},"name":"田羽生","id":"1332171"}],"year":"2017","images":{"small":"https://img1.doubanio.com/view/photo/s_ratio_poster/public/p2508926717.webp","large":"https://img1.doubanio.com/view/photo/s_ratio_poster/public/p2508926717.webp","medium":"https://img1.doubanio.com/view/photo/s_ratio_poster/public/p2508926717.webp"},"alt":"https://movie.douban.com/subject/26662193/","id":"26662193"},{"rating":{"max":10,"average":5.4,"stars":"30","min":0},"genres":["剧情","奇幻"],"title":"解忧杂货店","casts":[{"alt":"https://movie.douban.com/celebrity/1339594/","avatars":{"small":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1503377320.23.webp","large":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1503377320.23.webp","medium":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1503377320.23.webp"},"name":"王俊凯","id":"1339594"},{"alt":"https://movie.douban.com/celebrity/1325127/","avatars":{"small":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1492095415.48.webp","large":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1492095415.48.webp","medium":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1492095415.48.webp"},"name":"迪丽热巴","id":"1325127"},{"alt":"https://movie.douban.com/celebrity/1330472/","avatars":{"small":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1385100761.5.webp","large":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1385100761.5.webp","medium":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1385100761.5.webp"},"name":"董子健","id":"1330472"}],"collect_count":17076,"original_title":"解忧杂货店","subtype":"movie","directors":[{"alt":"https://movie.douban.com/celebrity/1316056/","avatars":{"small":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p34888.webp","large":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p34888.webp","medium":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p34888.webp"},"name":"韩杰","id":"1316056"}],"year":"2017","images":{"small":"https://img3.doubanio.com/view/photo/s_ratio_poster/public/p2503544593.webp","large":"https://img3.doubanio.com/view/photo/s_ratio_poster/public/p2503544593.webp","medium":"https://img3.doubanio.com/view/photo/s_ratio_poster/public/p2503544593.webp"},"alt":"https://movie.douban.com/subject/26654146/","id":"26654146"},{"rating":{"max":10,"average":6.9,"stars":"35","min":0},"genres":["剧情","悬疑","奇幻"],"title":"妖猫传","casts":[{"alt":"https://movie.douban.com/celebrity/1276105/","avatars":{"small":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1403053084.22.webp","large":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1403053084.22.webp","medium":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1403053084.22.webp"},"name":"黄轩","id":"1276105"},{"alt":"https://movie.douban.com/celebrity/1315737/","avatars":{"small":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p50940.webp","large":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p50940.webp","medium":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p50940.webp"},"name":"染谷将太","id":"1315737"},{"alt":"https://movie.douban.com/celebrity/1274494/","avatars":{"small":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1510497293.38.webp","large":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1510497293.38.webp","medium":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1510497293.38.webp"},"name":"张雨绮","id":"1274494"}],"collect_count":147235,"original_title":"妖猫传","subtype":"movie","directors":[{"alt":"https://movie.douban.com/celebrity/1023040/","avatars":{"small":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1451727734.81.webp","large":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1451727734.81.webp","medium":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1451727734.81.webp"},"name":"陈凯歌","id":"1023040"}],"year":"2017","images":{"small":"https://img1.doubanio.com/view/photo/s_ratio_poster/public/p2507024497.webp","large":"https://img1.doubanio.com/view/photo/s_ratio_poster/public/p2507024497.webp","medium":"https://img1.doubanio.com/view/photo/s_ratio_poster/public/p2507024497.webp"},"alt":"https://movie.douban.com/subject/5350027/","id":"5350027"},{"rating":{"max":10,"average":9.1,"stars":"45","min":0},"genres":["喜剧","动画","冒险"],"title":"寻梦环游记","casts":[{"alt":"https://movie.douban.com/celebrity/1370411/","avatars":{"small":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1489594517.9.webp","large":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1489594517.9.webp","medium":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1489594517.9.webp"},"name":"安东尼·冈萨雷斯","id":"1370411"},{"alt":"https://movie.douban.com/celebrity/1041009/","avatars":{"small":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1510634028.69.webp","large":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1510634028.69.webp","medium":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1510634028.69.webp"},"name":"盖尔·加西亚·贝纳尔","id":"1041009"},{"alt":"https://movie.douban.com/celebrity/1036383/","avatars":{"small":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1416762292.89.webp","large":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1416762292.89.webp","medium":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1416762292.89.webp"},"name":"本杰明·布拉特","id":"1036383"}],"collect_count":330788,"original_title":"Coco","subtype":"movie","directors":[{"alt":"https://movie.douban.com/celebrity/1022678/","avatars":{"small":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p13830.webp","large":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p13830.webp","medium":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p13830.webp"},"name":"李·昂克里奇","id":"1022678"},{"alt":"https://movie.douban.com/celebrity/1370425/","avatars":{"small":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1497195148.21.webp","large":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1497195148.21.webp","medium":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1497195148.21.webp"},"name":"阿德里安·莫利纳","id":"1370425"}],"year":"2017","images":{"small":"https://img1.doubanio.com/view/photo/s_ratio_poster/public/p2503997609.webp","large":"https://img1.doubanio.com/view/photo/s_ratio_poster/public/p2503997609.webp","medium":"https://img1.doubanio.com/view/photo/s_ratio_poster/public/p2503997609.webp"},"alt":"https://movie.douban.com/subject/20495023/","id":"20495023"},{"rating":{"max":10,"average":0,"stars":"00","min":0},"genres":["儿童","动画","奇幻"],"title":"咕噜咕噜美人鱼2","casts":[{"alt":"https://movie.douban.com/celebrity/1386282/","avatars":{"small":"https://img1.doubanio.com/f/movie/ca527386eb8c4e325611e22dfcb04cc116d6b423/pics/movie/celebrity-default-small.png","large":"https://img3.doubanio.com/f/movie/63acc16ca6309ef191f0378faf793d1096a3e606/pics/movie/celebrity-default-large.png","medium":"https://img1.doubanio.com/f/movie/8dd0c794499fe925ae2ae89ee30cd225750457b4/pics/movie/celebrity-default-medium.png"},"name":"丁莹莹","id":"1386282"},{"alt":"https://movie.douban.com/celebrity/1356706/","avatars":{"small":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1481650266.82.webp","large":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1481650266.82.webp","medium":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1481650266.82.webp"},"name":"杨进","id":"1356706"},{"alt":"https://movie.douban.com/celebrity/1348773/","avatars":{"small":"https://img1.doubanio.com/f/movie/ca527386eb8c4e325611e22dfcb04cc116d6b423/pics/movie/celebrity-default-small.png","large":"https://img3.doubanio.com/f/movie/63acc16ca6309ef191f0378faf793d1096a3e606/pics/movie/celebrity-default-large.png","medium":"https://img1.doubanio.com/f/movie/8dd0c794499fe925ae2ae89ee30cd225750457b4/pics/movie/celebrity-default-medium.png"},"name":"卢瑶","id":"1348773"}],"collect_count":52,"original_title":"咕噜咕噜美人鱼2","subtype":"movie","directors":[{"alt":"https://movie.douban.com/celebrity/1340240/","avatars":{"small":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1483596470.52.webp","large":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1483596470.52.webp","medium":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1483596470.52.webp"},"name":"杨广福","id":"1340240"}],"year":"2017","images":{"small":"https://img3.doubanio.com/view/photo/s_ratio_poster/public/p2508071436.webp","large":"https://img3.doubanio.com/view/photo/s_ratio_poster/public/p2508071436.webp","medium":"https://img3.doubanio.com/view/photo/s_ratio_poster/public/p2508071436.webp"},"alt":"https://movie.douban.com/subject/27193475/","id":"27193475"},{"rating":{"max":10,"average":8.3,"stars":"45","min":0},"genres":["喜剧","动画","家庭"],"title":"帕丁顿熊2","casts":[{"alt":"https://movie.douban.com/celebrity/1025149/","avatars":{"small":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1397997449.99.webp","large":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1397997449.99.webp","medium":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1397997449.99.webp"},"name":"本·卫肖","id":"1025149"},{"alt":"https://movie.douban.com/celebrity/1003493/","avatars":{"small":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1871.webp","large":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1871.webp","medium":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1871.webp"},"name":"休·格兰特","id":"1003493"},{"alt":"https://movie.douban.com/celebrity/1041224/","avatars":{"small":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1416053013.86.webp","large":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1416053013.86.webp","medium":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1416053013.86.webp"},"name":"休·博内威利","id":"1041224"}],"collect_count":54486,"original_title":"Paddington 2","subtype":"movie","directors":[{"alt":"https://movie.douban.com/celebrity/1313689/","avatars":{"small":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1425285993.52.webp","large":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1425285993.52.webp","medium":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1425285993.52.webp"},"name":"保罗·金","id":"1313689"}],"year":"2017","images":{"small":"https://img1.doubanio.com/view/photo/s_ratio_poster/public/p2506466229.webp","large":"https://img1.doubanio.com/view/photo/s_ratio_poster/public/p2506466229.webp","medium":"https://img1.doubanio.com/view/photo/s_ratio_poster/public/p2506466229.webp"},"alt":"https://movie.douban.com/subject/26340419/","id":"26340419"},{"rating":{"max":10,"average":6.2,"stars":"30","min":0},"genres":["动作","犯罪","悬疑"],"title":"心理罪之城市之光","casts":[{"alt":"https://movie.douban.com/celebrity/1274235/","avatars":{"small":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p805.webp","large":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p805.webp","medium":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p805.webp"},"name":"邓超","id":"1274235"},{"alt":"https://movie.douban.com/celebrity/1259866/","avatars":{"small":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p21006.webp","large":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p21006.webp","medium":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p21006.webp"},"name":"阮经天","id":"1259866"},{"alt":"https://movie.douban.com/celebrity/1274533/","avatars":{"small":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p35797.webp","large":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p35797.webp","medium":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p35797.webp"},"name":"刘诗诗","id":"1274533"}],"collect_count":30498,"original_title":"心理罪之城市之光","subtype":"movie","directors":[{"alt":"https://movie.douban.com/celebrity/1317195/","avatars":{"small":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p38956.webp","large":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p38956.webp","medium":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p38956.webp"},"name":"徐纪周","id":"1317195"}],"year":"2017","images":{"small":"https://img3.doubanio.com/view/photo/s_ratio_poster/public/p2506228506.webp","large":"https://img3.doubanio.com/view/photo/s_ratio_poster/public/p2506228506.webp","medium":"https://img3.doubanio.com/view/photo/s_ratio_poster/public/p2506228506.webp"},"alt":"https://movie.douban.com/subject/26774722/","id":"26774722"},{"rating":{"max":10,"average":5,"stars":"25","min":0},"genres":["剧情","动作","科幻"],"title":"机器之血","casts":[{"alt":"https://movie.douban.com/celebrity/1054531/","avatars":{"small":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p694.webp","large":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p694.webp","medium":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p694.webp"},"name":"成龙","id":"1054531"},{"alt":"https://movie.douban.com/celebrity/1274317/","avatars":{"small":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p3083.webp","large":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p3083.webp","medium":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p3083.webp"},"name":"罗志祥","id":"1274317"},{"alt":"https://movie.douban.com/celebrity/1336314/","avatars":{"small":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1499104651.08.webp","large":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1499104651.08.webp","medium":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1499104651.08.webp"},"name":"欧阳娜娜","id":"1336314"}],"collect_count":10002,"original_title":"机器之血","subtype":"movie","directors":[{"alt":"https://movie.douban.com/celebrity/1324053/","avatars":{"small":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1495169894.72.webp","large":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1495169894.72.webp","medium":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1495169894.72.webp"},"name":"张立嘉","id":"1324053"}],"year":"2017","images":{"small":"https://img1.doubanio.com/view/photo/s_ratio_poster/public/p2505785547.webp","large":"https://img1.doubanio.com/view/photo/s_ratio_poster/public/p2505785547.webp","medium":"https://img1.doubanio.com/view/photo/s_ratio_poster/public/p2505785547.webp"},"alt":"https://movie.douban.com/subject/26729868/","id":"26729868"},{"rating":{"max":10,"average":8.6,"stars":"45","min":0},"genres":["剧情","传记","动画"],"title":"至爱梵高·星空之谜","casts":[{"alt":"https://movie.douban.com/celebrity/1314461/","avatars":{"small":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p43221.webp","large":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p43221.webp","medium":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p43221.webp"},"name":"道格拉斯·布斯","id":"1314461"},{"alt":"https://movie.douban.com/celebrity/1376200/","avatars":{"small":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1498554583.31.webp","large":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1498554583.31.webp","medium":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1498554583.31.webp"},"name":"罗伯特·古拉奇克","id":"1376200"},{"alt":"https://movie.douban.com/celebrity/1275043/","avatars":{"small":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1360941730.7.webp","large":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1360941730.7.webp","medium":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1360941730.7.webp"},"name":"埃莉诺·汤姆林森","id":"1275043"}],"collect_count":81929,"original_title":"Loving Vincent","subtype":"movie","directors":[{"alt":"https://movie.douban.com/celebrity/1326282/","avatars":{"small":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1393488180.49.webp","large":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1393488180.49.webp","medium":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1393488180.49.webp"},"name":"多洛塔·科别拉","id":"1326282"},{"alt":"https://movie.douban.com/celebrity/1306202/","avatars":{"small":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1498554460.64.webp","large":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1498554460.64.webp","medium":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1498554460.64.webp"},"name":"休·韦尔什曼","id":"1306202"}],"year":"2017","images":{"small":"https://img1.doubanio.com/view/photo/s_ratio_poster/public/p2506935748.webp","large":"https://img1.doubanio.com/view/photo/s_ratio_poster/public/p2506935748.webp","medium":"https://img1.doubanio.com/view/photo/s_ratio_poster/public/p2506935748.webp"},"alt":"https://movie.douban.com/subject/25837262/","id":"25837262"},{"rating":{"max":10,"average":4.8,"stars":"25","min":0},"genres":["动作","奇幻"],"title":"奇门遁甲","casts":[{"alt":"https://movie.douban.com/celebrity/1324043/","avatars":{"small":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1490342249.11.webp","large":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1490342249.11.webp","medium":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1490342249.11.webp"},"name":"大鹏","id":"1324043"},{"alt":"https://movie.douban.com/celebrity/1315861/","avatars":{"small":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1368598869.24.webp","large":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1368598869.24.webp","medium":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1368598869.24.webp"},"name":"倪妮","id":"1315861"},{"alt":"https://movie.douban.com/celebrity/1274292/","avatars":{"small":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p10299.webp","large":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p10299.webp","medium":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p10299.webp"},"name":"李治廷","id":"1274292"}],"collect_count":22624,"original_title":"奇门遁甲","subtype":"movie","directors":[{"alt":"https://movie.douban.com/celebrity/1275026/","avatars":{"small":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p9332.webp","large":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p9332.webp","medium":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p9332.webp"},"name":"袁和平","id":"1275026"}],"year":"2017","images":{"small":"https://img3.doubanio.com/view/photo/s_ratio_poster/public/p2507566212.webp","large":"https://img3.doubanio.com/view/photo/s_ratio_poster/public/p2507566212.webp","medium":"https://img3.doubanio.com/view/photo/s_ratio_poster/public/p2507566212.webp"},"alt":"https://movie.douban.com/subject/26661191/","id":"26661191"},{"rating":{"max":10,"average":8.6,"stars":"45","min":0},"genres":["剧情","传记","历史"],"title":"至暗时刻","casts":[{"alt":"https://movie.douban.com/celebrity/1010507/","avatars":{"small":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p33896.webp","large":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p33896.webp","medium":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p33896.webp"},"name":"加里·奥德曼","id":"1010507"},{"alt":"https://movie.douban.com/celebrity/1021997/","avatars":{"small":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1512573236.06.webp","large":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1512573236.06.webp","medium":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1512573236.06.webp"},"name":"克里斯汀·斯科特·托马斯","id":"1021997"},{"alt":"https://movie.douban.com/celebrity/1318674/","avatars":{"small":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1426508419.1.webp","large":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1426508419.1.webp","medium":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1426508419.1.webp"},"name":"莉莉·詹姆斯","id":"1318674"}],"collect_count":50108,"original_title":"Darkest Hour","subtype":"movie","directors":[{"alt":"https://movie.douban.com/celebrity/1275041/","avatars":{"small":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1424001982.32.webp","large":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1424001982.32.webp","medium":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1424001982.32.webp"},"name":"乔·赖特","id":"1275041"}],"year":"2017","images":{"small":"https://img3.doubanio.com/view/photo/s_ratio_poster/public/p2504277551.webp","large":"https://img3.doubanio.com/view/photo/s_ratio_poster/public/p2504277551.webp","medium":"https://img3.doubanio.com/view/photo/s_ratio_poster/public/p2504277551.webp"},"alt":"https://movie.douban.com/subject/26761416/","id":"26761416"},{"rating":{"max":10,"average":5.5,"stars":"30","min":0},"genres":["剧情"],"title":"咖啡风暴","casts":[{"alt":"https://movie.douban.com/celebrity/1083010/","avatars":{"small":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p21789.webp","large":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p21789.webp","medium":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p21789.webp"},"name":"恩尼奥·凡塔斯蒂基尼","id":"1083010"},{"alt":"https://movie.douban.com/celebrity/1315585/","avatars":{"small":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1380440677.6.webp","large":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1380440677.6.webp","medium":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1380440677.6.webp"},"name":"芦芳生","id":"1315585"},{"alt":"https://movie.douban.com/celebrity/1312976/","avatars":{"small":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p38291.webp","large":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p38291.webp","medium":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p38291.webp"},"name":"谭卓","id":"1312976"}],"collect_count":91,"original_title":"Caffè","subtype":"movie","directors":[{"alt":"https://movie.douban.com/celebrity/1010292/","avatars":{"small":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p24213.webp","large":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p24213.webp","medium":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p24213.webp"},"name":"克里斯蒂亚诺·博尔托内","id":"1010292"}],"year":"2016","images":{"small":"https://img1.doubanio.com/view/photo/s_ratio_poster/public/p2508258787.webp","large":"https://img1.doubanio.com/view/photo/s_ratio_poster/public/p2508258787.webp","medium":"https://img1.doubanio.com/view/photo/s_ratio_poster/public/p2508258787.webp"},"alt":"https://movie.douban.com/subject/26649225/","id":"26649225"},{"rating":{"max":10,"average":6.8,"stars":"35","min":0},"genres":["喜剧","家庭"],"title":"圣诞奇妙公司","casts":[{"alt":"https://movie.douban.com/celebrity/1009332/","avatars":{"small":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p39998.webp","large":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p39998.webp","medium":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p39998.webp"},"name":"格什菲·法拉哈尼","id":"1009332"},{"alt":"https://movie.douban.com/celebrity/1004860/","avatars":{"small":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1973.webp","large":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1973.webp","medium":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1973.webp"},"name":"奥黛丽·塔图","id":"1004860"},{"alt":"https://movie.douban.com/celebrity/1036749/","avatars":{"small":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p4917.webp","large":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p4917.webp","medium":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p4917.webp"},"name":"阿兰·夏巴","id":"1036749"}],"collect_count":1654,"original_title":"Santa & Cie","subtype":"movie","directors":[{"alt":"https://movie.douban.com/celebrity/1036749/","avatars":{"small":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p4917.webp","large":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p4917.webp","medium":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p4917.webp"},"name":"阿兰·夏巴","id":"1036749"}],"year":"2017","images":{"small":"https://img3.doubanio.com/view/photo/s_ratio_poster/public/p2505778884.webp","large":"https://img3.doubanio.com/view/photo/s_ratio_poster/public/p2505778884.webp","medium":"https://img3.doubanio.com/view/photo/s_ratio_poster/public/p2505778884.webp"},"alt":"https://movie.douban.com/subject/27011205/","id":"27011205"},{"rating":{"max":10,"average":4.8,"stars":"25","min":0},"genres":["喜剧","犯罪"],"title":"这就是命","casts":[{"alt":"https://movie.douban.com/celebrity/1317139/","avatars":{"small":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1371453539.51.webp","large":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1371453539.51.webp","medium":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1371453539.51.webp"},"name":"王迅","id":"1317139"},{"alt":"https://movie.douban.com/celebrity/1002862/","avatars":{"small":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1161.webp","large":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1161.webp","medium":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1161.webp"},"name":"曾志伟","id":"1002862"},{"alt":"https://movie.douban.com/celebrity/1340702/","avatars":{"small":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1402633807.18.webp","large":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1402633807.18.webp","medium":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1402633807.18.webp"},"name":"梁超","id":"1340702"}],"collect_count":1432,"original_title":"这就是命","subtype":"movie","directors":[{"alt":"https://movie.douban.com/celebrity/1335838/","avatars":{"small":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1507906324.74.webp","large":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1507906324.74.webp","medium":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1507906324.74.webp"},"name":"王丹","id":"1335838"}],"year":"2017","images":{"small":"https://img1.doubanio.com/view/photo/s_ratio_poster/public/p2506823369.webp","large":"https://img1.doubanio.com/view/photo/s_ratio_poster/public/p2506823369.webp","medium":"https://img1.doubanio.com/view/photo/s_ratio_poster/public/p2506823369.webp"},"alt":"https://movie.douban.com/subject/27037539/","id":"27037539"},{"rating":{"max":10,"average":6.6,"stars":"35","min":0},"genres":["动作","科幻","奇幻"],"title":"正义联盟","casts":[{"alt":"https://movie.douban.com/celebrity/1054417/","avatars":{"small":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p7622.webp","large":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p7622.webp","medium":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p7622.webp"},"name":"本·阿弗莱克","id":"1054417"},{"alt":"https://movie.douban.com/celebrity/1044713/","avatars":{"small":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1371934661.95.webp","large":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1371934661.95.webp","medium":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1371934661.95.webp"},"name":"亨利·卡维尔","id":"1044713"},{"alt":"https://movie.douban.com/celebrity/1044996/","avatars":{"small":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1467908677.92.webp","large":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1467908677.92.webp","medium":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1467908677.92.webp"},"name":"盖尔·加朵","id":"1044996"}],"collect_count":121703,"original_title":"Justice League","subtype":"movie","directors":[{"alt":"https://movie.douban.com/celebrity/1031904/","avatars":{"small":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p23346.webp","large":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p23346.webp","medium":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p23346.webp"},"name":"扎克·施奈德","id":"1031904"}],"year":"2017","images":{"small":"https://img3.doubanio.com/view/photo/s_ratio_poster/public/p2504027804.webp","large":"https://img3.doubanio.com/view/photo/s_ratio_poster/public/p2504027804.webp","medium":"https://img3.doubanio.com/view/photo/s_ratio_poster/public/p2504027804.webp"},"alt":"https://movie.douban.com/subject/2158490/","id":"2158490"},{"rating":{"max":10,"average":6.2,"stars":"30","min":0},"genres":["惊悚","冒险"],"title":"鲨海","casts":[{"alt":"https://movie.douban.com/celebrity/1000013/","avatars":{"small":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p23017.webp","large":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p23017.webp","medium":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p23017.webp"},"name":"曼迪·摩尔","id":"1000013"},{"alt":"https://movie.douban.com/celebrity/1232340/","avatars":{"small":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p46657.webp","large":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p46657.webp","medium":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p46657.webp"},"name":"克莱尔·霍尔特","id":"1232340"},{"alt":"https://movie.douban.com/celebrity/1050080/","avatars":{"small":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p11804.webp","large":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p11804.webp","medium":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p11804.webp"},"name":"克里斯·J.约翰逊","id":"1050080"}],"collect_count":11033,"original_title":"47 Meters Down","subtype":"movie","directors":[{"alt":"https://movie.douban.com/celebrity/1019388/","avatars":{"small":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1498455992.67.webp","large":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1498455992.67.webp","medium":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1498455992.67.webp"},"name":"约翰内斯·罗伯茨","id":"1019388"}],"year":"2016","images":{"small":"https://img3.doubanio.com/view/photo/s_ratio_poster/public/p2504892832.webp","large":"https://img3.doubanio.com/view/photo/s_ratio_poster/public/p2504892832.webp","medium":"https://img3.doubanio.com/view/photo/s_ratio_poster/public/p2504892832.webp"},"alt":"https://movie.douban.com/subject/26845362/","id":"26845362"}]
     * title : 正在上映的电影-北京
     */

    private int count;
    private int start;
    private int total;
    private String title;
    private List<SubjectsBean> subjects;

    public int getCount() {
        return count;
    }

    public void setCount(int count) {
        this.count = count;
    }

    public int getStart() {
        return start;
    }

    public void setStart(int start) {
        this.start = start;
    }

    public int getTotal() {
        return total;
    }

    public void setTotal(int total) {
        this.total = total;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public List<SubjectsBean> getSubjects() {
        return subjects;
    }

    public void setSubjects(List<SubjectsBean> subjects) {
        this.subjects = subjects;
    }

    public static class SubjectsBean {
        /**
         * rating : {"max":10,"average":4.9,"stars":"25","min":0}
         * genres : ["喜剧"]
         * title : 妖铃铃
         * casts : [{"alt":"https://movie.douban.com/celebrity/1127819/","avatars":{"small":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p45539.webp","large":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p45539.webp","medium":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p45539.webp"},"name":"吴君如","id":"1127819"},{"alt":"https://movie.douban.com/celebrity/1325700/","avatars":{"small":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1356510694.28.webp","large":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1356510694.28.webp","medium":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p1356510694.28.webp"},"name":"沈腾","id":"1325700"},{"alt":"https://movie.douban.com/celebrity/1317663/","avatars":{"small":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p40756.webp","large":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p40756.webp","medium":"https://img3.doubanio.com/view/celebrity/s_ratio_celebrity/public/p40756.webp"},"name":"岳云鹏","id":"1317663"}]
         * collect_count : 10595
         * original_title : 妖铃铃
         * subtype : movie
         * directors : [{"alt":"https://movie.douban.com/celebrity/1127819/","avatars":{"small":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p45539.webp","large":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p45539.webp","medium":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p45539.webp"},"name":"吴君如","id":"1127819"}]
         * year : 2017
         * images : {"small":"https://img3.doubanio.com/view/photo/s_ratio_poster/public/p2506825662.webp","large":"https://img3.doubanio.com/view/photo/s_ratio_poster/public/p2506825662.webp","medium":"https://img3.doubanio.com/view/photo/s_ratio_poster/public/p2506825662.webp"}
         * alt : https://movie.douban.com/subject/26966580/
         * id : 26966580
         */

        private RatingBean rating;
        private String title;
        private int collect_count;
        private String original_title;
        private String subtype;
        private String year;
        private ImagesBean images;
        private String alt;
        private String id;
        private List<String> genres;
        private List<CastsBean> casts;
        private List<DirectorsBean> directors;

        public RatingBean getRating() {
            return rating;
        }

        public void setRating(RatingBean rating) {
            this.rating = rating;
        }

        public String getTitle() {
            return title;
        }

        public void setTitle(String title) {
            this.title = title;
        }

        public int getCollect_count() {
            return collect_count;
        }

        public void setCollect_count(int collect_count) {
            this.collect_count = collect_count;
        }

        public String getOriginal_title() {
            return original_title;
        }

        public void setOriginal_title(String original_title) {
            this.original_title = original_title;
        }

        public String getSubtype() {
            return subtype;
        }

        public void setSubtype(String subtype) {
            this.subtype = subtype;
        }

        public String getYear() {
            return year;
        }

        public void setYear(String year) {
            this.year = year;
        }

        public ImagesBean getImages() {
            return images;
        }

        public void setImages(ImagesBean images) {
            this.images = images;
        }

        public String getAlt() {
            return alt;
        }

        public void setAlt(String alt) {
            this.alt = alt;
        }

        public String getId() {
            return id;
        }

        public void setId(String id) {
            this.id = id;
        }

        public List<String> getGenres() {
            return genres;
        }

        public void setGenres(List<String> genres) {
            this.genres = genres;
        }

        public List<CastsBean> getCasts() {
            return casts;
        }

        public void setCasts(List<CastsBean> casts) {
            this.casts = casts;
        }

        public List<DirectorsBean> getDirectors() {
            return directors;
        }

        public void setDirectors(List<DirectorsBean> directors) {
            this.directors = directors;
        }

        public static class RatingBean {
            /**
             * max : 10
             * average : 4.9
             * stars : 25
             * min : 0
             */

            private int max;
            private double average;
            private String stars;
            private int min;

            public int getMax() {
                return max;
            }

            public void setMax(int max) {
                this.max = max;
            }

            public double getAverage() {
                return average;
            }

            public void setAverage(double average) {
                this.average = average;
            }

            public String getStars() {
                return stars;
            }

            public void setStars(String stars) {
                this.stars = stars;
            }

            public int getMin() {
                return min;
            }

            public void setMin(int min) {
                this.min = min;
            }
        }

        public static class ImagesBean {
            /**
             * small : https://img3.doubanio.com/view/photo/s_ratio_poster/public/p2506825662.webp
             * large : https://img3.doubanio.com/view/photo/s_ratio_poster/public/p2506825662.webp
             * medium : https://img3.doubanio.com/view/photo/s_ratio_poster/public/p2506825662.webp
             */

            private String small;
            private String large;
            private String medium;

            public String getSmall() {
                return small;
            }

            public void setSmall(String small) {
                this.small = small;
            }

            public String getLarge() {
                return large;
            }

            public void setLarge(String large) {
                this.large = large;
            }

            public String getMedium() {
                return medium;
            }

            public void setMedium(String medium) {
                this.medium = medium;
            }
        }

        public static class CastsBean {
            /**
             * alt : https://movie.douban.com/celebrity/1127819/
             * avatars : {"small":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p45539.webp","large":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p45539.webp","medium":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p45539.webp"}
             * name : 吴君如
             * id : 1127819
             */

            private String alt;
            private AvatarsBean avatars;
            private String name;
            private String id;

            public String getAlt() {
                return alt;
            }

            public void setAlt(String alt) {
                this.alt = alt;
            }

            public AvatarsBean getAvatars() {
                return avatars;
            }

            public void setAvatars(AvatarsBean avatars) {
                this.avatars = avatars;
            }

            public String getName() {
                return name;
            }

            public void setName(String name) {
                this.name = name;
            }

            public String getId() {
                return id;
            }

            public void setId(String id) {
                this.id = id;
            }

            public static class AvatarsBean {
                /**
                 * small : https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p45539.webp
                 * large : https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p45539.webp
                 * medium : https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p45539.webp
                 */

                private String small;
                private String large;
                private String medium;

                public String getSmall() {
                    return small;
                }

                public void setSmall(String small) {
                    this.small = small;
                }

                public String getLarge() {
                    return large;
                }

                public void setLarge(String large) {
                    this.large = large;
                }

                public String getMedium() {
                    return medium;
                }

                public void setMedium(String medium) {
                    this.medium = medium;
                }
            }
        }

        public static class DirectorsBean {
            /**
             * alt : https://movie.douban.com/celebrity/1127819/
             * avatars : {"small":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p45539.webp","large":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p45539.webp","medium":"https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p45539.webp"}
             * name : 吴君如
             * id : 1127819
             */

            private String alt;
            private AvatarsBeanX avatars;
            private String name;
            private String id;

            public String getAlt() {
                return alt;
            }

            public void setAlt(String alt) {
                this.alt = alt;
            }

            public AvatarsBeanX getAvatars() {
                return avatars;
            }

            public void setAvatars(AvatarsBeanX avatars) {
                this.avatars = avatars;
            }

            public String getName() {
                return name;
            }

            public void setName(String name) {
                this.name = name;
            }

            public String getId() {
                return id;
            }

            public void setId(String id) {
                this.id = id;
            }

            public static class AvatarsBeanX {
                /**
                 * small : https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p45539.webp
                 * large : https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p45539.webp
                 * medium : https://img1.doubanio.com/view/celebrity/s_ratio_celebrity/public/p45539.webp
                 */

                private String small;
                private String large;
                private String medium;

                public String getSmall() {
                    return small;
                }

                public void setSmall(String small) {
                    this.small = small;
                }

                public String getLarge() {
                    return large;
                }

                public void setLarge(String large) {
                    this.large = large;
                }

                public String getMedium() {
                    return medium;
                }

                public void setMedium(String medium) {
                    this.medium = medium;
                }
            }
        }
    }
}
```
使用的是豆瓣电影API，直接用Gsonformat，内容比较多
3. 创建描述网络请求的接口
```
package com.paulniu.rxjavademo;

import io.reactivex.Observable;
import retrofit2.http.GET;

/**
 * Created by niupule on 2017/12/30.
 */

public interface GerRequest_Interface {

    @GET("v2/movie/in_theaters")
    Observable<MovieBean> getCall();
    // 注解里传入 网络请求 的部分URL地址
    // Retrofit把网络请求的URL分成了两部分：一部分放在Retrofit对象里，另一部分放在网络请求接口里
    // 如果接口里的url是一个完整的网址，那么放在Retrofit对象里的URL可以忽略
    // 采用Observable<...>接口
    // getCall()是接受网络请求数据的方法

}
```
4. 在Activity中实现
```
package com.paulniu.rxjavademo;

import android.app.Activity;
import android.os.Bundle;
import android.support.annotation.Nullable;
import android.util.Log;
import android.widget.ListView;

import com.jakewharton.retrofit2.adapter.rxjava2.RxJava2CallAdapterFactory;
import com.orhanobut.logger.AndroidLogAdapter;
import com.orhanobut.logger.Logger;


import io.reactivex.Observable;
import io.reactivex.ObservableEmitter;
import io.reactivex.ObservableOnSubscribe;
import io.reactivex.Observer;
import io.reactivex.Scheduler;
import io.reactivex.android.schedulers.AndroidSchedulers;
import io.reactivex.disposables.Disposable;
import io.reactivex.functions.Consumer;
import io.reactivex.schedulers.Schedulers;
import retrofit2.Retrofit;
import retrofit2.converter.gson.GsonConverterFactory;

/**
 * Created by niupule on 2017/12/30.
 */

public class RxJava_Douban extends Activity {

    private ListView listview;

    public static final String TAG = "RxJava_Douban";

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.rxjava_douban);
        listview = findViewById(R.id.listview);
        Logger.addLogAdapter(new AndroidLogAdapter());
        getData();
    }

    private void getData() {
        /**
         * 通过RxJava发送网络请求
         */
        Retrofit retrofit = new Retrofit.Builder()
                .baseUrl("https://api.douban.com/")
                .addConverterFactory(GsonConverterFactory.create())
                .addCallAdapterFactory(RxJava2CallAdapterFactory.create())
                .build();

        GerRequest_Interface request = retrofit.create(GerRequest_Interface.class);
        Observable<MovieBean> call = request.getCall();
        call.subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Observer<MovieBean>() {
                    @Override
                    public void onSubscribe(Disposable d) {

                    }

                    @Override
                    public void onNext(MovieBean value) {
                        //获取服务器返回数据
                        Logger.d("hello");
                        Logger.d(value.getSubjects().size());
                    }

                    @Override
                    public void onError(Throwable e) {

                    }

                    @Override
                    public void onComplete() {

                    }
                });

    }

}
```

暂时先写这么多，后面关于别的内容，以后再补充

# 参考资料
[轻易&易懂的RxJava入门](https://www.jianshu.com/p/a406b94f3188)
[RxJava基础操作符-创建操作符](https://www.jianshu.com/p/e19f8ed863b1)
[给初学者的RxJava教程](https://www.jianshu.com/p/464fa025229e)
[功能强大的Logger](https://github.com/orhanobut/logger)
[RxJava中文文档]
(https://mcxiaoke.gitbooks.io/rxdocs/content/)
