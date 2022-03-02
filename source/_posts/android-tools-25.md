---
title: android奇技淫巧 25 Kotlin和Lambda表达式 使用技巧
date: 2021-09-25 21:38:10
tags:
  - android
---
Kotlin，lambda表达式 在使用中的一些常见问题和解决方法

<!--more-->

### 尽量少使用toLowerCase和toUpperCase方法
当我们比较两个字符串，需要忽略大小写的时候，通常会调用**toLowerCase**方法或者**toUpperCase**方法转换成大写或小写，然后再进行比较，但是这样的话有一个不好的地方，每次调用**toLowerCase**方法或者**toUpperCase**方法的时候，会创建一个新的字符串，然后再进行比较
```
fun main(args:Array<String>){
  val oldName = "paulniu"
  val newName = "PaulNiu"
  val result = oldName.toLowerCase() == newName.toLowerCase()
}
```
**toLowerCase**方法编译之后的Java代码
![编译之后的java代码](/assets/tools/tools-lambda-01.png)

如上图所示首先会生成一个新的字符串，然后再进行字符串的比较

如果我们想要忽略大小写比较字符串内容，可以通过下面的方式
```
fun main(args:Array<String>){
  val oldName = "paulniu"
  val newName = "PaulNiu"
  val result = oldName.equals(newName,ignoreCase = true)
}
```
这样equals编译之后的java代码如下所示
![equals编译之后的java代码](/assets/tools/tools-lambda-02.png)

### Kotlin处理空字符串
当字符串为空字符串的时候，返回一个默认值，比如下面的写法
```
val target = ""
val name = if (target.isEmpty()) "npl" else target
```
其实我们可以通过另外一种方法
```
val name = target.ifEmpty{ "npl" }
```
这个地方的原理跟我们使用**if**表达式是一样的，源码内容如下
```
public inline fun <C, R> C.ifEmpty(defaultValue: () -> R): R where C : CharSequence, C : R =
    if (isEmpty()) defaultValue() else this
```
**ifEmpty**方法是一个扩展方法，接受一个lambda表达式**defaultValue**，如果是空字符串，则返回**defaultValue**，否则不为空，返回调用者本身
处理**ifEmpty**方法之后，kotlin库中还封装了很多其他有用的方法，如将字符串转换为数字
```
val input = "123"
val number = input.toIntOrNull()?:0
```

### 避免将解构声明和数据类一起使用
这个是Kotlin团队的一个建议：避免将解构声明和数据类一起使用，如果以后王数据类添加新的属性，很容易破坏戴梦得解构，我们看一个例子
```
// 数据类
data class People(
  val name:String,
  val city:String
)
fun main(args:Array<String>){
  printlnPeople(People("paulniu","beiJing"))
}
fun printlnPeople(people:People){
  // 解构声明，获取name和city
  val (name,city) = people
  println("name = ${name}")
  println("city = ${city}")
}
```
输出结果类似于：
```
name = paulniu
city = beiJing
```
这时我们想要给People类新增一个字段age，那么我们需要修改People数据类
```
data class People(
  val name:String,
  val age:Int,
  val city:String
)
fun main(args:Array<String>){
  printlnPeople(People("paulniu",18,"beiJing"))
}
```
此时没有更改解构声明，也不会有任何错误，但是输出结果肯定不是我们想要的
数据结果可能类似于：
```
name = paulniu
city = 18
```
因此这时我们需要修改解构声明，但如果代码中有多次解构声明，而我们只是增加了一个新属性，就有可能需要改动所有使用解构声明的地方。所以这种情况是非常不合理的，我们一定要避免将解构声明和数据类一起使用，当我们使用不规范的时候编译器也会给出警告


### 文件的扩展方法
Kotlin提供了很多文件扩展方法：**forEachLine**,**readLines**,**readText**,**useLines**等，帮助我们简化文件的操作，而且完成使用之后，他们会自动关闭，如下
```
File("dhl.txt").useLines { line ->
    println(line)
}
```
**userLines**是File的扩展方法，调用**useLines**会返回一个we你按中所有行的Sequence，当文件内容读取完毕之后，他会自动关闭
```
public inline fun <T> File.useLines(charset: Charset = Charsets.UTF_8, block: (Sequence<String>) -> T): T =
    bufferedReader(charset).use { block(it.lineSequence()) }
```
- useLines 是File的一个扩展方法
- useLines 接受一个Lambda表达式的block
- 调用了BufferedReader读取文件内容，之后调用block返回文件中所有行的Sequence给调用者

**useLines**方法内部调用了**use**方法，**use**方法也是一个扩展方法，源码如下
```
public inline fun <T : Closeable?, R> T.use(block: (T) -> R): R {
    var exception: Throwable? = null
    try {
        return block(this)
    } catch (e: Throwable) {
        exception = e
        throw e
    } finally {
        when {
            apiVersionIsAtLeast(1, 1, 0) -> this.closeFinally(exception)
            this == null -> {}
            exception == null -> close()
            else ->
                try {
                    close()
                } catch (closeException: Throwable) {
                    // cause.addSuppressed(closeException) // ignored here
                }
        }
    }
}
```
其实就是调用了**try...catch...finally**最后在finally内部进行close

### joinToString方法的使用
**joinToString**方法提供了一组丰富的可选项(分隔符，前缀，后缀，数量限制等)可用于将可迭代对象转换为字符串
```
val data = listOf("Java", "Kotlin", "C++", "Python")
        .joinToString(
                separator = " | ",
                prefix = "{",
                postfix = "}"
        ) {
            it.toUpperCase()
        }

println(data) // {JAVA | KOTLIN | C++ | PYTHON}
```
这是种很常见的方法，将集合转换成字符串，高效利用便捷的**joinToString**方法，开发的时候能够事半功倍。同时Kotlin库函数提供了一些删除的方法，如下所示
```
var data = "**hi dhl**"

// 移除前缀
println(data.removePrefix("**")) //  hi dhl**
// 移除后缀
println(data.removeSuffix("**")) //  **hi dhl
// 移除前缀和后缀
println(data.removeSurrounding("**")) // hi dhl

// 返回第一次出现分隔符后的字符串
println(data.substringAfter("**")) // hi dhl**
// 如果没有找到，返回原始字符串
println(data.substringAfter("--")) // **hi dhl**
// 如果没有找到，返回默认字符串 "no match"
println(data.substringAfter("--","no match")) // no match

data = "{JAVA | KOTLIN | C++ | PYTHON}"

// 移除前缀和后缀
println(data.removeSurrounding("{", "}")) // JAVA | KOTLIN | C++ | PYTHON
```
有了这些Kotlin库函数，我们不需要再做**startsWith**和**endsWith**的检查，如果让我们自己实现，可能需要花一些功夫实现，但是源码实现的就比较有特点，我们可以学习一下
```
public String substring(int beginIndex, int endIndex) {
    if (beginIndex < 0) {
        throw new StringIndexOutOfBoundsException(beginIndex);
    }
    if (endIndex > value.length) {
        throw new StringIndexOutOfBoundsException(endIndex);
    }
    int subLen = endIndex - beginIndex;
    if (subLen < 0) {
        throw new StringIndexOutOfBoundsException(subLen);
    }
    return ((beginIndex == 0) && (endIndex == value.length)) ? this
            : new String(value, beginIndex, subLen);
}
```

### plus操作符
我们在Java中的算术运算符只能应用于基本类型，+号运算符可以和String值一起使用，但是不能在集合中使用，在Kotlin中可以应用在任何类型。如下所示，利用plus和minus对map集合进行运算
```
fun main() {
    val numbersMap = mapOf("one" to 1, "two" to 2, "three" to 3)

    // plus (+)
    println(numbersMap + Pair("four", 4)) // {one=1, two=2, three=3, four=4}
    println(numbersMap + Pair("one", 10)) // {one=10, two=2, three=3}
    println(numbersMap + Pair("five", 5) + Pair("one", 11)) // {one=11, two=2, three=3, five=5}

    // minus (-)
    println(numbersMap - "one") // {two=2, three=3}
    println(numbersMap - listOf("two", "four")) // {one=1, three=3}
}
```
其实这里用到了运算符的重载，Kotlin在Maps.kt文件中，定义了一系列用关键字**operator**声明的Map的扩展函数
用**operator**关键字声明plus函数，可以直接使用+号来做运算，使用**operator**修饰符声明minus函数，可以直接使用-号来做运算
```
data class Salary(var base: Int = 100){
    override fun toString(): String = base.toString()
}

operator fun Salary.plus(other: Salary): Salary = Salary(base + other.base)
operator fun Salary.minus(other: Salary): Salary = Salary(base - other.base)

val s1 = Salary(10)
val s2 = Salary(20)
println(s1 + s2) // 30
println(s1 - s2) // -10
```

### Map集合的默认值
在Map集合中，可以使用withDefault设置一个默认值，当键不在Map集合中，通过getValue返回默认值
```
val map = mapOf(
        "java" to 1,
        "kotlin" to 2,
        "python" to 3
).withDefault { "?" }

println(map.getValue("java")) // 1
println(map.getValue("kotlin")) // 2
println(map.getValue("c++")) // ?
```
这里我们使用**withDefault**操作符，设置当Map中key对应的值为null的时候，可以返回一个null，源码如下：
```
internal inline fun <K, V> Map<K, V>.getOrElseNullable(key: K, defaultValue: () -> V): V {
    val value = get(key)
    if (value == null && !containsKey(key)) {
        return defaultValue()
    } else {
        @Suppress("UNCHECKED_CAST")
        return value as V
    }
}
```