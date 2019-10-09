---
title: JavaEE 02 Java语言基础
date: 2019-09-29 12:37:03
tags:
  - JavaEE
---

是不是以为我会长篇累牍的描述Java语言的各种特性，语法知识点等内容？想多了！Java基础那么多一个个的写，等我写完，假期也就结束了，所以我更愿意将这部分内容改变成Java语法基础知识的查漏补缺。如果Java基础不好的同学请移步到[菜鸟教程](https://www.runoob.com/java/java-tutorial.html)。这个网站真的很好，各种学习资料都是免费的。

<!--more-->

总之就是查找自己有哪些知识点是掌握不牢固的，有哪些内容是已经忘记了的

# 数据结构
java工具包提供了强大的数据结构，在java中数据结构主要包括以下几种接口和类

1. 枚举(Enum)
2. 位集合(BitSet)
3. 向量(Vector)
4. 栈(Stack)
5. 字典(Dictionary)
6. 哈希表(Hashtable)
7. 属性(Properties)

以上是传统的数据结构，在Java2中引入了一种新的框架--集合框架(Collection)

## 枚举
枚举结构虽然本身不属于数据结构，但它在其他数据结构的范畴中应用很广，枚举结构定义了一种从数据结构中取回连续元素的方式。

枚举接口中定义了一些方法，通过这些方法可以枚举(一次获得一个)对象集合中的元素
这种传统的接口已经被迭代器取代。虽然在现在代码中很少使用，但是还是使用在Vector和Properties中，除此之外，还有一些API类中也在使用

|方法|描述|
|---|---|
|boolean hasMoreElements|测试次枚举是否包含更多元素|
|Object nextElement()|如果枚举对象至少还有一个可提供的元素，则返回才枚举的下一个元素|

```
Enmueration<String> days;
Vector<String> dayNames = new Vector<String>();
dayNames.add("Sunday");
dayNames.add("Monday");
dayNames.add("Tuesday");
dayNames.add("Wednesday");
dayNames.add("Thursday");
dayNames.add("Friday");
dayNames.add("Saturday");
days = dayNames.elements();
while(days.hasMoreElements()){
  System.out.print(day.nextElement());
}
```

## 位集合
位集合类实现了一组可以单独设置和清除的位或标志
该类在处理一组布尔值的时候非常有用，只需要给每个值赋值一个"位"，然后对位进行适当的设置或清除，就可以对布尔值进行操作了。
BitSet定义了两个构造方法：BitSet()和BitSet(int size)

BitSet中实现了Cloneable接口中定义的方法，如下所示

|方法|描述|
|---|---|
|void and(BitSet set)|对此目标位set和参数位set执行逻辑与操作|
|void andNot(BitSet set)|清除此BitSet中所有的位，其相应的位在置顶的BitSet中已设置|
|int cardinality()|返回此BitSet中设置为true的位数|
|void clear()|将此BitSet中所有位设置为false|
|void clear(int index)|将索引指定处的位置设置为false|
|Object Clone()|复制此BitSet生成一个与之相等的新BitSet|
|boolean equals(Object bitSet)|将此对象与指定的对象进行比较|
|void flip(int index)|将指定索引出的位设置为当前值的补码|
|void flip(int startIndex,int endIndex)|将制定的fromIndex(包括)到制定的toIndex(不包括)范围内的每一个位设置为当前值的补码|
|boolean get(int index)|返回指定索引处的位值|
|BitSet get(int startIndex,int endIndex)|返回一个新的BitSet，它由此BitSet中从fromIndex(包括)到toIndex(不包括)范围内的位组成|
|int hashCode()|返回此位set的哈希码值|
|Boolean intersects(BitSet bitSet)|如果指定的BitSet中有设置为true的位，则返回true|
|boolean isEmpty()|如果此BitSet中没有包含任何设置为ture的位，返回true|
|int length()|返回此BitSet的逻辑大小，BitSet中最高设置位的索引加1|
|int nextClearBit(int startIndex)|返回第一个设置为false的位的索引，这发生在指定的其实索引或之后的索引上|
|int nextSetBit(int startIndex)|返回第一个设置为true的位的索引，这发生在置顶的其实索引或之后的索引上|
|void or(BitSet bitSet)|对此为set和为set参数执行逻辑或操作|
|void set(int index)|将指定索引处的位置设置为true|
|void set(int index,boolean v)|将制定索引处的位设置为指定的值|
|void set(int startIndex,int endIndex)|将指定的fromIndex(包括)到指定的toIndex(不包括)范围内的位设置为true|
|void set(int startIndex,int endIndex,boolean v)|将制定的fromIndex(包括)到指定的toIndex(不包括)范围内的位置设置为指定的值|
|int size()|返回此位set的字符串表示形式|
|String toString()|返回此位set的字符串表示形式|
|void xor(BitSet bitSet)|对此位set和位set参数执行逻辑异或操作|


```
BitSet bits1 = new BitSet(16);
     BitSet bits2 = new BitSet(16);
      
     // set some bits
     for(int i=0; i<16; i++) {
        if((i%2) == 0) bits1.set(i);
        if((i%5) != 0) bits2.set(i);
     }
     System.out.println("Initial pattern in bits1: ");
     System.out.println(bits1);
     System.out.println("\nInitial pattern in bits2: ");
     System.out.println(bits2);
 
     // AND bits
     bits2.and(bits1);
     System.out.println("\nbits2 AND bits1: ");
     System.out.println(bits2);
 
     // OR bits
     bits2.or(bits1);
     System.out.println("\nbits2 OR bits1: ");
     System.out.println(bits2);
 
     // XOR bits
     bits2.xor(bits1);
     System.out.println("\nbits2 XOR bits1: ");
     System.out.println(bits2);
```

## 向量
Vector类实现了一个动态数组，和ArrayList很相似，但两者有不同

- Vector是同步访问的
- Vector包含了很多传统方法，这些方法不属于集合框架

Vector主要用在实现不知道数组的大小，或者只是需要一个可以改变大小的数组的情况
Vector有四个构造方法：Vector(),Vector(int size),Vector(int size,int incr),Vector(Collection c)
其中Vector(int size)是创建指定大小的向量，Vector(int size,int incr)创建指定大小的向量并且增量用incr指定。增量表示向量每次增加的元素数目

Vector常用方法

|方法|描述|
|---|---|
|void add(int index,Object element)|在此向量的指定位置插入指定的元素|
|boolean add(Object o)|将指定元素添加到向量的末尾|
|boolean addAll(Collection c)|将制定Collection中的所有元素添加到此向量的末尾，按照指定collection的迭代器所返回的顺序添加这些元素|
|boolean addAll(int index,Collection c)|在置顶位置将制定Collection中的所有元素插入到向量中|
|void addElement(Object o)|将制定的组件添加到此向量的末尾，将其大小增加1|
|int capacity()|返回此向量的当前容量|
|void clear()|从此向量中移除所有元素|
|Object clone()|返回向量的一个副本|
|boolean contains(Object elem)|如果此向量包含指定的元素，则返回true|
|boolean containsAll(Collection c)|如果此向量包含指定Collection中的所有元素，返回true|
|void copyInto(Object[] anArray)|将此向量的组件复制到指定的数组中|
|Object elementAt(int index)|返回指定索引处的组件|
|Enumeration elements()|返回此向量的组件的枚举|
|void ensureCapacity(int minCapacity)|增加此向量的容量，以确保其至少能够保存最小容量参数指定的组件数|
|boolean equals(Object o)|比较置顶对象与此向量的相等性|
|Object firstElement()|返回此向量的第一个组件(下标从0开始)|
|Object get(int index)|返回向量中指定位置的元素|
|int hashCode()|返回此向量的哈希码值|
|int indexOf(Object elem)|返回词向量中第一次出现的指定元素的索引，如果此向量不包含该元素，返回-1|
|int indexOf(Obect element,int index)|返回此向量中第一次出现指定元素的索引，从index出正向搜索，如果未找到该元素，则返回-1|
|void insertElementAt(Object obj,int index)|将制定对象作为词向量中的组件插入到指定的index处|
|boolean isEmpty()|测试词向量是否不包含组件|
|Object lastElement()|返回此向量的最后一个组件|
|int lastIndexOf(Object elem)|返回此向量中最后一次出现的指定元素的索引，如果不包含该元素，返回-1|
|int lastIndexOf(Object elem,int index)|返回词向量中最后一次出现的指定元素的索引，从index处逆向搜索，如果未找到该元素，返回-1|
|Object remove(index)|移除此向量中置顶位置的元素|
|boolean remove(Object o)|移除此向量中指定元素的第一个匹配项，如果向量不包含该元素，则元素保持不变|
|boolean removeAll(Collection c)|从词向量中移除包含在指定Collection中全部元素|
|void removeAllElements()|从此向量中移除全部组件，并将其大小设置为0|
|boolean removeElement(Object o)|从此向量中移除变量的第一个(索引最小的)匹配项|
|void removeElementAt(int index)|删除指定索引处的组件|
|protected void removeRange(int formIndex,int toIndex)|从此List中移除索引位于formIndex(包括)与toIndex(不包括)之间的所有元素|
|boolean retainAll(Collection c)|在此向量中仅包括包含指定collection中的元素|
|Object set(int index,Object element)|用指定的元素替换此向量中指定位置处的元素|
|void setElementAt(Object obj,int index)|向此向量指定index处的组件设置为指定的对象|
|void setSize(int newSize)|设置此向量的大小|
|List subList(int fromIndex,int toIndex)|返回此List的部分视图，从fromIndex(包括)至toIndex(不包括)|
|Object[] toArray()|返回一个数组，包含此向量中以前当顺序存放的所有元素|
|Object[] toArray(Object[] a)|返回一个数组，包含此向量中以恰当顺序存放的所有元素，返回数组的运行时类型为置顶数组的类型|
|String toString()| 返回此向量的字符串表示形式，其中包含每个元素的String表示形式|
|void trimToSize()|对此向量的容量进行微调，使其等于向量的当前大小|

```
Vector v = new Vector(3,2);
System.out.print("初始化大小是"+v.size());
System.out.print("初始化Capacity"+v.capacity());
v.addElement(new Integer(1));
v.addElement(new Integer(2));
v.addElement(new Integer(3));
v.addElement(new Integer(4));
System.out.println("当前的capacity"+v.capacity());
v.addElement(new Double(5.45));
System.out.println("当前capacity"+v.capacity());
v,addElement(new Double(6.08));
v.addElement(new Integer(7));
System.out.println("Current capacity: " +
v.capacity());
v.addElement(new Float(9.4));
v.addElement(new Integer(10));
System.out.println("Current capacity: " +
v.capacity());
v.addElement(new Integer(11));
v.addElement(new Integer(12));
System.out.println("First element: " +
  (Integer)v.firstElement());
System.out.println("Last element: " +
  (Integer)v.lastElement());
if(v.contains(new Integer(3)))
    System.out.println("Vector contains 3.");
    // enumerate the elements in the vector.
    Enumeration vEnum = v.elements();
    System.out.println("\nElements in vector:");
    while(vEnum.hasMoreElements())
        System.out.print(vEnum.nextElement() + " ");
    System.out.println();
```

## 栈
栈是Vector的一个子类，他实现了一个标准的后进先出的栈
堆栈之定义了默认的构造函数，用来创建一个空栈，堆栈除了包括有Vector定义的所有方法，也定义了自己的一些方法

|方法|描述|
|---|---|
|boolean empty()|测试堆栈是否为空|
|Object peek()|查看堆栈顶部的对象，但不从堆栈中移除他|
|Object pop()|移除堆栈顶部的对象，并作为此函数的值返回该对象|
|Object push(Object element)|把项压入堆栈顶部|
|int search(Object element)|返回对象在堆栈中的位置，以1为基数|

```
public class StackDemo {
 
    static void showpush(Stack<Integer> st, int a) {
        st.push(new Integer(a));
        System.out.println("push(" + a + ")");
        System.out.println("stack: " + st);
    }
 
    static void showpop(Stack<Integer> st) {
        System.out.print("pop -> ");
        Integer a = (Integer) st.pop();
        System.out.println(a);
        System.out.println("stack: " + st);
    }
 
    public static void main(String args[]) {
        Stack<Integer> st = new Stack<Integer>();
        System.out.println("stack: " + st);
        showpush(st, 42);
        showpush(st, 66);
        showpush(st, 99);
        showpop(st);
        showpop(st);
        showpop(st);
        try {
            showpop(st);
        } catch (EmptyStackException e) {
            System.out.println("empty stack");
        }
    }
}
```

## 字典
Dictionary类是一个抽象类，用来存储键值对，作用和Map类似。
给出键和值，我们可以将只存储早字典对象中，一旦值被存储，就可以通过它的键来获取

字典常用方法

|方法|描述|
|---|---|
|Enumeration elements()|返回此字典中值的枚举|
|Object get(Object key)|返回此字典中该键所映射到的值|
|boolean isEmpty()|测试次字典是否不存在从键到值的映射|
|Enumeration keys()|返回此字典中键的枚举|
|Object put(Object key,Object value)|将制定key映射到此字典中指定的value|
|Object remove(Object key)|从此字典中移除key(以及相应的value)|
|int size()|返回此字典中条目(不同键)的数量|

> 字典Dictionary类已经过时了，在实际开发中使用欧冠Map集合来代替

# 泛型
Java泛型是JDK5中引入的新特性，泛型提供了编译时类型安全检测机制，该机制允许程序员在编译时检测到非法的类型
泛型的本质是参数化类型，也就是说操作的数据类型被指定为一个参数

## 泛型方法
我们可以写一个泛型方法，该方法在调用时可以接收不同类型的参数，根据传递给泛型方法的参数类型，编译器适当的处理每一个方法调用
定义泛型方法的规则
1. 所有泛型方法的声明都有一个类型参数声明部分(由尖括号分割)，该类型参数声明部分在方法返回类型之前
2. 每一个类型参数声明部分包含一个或多个类型参数，参数间用逗号隔开。一个泛型参数也被称为一个类型变量，适用于制定一个泛型类型名称的标识符
3. 类型参数能被用来声明返回值类型，并且能作为泛型方法得到的实际参数类型的占位符。
4. 泛型方法体声明和其他方法一样，注意类型参数只能代表引用型类型，不能是原始类型

```
public class GenericMethodTest{
  public static <E> void printArray(E[] intputArray){
    // 输入数组元素
    for(E element:inputArray){
      System.out.println("%s",element);
    }
    System.out.println();
  }

  public static void main(String[] args){
    // 创建不同类型数组： Integer, Double 和 Character
        Integer[] intArray = { 1, 2, 3, 4, 5 };
        Double[] doubleArray = { 1.1, 2.2, 3.3, 4.4 };
        Character[] charArray = { 'H', 'E', 'L', 'L', 'O' };
 
        System.out.println( "整型数组元素为:" );
        printArray( intArray  ); // 传递一个整型数组
 
        System.out.println( "\n双精度型数组元素为:" );
        printArray( doubleArray ); // 传递一个双精度型数组
 
        System.out.println( "\n字符型数组元素为:" );
        printArray( charArray ); // 传递一个字符型数组
  }
}
```
有界类型参数：
有时候我们想要限制那些被允许传到一个类型参数的参数类型。例如，我们只想接受一个传递的是Number类型或者是Number类型的子类，那么我们就可以约束成有界参数，实现我们想要的功能：
```
public class MaximumTest{
  // 比较三个值，然后返回最大值
  public static <T extends Comparable<T>> T maximum(T x,T y,T z){
    T max = x;
    if(y.compareTo(max) > 0){
      // 说明y更大
      max = y；
    }
    if(z.compareTo(max) > 0){
      // 说明z更大
      max = z;
    }
    return max;
  }

  public static void main(String [] args){
    System.out.printf( "%d, %d 和 %d 中最大的数为 %d\n\n",
                   3, 4, 5, maximum( 3, 4, 5 ) );
 
      System.out.printf( "%.1f, %.1f 和 %.1f 中最大的数为 %.1f\n\n",
                   6.6, 8.8, 7.7, maximum( 6.6, 8.8, 7.7 ) );
 
      System.out.printf( "%s, %s 和 %s 中最大的数为 %s\n","pear",
         "apple", "orange", maximum( "pear", "apple", "orange" ) );
  }
}
```

## 泛型类
泛型类的声明和非泛型类的声明类似，除了在类名后面需要添加类型参数声明部分。
和泛型方法一样，泛型类的类型参数声明部分也包含了一个或多个类型参数，参数之间用逗号隔开。一个泛型参数也被称为类型变量，是一个用来指定泛型参数类型的标志符。因为他们接受一个或多个参数，这些类被称为参数化类或参数化类性

```
public class Box<T>{
  private T t;
  public void setValue(T t){
    this.t = t;
  }
  public T getValue(){
    return this.t;
  }
  public static void main(String [] args){
    Box<Integer> box = new Box<Integer>();
    Box<String> strBox = new Box<String>();

    box.setValue(new Integer(100));
    strBox.setValue("hello world");

  }
}
```

## 类型通配符
 
- 类型通配符通常使用?来代替具体的参数类型，例如List<?>

```
public class GenericTest {
     
    public static void main(String[] args) {
        List<String> name = new ArrayList<String>();
        List<Integer> age = new ArrayList<Integer>();
        List<Number> number = new ArrayList<Number>();
        
        name.add("icon");
        age.add(18);
        number.add(314);
 
        getData(name);
        getData(age);
        getData(number);
       
   }
 
   public static void getData(List<?> data) {
      System.out.println("data :" + data.get(0));
   }
}
```

> 因为getData方法的参数类型是list，所以name，age，number都可以作为方法的实参，这就是通配符的作用

-  类型通配符上限通过形如List来定义，如此定义就是通配符泛型类型接受Number以及子类类型

```
public class GenericTest {
     
    public static void main(String[] args) {
        List<String> name = new ArrayList<String>();
        List<Integer> age = new ArrayList<Integer>();
        List<Number> number = new ArrayList<Number>();
        
        name.add("icon");
        age.add(18);
        number.add(314);
 
        //getUperNumber(name);//1
        getUperNumber(age);//2
        getUperNumber(number);//3
       
   }
 
   public static void getData(List<?> data) {
      System.out.println("data :" + data.get(0));
   }
   
   public static void getUperNumber(List<? extends Number> data) {
          System.out.println("data :" + data.get(0));
       }
}
```

> 在(//1)处会出现错误，因为getUperNumber()方法中的参数已经限定了参数泛型上限为Number，所以泛型为String是不在这个范围之内，所以会报错

- 类型通配符下限通过形如List<? super Number>来定义，表示只接受Number以及其三层父类型

### 补充

- <? extends T>和<? super T>的区别
<? extends T>表示泛型只接受T类型或T类型的子类
<？ super T>表示泛型只接受T类型或T类型的父类

- 对于泛型，只是允许程序员在编译时检测到非法的类型而已。但是在运行期时，其中的泛型标志会变化为 Object 类型。
```
List<Integer> list = new ArrayList<>();

list.add(12);
//这里直接添加会报错
list.add("a");
Class<? extends List> clazz = list.getClass();
Method add = clazz.getDeclaredMethod("add", Object.class);
//但是通过反射添加，是可以的
add.invoke(list, "kl");

System.out.println(list)
```