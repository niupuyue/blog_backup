---
title: kotlin let,run,with,apply和also的使用
date: 2021-06-21 16:14:22
tags:
  - kotlin
---

总结一下kotlin中let，also，apply，with，run的使用方法

<!--more-->

作用域函数在kotlin中是非常重要的特点，共分为以下五种：let,run,with,apply和also,这五个函数的工作方式可以说非常相似，但是我们需要了解这五个函数的差异，以便在不同的场景更好的利用它

### Kotlin的作用域函数
简单来说，作用域函数是为了方便对一个对象进行访问和操作，可以对他们进行空检查或者修改它的属性或者直接返回它的值等操作，具体来说

#### let

```
public inline fun <T, R> T.let(block: (T) -> R): R
```

let函数是参数化类型T的扩展函数，在let块内可以通过it指代该对象，返回值为let块的最后一行或自定return表达式
如下所示：

```
class Book(){
    var name = "《魁拔之书》"
    var price = 60
    fun displayInfo() = print("书 $name ,价格是 $price")
}
```
```
val book = Book().let{
    it.name = "魁拔之书"
    "这本书是${it.name}"
}
print(book)
```
在控制台中输出的结果是
```
这本书是魁拔之书
```
这里我们对Book对象使用let作用域函数，在函数块的最后一行添加了一行字符串，并且对Book对象进行了打印，我们看到最后输出的结果是字符串。正常的逻辑，打印一个对象，必定输出的是一个对象，但是在使用let函数之后，输出的是最后一行字符串，这就是let函数的特性导致的。在kotlin中，如果let块中的最后一条语句是非赋值语句，则默认情况下它就是返回语句，相应的如果我们把最后一个语句改为赋值语句，则会变成一个对象
```
val book = Book().let{
    it.name = "魁拔之书"
}
print(book)
```
控制台输出结果为
```
kotlin.Unit
```
由此可见，let的函数的特点：

1.let块中最后一条语句如果是非赋值语句，则默认情况下他是返回语句，否则返回的是一个Unit类型

2.let可以用于空安全检查
如果需要对空对象进行操作，可以对其使用安全调用操作符?.，并调用let在lambda表达式中执行操作，如下所示
```
var name:String? = null
val nameLen = names?.let{
    it.lenght
}?: "name为null"
```
当那么不为null时，会给nameLen赋值为字符串长度，否则会赋值为字符串

3.let可以对调用链的结果记性操作
例如，我们想获取一个数组中字符串长度大于3的字符串，正常的写法应该是这样的
```
val numbers = arrayListOf("one","two","three","four","five")
val resultList = numbers.map { 
    it.length
 }.filter {
     it > 3
 }
 print(resultList)
```
如果想用let操作符进行简化，可以这样写
```
val numbers = arrayListOf("one","two","three","four","five")
numbers.map{
    it.length
}.filter {
    it > 3
}?.let{
    print(it)
}
```
使用let之后，可以直接对数组列表中长度大于3的字符串进行打印，去掉变量赋值

4.let可以重命名it为一个刻度的lambda参数
```
val book = Book().let{ book -> 
    book.name = "《最后的魁拔》"
}
```
let通过使用关键字it来引用对象的上下文，不过也可以重命名为一个可读的lambda参数



#### run

```
public inline fun <T, R> T.run(block: T.() -> R): R
```
run函数比较简单，以this作为上下文对象，且他的调用方式与let一致

特点：

1.当lambda表达式同时包含对象初始化和返回值的计算时，使用run更合适
简单来说，在run方法中，我们可以对当前对象同时执行多个操作，如下所示
```
Book().run{
    name = "魁拔之书"
    price = 35
    displayInfo()
}
```
控制台输出结果是
```
书 魁拔之书 ,价格是 35
```
我们发现即使我们没有手动执行print方法，但是因为displayInfo方法本身就有打印功能，所以最终还是把信息打印了出来。这里我们可以认为，在run代码块中，每一行代码都会被执行到，就算是一个方法的调用，也会被执行到。
如果不适用run函数，则需要这样操作
```
val book = Book()
book.name = "《魁拔之书》"
book.price = 35
book.displayInfo()
```
2.除了上面给出的声明方式外，我们还可以写成这样
```
run {
    val book = Book()
    book.name = "《魁拔之书》"
    book.price = 35
    book.displayInfo()
}
```
当时这样的声明时，使用的声明方式是如下所示
```
public inline fun <R> run(block: () -> R): R
```


#### with
```
public inline fun <T,R> with (receiver: T, block:T.() -> R):R
```
with函数属于非扩展函数，直接输入一个对象receiver，当输入receiver后，便可以改变receiver的属性，同时也与run做了同样的事情
```
val book = Book()
with(book){
    name = "魁拔之书"
    price = 35
}
```
这里我们使用with时传入了一个参数book，可以直接在with代码块中访问book中的属性和方法。

> with使用的是非null对象，当函数块中不需要返回值时，可以使用with



#### apply
```
public inline fun <T> T.apply(block: T() -> Unit): T
```
apply是T的扩展函数，这点和run有点类似，它将对象的上下文引用为this而不是it，并且提供了空安全检查，不同的是，apply不接受函数块中的返回值，返回值是自己的T类型对象
```
val book = Book().apply{
    name = "《魁拔之书》"
    price = 33
}
print(book)
```
此处打印的结果 book是一个第一项，返回内存地址。其实我们不难发现，let，with,run函数返回值都是R，而apply和also返回值是T。比如在let中，没有在函数块中返回具体的值，最终会成为Unit类型。而apply中，最后返回对象本身T时，他就是Book类型


#### also
```
public inline fun <T> T.alse(block: (T) -> Unit): T
```
alse是T的扩展函数，返回值与apply一致，直接返回T。alse函数的用法类似于let函数，将对象的上下文引用为it，并提供空安全检查
```
val book = Book().also{
    it.name = "《魁拔之书》"
    it.price = 32
}
print(book)
```
控制台最终输出的结果是book的内存地址

### let,run,with,apply,alse的使用场景
简单罗列了几个常见的使用场景
- 初始化对象/更改对象属性，可使用apply
- 如果将数据指派给接收对象的属性之前需要验证对象，使用also
- 如果将对象进行空检查并访问或修改其属性，使用let
- 如果是非null的对象并且函数块中不需要返回值，可以使用with
- 如果想计算某个值，或者限制多个本地变量的范围，使用run

```
// 显示弹窗时，保证context上下文对象不能为null(空检查)
context?.let {
    val teacherInfoDialog = AICourseTeacherInfoDialog(it, t.data)
    teacherInfoDialog.show()
}

// 修改对象的属性
UserManager.user?.apply {
    this.grade = grade
    this.edition = edition
}

// 计算某个值
val user = User("Kotlin", 1, "1111111")
val result = user.run {
    println("my name is $name, I am $age years old, my phone number is $phoneNum")
    1000
}
```

# 参考资料
[(kotlin篇)差异化分析，let，run，with，apply及also](https://juejin.cn/post/6975384870675546126?share_token=d2308375-e0e4-43b8-8679-8b7a8e1155c0)