---
title: 源码分析(二) HashMap
date: 2020-03-03 22:09:13
tags:
 - 源码分析
---

HashMap源码分析
<!--more-->

HashMap是位于java.util包下的工具类，帮我们实现了继承自Map接口的数据集。HashMap直接继承自AbstractMap，并且实现了Cloneable和Serializable接口，所以我们可以将HashMap执行深复制/浅复制和序列化等操作。

```
public class HashMap<K,V>
    extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable
```

在HashMap中有一些变量需要我们注意
1. DEFAULT_INITIAL_CAPACITY
这个变量表示HashMap初始值大小，后面我们会说HashMap其实也是跟数组有关的，既然是数组那就必须有一个初始值，HashMap的初始值就是4
```
    /**
     * The default initial capacity - MUST be a power of two.
     */
    static final int DEFAULT_INITIAL_CAPACITY = 4;

```
2. EMPTY_TABLE
这个表示在数据没有执行初始化的时候，默认的数组
```
    /**
     * An empty table instance to share when the table is not inflated.
     */
    static final HashMapEntry<?,?>[] EMPTY_TABLE = {};

```
3. table
这个就是HashMap实际存储数据的数组，默认情况下使用EMPTY_TABLE进行初始化
```
    /**
     * The table, resized as necessary. Length MUST Always be a power of two.
     */
    transient HashMapEntry<K,V>[] table = (HashMapEntry<K,V>[]) EMPTY_TABLE;

```
其中HashMapEntry是一个实体，表示的就是键值对。我们可以看一下HashMapEntry的类实体
```
    static class HashMapEntry<K,V> implements Map.Entry<K,V> {
        final K key;
        V value;
        HashMapEntry<K,V> next;
        int hash;
        ...
    }
```
在这个类中有四个成员变量，key，value，next，hash。其中next也是一个HashMapEntry对象，由此我们可以知道，HashMapEntry其实就是一个单链表
所以通过上面的代码我们知道，我们是将需要存入的key-value作为一个对象进行封装，装成entry之后，将entry作为数组的数据存入。我们可以这样说：HashMap是以数组为存储结构的数组-链表集合。

HashMap的构造方法有多个，但是最终都会调用到一个传入了初始化大小和增长因子的构造方法中，如下所示
```
    public HashMap(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
        if (initialCapacity > MAXIMUM_CAPACITY) {
            initialCapacity = MAXIMUM_CAPACITY;
        } else if (initialCapacity < DEFAULT_INITIAL_CAPACITY) {
            initialCapacity = DEFAULT_INITIAL_CAPACITY;
        }

        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);
        // Android-Note: We always use the default load factor of 0.75f.

        // This might appear wrong but it's just awkward design. We always call
        // inflateTable() when table == EMPTY_TABLE. That method will take "threshold"
        // to mean "capacity" and then replace it with the real threshold (i.e, multiplied with
        // the load factor).
        threshold = initialCapacity;
        init();
    }

```
两个参数，第一个参数initialCapacity就是初始化数据大小，第二个参数loadFactor是增长因子。里面关于这两个变量的赋值改变等操作我们不用去看，只看最后一个init()方法的调用。这个init方法默认HashMap是不帮我们实现的，如果我们想要对HashMap有一个初始化的操作，可以重写该方法。

同其他集合也是一样，我们使用集合就是为了存放数据，获取数据，修改数据，删除数据等操作的，所以先看第一个方法put方法。
```
    public V put(K key, V value) {
        if (table == EMPTY_TABLE) {
            inflateTable(threshold);
        }
        if (key == null)
            return putForNullKey(value);
        int hash = sun.misc.Hashing.singleWordWangJenkinsHash(key);
        int i = indexFor(hash, table.length);
        for (HashMapEntry<K,V> e = table[i]; e != null; e = e.next) {
            Object k;
            if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
                V oldValue = e.value;
                e.value = value;
                e.recordAccess(this);
                return oldValue;
            }
        }

        modCount++;
        addEntry(hash, key, value, i);
        return null;
    }

```
在执行put方法时，我们会先检测当前table是否为空，如果为空，则会执行inflateTable这个方法。这个方法很简单就是说如果HashMap为空的话，则初始化HashMap的长度。
```
    /**
     * Inflates the table.
     */
    private void inflateTable(int toSize) {
        // Find a power of 2 >= toSize
        int capacity = roundUpToPowerOf2(toSize);

        // Android-changed: Replace usage of Math.min() here because this method is
        // called from the <clinit> of runtime, at which point the native libraries
        // needed by Float.* might not be loaded.
        float thresholdFloat = capacity * loadFactor;
        if (thresholdFloat > MAXIMUM_CAPACITY + 1) {
            thresholdFloat = MAXIMUM_CAPACITY + 1;
        }

        threshold = (int) thresholdFloat;
        table = new HashMapEntry[capacity];
    }

```
还是put方法，紧接着我们判断key是否为空，如果为空则调用putForNullKey()方法。这个方法其实就是保证HashMap中只有一个key=null。说明HashMap允许key=null。
如果key不为空，则时候我们调用了一个方法singleWordWangJenkinsHash(),这个方法在我的代码里无法直接访问到，但是我在网上查找了一下，其实这个方法就是将key的值转换成hash值。我们知道hash值跟我们的数据是有关系的，如果两个对象的值相同，那么hash值也一定是相同的。在拿到hash值之后根据hash值判断当前的hash值在数组中的下标，返回值是i，通过for循环，从数组中获取HashMapEntry对象，并且比较Entry对象和需要添加的key，value值是否相等，这里需要注意我们for循环的判断条件是next(),其实就是说遍历i下标的数组对象(链表中)是否存在key-value相等的对象。如果存在，则返回旧的数据，如果不存在，则执行完for循环之后执行addEntry方法。
```
    void addEntry(int hash, K key, V value, int bucketIndex) {
        if ((size >= threshold) && (null != table[bucketIndex])) {
            resize(2 * table.length);
            hash = (null != key) ? sun.misc.Hashing.singleWordWangJenkinsHash(key) : 0;
            bucketIndex = indexFor(hash, table.length);
        }

        createEntry(hash, key, value, bucketIndex);
    }
```
addEntry方法需要传递的数据包括hash值，这个值是添加数据时在数组中的标志，key和value值，bucketIndex。然后在createEntry方法的具体内容如下：
```
    void createEntry(int hash, K key, V value, int bucketIndex) {
        HashMapEntry<K,V> e = table[bucketIndex];
        table[bucketIndex] = new HashMapEntry<>(hash, key, value, e);
        size++;
    }
```
这里是直接将hash值在数组中对应的对象拿过来作为参数传递到HashMapEntry的构造方法中，在构造方法中其实就是将需要新创建的对象放在链表的头部。这样就是新的数据放在链表的前面，以前的对象放在链表的最后。
```
        HashMapEntry(int h, K k, V v, HashMapEntry<K,V> n) {
            value = v;
            next = n;
            key = k;
            hash = h;
        }

```

关于剩下的删除，获取等方法，这里就不一一介绍，主要还是添加方法的逻辑。