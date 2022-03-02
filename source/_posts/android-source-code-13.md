---
title: 源码分析(十三) LinkedList
date: 2021-09-06 20:09:13
tags:
 - 源码分析
---

LinkedList

<!--more-->

LinkedList是同时实现了List接口和Deque接口，也就说它既可以看做是一个顺序容器，也可以看做是一个队列，同时也可以看做是一个栈

```
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
{
    //长度
    transient int size = 0;
    //指向头结点
    transient Node<E> first;
    //指向尾结点
    transient Node<E> last;
}
```

这样看来LinkedList几乎无敌，当需要使用栈或队列时，可以考虑用LinkedList，一方面是因为Java官方已经声明了不建议使用Stack类，而且Java中根本就没有衣蛾叫做Queue的类(只有一个接口的名字)

关于栈或队列 首选是ArrayDeque，他有比LinkedList(当做队列或者栈使用时)更好的性能，LinkedList的实现方式决定了所有跟下面图片相关的操作都是线性时间，而在首段或者末未删除元素还需要常数的时间。为了追求更好的效率，LinkedList没有实现同步(synchronized)，如果需要多个线程并发访问，可以先采用**Collection.synchronizedList**方法对齐进行包装

![集合种类](/assets/collection/linkedlist_01.png)

基本属性
- transient int size = 0 // LinkedList中存放的元素个数
- transient Node<E> first // 头结点
- transient Node<E> last // 尾结点
- Collection接口：Collection接口是所有集合类的根节点，表示一种规则

继承的类和实现的接口
- List接口：List是Collection的子接口，他是一个元素有序(按照插入的顺序维护元素顺序),可重复，可以为null的集合
- AbstractCollection类： Collection接口的骨架实现类，最小化实现了Collection接口所需要实现的工作量
- AbstractList类：List接口的骨架实现类，最小化实现了List接口所需要实现的工作量
- Cloneable接口：实现了该接口的类可以显示的调用**Object.Clone**方法，合法的对该对象实例进行字段复制，如果没有实现Cloneable接口的实例上调用了**Object.clone**方法，会抛出**CloneNotSupportException**异常，正常情况下，实现了Cloneable接口的类会以公共方法重写**Object.clone**方法
- Deque接口：定义了一个线性的Collection，支持在两端插入和删除元素，Deque实际上就是双端队列的简称，大多数Deque接口的实现都不会限制元素的数量，但是这个队列支持有容量限制的实现，比如LinkedList就是有容量限制的实现，其最大容量是Interger.MAX_VALUE
- Serialize接口：实现了该接口表示类可以被序列化
- AbstractSequentialList类：提供了List接口的主要实现，最大限度的减少了实现受"连续访问"数据存储(如链表)支持的此接口所需要的工作，对于随机访问数据(如数组),应该优先使用AbstractList

底层源码分析
![LinkedList底层实现示意图](/assets/collection/linkedlist_02.png)

此处省略了LinkedList的源码，可以自己通过AndroidStudio查看

列出几个比较重要的方法

构造方法
```
LinkedList()
LinkedList(Collection<? extends E>c)
```
LinkedList没有长度的概念，所以不存在容量不足的问题，因此不需要提供大量初始化大小的构造方法，因此只提供了两个，一个是无参构造，出示一个LinkedList对象，和将制定元素转化为LinkedList的构造方法


```
// 插入头节点
private void linkFirst(E e) {
        final Node<E> f = first;  //将头节点赋值给f节点
        //new 一个新的节点，此节点的data = e , pre = null , next - > f 
        final Node<E> newNode = new Node<>(null, e, f);
        first = newNode; //将新创建的节点地址复制给first
        if (f == null)  //f == null，表示此时LinkedList为空
            last = newNode;  //将新创建的节点赋值给last
        else
            f.prev = newNode;  //否则f.前驱指向newNode
        size++;
        modCount++;
}
```

在succ节点前插入e节点，并修改各个节点之间的前驱后继
```
private void linkBefore(E e, Node<E> succ) {
        // assert succ != null;
        final Node<E> pred = succ.prev;
        final Node<E> newNode = new Node<>(pred, e, succ);
        succ.prev = newNode;
        if (pred == null)
            first = newNode;
        else
            pred.next = newNode;
        size++;
        modCount++;
    }
```

获取LinkedList中的第一个节点信息
```
public E getFirst(){
    final Node<E> f = first;
    if(f == null){
        throw new NoSuchElementException();
    }
    return f.item;
}
```

添加节点
```
publick boolean add(E e){
    linkLast(e);
    return true;
}
//插入尾节点
private void linkLast(E e) {
        final Node<E> l = last; 
        final Node<E> newNode = new Node<>(l, e, null);
        last = newNode;
        if (l == null)
            first = newNode;
        else
            l.next = newNode;
        size++;
        modCount++;
}
```
添加节点的过程如下：
- 记录当前末尾结点，通过构造另外一个指向末尾结点的指针l
- 产生新的节点:注意由于是添加到链表的末尾，next为null
- last指向新节点
- 判断是判断是否为第一个元素(当l为null时，则表示链表中没有节点)
- 如果是第一个节点，则使用first指向这个节点，若不是则当前节点的next指向新增的节点
- size增加

![添加节点示意图](/assets/collection/linkedlist_03.png)

删除节点
两种方法删除节点
```
// 方法一 删除制定索引上的节点
public E remove(int index){
    // 检查索引是否正确
    checkElementIndex(index);
    return unlink(node(index));
}
// 方法二 删除制定值的节点
public boolean remove(Object o){
    // 判断删除的元素是否为null
    if(o == null){
        // 如果是null，则遍历所有的数据 执行删除操作
        for(Node<E> x = first;x != null;x = x.next){
            if(x.item == null){
                unlink(x);
                return true;
            }
        }
    }else{
        // 如果不是空，则遍历所有的数据
        for(Node<E> x = first; x != null; x = x.next){
            if(o.equals(x.item)){
                unlink(x);
                return true;
            }
        }
    }
    retrun false;
}
// 删除指定节点
public E unlink(Node<E> x){
    final E element = x.item;
    final Node<E> next = x.next;
    final Node<E> prev = x.prev;
    if(prev == null){
        first = next;
    }else{
        prev.next = next;
        x.prev = null;
    }
    if(next == null){
        last = prev;
    }else{
        next.prev = prev;
        x.next = null;
    }
    x.item = null;
    size--;
    modCount++;
    return element;
}
```

从源码中可以看出
- 获取到需要删除元素当前的值，指向他前一个节点的引用，以及指向它后一个节点的引用
- 判断删除的是否是第一个节点，如果是，则first向后移动，若不是，则将当前节点的前一个节点的next指向当前节点的后一个节点
- 判断删除的是否是最后一个节点，如果是，则last向前移动，若不是，则将当前节点的后一个节点的prev指向当前节点的前一个节点
- 当前节点的值为null
- size减少并返回删除节点的值

关于ArrayList和LinkedList的区别
1. 底层实现:ArrayList内部是数组实现，而LinkedList内部实现是双向链表结构；
2. 接口实现：都实现了List接口，都是线性列表的实现，ArrayList实现了RandomAccess可以支持随机元素访问，而LinkedList实现了Deque可以当做队列使用；
3. 性能：新增、删除元素时ArrayList需要使用到拷贝原数组，而LinkedList只需移动指针，查找元素 ArrayList支持随机元素访问,而LinkedList只能一个节点的去遍历；
4. 线程安全：都是线程不安全的；
5. LinkedList下插入、删除是性能优于ArrayList，这是由于插入、删除元素时ArrayList中需要额外的开销去移动、拷贝元素(但是使用removeElements2方法所示去遍历删除是速度异常的快，这种方式的做法是从末尾开始删除，不存在移动、拷贝元素，从而速度非常快)；
6. ArrayList在查询元素的性能上要由于LinkedList；
