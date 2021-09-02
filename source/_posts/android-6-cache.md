---
title: 重拾android路(六) 缓存
date: 2016-04-15 23:23:34
tags:
  - android
---
Android中的缓存
<!--more-->
# LruCache

LruCache 位于 android.util 包下，属于 SDK 自动的工具类。Lru 的英文为 Least recently used，近期最少使用算法。设计的思路大致是这样的，在一个特定的缓存大小限制下，最近被使用的内容，很可能在未来再次被使用，而越长时间没被使用的内容，被使用的概率越底，基于这样的思路，最近被使用的内容被放在缓存的头部，这样减少了下次使用的查找时间，很长时间不被使用的内容被放在缓存的尾部，甚至在缓存超出预设的大小的时候，把尾部的内容从缓存中清楚掉

## 实现原理
根据LRU算法的思想，要实现LRU最核心的是要有一种数据结构能够基于访问顺序来保存缓存中的对象，这样我们就能够很方便的知道哪个对象是最近访问的，哪个对象是最长时间未访问的。LruCache选择的是LinkedHashMap这个数据结构，LinkedHashMap是一个双向循环链表，在构造LinkedHashMap时，通过一个boolean值来指定LinkedHashMap中保存数据的方式，LinkedHashMap的一个构造方法如下：    
```
/* 
     * 初始化LinkedHashMap 
     * 第一个参数：initialCapacity，初始大小 
     * 第二个参数：loadFactor，负载因子=0.75f 
     * 第三个参数：accessOrder=true，基于访问顺序；accessOrder=false，基于插入顺序 
     */  
    public LinkedHashMap(int initialCapacity, float loadFactor, boolean accessOrder) {  
        super(initialCapacity, loadFactor);  
        init();  
        this.accessOrder = accessOrder;  
    } 
```
显然，在LruCache中选择的是accessOrder = true；此时，当accessOrder 设置为 true时，每当我们更新（即调用put方法）或访问（即调用get方法）map中的结点时，LinkedHashMap内部都会将这个结点移动到链表的尾部，因此，在链表的尾部是最近刚刚使用的结点，在链表的头部是是最近最少使用的结点，当我们的缓存空间不足时，就应该持续把链表头部结点移除掉，直到有剩余空间放置新结点。
可以看到，LinkedHashMap完成了LruCache中的核心功能，那LruCache中剩下要做的就是定义缓存空间总容量，当前保存数据已使用的容量，对外提供put、get方法。

## field
android.util.LruCache 的代码很简洁，其重要的属性有下面三个。整个类的设计都是围绕着这三个变量来进行的。

- LinkedHashMap map //核心数据结构 
- int maxSize  //缓存空间总容量 
- int size  // 当前缓存数据所占的大小  
要注意的是size字段，因为map中可以存放各种类型的数据，这些数据的大小测量方式也是不一样的，比如Bitmap类型的数据和String类型的数据计算他们的大小方式肯定不同，因此，LruCache中在计算放入数据大小的方法sizeOf中，只是简单的返回了1，需要我们重写这个方法，自己去定义数据的测量方式。因此，我们在使用LruCache的时候，经常会看到这种方式
```
private static final int CACHE_SIZE = 4 * 1024 * 1024;//4Mib  
    LruCache<String,Bitmap> bitmapCache = new LruCache<String,Bitmap>(CACHE_SIZE){  
        @Override  
        protected int sizeOf(String key, Bitmap value) {  
            return value.getByteCount();//自定义Bitmap数据大小的计算方式  
        }  
    }; 
```
LinkedHashMap 是这个类的核心，LinkedHashMap 是 HashMap 的子类，从名字上就可以看出，它首先是一个 HashMap，然后是一个 Linked 的 HashMap，Linked 的设计，也让它更加适合 LruCache 的实现。被 get() 的实例会被放到链表的开头。
## methods
### 构造方法
```
public LruCache(int maxSize) {
    if (maxSize <= 0) {
        throw new IllegalArgumentException("maxSize <= 0");
    }
    this.maxSize = maxSize;
    this.map = new LinkedHashMap<K, V>(0, 0.75f, true);
}
```
LruCache只有一个唯一的构造方法，在构造方法中，给定了缓存空间的总大小，初始化了LinkedHashMap核心数据结构，在LinkedHashMap中的第三个参数指定为true，也就设置了accessOrder=true，表示这个LinkedHashMap将是基于数据的访问顺序进行排序。
### sizeOf()和safeSizeOf()
 根据上面的解释，由于各种数据类型大小测量的标准不统一，具体测量的方法应该由使用者来实现，如上面给出的一个在实现LruCache时重写sizeOf的一种常用实现方式。通过多态的性质，再具体调用sizeOf时会调用我们重写的方法进行测量，LruCache对sizeOf()的调用进行一层封装，如下：
 ```
 private int safeSizeOf(K key, V value) {
    int result = sizeOf(key, value);
    if (result < 0) {
        throw new IllegalStateException("Negative size: " + key + "=" + value);
    }
    return result;
}
```
里面其实就是调用sizeOf()方法，返回sizeOf计算的大小。
### get()和put()方法
get() 和 put() 方法是咱们使用 LruCache 最经常用的方法，remove() 咱们直接使用的就比较少了。trimToSize() 是 LruCache 管理缓存内容的核心了，配合 size 和 maxSize 这两个变量，维护这这个缓存的大小。
get()
```
/**
 * Returns the value for {@code key} if it exists in the cache or can be
 * created by {@code #create}. If a value was returned, it is moved to the
 * head of the queue. This returns null if a value is not cached and cannot
 * be created.
 */
public final V get(K key) {
    if (key == null) {
        throw new NullPointerException("key == null");
    }

    V mapValue;
    synchronized (this) {
        mapValue = map.get(key);
        if (mapValue != null) {
            hitCount++;
            return mapValue;
        }
        missCount++;
    }

    /*
     * Attempt to create a value. This may take a long time, and the map
     * may be different when create() returns. If a conflicting value was
     * added to the map while create() was working, we leave that value in
     * the map and release the created value.
     */

    // 创建一个默认的对象，create() 默认返回 null ，可自定义实现，返回一个默认的对象
    V createdValue = create(key);
    if (createdValue == null) {
        return null;
    }

    // 如果有默认对象，那么这个对象也是占用空间的
    synchronized (this) {
        createCount++;
        // 把创建的对象放入 map 中，返回 map 中的原对象
        mapValue = map.put(key, createdValue);
        // 由于 create() 方法没有同步保护，所以这里需要再次检查，一般情况下 mapValue 会是空的，但是在多线程下，有可能不为空。 
        if (mapValue != null) {
            // There was a conflict so undo that last put
            // 如果不为空，保留原对象
            map.put(key, mapValue);
        } else {
            // 计算创建的默认对象占用的空间
            size += safeSizeOf(key, createdValue);
        }
    }

    
    if (mapValue != null) {
        // 刚创建的默认对象需要做一些回收的操作，如果需要的话。
        entryRemoved(false, key, createdValue, mapValue);
        return mapValue;
    } else {
        // 如果默认的对象被放入到 map 中，需要重新检查和裁剪 map 大小。
        trimToSize(maxSize);
        return createdValue;
    }
}
```
get()方法的思路就是：
1. 先尝试从map缓存中获取value，即mapVaule = map.get(key);如果mapVaule != null，说明缓存中存在该对象，直接返回即可；
2. 如果mapVaule == null，说明缓存中不存在该对象，大多数情况下会直接返回null；但是如果我们重写了create()方法，在缓存没有该数据的时候自己去创建一个，则会继续往下走，中间可能会出现冲突，看注释；
3. 注意：在我们通过LinkedHashMap进行get(key)或put(key,value)时都会对链表进行调整，即将刚刚访问get或加入put的结点放入到链表尾部。
put()
```
/**
 * Caches {@code value} for {@code key}. The value is moved to the head of
 * the queue.
 *
 * @return the previous value mapped by {@code key}.
 */
public final V put(K key, V value) {
    if (key == null || value == null) {
        throw new NullPointerException("key == null || value == null");
    }

    V previous;
    synchronized (this) {
        putCount++;
        // 计算将要放入的对象的大小
        size += safeSizeOf(key, value);
        // 把对象放入到 map 中，获取改 key 原对象
        previous = map.put(key, value);
        // 如果该 key 有原对象，减掉它占用的空间
        if (previous != null) {
            size -= safeSizeOf(key, previous);
        }
    }

    if (previous != null) {
        entryRemoved(false, key, previous, value);
    }
    // 重新检查和裁剪 map 大小
    trimToSize(maxSize);
    return previous;
}
```
可以看到，put()方法主要有以下几步：
1. key和value判空，说明LruCache中不允许key和value为null；
2. 通过safeSizeOf()获取要加入对象数据的大小，并更新当前缓存数据的大小；
3. 将新的对象数据放入到缓存中，即调用LinkedHashMap的put方法，如果原来存在该key时，直接替换掉原来的value值，并返回之前的value值，得到之前value的大小，更新当前缓存数据的size大小；如果原来不存在该key，则直接加入缓存即可；
4. 清理缓存空间，如下；
```
public void trimToSize(int maxSize) {
    /*
     * 循环进行LRU，直到当前所占容量大小没有超过指定的总容量大小
     */
    while (true) {
        K key;
        V value;
        synchronized (this) {
            // 一些异常情况的处理
            if (size < 0 || (map.isEmpty() && size != 0)) {
                throw new IllegalStateException(
                        getClass().getName() + ".sizeOf() is reporting inconsistent results!");
            }
            // 首先判断当前缓存数据大小是否超过了指定的缓存空间总大小。如果没有超过，即缓存中还可以存入数据，直接跳出循环，清理完毕
            if (size <= maxSize || map.isEmpty()) {
                break;
            }
            /**
             * 执行到这，表示当前缓存数据已超过了总容量，需要执行LRU，即将最近最少使用的数据清除掉，直到数据所占缓存空间没有超标;
             * 根据前面的原理分析，知道，在链表中，链表的头结点是最近最少使用的数据，因此，最先清除掉链表前面的结点
             */
            Map.Entry<K, V> toEvict = map.entrySet().iterator().next();
            key = toEvict.getKey();
            value = toEvict.getValue();
            map.remove(key);
            // 移除掉后，更新当前数据缓存的大小
            size -= safeSizeOf(key, value);
            // 更新移除的结点数量
            evictionCount++;
        }
        /*
         * 通知某个结点被移除，类似于回调
         */
        entryRemoved(true, key, value, null);
    }
```
trimToSize()方法的作用就是为了保证当前数据的缓存大小不能超过我们指定的缓存总大小，如果超过了，就会开始移除最近最少使用的数据，直到size符合要求。trimToSize()方法在put()的时候一定会调用，在get()的时候有可能会调用。
### create()
在 get() 调用中，可能需要调用 create() 创建默认对象。但是这个方法不是在一个同步语句块中调用的，所以要注意线程安全的问题，具体建议详细看看 get() 的方法是怎么处理的
```
/**
 * Called after a cache miss to compute a value for the corresponding key.
 * Returns the computed value or null if no value can be computed. The
 * default implementation returns null.
 *
 * <p>The method is called without synchronization: other threads may
 * access the cache while this method is executing.
 *
 * <p>If a value for {@code key} exists in the cache when this method
 * returns, the created value will be released with {@link #entryRemoved}
 * and discarded. This can occur when multiple threads request the same key
 * at the same time (causing multiple values to be created), or when one
 * thread calls {@link #put} while another is creating a value for the same
 * key.
 */
protected V create(K key) {
    return null;
}
```

### entryRemoved()
```
/**
 * 1.当被回收或者删掉时调用。该方法当value被回收释放存储空间时被remove调用
* 或者替换条目值时put调用，默认实现什么都没做。
* 2.该方法没用同步调用，如果其他线程访问缓存时，该方法也会执行。
* 3.evicted=true：如果该条目被删除空间 （表示 进行了trimToSize or remove）  evicted=false：put冲突后 或 get里成功create后
* 导致
* 4.newValue!=null，那么则被put()或get()调用。
*/
protected void entryRemoved(boolean evicted, K key, V oldValue, V newValue) {
}
```
可以发现entryRemoved方法是一个空方法，说明这个也是让开发者自己根据需求去重写的。entryRemoved()主要作用就是在结点数据value需要被删除或回收的时候，给开发者的回调。开发者就可以在这个方法里面实现一些自己的逻辑：
1. 可以进行资源的回收；
2. 可以实现二级内存缓存，可以进一步提高性能，思路如下：重写LruCache的entryRemoved()函数，把删除掉的item，再次存入另外一个LinkedHashMap<String, SoftWeakReference<Bitmap>>中，这个数据结构当做二级缓存，每次获得图片的时候，先判断LruCache中是否缓存，没有的话，再判断这个二级缓存中是否有，如果都没有再从sdcard上获取。sdcard上也没有的话，就从网络服务器上拉取。
### 线程
LruCache是线程安全的，因为在put、get、trimToSize、remove的方法中都加入synchronized进行同步控制
### LRU的使用
注意：
1. 在构造LruCache时提供一个总的缓存大小；
2. 重写sizeOf方法，对存入map的数据大小进行自定义测量；
3. 根据需要，决定是否要重写entryRemoved()方法；
4. 使用LruCache提供的put和get方法进行数据的缓存
> LruCache 自身并没有释放内存，只是 LinkedHashMap中将数据移除了，如果数据还在别的地方被引用了，还是有泄漏问题，还需要手动释放内存；
> 覆写 entryRemoved 方法能知道 LruCache 数据移除是是否发生了冲突（冲突是指在map.put()的时候，对应的key中是否存在原来的值），也可以去手动释放资源；

# DiskLruCache
我们现在来假设一种情况，当前我们的手机应用需要对数据进行大量的读取，比如一个网易新闻，那么这个时候我们需要存储的不仅包括了文字，还有图片，甚至还有可能是视频。那么当这些文件如果也通过LruCache的方式进行存储的时候，我们就需要对他的内容进行大量的修改，那么这样的操作肯定是不合理的
![网易新闻](/assets/cache/cache01.png)
那么这个时候我们的DiskLruCache就起到了作用，他我们可以认为就是把当前的数据保存到了本地存储空间中(现在的手机大部分都不在使用外部存储卡，这样也大大提高了我们对于外部存储数据的读写速度)。

当我们在有网络连接的情况下查看了某些新闻信息，当我们在没有网络情况下依然可以可以去访问我们之前看过的内容，那么这些东西就是使用DiskLruCache。既然保存在了本地，那么我们遇到的第一个问题就是存储的位置(说来惭愧，之前我一直对存储位置一知半解).
其实DiskLruCache并没有限制数据的缓存位置，可以自由地进行设定，但是通常情况下多数应用程序都会将缓存的位置选择为 
```
/sdcard/Android/data/<application package>/cache 
```
这个路径。选择在这个位置有两点好处：第一，这是存储在SD卡上的，因此即使缓存再多的数据也不会对手机的内置存储空间有任何影响，只要SD卡空间足够就行。第二，这个路径被Android系统认定为应用程序的缓存路径，当程序被卸载的时候，这里的数据也会一起被清除掉，这样就不会出现删除程序之后手机上还有很多残留数据的问题。

那么这里还是以网易新闻为例，它的客户端的包名是com.netease.newsreader.activity，因此数据缓存地址就应该是 /sdcard/Android/data/com.netease.newsreader.activity/cache ，我们进入到这个目录中看一下，结果如下图所示：
![网易新闻](/assets/cache/cache02.png)
可以看到有很多个文件夹，因为网易新闻对多种类型的数据都进行了缓存，这里简单起见我们只分析图片缓存就好，所以进入到bitmap文件夹当中。然后你将会看到一堆文件名很长的文件，这些文件命名没有任何规则，完全看不懂是什么意思，但如果你一直向下滚动，将会看到一个名为journal的文件，如下图所示
![网易新闻](/assets/cache/cache03.png)
那么这些文件到底都是什么呢？看到这里相信有些朋友已经是一头雾水了，这里我简单解释一下。上面那些文件名很长的文件就是一张张缓存的图片，每个文件都对应着一张图片，而journal文件是DiskLruCache的一个日志文件，程序对每张图片的操作记录都存放在这个文件中，基本上看到journal这个文件就标志着该程序使用DiskLruCache技术了

下面就是我们需要执行的步骤
## 申请权限
```
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.INTERNET" />
```
## 判断外部存储设备是否存在
```
/**
   * Get a usable cache directory (external if available, internal otherwise).
   * external：如：/storage/emulated/0/Android/data/package_name/cache
   * internal 如：/data/data/package_name/cache
   *
   * @param context    The context to use
   * @param uniqueName A unique directory name to append to the cache dir
   * @return The cache dir
   */
  public static File getDiskCacheDir(Context context, String uniqueName) {
      final String cachePath = Environment.MEDIA_MOUNTED.equals(Environment.getExternalStorageState()) ||!isExternalStorageRemovable() 
              ? context.getExternalCacheDir().getPath()
              : context.getCacheDir().getPath();
      return new File(cachePath + File.separator + uniqueName);
  }
```
1. 首先判断外部缓存是否被移除或已存满，如果已存满或者外存储被移除，则缓存目录=context.getCacheDir().getPath()，即存到 /data/data/package_name/cache 这个文件系统目录下；
2. 反之缓存目录=context.getExternalCacheDir().getPath()，即存到 /storage/emulated/0/Android/data/package_name/cache 这个外部存储目录中，PS：外部存储可以分为两种：一种如上面这种路径 (/storage/emulated/0/Android/data/package_name/cache)， 当应用卸载后，存储数据也会被删除，另外一种是永久存储，即使应用被卸载，存储的数据依然存在，存储路径如：/storage/emulated/0/mDiskCache，可以通过Environment.getExternalStorageDirectory().getAbsolutePath() + "/mDiskCache" 来获得路径。
## 下载一个现在的图片，并且把它写到输出流
```
/**
     * Download a bitmap from a URL and write the content to an output stream.
     *
     * @param urlString The URL to fetch
     * @return true if successful, false otherwise
     */
    private boolean downloadUrlToStream(String urlString, OutputStream outputStream) {
        HttpURLConnection urlConnection = null;
        BufferedOutputStream out = null;
        BufferedInputStream in = null;

        try {
            final URL url = new URL(urlString);
            urlConnection = (HttpURLConnection) url.openConnection();
            in = new BufferedInputStream(urlConnection.getInputStream(), IO_BUFFER_SIZE);
            out = new BufferedOutputStream(outputStream, IO_BUFFER_SIZE);
            int b;
            while ((b = in.read()) != -1) {
                out.write(b);
            }
            return true;
        } catch (Exception e) {
            Log.e(TAG, "Error in downloadBitmap - " + e);
        } finally {
            if (urlConnection != null) {
                urlConnection.disconnect();
            }
            try {
                if (out != null) {
                    out.close();
                }
                if (in != null) {
                    in.close();
                }
            } catch (final IOException e) {
            }
        }
        return false;
    }
```
## 初始化DiskLruCache，并且使用DiskLruCache.Editor方法准备缓存
```
private static final int MAX_SIZE = 10 * 1024 * 1024;//10MB
    private DiskLruCache diskLruCache;
    private void initDiskLruCache() {
        if (diskLruCache == null || diskLruCache.isClosed()) {
            try {
                File cacheDir = CacheUtil.getDiskCacheDir(this, "CacheDir");
                if (!cacheDir.exists()) {
                    cacheDir.mkdirs();
                }
                //初始化DiskLruCache
                diskLruCache = DiskLruCache.open(cacheDir, 1, 1, MAX_SIZE);
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
```
下载图片时需要放在异步线程中，AsyncTask中的DoInBackground中
```
@Override
protected Boolean doInBackground(Object... params) {
    try {
        String key = Util.hashKeyForDisk(Util.IMG_URL);
        DiskLruCache diskLruCache = (DiskLruCache) params[0];
         //得到DiskLruCache.Editor
        DiskLruCache.Editor editor = diskLruCache.edit(key);
        if (editor != null) {
            OutputStream outputStream = editor.newOutputStream(0);
            if (downloadUrlToStream(Util.IMG_URL, outputStream)) {
                publishProgress("");
                //写入缓存
                editor.commit();
            } else {
                 //写入失败
                editor.abort();
            }
        }
        diskLruCache.flush();
    } catch (IOException e) {
        e.printStackTrace();
        return false;
    }
    return true;
}
```
上面代码中有个hashKeyForDisk()方法，其作用是把图片URL经过MD5加密生成唯一的key值，避免了URL中可能含有非法字符问题，hashKeyForDisk()代码如下
```
/**
  * A hashing method that changes a string (like a URL) into a hash suitable for using as a
  * disk filename.
  */
 public static String hashKeyForDisk(String key) {
     String cacheKey;
     try {
         final MessageDigest mDigest = MessageDigest.getInstance("MD5");
         mDigest.update(key.getBytes());
         cacheKey = bytesToHexString(mDigest.digest());
     } catch (NoSuchAlgorithmException e) {
         cacheKey = String.valueOf(key.hashCode());
     }
     return cacheKey;
 }

 private static String bytesToHexString(byte[] bytes) {
     // http://stackoverflow.com/questions/332079
     StringBuilder sb = new StringBuilder();
     for (int i = 0; i < bytes.length; i++) {
         String hex = Integer.toHexString(0xFF & bytes[i]);
         if (hex.length() == 1) {
             sb.append('0');
         }
         sb.append(hex);
     }
     return sb.toString();
 }
```
经过上面的代码，我们已经可以看到图片已经缓存到 /storage/emulated/0/Android/data/package_name/cache/CacheDir 这个目录下了：
![图片](/assets/cache/cache04.png)
第一个标识为110.78kb大小的就是我们缓存下来的图片，它的名字正是由图片的URL经过MD5加密得到的，它下面的journal文件是用来记录的，来看里面的内容
![图片](/assets/cache/cache05.png)
第一行：libcore.io.DiskLruCache固定写死
第二行：DiskLruCache版本号
第三行：APP版本号，由open()方法的参数appVersion传入
第四行：同一个key可以对应多少文件，由open()方法的参数valueCount传入，一般为1
第五行：空格
第六行：以DIRTY开头，后面跟着的是图片的key值，表示准备缓存这张图片，当调用DiskLruCache的edit()时就会生成这行记录
第七行： 以CLEAN开头,后面跟着的是图片的Key值和大小，当调用editor.commit()时会生成这条记录，表示缓存成功；如果调用editor.abort()表示缓存失败，则会生成REMOVE开头的表示删除这条数据。
## 获取存储图片
通过diskLruCache.get(key)得到DiskLruCache.Snapshot，key是经过MD5加密后那个唯一的key，接着使用Snapshot.getInputStream()可以得到输入流InputStream ，进而得到缓存图片
```
private Bitmap getCache() {
     try {
         String key = Util.hashKeyForDisk(Util.IMG_URL);
         DiskLruCache.Snapshot snapshot = diskLruCache.get(key);
         if (snapshot != null) {
             InputStream in = snapshot.getInputStream(0);
             return BitmapFactory.decodeStream(in);
         }
     } catch (IOException e) {
         e.printStackTrace();
     }
     return null;
 }
```
使用：
```
Bitmap bitmap = getCache();
     if (bitmap != null) {
         iv_img.setImageBitmap(bitmap);
     }
```
## 移除缓存
在网易新闻的手机应用中我们会发现有清除缓存的操作，那么这个地方就是移除缓存
移除缓存主要是借助DiskLruCache的remove()方法实现的，接口如下所示：
```
public synchronized boolean remove(String key) throws IOException  
```
在remove()方法中会要求传入一个key值，这样就会移除对应的缓存
```
try {  
    String imageUrl = "http://img.my.csdn.net/uploads/201309/01/1378037235_7476.jpg";    
    String key = hashKeyForDisk(imageUrl);    
    mDiskLruCache.remove(key);  
} catch (IOException e) {  
    e.printStackTrace();  
}  
```
当然这个地方我们其实使用这种方式是非常复杂的，就是因为这个key值得存在，那么其实我们是可以通过别的方法去删除的，这里把其他的一些API也说一下
| 方法 | 说明 |
|----|----|
|size()|这个方法会返回当前缓存路径下所有缓存数据的总字节数，以byte为单位，如果应用程序中需要在界面上显示当前缓存数据的总大小，就可以通过调用这个方法计算出来|
|flush() | 这个方法用于将内存中的操作记录同步到日志文件（也就是journal文件）当中。这个方法非常重要，因为DiskLruCache能够正常工作的前提就是要依赖于journal文件中的内容。前面在讲解写入缓存操作的时候我有调用过一次这个方法，但其实并不是每次写入缓存都要调用一次flush()方法的，频繁地调用并不会带来任何好处，只会额外增加同步journal文件的时间。比较标准的做法就是在Activity的onPause()方法中去调用一次flush()方法就可以了|
|close()| 这个方法用于将DiskLruCache关闭掉，是和open()方法对应的一个方法。关闭掉了之后就不能再调用DiskLruCache中任何操作缓存数据的方法，通常只应该在Activity的onDestroy()方法中去调用close()方法。|
|delete()|这个方法用于将所有的缓存数据全部删除，比如说网易新闻中的那个手动清理缓存功能，其实只需要调用一下DiskLruCache的delete()方法就可以实现了

# 关于缓存的框架
我们的换粗大部分情况下都是缓存图片，所以在一些图片请求框架中就已经包含了图片缓存，这个我以后在总结，留下个尾巴

# 参考资料
[android缓存](http://blog.csdn.net/shakespeare001/article/details/51695358)
[android Lru源码分析](http://www.binkery.com/archives/561.html)
[android DiskLruCache](http://blog.csdn.net/guolin_blog/article/details/28863651)
[jake大神的分析](https://github.com/JakeWharton/DiskLruCache)