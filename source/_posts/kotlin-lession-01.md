---
title: kotlin 基础语法
date: 2018-02-08 14:57:51
tags:
  - kotlin
---

最近的一段时间在找工作。其实并没有真正的开始找，因为，毕竟马上就要开始过年了，所以，不管是公司，还是我自己，都把中心放在了过年这个事情上。但是也有一些非常勤劳的工作者，在孜孜不倦的在自己的岗位上付出着，所以，我也陆续收到了一些关于面试的信息。真的感觉面试的内容真的超级多，如果你没有自己的明确定位，很容易就被带跑偏。这也就是我发现很多我之前的本专业的同学有一部分转行了，而且是和我们的计算机没有任何关系的行业，原来是这么回事。
<!--more-->
但是，我也从这些公司的要求中，找到了一些比较有意思，或者以后可能会成为业界趋势的一个征兆，所以，今天我就要来说一下，就是kotlin语言。说着这里可能大家会说，切，什么啊，这个东西很早就出来了，你现在才说。好吧，确实是很早就出来了，但是我想问一下，你们都公司有要求你们把所有的Android代码都是使用kotlin语言写的吗？我估计没有公司这么做。先不管这些名利的问题，我们就直说这个kotlin语言的特点。
网上说这个语言很好学，也有人说很难学的。这么跟大家说一下，如果你之前学过类似Typescript，CoffeeScript这些JS扩展语言的话，那么kotlin将会非常好学。但如果你之前只学过java，并且对java的掌握并没有那么的全面的时候，就会是比较难。难并不在于语法结构，逻辑概念，而在于因为和java有很多相似和不同的地方，很容易让人搞混，所以，我今天就结合着之前看过的一本书《kotlin for android developer》，这本书写的真的很不错。之前一直想在网上找到资源，无奈手残党。然后某CSDN的网站一个coder竟然要价15个积分，我连看都不看，直接拉黑，因为我知道如果一程序员没有开源共享的态度，那么他是不会在这条道路上走太久的。所以，今天我把我在网上找到的分享给大家,并且我自己也要总结一下。
连接:<a href="链接:https://pan.baidu.com/s/1qZPMJ1M">连接</a>密码：cr18
> 郑重声明，本篇文章的翻译只作为学习参考之用，不能应用于各种商业用途，在参考学习完成之后，请自觉删除，所有和本文有关的纠纷，与本人无关，特此声明

# kotlin
## 什么是kotlin
Kotlin，是由jetBrains开发的基于JVM的语言。Kotlin是使用java语言开发者的思维创建出来的，那么我们的Android使用Kotlin有什么样的优点：
1. 对于java开发者而言，Kotlin语言是非常直觉话的，并且容易学习
2. 它与我们日常使用的IDE完全吻合，as可以非常完美的编译，运行Kotlin语言

那么其实这些都是我们自己总结的，那么其实官方也有为我们总结关于Kotlin的具体特点
1. 易表现：通过Kotlin可以很容易的避免模板代码因为因为大部分的典型情况都在语言中默认覆盖了，如下的例子：
```
public class Article{
    private long id;
    private String name;
    private String url;
    private String mbid;
     
    public long getId(){
        return this.id;
    }
    public void setId(long id){
        this.id = id;
    }

    public String getName(){
        return this.name;
    }
    public void setName(String name){
        this.name = name;
    }

    public String getUrl(){
        return this.url;
    }
    publiv void setUrl(String url){
        this.url = url;
    }

    public String getMbid(){
        return this.mbid;
    }
    public void setMbid(String mbid){
        this.mbid = mbid;
    }
}
```
但是如果我们通过Kotlin语言来实现，代码就会减少很多
```
data class Article(
    var id:Long,
    var name:String,
    var url:String,
    var mbid:String
)
```
这个类它就会自动生成所有的属性和他们的访问器，以及一些有用的方法，比如toString()方法
2. 空安全：java语言我们可以称之为防御性语言，如果我们不想遇到空指针异常，那么我们需要在使用它之前不停地判断他是否为空，如果为空则不能执行操作。Kotlin语言和其他的现代语言是相类似的，都是空安全的，因为我们需要通过一个安全调用操作符(用?标识)明确的来指定一个对象是否为空
```
//这里是不能通过编译的，因为我们这规定了，notNullArticle不能为空
var notNullArticle:Article = null

//可以编译，因为我们这里变量可以为空
var artist:Article? = null

//无法编译，因为artist可能为空，那么如果是空，则不能调用方法
artist.print()

//可以编译，这里只有当artist不为空的时候才可以执行响应的函数
artist?.print()

//如果我们在使用之前就调用判断是否为空，那么我们这里可以直接使用
if(artist != null){
    artist.print()
}

//只有确保artis不为空的情况下才能调用print，否则不给调用
artist!!.print()

//我们有时候可以使用Elvis来判断当前是否为空，然后如果为空需要赋予额一个值，我们也可以成为是三元运算符
var name = artist?.name ? : "empty"
```
3. 拓展方法：我们可以给任何类添加方法，他比我们那些在项目中使用额工具类拥有更加良好的可读性，下面是书里面给举得例子
```
fun Fragment.Toast(message:CharSequeue,duration:Int = Toast.SHORT){
    Toast.makeText(getActivity(),message,duration).show()
}
```
那么我们紧接着就可以调用这个方法
```
fragment.toast("你好");
```
4. 函数式支持：有时候我们做开发的时候总会遇到这样的情况：当我们添加点击事件的时候，总是不得不使用一个匿名函数作为接口回调，实现点击之后的操作。其实最终我们可以这样实现
```
view.setOnclickListener(
    toast("hello world")
)
```
## android studio
android studio是被google公司承认的Android开发官方软件。Android studio3.0以前的版本不支持Kotlin，那么我们需要使用一些插件，来完成，如果使用额是Android studio3.0以后的版本，那么我们就可以直接使用了。
那么我们就按照书籍上的例子先去创建一个Android项目，并且为了能够正常