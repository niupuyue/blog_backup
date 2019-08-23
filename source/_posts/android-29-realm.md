---
title: 重拾android路(二十九) Realm数据库
date: 2019-07-14 11:13:26
tags:
  - android
  - realm
---

Realm简介
<!--more-->
数据库Realm，是用来替代sqlite的一种解决方案，它有一套自己的数据库存储引擎，比sqlite更轻量级，拥有更快的速度，并且具有很多现代数据库的特性，比如支持JSON，流式api，数据变更通知，自动数据同步,简单身份验证，访问控制，事件处理，最重要的是跨平台，目前已有Java，Objective C，Swift，React-Native，Xamarin这五种实现。

# 环境配置

1. 在项目中build文件加上
```
buildscript {
 repositories {
     jcenter()
 }
 dependencies {
     ...
     classpath "io.realm:realm-gradle-plugin:1.2.0"
 }
```
如下图所示：
![配置在build](/assets/realm/realm01.png)

2. 在app的build文件中加入如下内容
```
apply plugin: 'realm-android'
```
如下图所示
![配置在app中的build](/assets/realm/realm02.png)

# 初始化Realm

1. 在Application中初始化Realm

```
public class MyApplication extends Application {
  @Override
  public void onCreate() {
    super.onCreate();
    Realm.init(this);
  }
}
```
2. 在Application的onCreate方法中对Realm进行初始化

- 使用默认配置

```
public class MyApplication extends Application {
  @Override
  public void onCreate() {
    super.onCreate();
    // The Realm file will be located in Context.getFilesDir() with name "default.realm"
    Realm.init(this);
    RealmConfiguration config = new RealmConfiguration.Builder().build();
    Realm.setDefaultConfiguration(config);
  }
}
```

- 使用默认配置

```
public class MyApplication extends Application {
  @Override 
  public void onCreate() {
    super.onCreate();
    Realm.init(this);
    RealmConfiguration config = new  RealmConfiguration.Builder()
                                         .name("myRealm.realm")
                                         .deleteRealmIfMigrationNeeded()
                                         .build();
    Realm.setDefaultConfiguration(config);
  }
}
```
下面是我的配置：
![Realm配置](/assets/realm/realm03.png)

# 创建实体

1. 创建一个类继承RealmObject

```
public class AffairModel extends RealmObject implements Serializable {

    @PrimaryKey
    public int id;

    public String affairNote;

    public long affairTime = 0L;

    public int affairType = 0;

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getAffairNote() {
        return affairNote;
    }

    public void setAffairNote(String affairNote) {
        this.affairNote = affairNote;
    }

    public long getAffairTime() {
        return affairTime;
    }

    public void setAffairTime(long affairTime) {
        this.affairTime = affairTime;
    }

    public int getAffairType() {
        return affairType;
    }

    public void setAffairType(int affairType) {
        this.affairType = affairType;
    }
}
```
如果存在多对多的关系
```
public class Contact extends RealmObject {
    public String name;
    public RealmList<Email> emails;
}
 
public class Email extends RealmObject {
    public String address;
    public boolean active;
}
```

2. 其他相关说明

- 支持数据类型

boolean, byte, short, int, long, float, double, String, Date and byte[]
在Realm中byte, short, int, long最终都被映射成long类型

- 注解说明

#### @PrimaryKey

①字段必须是String、 integer、byte、short、 int、long 以及它们的封装类Byte, Short, Integer, and Long

②使用了该注解之后可以使用copyToRealmOrUpdate()方法，通过主键查询它的对象，如果查询到了，则更新它，否则新建一个对象来代替。

③使用了该注解将默认设置（@index）注解

④使用了该注解之后，创建和更新数据将会慢一点，查询数据会快一点。

#### @Required

数据不能为null

#### @Ignore

忽略，即该字段不被存储到本地

#### @Index

为这个字段添加一个搜索引擎，这将使插入数据变慢、数据增大，但是查询会变快。建议在需要优化读取性能的情况下使用。

# 数据库操作(增删改查) 非异步操作

## 曾

### 事务操作一

创建一个对象，并进行存储

```
Realm realm=Realm.getDefaultInstance();
realm.beginTransaction();
User user = realm.createObject(User.class); // Create a new object
user.setName("John");
user.setEmail("john@corporation.com");
realm.commitTransaction();
```

复制一对象进行存储
```
Realm realm=Realm.getDefaultInstance();
 
User user = new User("John");
user.setEmail("john@corporation.com");
 
// Copy the object to Realm. Any further changes must happen on realmUser
realm.beginTransaction();
realm.copyToRealm(user);
realm.commitTransaction();
```

### 事务操作二

```
Realm  mRealm=Realm.getDefaultInstance();
 
final User user = new User("John");
user.setEmail("john@corporation.com");
 
mRealm.executeTransaction(new Realm.Transaction() {
            @Override
            public void execute(Realm realm) {
            
            realm.copyToRealm(user);
               
            }
        });
```

## 删除

```
Realm  mRealm=Realm.getDefaultInstance();
 
    final RealmResults<Dog> dogs=  mRealm.where(Dog.class).findAll();
 
        mRealm.executeTransaction(new Realm.Transaction() {
            @Override
            public void execute(Realm realm) {
            
                Dog dog=dogs.get(5);
                dog.deleteFromRealm();
                //删除第一个数据
                dogs.deleteFirstFromRealm();
                //删除最后一个数据
                dogs.deleteLastFromRealm();
                //删除位置为1的数据
                dogs.deleteFromRealm(1);
                //删除所有数据
                dogs.deleteAllFromRealm();
            }
        });
```

> 同样也可以使用同上的beginTransaction和commitTransaction方法进行删除

## 改

```
Realm  mRealm=Realm.getDefaultInstance();
 
Dog dog = mRealm.where(Dog.class).equalTo("id", id).findFirst();
mRealm.beginTransaction();
dog.setName(newName);
mRealm.commitTransaction();
```

> 也可以用事物块来更新数据

## 查

### 查询全部

查询结果为RealmResults

```
public List<Dog> queryAllDog() {
        Realm  mRealm=Realm.getDefaultInstance();
    
        RealmResults<Dog> dogs = mRealm.where(Dog.class).findAll();
        
        return mRealm.copyFromRealm(dogs);
    }
```

### 条件查询

```
public Dog queryDogById(String id) {
        Realm  mRealm=Realm.getDefaultInstance();
    
        Dog dog = mRealm.where(Dog.class).equalTo("id", id).findFirst();
        return dog;
    }
```

常见的条件如下（详细资料请查官方文档）：

between(), greaterThan(), lessThan(), greaterThanOrEqualTo() & lessThanOrEqualTo()

equalTo() & notEqualTo()

contains(), beginsWith() & endsWith()

isNull() & isNotNull()

isEmpty() & isNotEmpty()

### 对查询结果进行排序

```
/**
     * query （查询所有）
     */
    public List<Dog> queryAllDog() {
        RealmResults<Dog> dogs = mRealm.where(Dog.class).findAll();
        /**
         * 对查询结果，按Id进行排序，只能对查询结果进行排序
         */
        //增序排列
        dogs=dogs.sort("id");
        //降序排列
        dogs=dogs.sort("id", Sort.DESCENDING);
        return mRealm.copyFromRealm(dogs);
    }
```

### 其他查询

sum，min，max，average只支持整型数据字段

```
**
     *  查询平均年龄
     */
    private void getAverageAge() {
         double avgAge=  mRealm.where(Dog.class).findAll().average("age");
    }
 
    /**
     *  查询总年龄
     */
    private void getSumAge() {
      Number sum=  mRealm.where(Dog.class).findAll().sum("age");
        int sumAge=sum.intValue();
    }
 
    /**
     *  查询最大年龄
     */
    private void getMaxId(){
      Number max=  mRealm.where(Dog.class).findAll().max("age");
        int maxAge=max.intValue();
    }
```

# 数据库操作(增删改查)  异步操作

大多数情况下，Realm的增删改查操作足够快，可以在UI线程中执行操作。但是如果遇到较复杂的增删改查，或增删改查操作的数据较多时，就可以子线程进行操作。

## 异步增

```
private void addCat(final Cat cat) {
      RealmAsyncTask  addTask=  mRealm.executeTransactionAsync(new Realm.Transaction() {
            @Override
            public void execute(Realm realm) {
                realm.copyToRealm(cat);
            }
        }, new Realm.Transaction.OnSuccess() {
            @Override
            public void onSuccess() {
                ToastUtil.showShortToast(mContext,"收藏成功");
            }
        }, new Realm.Transaction.OnError() {
            @Override
            public void onError(Throwable error) {
                ToastUtil.showShortToast(mContext,"收藏失败");
            }
        });
 
    }
```

最后在销毁Activity或Fragment时，要取消掉异步任务

```
@Override
    protected void onDestroy() {
        super.onDestroy();
       if (addTask!=null&&!addTask.isCancelled()){
            addTask.cancel();
        }
    }
```

## 异步删

```
private void deleteCat(final String id, final ImageView imageView){
      RealmAsyncTask  deleteTask=   mRealm.executeTransactionAsync(new Realm.Transaction() {
            @Override
            public void execute(Realm realm) {
                Cat cat=realm.where(Cat.class).equalTo("id",id).findFirst();
                cat.deleteFromRealm();
 
            }
        }, new Realm.Transaction.OnSuccess() {
            @Override
            public void onSuccess() {
                ToastUtil.showShortToast(mContext,"取消收藏成功");
            }
        }, new Realm.Transaction.OnError() {
            @Override
            public void onError(Throwable error) {
                ToastUtil.showShortToast(mContext,"取消收藏失败");
            }
        });
 
    }
```

最后在销毁Activity或Fragment时，要取消掉异步任务

```
@Override
    protected void onDestroy() {
        super.onDestroy();
       if (deleteTask!=null&&!addTask.isCancelled()){
            deleteTask.cancel();
        }
    }
```

## 异步改

```
RealmAsyncTask  updateTask=   mRealm.executeTransactionAsync(new Realm.Transaction() {
            @Override
            public void execute(Realm realm) {
                Cat cat=realm.where(Cat.class).equalTo("id",mId).findFirst();
                cat.setName(name);
            }
        }, new Realm.Transaction.OnSuccess() {
            @Override
            public void onSuccess() {
                ToastUtil.showShortToast(UpdateCatActivity.this,"更新成功");
             
            }
        }, new Realm.Transaction.OnError() {
            @Override
            public void onError(Throwable error) {
                ToastUtil.showShortToast(UpdateCatActivity.this,"失败成功");
            }
        });
```

最后在销毁Activity或Fragment时，要取消掉异步任务

```
@Override
    protected void onDestroy() {
        super.onDestroy();
       if (updateTask!=null&&!addTask.isCancelled()){
            updateTask.cancel();
        }
    }
```

## 异步查

```
RealmResults<Cat>   cats=mRealm.where(Cat.class).findAllAsync();
        cats.addChangeListener(new RealmChangeListener<RealmResults<Cat>>() {
            @Override
            public void onChange(RealmResults<Cat> element) {
               element= element.sort("id");
                List<Cat> datas=mRealm.copyFromRealm(element);
              
            }
        });
```

最后在销毁Activity或Fragment时，要取消掉异步任务

```
 @Override
    protected void onDestroy() {
        super.onDestroy();
        cats.removeChangeListeners();
  
    }
```

# 使用json

在实际开发中我们和json打交道的机会比较多，所以直接从json去创建对象是十分有用的，下面的代码片段展示了怎么去用。

```
private void testAddFromJson() {

   realm.executeTransaction(new Realm.Transaction() {

        @Override
        public void execute(Realm realm) {
            String json = "{\n" +
                    "    \"id\": \"uuid1\",\n" +
                    "    \"name\": \"solid\",\n" +
                    "    \"age\": 20\n" +
                    "}";
            String jsons = "[\n" +
                    "    {\n" +
                    "        \"id\": \"uuid1\",\n" +
                    "        \"name\": \"solid\",\n" +
                    "        \"age\": 20\n" +
                    "    },\n" +
                    "    {\n" +
                    "        \"id\": \"uuid2\",\n" +
                    "        \"name\": \"jhack\",\n" +
                    "        \"age\": 21\n" +
                    "    },\n" +
                    "    {\n" +
                    "        \"id\": \"uuid3\",\n" +
                    "        \"name\": \"tom\",\n" +
                    "        \"age\": 22\n" +
                    "    }\n" +
                    "]";
            //realm.createObjectFromJson(User.class, json);
            realm.createAllFromJson(User.class, jsons);
        }
    });
}
```

# 数据模型改变

开发中数据模型不可能从一开始创建了，就保证后面的开发过程中不会更改，对于Realm如果其中的某个实体类改变了，而我们没有做任何的处理，就会报错，如果还处于应用的开发的初期，这无所谓，直接清空数据即可，但是如果应用已经发布了，我们就需要去寻找一种解决方案了。

这里的解决方案如下：

```
public class MyMigration implements RealmMigration {
    @Override
    public void migrate(DynamicRealm realm, long oldVersion, long newVersion) {
        Log.e(MainActivity.TAG, "oldVersion:" + oldVersion + " newVersion:" + newVersion);
 
        RealmSchema schema = realm.getSchema();
 
        if (newVersion == 2) {
            schema.get("User")
                    .addField("desc", String.class);
        }
    }
 
    @Override
    public boolean equals(Object obj) {
        return obj instanceof MyMigration;
    }
 
    @Override
    public int hashCode() {
        return super.hashCode();
    }
}
```

> 在用Realm进行操作之前需要对Realm作相关的配置操作，Realm中所有的写操作都必须在事务中进行，不然就会报错， 记得在Activity的onDestory中调用realm.close()释放资源。

# 参考资料 
[android之Realm数据库使用](https://blog.csdn.net/dengpeng_/article/details/54895066)



