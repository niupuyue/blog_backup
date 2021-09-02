---
title: android奇技淫巧 09 gradle统一配置版本
date: 2019-07-11 22:42:10
tags:
  - android
---

在项目开发中，随着项目的逐渐庞大，引用的依赖库也会随之增多，那么我们需要有一个文件来统一管理gradl的版本
<!--more-->
在Project/build.gradle中定义，在module/build.gradle中使用
最终的效果如图所示：

![gradle同一版本配置](/assets/tools/tools-gradle-01.png)

我们可以单独配置一个config.gradle的文件，在这个文件中，将我们所有需要使用到的版本进行统一管理

1. 在project层级下新建一个名为config.gradle的文件，这个文件就是我们需要配置版本的地方

2. 在配置文件中我们书写一下代码，如下所示：
```
// gradle全局配置文件

// 定义所有的project参数
ext {

    android = [
            compileSdk : 28,
            buildTools : "28.0.3",
            minSdk     : 19,
            minLimitSdk: 15,  // 限制低版本用户安装
            targetSdk  : 28,
    ]

    dependencies = [
            // app依赖库
            junit                : '4.12',
            supportLibraryVersion: '27.1.1',
            supportParentVersion : '25.3.1',
            // 自己的依赖库
            IYingLibrary         : '0.0.7',
            // 第三方依赖库
            // 检测内存泄漏
            leakcanary           : '1.5.4',
            // 高德地图
            amapLocation         : 'latest.integration',
            // Logger日志
            log                  : '2.2.0',
            // RxPermission
            rxpermission         : '0.10.2',
            // Glide
            glide                : '4.9.0',
            glidecompiler        : '4.9.0',
            // rxjava rxAndroid
            rxjava               : '2.2.7',
            rxAndroid            : '2.0.1'

    ]
}
```

3. 在project层级下的build.gradle文件中将config.gradle引入到该文件中，如下所示

```
apply from: 'config.gradle'
```

![将配置文件引入到build.gradle文件中](/assets/tools/tools-gradle-02.png)

4. 引用

在引用的时候分为两种，一种是系统配置，如minSdk，另外一种是依赖，比如我们引入rxJava

- 数字引入
```
compileSdkVersion rootProject.ext.android.compileSd
```

- 第三方库版本号引入
```
api "com.android.support:appcompat-v7:$rootProject.ext.dependencies.supportLibraryVersion"
```

最终的效果如下图所示
![module中使用](/assets/tools/tools-gradle-03.png)
