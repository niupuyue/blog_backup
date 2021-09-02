---
title: 源码分析(一) ArrayList
date: 2020-03-02 22:09:13
tags:
 - 源码分析
---

ArrayList源码分析
<!--more-->

ArrayList在实际开发中使用到的机会特别多，但是如果我们对其底层实现机制不是很了解的话，可能会给我们的程序带来很多负面的影响。例如，当我们需要放在集合中的数据，面临大量的删除，增加等操作的时候，如果依然使用ArrayList，就会暂用大量的系统资源，因为ArrayList底层是通过数组实现的。

如上所说，ArrayList底层是通过数组实现的，我们可以通过源码查找到我们需要的内容。

ArrayList是在java.util包下面的类，继承自AbstractList类，而在这个类里面主要是帮我们声明了一些List集合需要使用到的方法，比如get，add方法等，AbstractList是一个抽象类，大部分的方法都是抽象方法。同时ArrayList还实现了Cloneable，Serialable接口，那就意味着我们的ArrayList可以执行浅复制/深复制，和序列化操作。

```
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
```

在ArrayList类里面，我们声明了几个变量，我们只说几个比较重要的

1. DEFAULT_CAPACITY

```
/**
 * Default initial capacity.
*/
private static final int DEFAULT_CAPACITY = 10;
```
这个变量表示我们当前ArrayList的初始大小是10个，因为ArrayList的底层是通过数组实现的，而数组在声明的时候的初始大小就是10个

2. EMPTY_ELEMENTDATA
```
    /**
     * Shared empty array instance used for empty instances.
     */
    private static final Object[] EMPTY_ELEMENTDATA = {};
```
这是一个数组，使我们声明的默认空数组，如果ArrayList在声明的时候没有指明ArrayList的大小，则默认讲这个数组指定为我们底层数组的赋值对象（在构造方法里我们可以看到）

3. elementData
```
    /**
     * The array buffer into which the elements of the ArrayList are stored.
     * The capacity of the ArrayList is the length of this array buffer. Any
     * empty ArrayList with elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA
     * will be expanded to DEFAULT_CAPACITY when the first element is added.
     */
    // Android-note: Also accessed from java.util.Collections
    transient Object[] elementData; // non-private to simplify nested class access
```
这个数组就是我们实际需要操作的底层数据，这里我们使用一个transient的关键字，他表示在我们执行序列化操作时，被transient修饰的变量不参与序列化的操作，具体可以参考这篇文章[transient关键字](https://blog.csdn.net/u010188178/article/details/83581506)

分析了这几个变量，我们来看一下构造方法，我们一般情况下使用ArrayList的方式无非就这么几种，如下
```
ArrayList<Object> objects = new ArrayList<>();
ArrayList<Object> objects = new ArrayList<>(5);
ArrayList<Object> objects = new ArrayList<>(Collection);
```

这三个方法分别对应着三个不同的构造方法，大致的操作都是相同的，如下所示
1. 无参构造
```
    /**
     * Constructs an empty list with an initial capacity of ten.
     */
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }
// 其中DEFAULTCAPACITY_EMPTY_ELEMENTDATA也是一个为空的数组，主要是为了在构造方法中区分如何初始化数组对象的
```

2. 传递个数的构造方法
```
    /**
     * Constructs an empty list with the specified initial capacity.
     *
     * @param  initialCapacity  the initial capacity of the list
     * @throws IllegalArgumentException if the specified initial capacity
     *         is negative
     */
    public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }

```
这里我们发现，在执行构造方法时，如果传递的数量大于0，则我们创建一个Object类型的数组，长度就是传递过来的数值，如果传递的大小为0，则将空的数组赋值给elementData对象

3. 传递一个Collection集合的构造方法
```
    /**
     * Constructs a list containing the elements of the specified
     * collection, in the order they are returned by the collection's
     * iterator.
     *
     * @param c the collection whose elements are to be placed into this list
     * @throws NullPointerException if the specified collection is null
     */
    public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray();
        if ((size = elementData.length) != 0) {
            // c.toArray might (incorrectly) not return Object[] (see 6260652)
            if (elementData.getClass() != Object[].class)
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // replace with empty array.
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }
```
在这个构造方法里面，我们抽象需要将当前的传递的Collection转换成byte()数组，如果数组长度为0，则初始化一个空的elementData，如果数组的长度不为空，则执行Arrays中的copyof方法，这个方法会返回给当前的elementData数组。
> 注意，这里需要进行类型的判定，可能数据会出错，注释中也给出了解答

在初始化完成之后，我们就要来看一下我们常用的一些方法具体是如何实现的

1. add()
add()方法是往集合中添加数据
```
   /**
     * Appends the specified element to the end of this list.
     *
     * @param e element to be appended to this list
     * @return <tt>true</tt> (as specified by {@link Collection#add})
     */
    public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }

```
在add方法中，我们需要执行两步操作，第一步是执行ensureCapacityIntrnal方法，第二步是执行数组的赋值操作。那么细心的同学可能会说，如果一开始我们声明的数组长度是10，而我这时候添加第11个元素，这样数组不就越界了吗？嗯，理论上来说是这样的，但是如果这时候我们的数组长度改变了呢？对，其实这里如果我再添加数据之后的数组长度大于目前的数组长度，就需要将数组扩容，那么判断其实就是通过ensureCapacityInternal方法执行的。并且细心的同学会发现，我们在调用这个方法的时候，传递的数据就是(size+1)看一下这个方法

```
    private void ensureCapacityInternal(int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }

        ensureExplicitCapacity(minCapacity);
    }

```
在这个方法中我们会看到，首先判断当前elementdata是否等于空，如果等于空，则将10和传递数据的最大值赋值给minCapacity，并且继续调用ensureExplicitCapacity()方法。这里我们分析三种情况，①如果我们是第一次添加数据，这时候minCapacity=10，②我们是第二次添加数据，也就是说添加之后的数量不超过默认值，这时候minCapacity=10，③添加之后的数量大于默认值，则返回传递过来的数据，我们假设minCapacity=11。紧接着我们看一下ensureExplicitCapacity方法是如何实现的

```
    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }
```
这个方法中，我们首先将modCount++，这个变量表示我们当前ArrayList更新了的次数，暂时先不考虑；紧接着我们有一个if语句判断传递过来的mincapacity减去elementdata的长度是否大于0，如果大于零则执行grow方法，如果小于/等于0，则不需要执行任何操作。如果不需要执行任何操作就会执行执行elementData[size++] = e这个语句。我们来看一下，在前一个方法中我们分析了，可能传递过来的数据minCapacity有两种情况：10或者11，如果是10的话，当前的elementData的长度也是10，就不会执行grow方法，只有当前传递过来的数据是11的时候，才会大于10，所以我们可以得出结论：如果需要添加的数据之后的数组长度小于等于当前数组长度，则直接赋值即可，如果大于则需要执行grow方法。而这个grow方法就是扩容方法。我们来看一下grow方法是如何实现的

```
    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```
在grow方法中我们传递过来的是minCapacity是扩容之后的长度，这个我们需要知道；首先我们先得到目前的elementData的长度，也就是oldCapacity = 10，紧接着我们执行了下面这句话
```
int new Capacity = oldCapacity + (oldCapacity >> 1);
```
这句话是计算在扩容之后的数组长度，想一下我们现在数组长度是10，新增加一个元素，如果只把数组长度增加到11的话，那么后面每次增加我们都需要再次扩容，这样很消耗资源，所以我们就按照一定的方法扩容一定的长度。至于具体扩容多少，需要牵扯到我们的位运算，其实就是将数组的长度扩容了原来的一般，如果之前的长度是10，那么第一次扩容之后的长度是15，第二次扩容之后的长度是23，以此类推。紧接着就是判断的操作，这个判断操作没什么好说的，直接看最后一行
```
elementData = Arrays.copyof(elementData,newCapacity);
```
这句是干啥呢？其实就是将elementData数组扩容到newCapacity的长度，那原来的数据呢？会通过赋值的形式重新传递给我们的elementData。
这样就完成了添加扩容的操作。
我们会发现，在我们执行添加操作的时候，不牵扯到扩容还好，牵扯到扩容之后，竟然需要把之前的数据拷贝一遍在赋值给新的数组，这样是很消耗资源的的，所以这个扩容的过程会很慢(虽然CPU运行会很快，但是他确实很消耗资源)。

> 这里可能有同学会说，我们可以声明一个非常大的数组，这样就不会有扩容的问题了。话是这么说的，但是我们知道数组在内存中是一个堤内地址连续的存储空间，申请一个比较大的数组就决定了我们需要一块很大的内存，这个对于内存来说，完完全全是负担。

关于get，set方法这里都是对数组执行的操作，没什么好说的。我们再来看一下remove方法。我们在执行remove方法的时候，一般会传递一个下标或者直接传递集合对象，我们先看一下实现

```
    public E remove(int index) {
        if (index >= size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));

        modCount++;
        E oldValue = (E) elementData[index];

        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work

        return oldValue;
    }
```
下标的删除比较简单，主要就是将数组中对应下标的内容设置为null，即可。但是我们需要注意的是，我们设置了为null，但是并不代表数据所占用的资源会被立刻设防，所以说我们的数组的长度之后的数据是不变的。

```
    public boolean remove(Object o) {
        if (o == null) {
            for (int index = 0; index < size; index++)
                if (elementData[index] == null) {
                    fastRemove(index);
                    return true;
                }
        } else {
            for (int index = 0; index < size; index++)
                if (o.equals(elementData[index])) {
                    fastRemove(index);
                    return true;
                }
        }
        return false;
    }
```
对象的删除方式就是通过for循环找到相应的对象，然后通过fastRemove方法达到删除的效果。至于fastRemove方法其实和下表删除基本一致
```
    private void fastRemove(int index) {
        modCount++;
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work
    }
```

