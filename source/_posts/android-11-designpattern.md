---
title: 重拾android路(十一) 设计模式
date: 2016-07-05 23:27:46
tags:
  - 设计模式
---
设计模式在Android中的应用
<!--more-->
# 单例

## 饿汉模式
```
public class App{
    private App(){}
    private static App app = new App();
    public static App getApp(){
        return app;
    }
}
```
## 懒汉模式
```
public class App{
    private App(){}
    private static App app = null;
    public static synchronized App getApp(){
        if(app == null){
            app = new App();
        }
        return app;
    }
}
```
## 另外一种
```
public class App{
    private App(){}
    private volatile static App app;
    public static App getApp(){
        if(app ==  null){
            synchronized(App.class){
                if(app == null){
                    app = new App();
                }
            }
        }
        return app;
    }
}
```
## 静态内部类
```
public class App{
    private static class GetAppInstance{
        public static App mApp = new App();
    }
    private App(){}
    public static App getApp(){
        return GetAppInstance.mApp;
    }
}
```