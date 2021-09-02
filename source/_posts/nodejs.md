---
title: nodejs之Express
date: 2018-01-28 19:13:28
tags:
  - nodejs
  - express
---

项目介绍：基于nodejs+mysql+express+jade的招聘信息类网站构建
<!--more-->
#node
## 定义
简单来说Nodejs就是运行在服务端的JavaScript
Nodejs是一个事件驱动I/O服务端JavaScript环境，基于Google浏览器V8引擎，V8引擎运行JavaScript的速度非常快
## node安装
直接在百度中搜索node，或者直接点击下载链接<a href="http://nodejs.cn">node中文网</a>
下载对应的版本之后，安装提示默认安装即可。

## nodejs的第一次体验
使用nodejs创建一个项目一般分为三部分，如下
1. 引入require模块，我们可以使用require模块来引入node模块
2. 创建一个服务器，这个服务器用于监听客户端，也就是浏览器的地址访问和端口访问，如果都能匹配上，就会监听到有客户端访问服务器，并且做出对应的操作
3. 接受请求和处理请求，一般会经过listen方法，监听某一个接口。

### 引入require模块
使用require模块引入http
```
var http = require('http');
```
### 创建一个服务器
通过http实例中的createServer方法，并且使用listen方法实现端口的监听
```
var http = require('http');

http.createServer(function(request,response){
  // 发送 HTTP 头部 
  // HTTP 状态值: 200 : OK
  // 内容类型: text/plain
  response.writeHead(200, {'Content-Type': 'text/html'});

  // 发送响应数据 "Hello World"
  response.end('Hello World\n');
}).listen(8080);
// 终端打印如下信息
console.log('Server running at http://127.0.0.1:8888/');
```
## nodejsREPL (交互解析器)
Node.js REPL(Read Eval Print Loop:交互式解释器) 表示一个电脑的环境，类似 Window 系统的终端或 Unix/Linux shell，我们可以在终端中输入命令，并接收系统的响应
Nodejs自带的交互式解析器，可以执行以下的内容
1. 读取 - 读取用户输入，解析输入了Javascript 数据结构并存储在内存中
2. 执行 - 执行输入的数据结构
3. 执行 - 执行输入的数据结构
4. 循环 - 循环操作以上步骤直到用户两次按下 ctrl-c 按钮退出。
Node 的交互式解释器可以很好的调试 Javascript 代码
![终端](/assets/express/nodejs02.png)
### nodejs简单表达式
![终端](/assets/express/nodejs01.png)
### 使用变量
你可以将数据存储在变量中，并在你需要的时候使用它。
变量声明需要使用 var 关键字，如果没有使用 var 关键字变量会直接打印出来。
使用 var 关键字的变量可以使用 console.log() 来输出变量。
![终端](/assets/express/nodejs03.png)
### 多行表达式
Node REPL 支持输入多行表达式，这就有点类似 JavaScript。接下来让我们来执行一个 do-while 循环
![终端](/assets/express/nodejs04.png)
### 下划线(_)
你可以使用下划线(_)获取上一个表达式的运算结果：
![终端](/assets/express/nodejs05.png)
### REPL命令
- ctrl + c - 退出当前终端。
- ctrl + c 按下两次 - 退出 Node REPL。
- ctrl + d - 退出 Node REPL.
- 向上/向下 键 - 查看输入的历史命令
- tab 键 - 列出当前命令
- .help - 列出使用命令
- .break - 退出多行表达式
- .clear - 退出多行表达式
- .save filename - 保存当前的 Node REPL 会话到指定文件
- .load filename - 载入当前 Node REPL 会话的文件内容
### 停止REPL命令
两次ctrl+C

## 事件循环
Node.js 是单进程单线程应用程序，但是通过事件和回调支持并发，所以性能非常高。
Node.js 的每一个 API 都是异步的，并作为一个独立线程运行，使用异步函数调用，并处理并发。
Node.js 基本上所有的事件机制都是用设计模式中观察者模式实现。
Node.js 单线程类似进入一个while(true)的事件循环，直到没有事件观察者退出，每个异步事件都生成一个事件观察者，如果有事件发生就调用该回调函数.
### 事件驱动程序
Node.js 使用事件驱动模型，当web server接收到请求，就把它关闭然后进行处理，然后去服务下一个web请求。
当这个请求完成，它被放回处理队列，当到达队列开头，这个结果被返回给用户。
这个模型非常高效可扩展性非常强，因为webserver一直接受请求而不等待任何读写操作。（这也被称之为非阻塞式IO或者事件驱动IO）
在事件驱动模型中，会生成一个主循环来监听事件，当检测到事件时触发回调函数
![终端](/assets/express/nodejs06.jpg)
整个事件驱动的流程就是这么实现的，非常简洁。有点类似于观察者模式，事件相当于一个主题(Subject)，而所有注册到这个事件上的处理函数相当于观察者(Observer)。
Node.js 有多个内置的事件，我们可以通过引入 events 模块，并通过实例化 EventEmitter 类来绑定和监听事件，如下实例
```
// 引入 events 模块
var events = require('events');
// 创建 eventEmitter 对象
var eventEmitter = new events.EventEmitter();
```
以下程序绑定事件处理程序
```
// 绑定事件及事件的处理程序
eventEmitter.on('eventName', eventHandler);
```
我们可以通过程序触发
```
// 触发事件
eventEmitter.emit('eventName');
```
实例：
```
// 引入 events 模块
var events = require('events');
// 创建 eventEmitter 对象
var eventEmitter = new events.EventEmitter();

// 创建事件处理程序
var connectHandler = function connected() {
   console.log('连接成功。');
  
   // 触发 data_received 事件 
   eventEmitter.emit('data_received');
}

// 绑定 connection 事件处理程序
eventEmitter.on('connection', connectHandler);
 
// 使用匿名函数绑定 data_received 事件
eventEmitter.on('data_received', function(){
   console.log('数据接收成功。');
});

// 触发 connection 事件 
eventEmitter.emit('connection');

console.log("程序执行完毕。");
```
## nodejs Buffer缓冲区
JavaScript 语言自身只有字符串数据类型，没有二进制数据类型。
但在处理像TCP流或文件流时，必须使用到二进制数据。因此在 Node.js中，定义了一个 Buffer 类，该类用来创建一个专门存放二进制数据的缓存区。
在 Node.js 中，Buffer 类是随 Node 内核一起发布的核心库。Buffer 库为 Node.js 带来了一种存储原始数据的方法，可以让 Node.js 处理二进制数据，每当需要在 Node.js 中处理I/O操作中移动的数据时，就有可能使用 Buffer 库。原始数据存储在 Buffer 类的实例中。一个 Buffer 类似于一个整数数组，但它对应于 V8 堆内存之外的一块原始内存。
### buffer与编码
Buffer 实例一般用于表示编码字符的序列，比如 UTF-8 、 UCS2 、 Base64 、或十六进制编码的数据。 通过使用显式的字符编码，就可以在 Buffer 实例与普通的 JavaScript 字符串之间进行相互转换
```
const buf = Buffer.from('runoob', 'ascii');

// 输出 72756e6f6f62
console.log(buf.toString('hex'));

// 输出 cnVub29i
console.log(buf.toString('base64'));
```
### 创建一个buffer类
- Buffer.alloc(size[, fill[, encoding]])： 返回一个指定大小的 Buffer 实例，如果没有设置 fill，则默认填满 0
- Buffer.allocUnsafe(size)： 返回一个指定大小的 Buffer 实例，但是它不会被初始化，所以它可能包含敏感的数据
- Buffer.allocUnsafeSlow(size)
- Buffer.from(array)： 返回一个被 array 的值初始化的新的 Buffer 实例（传入的 array 的元素只能是数字，不然就会自动被 0 覆盖）
- Buffer.from(arrayBuffer[, byteOffset[, length]])： 返回一个新建的与给定的 ArrayBuffer 共享同一内存的 Buffer。
- Buffer.from(buffer)： 复制传入的 Buffer 实例的数据，并返回一个新的 Buffer 实例
- Buffer.from(string[, encoding])： 返回一个被 string 的值初始化的新的 Buffer 实例
```
// 创建一个长度为 10、且用 0 填充的 Buffer。
const buf1 = Buffer.alloc(10);

// 创建一个长度为 10、且用 0x1 填充的 Buffer。 
const buf2 = Buffer.alloc(10, 1);

// 创建一个长度为 10、且未初始化的 Buffer。
// 这个方法比调用 Buffer.alloc() 更快，
// 但返回的 Buffer 实例可能包含旧数据，
// 因此需要使用 fill() 或 write() 重写。
const buf3 = Buffer.allocUnsafe(10);

// 创建一个包含 [0x1, 0x2, 0x3] 的 Buffer。
const buf4 = Buffer.from([1, 2, 3]);

// 创建一个包含 UTF-8 字节 [0x74, 0xc3, 0xa9, 0x73, 0x74] 的 Buffer。
const buf5 = Buffer.from('tést');

// 创建一个包含 Latin-1 字节 [0x74, 0xe9, 0x73, 0x74] 的 Buffer。
const buf6 = Buffer.from('tést', 'latin1');
```
### 写入缓存区
#### 语法
```
buf.write(string[, offset[, length]][, encoding])
```
#### 参数
1. string - 写入缓冲区的字符串。
2. offset - 缓冲区开始写入的索引值，默认为 0 。
3. length - 写入的字节数，默认为 buffer.length
4. encoding - 使用的编码。默认为 'utf8' 。
根据 encoding 的字符编码写入 string 到 buf 中的 offset 位置。 length 参数是写入的字节数。 如果 buf 没有足够的空间保存整个字符串，则只会写入 string 的一部分。 只部分解码的字符不会被写入。
#### 返回值
返回实际写入的大小。如果 buffer 空间不足， 则只会写入部分字符串。
实例：
```
buf = Buffer.alloc(256);
len = buf.write("www.runoob.com");

console.log("写入字节数 : "+  len);
```
### 从缓存区读取数据
#### 语法
```
buf.toString([encoding[, start[, end]]])
```
#### 参数
1. encoding - 使用的编码。默认为 'utf8' 。
2. start - 指定开始读取的索引位置，默认为 0。
3. end - 结束位置，默认为缓冲区的末尾。
### 返回值
解码缓冲区数据并使用指定的编码返回字符串。
实例：
```
buf = Buffer.alloc(26);
for (var i = 0 ; i < 26 ; i++) {
  buf[i] = i + 97;
}

console.log( buf.toString('ascii'));       // 输出: abcdefghijklmnopqrstuvwxyz
console.log( buf.toString('ascii',0,5));   // 输出: abcde
console.log( buf.toString('utf8',0,5));    // 输出: abcde
console.log( buf.toString(undefined,0,5)); // 使用 'utf8' 编码, 并输出: abcde
```
### 将buffer转换成json
#### 语法
```
buf.toJSON()
```
> 当字符串化一个 Buffer 实例时，JSON.stringify() 会隐式地调用该 toJSON()
#### 返回值
返回 JSON 对象。
实例：
```
const buf = Buffer.from(["google", "runoob", "taobao"]);
const json = JSON.stringify(buf);

// 输出: {"type":"Buffer","data":[1,2,3,4,5]}
console.log(json);

const copy = JSON.parse(json, (key, value) => {
  return value && value.type === 'Buffer' ?
    Buffer.from(value.data) :
    value;
});

// 输出: <Buffer 01 02 03 04 05>
console.log(copy);
```
### 缓存区合并
```
Buffer.concat(list[, totalLength])
```
1. list - 用于合并的 Buffer 对象数组列表。
2. totalLength - 指定合并后Buffer对象的总长度。
实例：
```
var buffer1 = Buffer.from(('菜鸟教程'));
var buffer2 = Buffer.from(('www.runoob.com'));
var buffer3 = Buffer.concat([buffer1,buffer2]);
console.log("buffer3 内容: " + buffer3.toString());
```
### 缓存区比较
```
buf.compare(otherBuffer);
```
1. otherBuffer - 与 buf 对象比较的另外一个 Buffer 对象。
#### 返回值
返回一个数字，表示 buf 在 otherBuffer 之前，之后或相同。
实例
```
var buffer1 = Buffer.from('ABC');
var buffer2 = Buffer.from('ABCD');
var result = buffer1.compare(buffer2);

if(result < 0) {
   console.log(buffer1 + " 在 " + buffer2 + "之前");
}else if(result == 0){
   console.log(buffer1 + " 与 " + buffer2 + "相同");
}else {
   console.log(buffer1 + " 在 " + buffer2 + "之后");
}
```
### 拷贝缓冲区
```
buf.copy(targetBuffer[, targetStart[, sourceStart[, sourceEnd]]])
```
1. targetBuffer - 要拷贝的 Buffer 对象。
2. targetStart - 数字, 可选, 默认: 0
3. sourceStart - 数字, 可选, 默认: 0
4. sourceEnd - 数字, 可选, 默认: buffer.length
实例：
```
var buf1 = Buffer.from('abcdefghijkl');
var buf2 = Buffer.from('RUNOOB');

//将 buf2 插入到 buf1 指定位置上
buf2.copy(buf1, 2);

console.log(buf1.toString());
```
### 缓存区裁剪
```
buf.slice([start[, end]])
```
1. start - 数字, 可选, 默认: 0
2. end - 数字, 可选, 默认: buffer.length
实例：
```
var buffer1 = Buffer.from('runoob');
// 剪切缓冲区
var buffer2 = buffer1.slice(0,2);
console.log("buffer2 content: " + buffer2.toString());
```
## nodejs Stream流
Stream 是一个抽象接口，Node 中有很多对象实现了这个接口。例如，对http 服务器发起请求的request 对象就是一个 Stream，还有stdout（标准输出）
Node.js，Stream 有四种流类型：
1. Readable - 可读操作。
2. Writable - 可写操作。
3. Duplex - 可读可写操作.
4. Transform - 操作被写入数据，然后读出结果。
所有的 Stream 对象都是 EventEmitter 的实例。常用的事件有：
1. data - 当有数据可读时触发。
2. end - 没有更多的数据可读时触发。
3. error - 在接收和写入过程中发生错误时触发。
4. finish - 所有数据已被写入到底层系统时触发。

### 从流中读取数据
```
var fs = require("fs");
var data = '';

// 创建可读流
var readerStream = fs.createReadStream('input.txt');

// 设置编码为 utf8。
readerStream.setEncoding('UTF8');

// 处理流事件 --> data, end, and error
readerStream.on('data', function(chunk) {
   data += chunk;
});

readerStream.on('end',function(){
   console.log(data);
});

readerStream.on('error', function(err){
   console.log(err.stack);
});

console.log("程序执行完毕");
```
### 写入流
```
var fs = require("fs");
var data = '菜鸟教程官网地址：www.runoob.com';

// 创建一个可以写入的流，写入到文件 output.txt 中
var writerStream = fs.createWriteStream('output.txt');

// 使用 utf8 编码写入数据
writerStream.write(data,'UTF8');

// 标记文件末尾
writerStream.end();

// 处理流事件 --> data, end, and error
writerStream.on('finish', function() {
    console.log("写入完成。");
});

writerStream.on('error', function(err){
   console.log(err.stack);
});

console.log("程序执行完毕");
```
### 管道流
管道提供了一个输出流到输入流的机制。通常我们用于从一个流中获取数据并将数据传递到另外一个流中。
```
var fs = require("fs");

// 创建一个可读流
var readerStream = fs.createReadStream('input.txt');

// 创建一个可写流
var writerStream = fs.createWriteStream('output.txt');

// 管道读写操作
// 读取 input.txt 文件内容，并将内容写入到 output.txt 文件中
readerStream.pipe(writerStream);

console.log("程序执行完毕");
```
### 链式流
链式是通过连接输出流到另外一个流并创建多个流操作链的机制。链式流一般用于管道操作。
接下来我们就是用管道和链式来压缩和解压文件。
```
var fs = require("fs");
var zlib = require('zlib');

// 压缩 input.txt 文件为 input.txt.gz
fs.createReadStream('input.txt')
  .pipe(zlib.createGzip())
  .pipe(fs.createWriteStream('input.txt.gz'));
  
console.log("文件压缩完成。");
```
执行完以上操作后，我们可以看到当前目录下生成了 input.txt 的压缩文件 input.txt.gz。
接下来，让我们来解压该文件，创建 decompress.js 文件，代码如下：
```
var fs = require("fs");
var zlib = require('zlib');

// 解压 input.txt.gz 文件为 input.txt
fs.createReadStream('input.txt.gz')
  .pipe(zlib.createGunzip())
  .pipe(fs.createWriteStream('input.txt'));
  
console.log("文件解压完成。");
```
以上的所有例子我们会发现都是替换，并没有追加，如果想要追加，还有另外的方式，后面会总结
## nodejs模块系统
为了让Node.js的文件可以相互调用，Node.js提供了一个简单的模块系统。
模块是Node.js 应用程序的基本组成部分，文件和模块是一一对应的。换言之，一个 Node.js 文件就是一个模块，这个文件可能是JavaScript 代码、JSON 或者编译过的C/C++ 扩展。
### 创建模块
```
var hello = require('./hello');
hello.world();
```
以上实例中，代码 require('./hello') 引入了当前目录下的 hello.js 文件（./ 为当前目录，node.js 默认后缀为 js）。
Node.js 提供了 exports 和 require 两个对象，其中 exports 是模块公开的接口，require 用于从外部获取一个模块的接口，即所获取模块的 exports 对象
接下来看一下hello.js文件
```
exports.world = function() {
  console.log('Hello World');
}
```
在以上示例中，hello.js 通过 exports 对象把 world 作为模块的访问接口，在 main.js 中通过 require('./hello') 加载这个模块，然后就可以直接访 问 hello.js 中 exports 对象的成员函数了。
有时候我们只是想把一个对象封装到模块中，格式如下：
```
module.exports = function() {
  // ...
}
```
例如：
```
//hello.js 
function Hello() { 
    var name; 
    this.setName = function(thyName) { 
        name = thyName; 
    }; 
    this.sayHello = function() { 
        console.log('Hello ' + name); 
    }; 
}; 
module.exports = Hello;
```
后面我们在express的路由中会经常看到这样的情况出现
模块接口的唯一变化是使用 module.exports = Hello 代替了exports.world = function(){}。 在外部引用该模块时，其接口对象就是要输出的 Hello 对象本身，而不是原先的 exports。
## nodejs路由
我们要为路由提供请求的 URL 和其他需要的 GET 及 POST 参数，随后路由需要根据这些数据来执行相应的代码。
因此，我们需要查看 HTTP 请求，从中提取出请求的 URL 以及 GET/POST 参数。这一功能应当属于路由还是服务器（甚至作为一个模块自身的功能）确实值得探讨，但这里暂定其为我们的HTTP服务器的功能。
我们需要的所有数据都会包含在 request 对象中，该对象作为 onRequest() 回调函数的第一个参数传递。但是为了解析这些数据，我们需要额外的 Node.JS 模块，它们分别是 url 和 querystring 模块
![终端](/assets/express/nodejs07.png)
现在我们来给 onRequest() 函数加上一些逻辑，用来找出浏览器请求的 URL 路径
```
var http = require("http");
var url = require("url");
 
function start() {
  function onRequest(request, response) {
    var pathname = url.parse(request.url).pathname;
    console.log("Request for " + pathname + " received.");
    response.writeHead(200, {"Content-Type": "text/plain"});
    response.write("Hello World");
    response.end();
  }
 
  http.createServer(onRequest).listen(8888);
  console.log("Server has started.");
}
 
exports.start = start;
```
好了，我们的应用现在可以通过请求的 URL 路径来区别不同请求了--这使我们得以使用路由（还未完成）来将请求以 URL 路径为基准映射到处理程序上。
在我们所要构建的应用中，这意味着来自 /start 和 /upload 的请求可以使用不同的代码来处理。稍后我们将看到这些内容是如何整合到一起的。
现在我们可以来编写路由了，建立一个名为 router.js 的文件，添加以下内容：
```
function route(pathname) {
  console.log("About to route a request for " + pathname);
}
 
exports.route = route;
```
我们的服务器应当知道路由的存在并加以有效利用。我们当然可以通过硬编码的方式将这一依赖项绑定到服务器上，但是其它语言的编程经验告诉我们这会是一件非常痛苦的事，因此我们将使用依赖注入的方式较松散地添加路由模块。
首先，我们来扩展一下服务器的 start() 函数，以便将路由函数作为参数传递过去，server.js 文件代码如下
```
var http = require("http");
var url = require("url");
 
function start(route) {
  function onRequest(request, response) {
    var pathname = url.parse(request.url).pathname;
    console.log("Request for " + pathname + " received.");
 
    route(pathname);
 
    response.writeHead(200, {"Content-Type": "text/plain"});
    response.write("Hello World");
    response.end();
  }
 
  http.createServer(onRequest).listen(8888);
  console.log("Server has started.");
}
 
exports.start = start;
```
router.js
```
var server = require("./server");
var router = require("./router");
 
server.start(router.route);
```
## nodejs常用工具
util 是一个Node.js 核心模块，提供常用函数的集合，用于弥补核心JavaScript 的功能 过于精简的不足
### util.inherits
util.inherits(constructor, superConstructor)是一个实现对象间原型继承 的函数。
JavaScript 的面向对象特性是基于原型的，与常见的基于类的不同。JavaScript 没有 提供对象继承的语言级别特性，而是通过原型复制来实现的。
```
var util = require('util'); 
function Base() { 
    this.name = 'base'; 
    this.base = 1991; 
    this.sayHello = function() { 
    console.log('Hello ' + this.name); 
    }; 
} 
Base.prototype.showName = function() { 
    console.log(this.name);
}; 
function Sub() { 
    this.name = 'sub'; 
} 
util.inherits(Sub, Base); 
var objBase = new Base(); 
objBase.showName(); 
objBase.sayHello(); 
console.log(objBase); 
var objSub = new Sub(); 
objSub.showName(); 
//objSub.sayHello(); 
console.log(objSub); 
```
我们定义了一个基础对象Base 和一个继承自Base 的Sub，Base 有三个在构造函数 内定义的属性和一个原型中定义的函数，通过util.inherits 实现继承
> 注意：Sub 仅仅继承了Base 在原型中定义的函数，而构造函数内部创造的 base 属 性和 sayHello 函数都没有被 Sub 继承。
同时，在原型中定义的属性不会被console.log 作 为对象的属性输出。如果我们去掉 objSub.sayHello(); 这行的注释，将会看到：
```
node.js:201 
throw e; // process.nextTick error, or 'error' event on first tick 
^ 
TypeError: Object #&lt;Sub&gt; has no method 'sayHello' 
at Object.&lt;anonymous&gt; (/home/byvoid/utilinherits.js:29:8) 
at Module._compile (module.js:441:26) 
at Object..js (module.js:459:10) 
at Module.load (module.js:348:31) 
at Function._load (module.js:308:12) 
at Array.0 (module.js:479:10) 
at EventEmitter._tickCallback (node.js:192:40) 
```
### util.inspect(会使用的比较多)
util.inspect(object,[showHidden],[depth],[colors])是一个将任意对象转换 为字符串的方法，通常用于调试和错误输出。它至少接受一个参数 object，即要转换的对象。
showHidden 是一个可选参数，如果值为 true，将会输出更多隐藏信息。
depth 表示最大递归的层数，如果对象很复杂，你可以指定层数以控制输出信息的多 少。如果不指定depth，默认会递归2层，指定为 null 表示将不限递归层数完整遍历对象。 如果color 值为 true，输出格式将会以ANSI 颜色编码，通常用于在终端显示更漂亮 的效果。
特别要指出的是，util.inspect 并不会简单地直接把对象转换为字符串，即使该对 象定义了toString 方法也不会调用。
```
var util = require('util'); 
function Person() { 
    this.name = 'byvoid'; 
    this.toString = function() { 
    return this.name; 
    }; 
} 
var obj = new Person(); 
console.log(util.inspect(obj)); 
console.log(util.inspect(obj, true)); 
```

# express
## 定义
基于Node.js平台，快速，开发，极简的web开发框架。
安装的时候需要借助node.js
```
npm install -g express
```
安装完成之后，就算是基本的全局安装已经搞定了，紧接着，为了能够快速上手express，我们借助官方提供的工具express-generator，实现创建一个express项目，并且在这个项目中有一些必备的基础代码
```
npm install -g express-generator
```
这样就搞定了最基本的操作。
## 文档结构介绍
如图所示的就是下载好之后的文件路径
![文档结构](/assets/express/express01.png)
这个项目运行起来的命令式
```
npm start
```
### bin目录
bin目录主要完成的内容是一些全局性的操作，比如设置当前本地服务器被访问的端口，监听设置，错误相应等。
### node_modules目录
node_modules目录标识的事当前在本地提供的依赖，所有的依赖结构都会放在这
### public目录
public目录一般存放一些静态文件，比如css，jpg，png等资源，有时候一些第三方框架也会放在这里，比如bootstrap，jquery。
### root路由目录
root目录主要实现当前页面所有的子元素都会放在这里。js文件里面包括实现网络请求，数据格式化
### views目录
views目录主要实现内容页面的展示，在官方提供的模板中使用的是jade。

## 创建一个基本的项目
```
express jobServer
```

## 路由
路由指的是如何定义应用端点和如何响应客户端请求
路由是由一个 URI、HTTP 请求（GET、POST等）和若干个句柄组成，它的结构如下： app.METHOD(path, [callback...], callback)， app 是 express 对象的一个实例， METHOD 是一个 HTTP 请求方法， path 是服务器上的路径， callback 是当路由匹配时要执行的函数。
实例：
```
var express = require('express');
var app = express();

// respond with "hello world" when a GET request is made to the homepage
app.get('/', function(req, res) {
  res.send('hello world');
});
```
### 路由方法
路由方法源于 HTTP 请求方法，和 express 实例相关联。

下面这个例子展示了为应用跟路径定义的 GET 和 POST 请求
```
// GET method route
app.get('/', function (req, res) {
  res.send('GET request to the homepage');
});

// POST method route
app.post('/', function (req, res) {
  res.send('POST request to the homepage');
});
```
Express 定义了如下和 HTTP 请求对应的路由方法： get, post, put, head, delete, options, trace, copy, lock, mkcol, move, purge, propfind, proppatch, unlock, report, mkactivity, checkout, merge, m-search, notify, subscribe, unsubscribe, patch, search, 和 connect。
> 有些路由方法名不是合规的 JavaScript 变量名，此时使用括号记法，比如： app['m-search']('/', function ...
app.all() 是一个特殊的路由方法，没有任何 HTTP 方法与其对应，它的作用是对于一个路径上的所有请求加载中间件。

在下面的例子中，来自 “/secret” 的请求，不管使用 GET、POST、PUT、DELETE 或其他任何 http 模块支持的 HTTP 请求，句柄都会得到执行。
```
app.all('/secret', function (req, res, next) {
  console.log('Accessing the secret section ...');
  next(); // pass control to the next handler
});
```
### 路由路径
路由路径和请求方法一起定义了请求的端点，它可以是字符串、字符串模式或者正则表达式
```
// 匹配根路径的请求
app.get('/', function (req, res) {
  res.send('root');
});

// 匹配 /about 路径的请求
app.get('/about', function (req, res) {
  res.send('about');
});

// 匹配 /random.text 路径的请求
app.get('/random.text', function (req, res) {
  res.send('random.text');
});
```
使用字符串模式匹配路径，这里使用的是正则表达式
```
// 匹配 acd 和 abcd
app.get('/ab?cd', function(req, res) {
  res.send('ab?cd');
});

// 匹配 abcd、abbcd、abbbcd等
app.get('/ab+cd', function(req, res) {
  res.send('ab+cd');
});

// 匹配 abcd、abxcd、abRABDOMcd、ab123cd等
app.get('/ab*cd', function(req, res) {
  res.send('ab*cd');
});

// 匹配 /abe 和 /abcde
app.get('/ab(cd)?e', function(req, res) {
 res.send('ab(cd)?e');
});
```
也可以这样
```
// 匹配任何路径中含有 a 的路径：
app.get(/a/, function(req, res) {
  res.send('/a/');
});

// 匹配 butterfly、dragonfly，不匹配 butterflyman、dragonfly man等
app.get(/.*fly$/, function(req, res) {
  res.send('/.*fly$/');
});
```
### 路由句柄
可以为请求处理提供多个回调函数，其行为类似 中间件。唯一的区别是这些回调函数有可能调用 next('route') 方法而略过其他路由回调函数。可以利用该机制为路由定义前提条件，如果在现有路径上继续执行没有意义，则可将控制权交给剩下的路径。

路由句柄有多种形式，可以是一个函数、一个函数数组，或者是两者混合，如下所示.

使用一个回调函数处理路由：
```
app.get('/example/a', function (req, res) {
  res.send('Hello from A!');
});
```
使用多个回调函数(这里要使用next，在中间件中会说到)
```
app.get('/example/b', function (req, res, next) {
  console.log('response will be sent by the next function ...');
  next();
}, function (req, res) {
  res.send('Hello from B!');
});
```
使用回调函数数组
```
var cb0 = function (req, res, next) {
  console.log('CB0');
  next();
}

var cb1 = function (req, res, next) {
  console.log('CB1');
  next();
}

var cb2 = function (req, res) {
  res.send('Hello from C!');
}

app.get('/example/c', [cb0, cb1, cb2]);
```
### 使用混合函数和函数数组
```
var cb0 = function (req, res, next) {
  console.log('CB0');
  next();
}

var cb1 = function (req, res, next) {
  console.log('CB1');
  next();
}

app.get('/example/d', [cb0, cb1], function (req, res, next) {
  console.log('response will be sent by the next function ...');
  next();
}, function (req, res) {
  res.send('Hello from D!');
});
```
### 响应方法

| 方法 | 描述 |
|----|----|
|res.download()	| 提示下载文件。|
|res.end() |	终结响应处理流程。|
|res.json()	| 发送一个 JSON 格式的响应。 |
|res.jsonp() |	发送一个支持 JSONP 的 JSON 格式的响应。|
|res.redirect()	| 重定向请求。 |
|res.render()	| 渲染视图模板。|
|res.send()	| 发送各种类型的响应。|
|res.sendFile	| 以八位字节流的形式发送文件。 |
|res.sendStatus()	|设置响应状态代码，并将其以字符串形式作为响应体的一部分发送。 |

### router()
可使用 app.route() 创建路由路径的链式路由句柄。由于路径在一个地方指定，这样做有助于创建模块化的路由，而且减少了代码冗余和拼写错误。请参考 <a href="http://www.expressjs.com.cn/4x/api.html#router">Router() 文档</a> 了解更多有关路由的信息。

下面这个示例程序使用 app.route() 定义了链式路由句柄。
```
app.route('/book')
  .get(function(req, res) {
    res.send('Get a random book');
  })
  .post(function(req, res) {
    res.send('Add a book');
  })
  .put(function(req, res) {
    res.send('Update the book');
  });
```
### express.Router()
可使用 express.Router 类创建模块化、可挂载的路由句柄。Router 实例是一个完整的中间件和路由系统，因此常称其为一个 “mini-app”。

下面的实例程序创建了一个路由模块，并加载了一个中间件，定义了一些路由，并且将它们挂载至应用的路径上。

在 app 目录下创建名为 birds.js 的文件，内容如下
```
var express = require('express');
var router = express.Router();

// 该路由使用的中间件
router.use(function timeLog(req, res, next) {
  console.log('Time: ', Date.now());
  next();
});
// 定义网站主页的路由
router.get('/', function(req, res) {
  res.send('Birds home page');
});
// 定义 about 页面的路由
router.get('/about', function(req, res) {
  res.send('About birds');
});

module.exports = router;
```
然后在应用中加载路由模块
```
var birds = require('./birds');
...
app.use('/birds', birds);
```


# 项目
目录结构：
![项目目录](/assets/express/nodejs08.png)
setting.js：设置mysql数据库的环境
```
(function() {
    var settings;
    settings = {
        db: {
            host: 'localhost',     //本地数据库
            port: '3306',
            user: 'root',          //数据库用户名
            password: '710298',          //数据库密码
            database: 'myjob'  //数据库名称
        }
    };
    module.exports = settings;

}).call(this);
```
databases.js：设置数据库连接
```
(function() {
    var client, mysql, settings;

    settings = require('./settings');

    client = null;

    mysql = require('mysql');

    exports.getDbCon = function() {
        var err;
        try {
            if (client) {
                client = mysql.createConnection(settings.db);
                client.connect();
            } else {
                client = new mysql.createConnection(settings.db);
                client.connect();
            }
        } catch (_error) {
            err = _error;
            throw err;
        }
        return client;
    };

}).call(this);
```
修改app.js文件
```
var express = require('express');
var path = require('path');
var favicon = require('serve-favicon');
var logger = require('morgan');
var cookieParser = require('cookie-parser');
var bodyParser = require('body-parser');

var index = require('./routes/index');
var users = require('./routes/users');
var login = require('./routes/login');
var regist = require('./routes/regist');
var getJobs = require('./routes/getJobs');
var search = require('./routes/search');

var fs = require('fs');
var multer = require('multer');
var uploadimg = require('./routes/uploadimg');
var DIR = './uploads/';
var upload = multer({dest: DIR});



var app = express();

// view engine setup
app.set('views', path.join(__dirname, 'views'));
app.set('view engine', 'jade');

// uncomment after placing your favicon in /public
//app.use(favicon(path.join(__dirname, 'public', 'favicon.ico')));
app.use(logger('dev'));
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: false }));
app.use(cookieParser());
app.use(express.static(path.join(__dirname, 'public')));
app.use(function (req, res, next) {
    res.setHeader('Access-Control-Allow-Origin', 'http://valor-software.github.io');
    res.setHeader('Access-Control-Allow-Methods', 'POST');
    res.setHeader('Access-Control-Allow-Headers', 'X-Requested-With,content-type');
    res.setHeader('Access-Control-Allow-Credentials', true);
    next();
});
app.use(multer({
    dest: DIR,
    rename: function (fieldname, filename) {
        return filename + Date.now();
    },
    onFileUploadStart: function (file) {
        console.log(file.originalname + ' is starting ...');
    },
    onFileUploadComplete: function (file) {
        console.log(file.fieldname + ' uploaded to  ' + file.path);
    }
}));

//配置路由
app.use('/', index);
app.use('/users', users);
app.use('/login',login);
app.use('/regist',regist);

app.use('/getJobs',getJobs);
app.use('/search',search);

app.get('/uploadimg', function (req, res) {
    res.end('file catcher example');
});

app.post('/uploadimg', function (req, res) {
    upload(req, res, function (err) {
        if (err) {
            return res.end(err.toString());
        }
        res.end('File is uploaded');
    });
});

// catch 404 and forward to error handler
app.use(function(req, res, next) {
  var err = new Error('Not Found');
  err.status = 404;
  next(err);
});

// error handler
app.use(function(err, req, res, next) {
  // set locals, only providing error in development
  res.locals.message = err.message;
  res.locals.error = req.app.get('env') === 'development' ? err : {};

  // render the error page
  res.status(err.status || 500);
  res.render('error');
});

module.exports = app;
```
登录页面login.js
```
/**
 * Created by paul on 2018/1/30.
 */
var express = require('express');
var router = express.Router();
var url = require('url');

var conn = require('../databases');

var mysql = conn.getDbCon();

//登录
router.get('/', function (req, res, next) {
    //获取当前传递过来的用户名和密码
    var message;
    var params = url.parse(req.url, true).query;
    var username = params.username;
    var password = params.password;
    var sql = "select * from users where user_name = ? and user_password = ?";
    mysql.query(sql, [username, password], function (error, result) {
        if (error) {
            //查询失败
        }
        if (result.length > 0) {
            message = {
                flag: "success",
                data: result
            }
        } else {
            message = {
                flag: "faild",
                data: "登录失败，请查看你的用户名和密码是否正确，如果还没有注册，请跳转到注册页面"
            }
        }
        res.setHeader('Access-Control-Allow-Origin', '*');
        res.send(JSON.stringify(message));
    });
});


module.exports = router;
```





# 参考资料
[菜鸟教程](http://www.runoob.com/nodejs/nodejs-tutorial.html)
[nodejs中文网](http://nodejs.cn)
[express中文网](http://www.expressjs.com.cn/starter/installing.html)