---
title: Flutter (一)  Flutter基础介绍
date: 2019-07-02 20:31:11
tags:
  - flutter
---

简介
<!--more-->
Flutter是谷歌的移动UI框架，可以快速在iOS和Android上构建高质量的原生用户界面。 Flutter可以与现有的代码一起工作。在全世界，Flutter正在被越来越多的开发者和组织使用，并且Flutter是完全免费、开源的 
[Flutter Dev](https://flutter.dev)


# 环境配置
我使用的是Mac系统，其实不管是Windows系统还是Linux系统，配置环境都是比较简单的。可以在网上找到相关的教程，这里我就不再赘述。如果你也是Mac系统，可以参考博客[Flutter的环境配置](https://www.jianshu.com/p/b50a92afbef1)

当然还有另外一种方式，就是首先你的电脑里要安装Android studio，然后再插件中搜索Flutter，安装完成之后，重启Android Studio，你就会在主界面中发现有这么一个创建Flutter的选项，如图

![](/assets/flutter_01/flutter01.png)

选中这个选项，进入到如图所示的页面

![安装FlutterSDK](/assets/flutter_01/flutter02.png)

在这里我们因为没有安装过Flutter的SDK，所以直接选中安装按钮即可，系统会自动帮我们下载好SDK，并且配置好。下载的速度可能会有点慢，当然我们可以使用VPN，这个情况就因人而异了。我使用的是第一种方法，在官网下载好SDK的压缩文件，自己通过解压，配置环境变量，就可以使用了。

安装并且配置好环境之后，在终端中输入: flutter -h 如果现实如下图所示的内容，就表示你的环境已经搭建好了

![检测FlutterSDK](/assets/flutter_01/flutter03.png)

这里的截图比较少，其实后面还有很多的内容，包括Flutter常用的一些命令，都会在这里显示出来。

紧接着我们可以检查一下当前的环境，因为Flutter不仅可以满足Android开发，还可以同时满足IOS开发，那么我们有没有将相应的开发环境配置好呢？需要让Flutter帮我们检测一下，检测的命令就是：flutter doctor

![flutter doctor](/assets/flutter_01/flutter04.png)

这里我的电脑里还有一些配置没有安装完成 有！或者 x 符号的表示内容没有安装完成。

首先运行这句：flutter doctor --android-licenses

![flutter doctor --android-licenses](/assets/flutter_01/flutter05.png)

这里面会有几个地方需要我们同意的，其实直接输入y，即可。

最后的样式就是这样的

![](/assets/flutter_01/flutter06.png)

表示对于Android部分的内容是没问题的了。

紧接着做其他的部分，可能根据每个人电脑的配置而进行修改。或者你可以直接根据提示进行修改如下图：

![需要修改的内容](/assets/flutter_01/flutter07.png)

这个过程可能会比较慢，请耐心等待，或者你可以在晚上的时候让他自己去下载，然后我们去休息。过程中如果出现什么错误，一般会给出修改的提示，然后按照提示修改即可，如果遇到一些比较难缠的问题，只能去google了。

# 配置编辑器
使用android Studio进行开发比较简单，我这里介绍的是VSCode的方式

## 安装Flutter插件

1. 启动VSCode
2. 调用View>Command Palette…
3. 输入 ‘install’, 然后选择 Extensions: Install Extension action
4. 在右侧输入框Flutter 选择其中的install安装即可
5. 重启VSCode

通过Flutter Doctor验证设置

调用 View>Command Palette…
输入 ‘doctor’, 然后选择 ‘Flutter: Run Flutter Doctor’ action
查看“OUTPUT”窗口中的输出是否有问题


# 一个Demo

## 创建Flutter App
创建一个简单的、基于模板的Flutter应用程序，这里我把它命名为Flutter_Demo
![创建Flutter](/assets/flutter_01/flutter08.png)

下一步，选择Flutter Application
![](/assets/flutter_01/flutter09.png)

下一步，设置项目名称和路径
![](/assets/flutter_01/flutter10.png)

下一步，设置包名
![](/assets/flutter_01/flutter11.png)

等待Android Studio帮我们生成好项目即可。项目结构如下所示：
![Flutter项目结构](/assets/flutter_01/flutter12.png)

在这个项目中，我们有很多包，可能特别多的内容会导致我们在刚开始学习的时候，总是不知所措，所以，我们先看一些比较关键的部分，后面的内容，慢慢来。

首先我们找到lib文件夹，这个文件夹就是我们在写项目的时候的主体功能文件夹，也就是说我们一般情况下所有的逻辑代码都会放在这里。
![Flutter项目的lib](/assets/flutter_01/flutter13.png)

在这个lib文件夹中只有一个文件---main.dart,它里面的代码如下所示
```
import 'package:flutter/material.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  // This widget is the root of your application.
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      theme: ThemeData(
        // This is the theme of your application.
        //
        // Try running your application with "flutter run". You'll see the
        // application has a blue toolbar. Then, without quitting the app, try
        // changing the primarySwatch below to Colors.green and then invoke
        // "hot reload" (press "r" in the console where you ran "flutter run",
        // or simply save your changes to "hot reload" in a Flutter IDE).
        // Notice that the counter didn't reset back to zero; the application
        // is not restarted.
        primarySwatch: Colors.blue,
      ),
      home: MyHomePage(title: 'Flutter Demo Home Page'),
    );
  }
}

class MyHomePage extends StatefulWidget {
  MyHomePage({Key key, this.title}) : super(key: key);

  // This widget is the home page of your application. It is stateful, meaning
  // that it has a State object (defined below) that contains fields that affect
  // how it looks.

  // This class is the configuration for the state. It holds the values (in this
  // case the title) provided by the parent (in this case the App widget) and
  // used by the build method of the State. Fields in a Widget subclass are
  // always marked "final".

  final String title;

  @override
  _MyHomePageState createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  int _counter = 0;

  void _incrementCounter() {
    setState(() {
      // This call to setState tells the Flutter framework that something has
      // changed in this State, which causes it to rerun the build method below
      // so that the display can reflect the updated values. If we changed
      // _counter without calling setState(), then the build method would not be
      // called again, and so nothing would appear to happen.
      _counter++;
    });
  }

  @override
  Widget build(BuildContext context) {
    // This method is rerun every time setState is called, for instance as done
    // by the _incrementCounter method above.
    //
    // The Flutter framework has been optimized to make rerunning build methods
    // fast, so that you can just rebuild anything that needs updating rather
    // than having to individually change instances of widgets.
    return Scaffold(
      appBar: AppBar(
        // Here we take the value from the MyHomePage object that was created by
        // the App.build method, and use it to set our appbar title.
        title: Text(widget.title),
      ),
      body: Center(
        // Center is a layout widget. It takes a single child and positions it
        // in the middle of the parent.
        child: Column(
          // Column is also layout widget. It takes a list of children and
          // arranges them vertically. By default, it sizes itself to fit its
          // children horizontally, and tries to be as tall as its parent.
          //
          // Invoke "debug painting" (press "p" in the console, choose the
          // "Toggle Debug Paint" action from the Flutter Inspector in Android
          // Studio, or the "Toggle Debug Paint" command in Visual Studio Code)
          // to see the wireframe for each widget.
          //
          // Column has various properties to control how it sizes itself and
          // how it positions its children. Here we use mainAxisAlignment to
          // center the children vertically; the main axis here is the vertical
          // axis because Columns are vertical (the cross axis would be
          // horizontal).
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            Text(
              'You have pushed the button this many times:',
            ),
            Text(
              '$_counter',
              style: Theme.of(context).textTheme.display1,
            ),
          ],
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: _incrementCounter,
        tooltip: 'Increment',
        child: Icon(Icons.add),
      ), // This trailing comma makes auto-formatting nicer for build methods.
    );
  }
}
```
虽然内容很多，但是里面每个函数的意义和作用都在里面说明的很详细，所以这里我就介绍了。
值得注意的是，Flutter使用的是Dart语言开发的，这样我们就需要学习一门新的语言，但是Dart语言很简单，基本上两三天就能上手，抽空我会在去把Dart语言总结一下。
首先我们来运行一下这个应用程序。这里我使用的是Android虚拟机。运行之后的效果如下图所示

![Flutter运行结果](/assets/flutter_01/flutter14.png)

