---
title: android-interview-2017
date: 2017-12-24 19:16:29
tags:
  - android
  - 面试
---

2017年结尾，我打算离开广州，前往北京发展，所以又一次面临找工作。
<!--more-->
好久都没有去考过面试方面的东西了，所以，这一次我要把之前面试的东西总结一下，同时结合别人的面试经验，为过完年之后的面试做好准备。这次总结会添加一些自己之前面试的经验，也会添加一些别人面试题的总结。

首先看一张图片，Android知识图谱
![Android知识图谱](/assets/interview/interview01.png)
面试其实也就是上面的这些内容，其实还是挺多知识点的。一般中高级Android开发会问的比较深，所以要有心理准备。下面是一些重点知识，必须掌握的。如果不想在面试的时候被别人鄙视，那就一定要好好的准备

1. 基础知识：四大组件，生命周期，使用场景，如何启动
2. java基础：线程，数据结构，MVC模式，设计模式
3. 通信：网络通信(httpclient,HttpUrlConnection),Socket
4. 数据持久化：SQLite，Sharepreference，ContentProvider
5. 性能优化：布局优化，内存优化，电池优化
6. 安全：数据加密，代码混淆，WebView/JS调用，https
7. UI：动画
8. 其他：JNI，AIDl，Handler，Intent
9. 开源框架：Volley，Gilde，Rxjava
10. 拓展：Android6 7 8 新特性，kotlin语言，I/O大会

回答问题的时候要有自己的理解，不要死记硬背，因为面试官一天面试了很多人，所以，要有自己的思维

# 具体面试题
1. activity启动过程
> 这个一般不是让你回答生命周期的。
> <a href='http://blog.csdn.net/huangqili1314/article/details/72792682'>Activity启动过程</a>
2. activity启动模式和应用场景
> 四种启动方式 两种应用方式 

3. Service两种启动方式
> startService
> bindService
4. 两种广播注册
> 静态注册
> 动态注册
5. HttpClient和HttpUrlConnection的区别

6. Http和Https的区别

7. 进程保活




# 面试题
## Java方面
1. 河南郑州逸休联盟
> 题目：
> 字符串
> 1.  A1，B1，C1，C2，B2，A2，D1，D2，D3
> 2.  E1，E2，E3，F1，F2，F3 
> 字符串类型特特点：字符串里面的每个TOKEN都是 字母+数字，需要对字符串做做一个函数处理得到下面结果
> 题中只给了两个符合上面规则的字符串，符合上面规则的字符串是无限的。
> 字符串1结果：
> A -->{A1，A2}
> B -->{B1,B2}
> C -->{C1,C2}
> D-->{D1,D2,D3}
> 字符串2结果：
> E -->{ E1,E2,E3}
> F -->{F1,F2,F3}
> 请写一个函数， 输入是字符串，输出打印结果如上图所示
> 请完成后发送代码截图

主要是考察关于字符串函数的灵活应用，代码如下
```
package com.paul.demo;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.Map.Entry;
import java.util.Scanner;

public class Test {

	public static void main(String[] args) {
		Map<String,List<String>> res = new HashMap();
		Scanner str = new Scanner(System.in);
		String ss = str.next();
		String [] arr = ss.split(",");
		for(int i=0;i<arr.length;i++) {
			List<String> value = new ArrayList();
			String s = arr[i].split("\\d")[0];
			for(int y=0;y<arr.length;y++){
				if(arr[y].contains(s)) {
					value.add(arr[y]);
				}
			}
			res.put(s, value);
		}

		for(Entry<String,List<String>> hh:res.entrySet()) {
			System.out.println(hh.getKey()+"--->"+hh.getValue().toString());
		}
		
	}

}
```
运行结果：
![逸休联盟](/assets/interview/interview02.png)


# 参考资料
[面试](http://blog.csdn.net/huangqili1314/article/details/72792682)
