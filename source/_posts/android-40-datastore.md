---
title: 重拾android路(四十) DataStore
date: 2020-06-23 21:35:51
tags:
  - android
  - datastore
---



DataStore 是一种新的数据存储方案。DataStore 以异步、一致的事务方式存储数据，克服了 SharedPreferences 的一些缺点。

<!--more-->

### 关于SharedPreference的坑
* 通过 getXXX() 方法获取数据，可能会导致主线程阻塞
* SharedPreference 不能保证类型安全
* SharedPreference 加载的数据会一直留在内存中，浪费内存
* apply() 方法虽然是异步的，可能会发生 ANR，在 8.0 之前和 8.0 之后实现各不相同
* apply() 方法无法获取到操作成功或者失败的结果


#### getXXX方法可能汇总成主线程的阻塞
在SP中所有的**getXXX**方法都是同步执行的，在主线程中调用get方法，就必须等待SP加载完成，这会导致线程阻塞，比如我们通过下面这种方式加载数据
```
val sp = getSharedPreference("test",Context.MODE_PRIVATE) // 异步加载sp文件
sp.getString("name","") // 等到sp加载完毕
```
调用**getSharedPreference()**方法，最终会调用**SharedPreferenceImpl#startLoadFromDisk()**方法开启一个线程异步读取数据，如下所示
```
private final Object mLock = new Object();
private boolean mLoaded = false;
private void startLoadFromDisk() {
    synchronized (mLock) {
        mLoaded = false;
    }
    new Thread("SharedPreferencesImpl-load") {
        public void run() {
            loadFromDisk();
        }
    }.start();
}
```
可见，开启一个线程异步读取数据，如果数据比较小，问题不大，但如果我们读取一个比较大的数据，还没有读完，直接调用了**getXXX**方法，就会触发如下的内容
```
public String getString(String key, @Nullable String defValue) {
    synchronized (mLock) {
        awaitLoadedLocked();
        String v = (String)mMap.get(key);
        return v != null ? v : defValue;
    }
}

private void awaitLoadedLocked() {
    ......
    while (!mLoaded) {
        try {
            mLock.wait();
        } catch (InterruptedException unused) {
        }
    }
    ......
}
```
在同步方法中由于调用了**wait()**方法，一直等待**getSharedPreferences()**方法开启的线程读取完数据之后才会继续执行

#### SP不能保证类型安全
调用**getXXX**方法的时候，可能会出现**ClassCastException**异常，这是因为在使用了相同的**key**进行操作的时候，**putXXX**方法可以使用不同类型的数据覆盖掉相同的**key**，如下面的代码
```
val key = "jitpack"
val sp = getSharedPreferences("test",Context.MODE_PRIVATE) // 异步加载sp文件

sp.edit{ putInt(key,0) } // 使用Int类型的数据覆盖相同的key
sp.getString(key,"") // 使用相同的key获取String类型的数据
```
此时编译时正确，但是运行就会发生异常

#### SP加载数据会一直存在内存中
通过**getSharedPreferences()**方法加载的数据，最后会将数据存储在静态的成员变量中
```
// 调用 getSharedPreferences 方法，最后会调用 getSharedPreferencesCacheLocked 方法
public SharedPreferences getSharedPreferences(File file, int mode) {
    ......
    final ArrayMap<File, SharedPreferencesImpl> cache = getSharedPreferencesCacheLocked();
    return sp;
}

// 通过静态的 ArrayMap 缓存 SP 加载的数据
private static ArrayMap<String, ArrayMap<File, SharedPreferencesImpl>> sSharedPrefsCache;

// 将数据保存在 sSharedPrefsCache 中
private ArrayMap<File, SharedPreferencesImpl> getSharedPreferencesCacheLocked() {
    ......
    
    ArrayMap<File, SharedPreferencesImpl> packagePrefs = sSharedPrefsCache.get(packageName);
    if (packagePrefs == null) {
        packagePrefs = new ArrayMap<>();
        sSharedPrefsCache.put(packageName, packagePrefs);
    }

    return packagePrefs;
}
```
在代码中我们发现**sSharedPrefsCache**是一个静态的**ArrayMap**，这就会导致缓存的每一个SP文件都保存在内存中

#### apply()方法是异步的，但是还会存在ANR
**apply()**方法是异步的，但是会阻塞主线程，虽然**apply()**本身是不存在任何问题的，但当生命周期处于**handleStopService()**,**handlePauseActivity()**,**handleStopActivity()**的时候会一直等待**apply()**方法将数据保存成功，否则会一直等待，从而阻塞主线程造成ANR，**apply()**方法的实现如下：
```
public void apply() {
    final long startTime = System.currentTimeMillis();

    final MemoryCommitResult mcr = commitToMemory();
    final Runnable awaitCommit = new Runnable() {
            @Override
            public void run() {
                mcr.writtenToDiskLatch.await(); // 等待
                ......
            }
        };
    // 将 awaitCommit 添加到队列 QueuedWork 中
    QueuedWork.addFinisher(awaitCommit);

    Runnable postWriteRunnable = new Runnable() {
            @Override
            public void run() {
                awaitCommit.run();
                QueuedWork.removeFinisher(awaitCommit);
            }
        };
    // 8.0 之前加入到一个单线程的线程池中执行
    // 8.0 之后加入 HandlerThread 中执行写入任务
    SharedPreferencesImpl.this.enqueueDiskWrite(mcr, postWriteRunnable);
}
```
将一个**awaitCommit**的**Runnable**任务添加到队列**QueuedWork**中，在**awaitCommit**中会调用**await()**方法等待，在**handleStopService**,**handleStopActivity**等等生命周期这个作为判断条件，等待任务执行完毕
将一个**postWriteRunnable**的**Runnable**写任务，通过**enqueueDiskWrite**方法，将写入任务加入到队列中，而写入任务在一个线程中执行
具体的内容可以看这篇博客，写的非常详细 [[Google] 再见 SharedPreferences 拥抱 Jetpack DataStore](https://juejin.cn/post/6881442312560803853)

### DataStore
DataStore是经过改进的新版数据存储解决方案，旨在取代SharedPreference。DataStore基于Kotlin协程和流程构建而成，提供了两种不同的实现

* Proto DataStore 可以存储类型化对象
* Preferences DataStore 存储键值对

Google的目的很明确，就是打算用DataStore取代SharedPreference。这里的源码还没有来得及去看，先看看如何使用的

#### DataStore导入
目前DataStore还处于alpha版本，使用时需要导入下面的依赖
```
dependencies {
  // Preferences DataStore
  implementation "androidx.datastore:datastore-preferences:1.0.0-alpha02"
 
  // Proto DataStore
  implementation "androidx.datastore:datastore-core:1.0.0-alpha02"
}
```
#### DataStore的使用
DataStore的使用分为两种**Preferences DataStore**和**Proto DataStore**，其中**Preferences DataStore**是键值对，是由DataStore和Preferences实现，用于存储简单的键值对到磁盘

* Preferences DataStore的创建
```
private val DATASTORE_PREFERENCE_NAME = "DataStorePreference"//定义 DataStore 的名字
mDataStorePre = this.createDataStore(
    name = DATASTORE_PREFERENCE_NAME
)
```
**createDataStore**是Context的扩展方法，源码如下所示
```
fun Context.createDataStore(
    name: String,
    corruptionHandler: ReplaceFileCorruptionHandler<Preferences>? = null,
    migrations: List<DataMigration<Preferences>> = listOf(),
    scope: CoroutineScope = CoroutineScope(Dispatchers.IO + SupervisorJob())
): DataStore<Preferences> =
    PreferenceDataStoreFactory.create(
        produceFile = {
            File(this.filesDir, "datastore/$name.preferences_pb")
        },
        corruptionHandler = corruptionHandler,
        migrations = migrations,
        scope = scope
    )
```

* Preferences DataStore 数据的写入和读取
```
    private suspend fun savePreInfo(value: String) {
        var preKey = preferencesKey<String>(PREFERENCE_KEY_NAME)
        mDataStorePre.edit { mutablePreferences ->
            mutablePreferences[preKey] = value
        }
    }
 
    private suspend fun readPreInfo(): String {
        var preKey = preferencesKey<String>(PREFERENCE_KEY_NAME)
        var value = mDataStorePre.data.map { preferences ->
            preferences[preKey] ?: ""
        }
        return value.first()
    }
```
**Preferences DataStore**以键值对的形式存储在本地，首先应该定义一个key
```
var preKey = preferencesKey<String>(PREFERENCE_KEY_NAME)
```
key的类型是**Preferences.Key<T>**，但只支持Int,String,Boolean,Float,Long类型
```
inline fun <reified T : Any> preferencesKey(name: String): Preferences.Key<T> {
    return when (T::class) {
        Int::class -> {
            Preferences.Key<T>(name)
        }
        String::class -> {
            Preferences.Key<T>(name)
        }
        Boolean::class -> {
            Preferences.Key<T>(name)
        }
        Float::class -> {
            Preferences.Key<T>(name)
        }
        Long::class -> {
            Preferences.Key<T>(name)
        }
        Set::class -> {
            throw IllegalArgumentException("Use `preferencesSetKey` to create keys for Sets.")
        }
        else -> {
            throw IllegalArgumentException("Type not supported: ${T::class.java}")
        }
    }
}
```
**Preferences DataStore**中是通过**DataStore.edit()**写入数据，edit方法是一个suspend函数，必须在携程中调用，通过**DataStore.data**去读取数据，返回的是一个**Flow<T>**
```
lifecycleScope.launch{
  savePreInfo(textPre)
}
lifecycleScope.launch{
  val value = readPreInfo()
}
```

* 从SharedPreference迁移数据
DataStore的目的是为了取代SharedPreference，对于老项目来说，需要从SharedPreference中进行数据迁移。在createDataStore方法中，我们传递的参数包括了一个这样的参数
```
migrations: List<DataMigration<Preferences>> = listOf()
```
那么我们只需要在使用create方法时，将该参数设置即可
```
mDataStorePre = this.createDataStore(
  name = DATASTORE_PREFERENCE_NAME,
  migrations = listOf(SharedPreferencesMigration(this, SP_PREFERENCE_NAME))
)
```

> 其中**SP_PREFERENCE_NAME**SP中存储文件的文件名

除了需要存储一些基本类型之外，我们是有时候还要存储对象，之前使用SP时，是将对象序列化或者转换成Json字符串之后存储。但是现在我们可以统一使用**Proto DataStore**来存储

* Proto DataStore

Proto DataStore存储数据是采用[Protocol Buffers](https://developers.google.cn/protocol-buffers),关于这个东西，只要知道是一种数据描述性语言，类似于XML，能够将结构化数据序列化，可用于数据存储，通信协议等方面，不过通XML相比，Protocol Buffers还是有很多有点的
- 更简单
- 描述文件只需要原来的1/10至1/3
- 解析速度是原来的20至100倍
- 减少了二义性
- 生成了更容易在编程中使用的数据访问类

* Proto DataStore创建
在创建Proto DataStore的时候，在AndroidStudio中必须做如下配置，添加依赖
```
classpath 'com.google.protobuf:protobuf-gradle-plugin:0.8.8'
```
在app的build.gradle中，修改的比较多，如下所示
```
apply plugin: 'com.android.application'
apply plugin: 'kotlin-android'
apply plugin: 'kotlin-android-extensions'
apply plugin: 'com.google.protobuf'
 
android {
    compileSdkVersion 30
    buildToolsVersion "30.0.2"
 
    defaultConfig {
        applicationId "cn.zzw.datastore"
        minSdkVersion 21
        targetSdkVersion 30
        versionCode 1
        versionName "1.0"
 
        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
    }
 
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
    buildFeatures {
        dataBinding true
    }
    kotlinOptions {
        jvmTarget = "1.8"
    }
    sourceSets {
        main {
            proto {
                srcDir 'src/main/proto'
                include '**/*.proto'
            }
        }
    }
}
 
dependencies {
    implementation fileTree(dir: "libs", include: ["*.jar"])
    implementation "org.jetbrains.kotlin:kotlin-stdlib:$kotlin_version"
    implementation 'androidx.core:core-ktx:1.3.2'
    implementation 'androidx.appcompat:appcompat:1.2.0'
    implementation 'androidx.constraintlayout:constraintlayout:2.0.2'
    testImplementation 'junit:junit:4.12'
    androidTestImplementation 'androidx.test.ext:junit:1.1.2'
    androidTestImplementation 'androidx.test.espresso:espresso-core:3.3.0'
    // Preferences DataStore
    implementation "androidx.datastore:datastore-preferences:1.0.0-alpha02"
    // Proto DataStore
    implementation "androidx.datastore:datastore-core:1.0.0-alpha02"
    implementation "androidx.lifecycle:lifecycle-runtime-ktx:2.2.0"
    implementation "com.google.protobuf:protobuf-javalite:3.10.0"
}
 
protobuf {
    protoc {
        artifact = "com.google.protobuf:protoc:3.10.0"
    }
 
    // Generates the java Protobuf-lite code for the Protobufs in this project. See
    // https://github.com/google/protobuf-gradle-plugin#customizing-protobuf-compilation
    // for more information.
    generateProtoTasks {
        all().each { task ->
            task.builtins {
                java {
                    option 'lite'
                }
            }
        }
    }
}
```
接着在目录app/src/main/proto创建文件**user_prefs.proto**
```
syntax = "proto3";
 
option java_package = "cn.zzw.datastore";
option java_multiple_files = true;
 
message UserPreferences {
      int32 id = 1;
      string name = 2;
      int32 age = 3;
      string phone = 4;
}
```
> 执行rebuild project
创建UserPreferencesSerializer
```
package cn.paulniu.datastore
 
import androidx.datastore.Serializer
import java.io.InputStream
import java.io.OutputStream
 
object UserPreferencesSerializer : Serializer<UserPreferences> {
    override fun readFrom(input: InputStream): UserPreferences {
        return UserPreferences.parseFrom(input)
    }
 
    override fun writeTo(t: UserPreferences, output: OutputStream) = t.writeTo(output)
 
}
```
最后创建Proto DataStore
```
mDataStorePro =
    this.createDataStore(
        fileName = "user_pros.pb",
        serializer = UserPreferencesSerializer
    )
```

* 数据的写入和读取
```
    private suspend fun saveProInfo(value: String) {
        mDataStorePro.updateData { preferences ->
            preferences.toBuilder().setId(110).setName(value).setAge(39).setPhone("119120").build()
        }
    }
 
    private suspend fun readProInfo(): String {+
        val userPreferencesFlow: Flow<UserPreferences> = mDataStorePro.data
        return userPreferencesFlow.first().toString()
    }
```

调用的时候也需要配合协去执行
```
lifecycleScope.launch {
                    saveProInfo(textPre)
                }

lifecycleScope.launch{
   var value = readProInfo()
}
```

> 目前DataStore还处于alpha版本，目前还不会考虑在项目中使用它，等正式版出来之后，会考虑。所以目前还是使用SapredPreference来存储。


### 参考资料

[Android Jetpack 之 DataStore](https://blog.csdn.net/zzw0221/article/details/109274610)

