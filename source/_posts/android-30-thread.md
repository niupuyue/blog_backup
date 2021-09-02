---
title: 重拾android路(三十) Android多线程
date: 2019-07-19 15:27:11
tags:
  - android
  - thread
---

多线程的应用在Android开发中是非常常见的，常用方法主要有
<!--more-->
![常见方法](/assets/thread/thread01.png)

# 多线程基础知识

## 线程

![总纲](/assets/thread/thread02.png)

定义：一个基本CPU执行单元

作用：减少程序在并发执行时所付出的时空开销，提高操作系统的并发性能

状态：拥有类似于进程的就绪、阻塞、运行3种基本状态，具体如下图

![线程状态](/assets/thread/thread03.png)

# 基础实用

1. 继承Thread类
2. 实现Runnable接口
3. handler

## 继承Thread类

![继承Thread类的知识点](/assets/thread/thread04.png)

实现步骤
![Thread类实现步骤](/assets/thread/thread05.png)

具体实现代码如下，分为两种一种是通过默认写法，也就是直接新建一个类，然后让这个类继承Thread，重写run方法，第二种是以内部类的方式去实现

```
// 步骤1：创建线程类 （继承自Thread类）
   class MyThread extends Thread{

// 步骤2：复写run（），内容 = 定义线程行为
    @Override
    public void run(){
    ... // 定义的线程行为
    }
}

// 步骤3：创建线程对象，即 实例化线程类
  MyThread mt=new MyThread(“线程名称”);

// 步骤4：通过 线程对象 控制线程的状态，如 运行、睡眠、挂起  / 停止
// 此处采用 start（）开启线程
  mt.start();
```
匿名内部类
```
// 步骤1：采用匿名类，直接 创建 线程类的实例
 new Thread("线程名称") {
                 // 步骤2：复写run（），内容 = 定义线程行为
                    @Override
                    public void run() {       
                  // 步骤3：通过 线程对象 控制线程的状态，如 运行、睡眠、挂起  / 停止   
                      }.start();
```

区别

![两种实现方法的区别](/assets/thread/thread06.png)

## 实现runnable接口

![实现Runnable接口](/assets/thread/thread07.png)

实现步骤：
![实现Runnable的步骤](/assets/thread/thread08.png)

> Java中真正能创建新线程的只有Thread类对象,通过实现Runnable的方式，最终还是通过Thread类对象来创建线程,所以对于 实现了Runnable接口的类，称为 线程辅助类；Thread类才是真正的线程类

具体实现代码如下，也是分为两种，一种是默认写法，另一个是匿名类写法

```
// 步骤1：创建线程辅助类，实现Runnable接口
 class MyThread implements Runnable{
    ....
    @Override
// 步骤2：复写run（），定义线程行为
    public void run(){

    }
}

// 步骤3：创建线程辅助对象，即 实例化 线程辅助类
  MyThread mt=new MyThread();

// 步骤4：创建线程对象，即 实例化线程类；线程类 = Thread类；
// 创建时通过Thread类的构造函数传入线程辅助类对象
// 原因：Runnable接口并没有任何对线程的支持，我们必须创建线程类（Thread类）的实例，从Thread类的一个实例内部运行
  Thread td=new Thread(mt);

// 步骤5：通过 线程对象 控制线程的状态，如 运行、睡眠、挂起  / 停止
// 当调用start（）方法时，线程对象会自动回调线程辅助类对象的run（），从而实现线程操作
  td.start();
```
匿名类实现方法
```
// 步骤1：通过匿名类 直接 创建线程辅助对象，即 实例化 线程辅助类
    Runnable mt = new Runnable() {
                    // 步骤2：复写run（），定义线程行为
                    @Override
                    public void run() {
                    }
                };

                // 步骤3：创建线程对象，即 实例化线程类；线程类 = Thread类；
                Thread mt1 = new Thread(mt, "窗口1");
           
                // 步骤4：通过 线程对象 控制线程的状态，如 运行、睡眠、挂起  / 停止
                mt1.start();
```

两者之间的区别
![两种方式的区别](/assets/thread/thread09.png)

Thread类和Runnable的两种方式的区别
![区别](/assets/thread/thread10.png)

## handler的方式

这有一个写的很详细的，我就不再去重复了

具体请看文章：[Android Handler：这是一份 全面、详细的Handler机制 学习攻略](https://www.jianshu.com/p/9fe944ee02f7)
具体使用：[Android：这是一份Handler消息传递机制 的使用教程](https://www.jianshu.com/p/e172a2d58905)
使用问题: [（内存泄漏）：Android 内存泄露：详解 Handler 内存泄露的原因](https://www.jianshu.com/p/ed9e15eff47a)
工作原理：[Android Handler：图文解析 Handler通信机制 的工作原理](https://www.jianshu.com/p/f0b23ee5a922)
源码分析：[Android Handler：手把手带你深入分析 Handler机制源码](https://www.jianshu.com/p/b4d745c7ff7a)

# 复合使用
Android多线程实现的复合使用包括：

AsyncTask
HandlerThread
IntentService

称为”复用“的主要原因是：这3种方式的本质原理都是Android多线程基础实现（继承Thread类、实现Runnable接口、Handler）的组合实现

## AsyncTask

![AsyncTask](/assets/thread/thread11.png)

具体使用 & 实例讲解：[Android 多线程：手把手教你使用AsyncTask](https://www.jianshu.com/p/ee1342fcf5e7)

工作原理 & 源码分析：[Android 多线程：AsyncTask的原理 及其源码分析](https://www.jianshu.com/p/37502bbbb25a)

## HandlerThread

![HandlerThread](/assets/thread/thread12.png)

具体使用 & 实例讲解：[Android多线程：手把手教你使用HandlerThread](https://www.jianshu.com/p/9c10beaa1c95)

工作原理 & 源码分析：[Android多线程：这是一份详细的HandlerThread源码分析攻略](https://www.jianshu.com/p/4a8dc2f50ae6)

## IntentService
![IntentService](/assets/thread/thread13.png)

具体使用 & 实例讲解：[Android多线程：IntentService使用教程（含实例讲解）](https://www.jianshu.com/p/af62781fefba)

工作原理 & 源码分析：[Android多线程：这是一份全面 & 详细的IntentService源码分析指南](https://www.jianshu.com/p/8a3c44a9173a)

# 高级实用

Android多线程的高级使用主要是线程池（ThreadPool）。

![ThreadPool](/assets/thread/thread14.png)

![使用](https://www.jianshu.com/p/0e4a5e70bf0e)

# 对比

![对比](/assets/thread/thread15.png)

# 参考资料
[android多线程编程指南](https://juejin.im/post/5d12c1c66fb9a07ee30e2821)
[android多线程Thread类的使用](https://www.jianshu.com/p/834f336855c4)