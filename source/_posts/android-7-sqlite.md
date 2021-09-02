---
title: 重拾Android之路(七) sqlite
date: 2016-04-30 23:23:55
tags:
  - android
---

之前在一个项目中使用到了sqlite数据库。说实话，在此之前真的没有用过，感觉在一般情况下，客户并没有要求要使用sqlite数据库，或者我也没有觉得要使用sqlite数据库的地方。不过，前一段时间，有一个需要是，在用户没有登录的情况下，可以将商品添加到购物车，然后在登陆之后，自动把之前没有登录时添加到购物车的物品自动添加到购物车中。这种情况下，我就想到了用sqlite保存数据，然后在用户登录的时候，再从数据库中拿出来，上传到服务器端，并且刷新购物车。
<!--more-->
由于之前的项目是真实项目，已经交付，无法显示代码，所以，今天我把之前用到的内容，在这里回顾一下，顺便复习sqlite数据库的知识。其中不会涉及任何真实案例，望请谅解。
# SQLite

## sqlite的基本sql语句
> 对于sqlite，最大的特点就是他可以允许我们存入数据而不需要关系数据的类型

例如：
1. 我们可以在Interget类型中存放字符串，boolean值等等各种各样的数据，但是有一中情况是例外的，就是如果一个
   字段被<pre>INTEGER PRIMARY KEY</pre>修饰的时候，只能存储64位整数。
2. 当sqlite解析数据库语句的时候，会自动忽略 CREATE TABLE 语句中跟在字段名后面的数据类型信息
3. sqlite可以解析大部分的sql语句
> select * from 表名 where 条件子句 group by 分组字句 having ... order by 排序子句 (顺序一定不能错)
```
        select * from person

        select * from person order by id desc

        select name from person group by name having count(*)>1
```
4. 分页SQL与mysql类似，下面SQL语句获取5条记录，跳过前面3条记录
```
select * from Account limit 5 offset 3 
```
5. 插入语句
```
insert into person(name, age) values(‘小明’,3)
```
6. 更新语句
```
update person set name=‘小明‘ where id=10
```
7. 删除语句
```
delete from person  where id=10
```
## 数据库版本管理
我们可以通过SqliteOpenHelper对数据库进行版本管理。
我们在编写数据库应用软件时，需要考虑这样的问题：因为我们开发的软件可能会安装在很多用户的手机上，如果应用使用到了SQLite数据库，我们必须在用户初次使用软件时创建出应用使用到的数据库表结构及添加一些初始化记录，另外在软件升级的时候，也需要对数据表结构进行更新。那么，我们如何才能实现在用户初次使用或升级软件时自动在用户的手机上创建出应用需要的数据库表呢？总不能让我们在每个需要安装此软件的手机上通过手工方式创建数据库表吧？因为这种需求是每个数据库应用都要面临的，所以在Android系统，为我们提供了一个名为SQLiteOpenHelper的抽象类，必须继承它才能使用，它是通过对数据库版本进行管理来实现前面提出的需求。 

为了实现对数据库版本进行管理，SQLiteOpenHelper类提供了两个重要的方法，分别是
onCreate(SQLiteDatabase db)和onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion)。前者用于初次使用软件时生成数据库表，后者用于升级软件时更新数据库表结构。当调用SQLiteOpenHelper的getWritableDatabase()或者getReadableDatabase()方法获取用于操作数据库的SQLiteDatabase实例的时候，如果数据库不存在，Android系统会自动生成一个数据库，接着调用onCreate()方法，onCreate()方法在初次生成数据库时才会被调用，在onCreate()方法里可以生成数据库表结构及添加一些应用使用到的初始化数据。onUpgrade()方法在数据库的版本发生变化时会被调用，一般在软件升级时才需改变版本号，而数据库的版本是由程序员控制的，假设数据库现在的版本是1，由于业务的变更，修改了数据库表结构，这时候就需要升级软件，升级软件时希望更新用户手机里的数据库表结构，为了实现这一目的，可以把原来的数据库版本设置为2(有同学问设置为3行不行？当然可以，如果你愿意，设置为100也行)，并且在onUpgrade()方法里面实现表结构的更新。当软件的版本升级次数比较多，这时在onUpgrade()方法里面可以根据原版号和目标版本号进行判断，然后作出相应的表结构及数据更新。

> getWritableDatabase()和getReadableDatabase()方法都可以获取一个用于操作数据库的SQLiteDatabase实例。但getWritableDatabase() 方法以读写方式打开数据库，一旦数据库的磁盘空间满了，数据库就只能读而不能写，倘若使用getWritableDatabase()打开数据库就会出错。getReadableDatabase()方法先以读写方式打开数据库，如果数据库的磁盘空间满了，就会打开失败，当打开失败后会继续尝试以只读方式打开数据库。

实例：
```
public class DatabaseHelper extends SQLiteOpenHelper {  
  
        //类没有实例化,是不能用作父类构造器的参数,必须声明为静态  
  
         private static final String name = "count"; //数据库名称  
  
         private static final int version = 1; //数据库版本  
  
         public DatabaseHelper(Context context) {  
  
              //第三个参数CursorFactory指定在执行查询时获得一个游标实例的工厂类,设置为null,代表使用系统默认的工厂类  
  
                super(context, name, null, version);  
  
         }  
  
        @Override  
        public void onCreate(SQLiteDatabase db) {  
  
              db.execSQL("CREATE TABLE IF NOT EXISTS person (personid integer primary key autoincrement, name varchar(20), age INTEGER)");     
  
         }  
  
        @Override   
        public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {  
  
               db.execSQL("ALTER TABLE person ADD phone VARCHAR(12)"); //往表中增加一列  
  
         }  
}  
```
> 当数据库表结构发生更新时，应该避免用户存放于数据库中的数据丢失

### 操作数据库
使用SqliteDataBase操作sqlite数据库
Android提供了一个名为SQLiteDatabase的类，该类封装了一些操作数据库的API，使用该类可以完成对数据进行添加(Create)、查询(Retrieve)、更新(Update)和删除(Delete)操作（这些操作简称为CRUD）。对SQLiteDatabase的学习，我们应该重点掌握execSQL()和rawQuery()方法。 execSQL()方法可以执行insert、delete、update和CREATE TABLE之类有更改行为的SQL语句； rawQuery()方法用于执行select语句

execSQL()方法的使用：
```
SQLiteDatabase db = ....;  
  
db.execSQL("insert into person(name, age) values('炸死特', 4)");  
  
db.close();  
```
执行上面SQL语句会往person表中添加进一条记录，在实际应用中， 语句中的“炸死特”这些参数值会由用户输入界面提供，如果把用户输入的内容原样组拼到上面的insert语句， 当用户输入的内容含有单引号时，组拼出来的SQL语句就会存在语法错误。要解决这个问题需要对单引号进行转义，也就是把单引号转换成两个单引号。有些时候用户往往还会输入像“ & ”这些特殊SQL符号，为保证组拼好的SQL语句语法正确，必须对SQL语句中的这些特殊SQL符号都进行转义，显然，对每条SQL语句都做这样的处理工作是比较烦琐的。 SQLiteDatabase类提供了一个重载后的execSQL(String sql, Object[] bindArgs)方法，使用这个方法可以解决前面提到的问题，因为这个方法支持使用占位符参数(?)
例如：
```
SQLiteDatabase db = ....;  
  
db.execSQL("insert into person(name, age) values(?,?)", new Object[]{"炸死特", 4});   
  
db.close();
```
execSQL(String sql, Object[] bindArgs)方法的第一个参数为SQL语句，第二个参数为SQL语句中占位符参数的值，参数值在数组中的顺序要和占位符的位置对应

SQLiteDatabase的rawQuery() 用于执行select语句，使用例子如下
```
SQLiteDatabase db = ....;  
  
Cursor cursor = db.rawQuery(“select * from person”, null);  
  
while (cursor.moveToNext()) {  
  
  int personid = cursor.getInt(0); //获取第一列的值,第一列的索引从0开始  
  
  String name = cursor.getString(1);//获取第二列的值  
  
  int age = cursor.getInt(2);//获取第三列的值  
  
}  
  
cursor.close();  
  
db.close();  
```
rawQuery()方法的第一个参数为select语句；第二个参数为select语句中占位符参数的值，如果select语句没有使用占位符，该参数可以设置为null。带占位符参数的select语句使用例子如下
```
Cursor cursor = db.rawQuery("select * from person where name like ? and age=?", new String[]{"%炸死特%", "4"});  
```
Cursor是结果集游标，用于对结果集进行随机访问，如果大家熟悉jdbc， 其实Cursor与JDBC中的ResultSet作用很相似。使用moveToNext()方法可以将游标从当前行移动到下一行，如果已经移过了结果集的最后一行，返回结果为false，否则为true。另外Cursor 还有常用的moveToPrevious()方法（用于将游标从当前行移动到上一行，如果已经移过了结果集的第一行，返回值为false，否则为true ）、moveToFirst()方法（用于将游标移动到结果集的第一行，如果结果集为空，返回值为false，否则为true ）和moveToLast()方法（用于将游标移动到结果集的最后一行，如果结果集为空，返回值为false，否则为true ） 。

除了前面给大家介绍的execSQL()和rawQuery()方法， SQLiteDatabase还专门提供了对应于添加、删除、更新、查询的操作方法： insert()、delete()、update()和query() 。这些方法实际上是给那些不太了解SQL语法的菜鸟使用的，对于熟悉SQL语法的程序员而言，直接使用execSQL()和rawQuery()方法执行SQL语句就能完成数据的添加、删除、更新、查询操作。

Insert()方法用于添加数据，各个字段的数据使用ContentValues进行存放。 ContentValues类似于MAP，相对于MAP，它提供了存取数据对应的put(String key, Xxx value)和getAsXxx(String key)方法，  key为字段名称，value为字段值，Xxx指的是各种常用的数据类型，如：String、Integer等。
```
SQLiteDatabase db = databaseHelper.getWritableDatabase();  
  
ContentValues values = new ContentValues();  
  
values.put("name", "炸死特");  
  
values.put("age", 4);  
  
long rowid = db.insert(“person”, null, values);//返回新添记录的行号，与主键id无关 
```
不管第三个参数是否包含数据，执行Insert()方法必然会添加一条记录，如果第三个参数为空，会添加一条除主键之外其他字段值为Null的记录。Insert()方法内部实际上通过构造insert SQL语句完成数据的添加，Insert()方法的第二个参数用于指定空值字段的名称，相信大家对该参数会感到疑惑，该参数的作用是什么？

是这样的：如果第三个参数values 为Null或者元素个数为0， 由于Insert()方法要求必须添加一条除了主键之外其它字段为Null值的记录，为了满足SQL语法的需要， insert语句必须给定一个字段名，如：insert into person(name) values(NULL)，倘若不给定字段名 ， insert语句就成了这样： insert into person() values()，显然这不满足标准SQL的语法。对于字段名，建议使用主键之外的字段，如果使用了INTEGER类型的主键字段，执行类似insert into person(personid) values(NULL)的insert语句后，该主键字段值也不会为NULL。如果第三个参数values 不为Null并且元素的个数大于0 ，可以把第二个参数设置为null

delete()方法的使用
```
SQLiteDatabase db = databaseHelper.getWritableDatabase();  
  
db.delete("person", "personid<?", new String[]{"2"});  
  
db.close(); 
```
上面代码用于从person表中删除personid小于2的记录

update()方法使用
```
SQLiteDatabase db = databaseHelper.getWritableDatabase();  
  
ContentValues values = new ContentValues();  
  
values.put(“name”, “炸死特”);//key为字段名，value为值  
  
db.update("person", values, "personid=?", new String[]{"1"});   
  
db.close();  
```
上面代码用于把person表中personid等于1的记录的name字段的值改为“炸死特”

query()方法

实际上是把select语句拆分成了若干个组成部分，然后作为方法的输入参数
```
SQLiteDatabase db = databaseHelper.getWritableDatabase();  
  
Cursor cursor = db.query("person", new String[]{"personid,name,age"}, "name like ?", new String[]{"%炸死特%"}, null, null, "personid desc", "1,2");  
  
while (cursor.moveToNext()) {  
  
         int personid = cursor.getInt(0); //获取第一列的值,第一列的索引从0开始  
  
          String name = cursor.getString(1);//获取第二列的值  
  
          int age = cursor.getInt(2);//获取第三列的值  
  
}  
  
cursor.close();  
  
db.close();  
```
上面代码用于从person表中查找name字段含有“炸死特”的记录，匹配的记录按personid降序排序，对排序后的结果略过第一条记录，只获取2条记录。

query(table, columns, selection, selectionArgs, groupBy, having, orderBy, limit)方法各参数的含义：

table：表名。相当于select语句from关键字后面的部分。如果是多表联合查询，可以用逗号将两个表名分开。

columns：要查询出来的列名。相当于select语句select关键字后面的部分。

selection：查询条件子句，相当于select语句where关键字后面的部分，在条件子句允许使用占位符“?”

selectionArgs：对应于selection语句中占位符的值，值在数组中的位置与占位符在语句中的位置必须一致，否则就会有异常。

groupBy：相当于select语句group by关键字后面的部分

having：相当于select语句having关键字后面的部分

orderBy：相当于select语句order by关键字后面的部分，如：personid desc, age asc;

limit：指定偏移量和获取的记录数，相当于select语句limit关键字后面的部分

### 例子：
数据库操作帮助类
```
class RestaurantHelper extends SQLiteOpenHelper {  
    private static final String DATABASE_NAME="lunchlist.db";//数据库名称  
    private static final int SCHEMA_VERSION=2;//版本号,则是升级之后的,升级方法请看onUpgrade方法里面的判断  
      
    public RestaurantHelper(Context context) {//构造函数,接收上下文作为参数,直接调用的父类的构造函数  
        super(context, DATABASE_NAME, null, SCHEMA_VERSION);  
    }  
      
    @Override  
    public void onCreate(SQLiteDatabase db) {//创建的是一个午餐订餐的列表,id,菜名,地址等等  
        db.execSQL("CREATE TABLE restaurants (_id INTEGER PRIMARY KEY AUTOINCREMENT, name TEXT, address TEXT, type TEXT, notes TEXT, phone TEXT);");  
    }  
  
    @Override  
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {  
        if (oldVersion==1 && newVersion==2) {//升级判断,如果再升级就要再加两个判断,从1到3,从2到3  
            db.execSQL("ALTER TABLE restaurants ADD phone TEXT;");  
        }  
    }  
  
    public Cursor getAll(String where, String orderBy) {//返回表中的数据,where是调用时候传进来的搜索内容,orderby是设置中传进来的列表排序类型  
        StringBuilder buf=new StringBuilder("SELECT _id, name, address, type, notes, phone FROM restaurants");  
          
        if (where!=null) {  
            buf.append(" WHERE ");  
            buf.append(where);  
        }  
          
        if (orderBy!=null) {  
            buf.append(" ORDER BY ");  
            buf.append(orderBy);  
        }  
          
        return(getReadableDatabase().rawQuery(buf.toString(), null));  
    }  
      
    public Cursor getById(String id) {//根据点击事件获取id,查询数据库  
        String[] args={id};  
  
        return(getReadableDatabase()  
                        .rawQuery("SELECT _id, name, address, type, notes, phone FROM restaurants WHERE _ID=?",  
                                            args));  
    }  
      
    public void insert(String name, String address, String type, String notes, String phone) {  
        ContentValues cv=new ContentValues();  
                      
        cv.put("name", name);  
        cv.put("address", address);  
        cv.put("type", type);  
        cv.put("notes", notes);  
        cv.put("phone", phone);  
          
        getWritableDatabase().insert("restaurants", "name", cv);  
    }  
      
    public void update(String id, String name, String address,  
                                         String type, String notes, String phone) {  
        ContentValues cv=new ContentValues();  
        String[] args={id};  
          
        cv.put("name", name);  
        cv.put("address", address);  
        cv.put("type", type);  
        cv.put("notes", notes);  
        cv.put("phone", phone);  
          
        getWritableDatabase().update("restaurants", cv, "_ID=?",  
                                                                 args);  
    }  
      
    public String getName(Cursor c) {  
        return(c.getString(1));  
    }  
      
    public String getAddress(Cursor c) {  
        return(c.getString(2));  
    }  
      
    public String getType(Cursor c) {  
        return(c.getString(3));  
    }  
      
    public String getNotes(Cursor c) {  
        return(c.getString(4));  
    }  
      
    public String getPhone(Cursor c) {  
        return(c.getString(5));  
    }  
} 
```
简单的一个查询操作
```
public class LunchList extends ListActivity {  
 Cursor model=null;//用来存储查询的数据  
 RestaurantAdapter adapter=null;//上面自定义的数据库帮助类  
 public void onCreate(Bundle savedInstanceState) {  
   super.onCreate(savedInstanceState);  
   setContentView(R.layout.main);  
  
   helper=new RestaurantHelper(this);//参数是上下文  
  
   initList();//初始化listview列表,省略了很多,大体能明白使用数据库流程就行啦  
  
}  
  
private void initList() {  
  if (model!=null) {  
   stopManagingCursor(model);  
   model.close();  
  }  
    
  String where=null;  
  
  if (Intent.ACTION_SEARCH.equals(getIntent().getAction())) {//如果有查询指令则根据查询内容从数据库获取表中所有满足条件数据  
   where="name LIKE \"%"+getIntent().getStringExtra(SearchManager.QUERY)+"%\"";  
  }  
  model=helper.getAll(where, prefs.getString("sort_order", "name"));//调用帮助类中的方法获取数据  
  startManagingCursor(model);  
  adapter=new RestaurantAdapter(model);//设置adapter  
  setListAdapter(adapter);//在列表中显示出来  
 } 
 ```

 > 在开发中，如果我们使用个SQlite比较少，那么自己写一个比较简单的实现就可以完成我们的需求。但是如果内容比较多，比较复杂，还是建议使用第三方框架，这里说的就是greenDao;

 # GreenDao
 ## 介绍
 GreenDAO是一个开源的安卓ORM框架，能够使SQLite数据库的开发再次变得有趣。它减轻开发人员处理低级数据库需求，同时节省开发时间。 SQLite是一个令人敬畏的内嵌的关系数据库，编写SQL和解析查询结果是相当乏味和耗时的任务。通过将Java对象映射到数据库表（称为ORM，“对象/关系映射”），GreenDAO可以将它们从这些映射中释放出来，这样，您可以使用简单的面向对象的API来存储，更新，删除和查询数据库
 > 简单来说：GreenDAO 是一个将对象映射到 SQLite 数据库中的轻量且快速的 ORM 解决方案

 ## ORM介绍
 对象-关系映射（OBJECT/RELATIONALMAPPING，简称ORM），是随着面向对象的软件开发方法发展而产生的。用来把对象模型表示的对象映射到基于SQL的关系模型数据库结构中去。这样，我们在具体的操作实体对象的时候，就不需要再去和复杂的 SQL 语句打交道，只需简单的操作实体对象的属性和方法。ORM 技术是在对象和关系之间提供了一条桥梁，前台的对象型数据和数据库中的关系型的数据通过这个桥梁来相互转化
 > 简单来说:就是JavaBean和我们的数据库进行一个关系映射，一个实例对象对应数据库的一条记录，每个对象的属性则对应着数据库表的字段

 ## 添加依赖
 在项目根依赖中
 ```
 // Top-level build file where you can add configuration options common to all sub-projects/modules.

buildscript {
    
    repositories {
        google()
        jcenter()
        mavenCentral()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:3.0.0'
        classpath 'org.greenrobot:greendao-gradle-plugin:3.2.2'

        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}

allprojects {
    repositories {
        google()
        jcenter()
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}
```
在项目依赖中
```
apply plugin: 'com.android.application'
apply plugin: 'org.greenrobot.greendao'

android {
    compileSdkVersion 26
    defaultConfig {
        applicationId "com.paulniu.greendao2"
        minSdkVersion 15
        targetSdkVersion 26
        versionCode 1
        versionName "1.0"
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}
greendao {
    schemaVersion 1
    daoPackage 'com.paulniu.greendaodemo.dao'
    targetGenDir 'src/main/java'
}
dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    implementation 'com.android.support:appcompat-v7:26.1.0'
    implementation 'com.android.support.constraint:constraint-layout:1.0.2'
    testImplementation 'junit:junit:4.12'
    androidTestImplementation 'com.android.support.test:runner:1.0.1'
    androidTestImplementation 'com.android.support.test.espresso:espresso-core:3.0.1'
    compile 'org.greenrobot:greendao:3.2.2'
}
```
> 注意：GreenDao 3.0采用注解的方式来定义实体类，通过gradle插件生成相应的代码。您可以使用greenDAO Gradle插件，无需任何其他配置,但至少要设置schema的版本

greenDao配置元素支持多种配置选项
1. schemaVersion：指定数据库schema版本号，迁移等操作会用到；
2. daoPackage：通过gradle插件生成的数据库相关文件的包名，默认为你的entity所在的包名；
3. targetGenDir：自定义生成数据库文件的目录，可以将生成的文件放到我们的java目录中，而不是build中，这样就不用额外的设置资源目录了
```
greendao {
    schemaVersion 1
    daoPackage 'com.paulniu.greendaodemo.dao'
    targetGenDir 'src/main/java'
}
```
### 注解
通过GreenDao3注解的语法来定义我们的一个数据库实体类及其数据库操作方法
1. 我们先生成一个实体类
```
public class Bean {

    private String name;
    private String password;
}
```
2. 通过添加注解为我们的Bean实体类生成对应的数据库操作方法
```
@Entity
public class Bean {

    private Long _id;
    private String source;
    private String url;
}
```
> 这里是几个注解的含义，有些没有用到
- @Entity：将我们的java普通类变为一个能够被greenDAO识别的数据库类型的实体类;
- @nameInDb：在数据库中的名字，如不写则为实体中类名；
- @Id：选择一个long / Long属性作为实体ID。 在数据库方面，它是主键。 参数autoincrement是设置ID值自增；
- @NotNull：使该属性在数据库端成为“NOT NULL”列。 通常使用@NotNull标记原始类型（long，int，short，byte）是有意义的；
- @Transient：表明这个字段不会被写入数据库，只是作为一个普通的java类字段，用来临时存储数据的，不会被持久化。

3. 通过点击AndroidStudio中的MakeProject，便发现GreenDao为我们的Meizi实体类生成了对应的Getter、Setter方法以及俩个构造函数，同时在我们配置的com.ping.greendao.gen包下生成了三个对应类文件DaoMaster、DaoSession和MeiziDao，之后所有相关的数据库操作都依靠这三个文件了
```
 * Created by niupule on 2017/12/30.
 */
@Entity
public class Bean {

    private String name;
    private String password;
    @Generated(hash = 1463487377)
    public Bean(String name, String password) {
        this.name = name;
        this.password = password;
    }
    @Generated(hash = 80546095)
    public Bean() {
    }
    public String getName() {
        return this.name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public String getPassword() {
        return this.password;
    }
    public void setPassword(String password) {
        this.password = password;
    }

}
```
这里同样会生成三个文件
- DaoMaster：使用greenDAO的切入点。DaoMaster保存数据库对象（SQLiteDatabase）并管理特定模式的DAO类（而不是对象）。 它具有静态方法来创建表或将它们删除。 其内部类OpenHelper和DevOpenHelper是在SQLite数据库中创建模式的SQLiteOpenHelper实现。一个DaoMaster就代表着一个数据库的连接。
- DaoSession：管理特定模式的所有可用DAO对象，您可以使用其中一个getter方法获取。 DaoSession还为实体提供了一些通用的持久性方法，如插入，加载，更新，刷新和删除。 DaoSession可以让我们使用一些Entity的基本操作和获取Dao操作类，DaoSession可以创建多个，每一个都是属于同一个数据库连接的。
- XxxDAO：数据访问对象（DAO）持续存在并查询实体。 对于每个实体，GreenDAO生成一个DAO。 它比DaoSession有更多的持久化方法，例如：count，loadAll和insertInTx。

### GreenDaoManager
添加一个GreenDaoManager的文件，用于创建数据库，创建表
```
public class DaoManager {

    public static final String TAG = "DaoManager";

    private static final String DB_NAME = "MyBean";

    private Context context;

    //再多线程中被共享的要使用volatile
    private volatile static DaoManager daoManager = new DaoManager();
    private static DaoMaster daoMaster;
    private static DaoMaster.DevOpenHelper helper;
    private static DaoSession session;

    /**
     * 单例获取daomanager对象
     */
    public static DaoManager getInstance() {
        return daoManager;
    }

    /**
     * 初始化上下文context
     */
    public void initContext(Context context) {
        this.context = context;
    }

    /**
     * 获得数据连接，如果没有连接，则创建连接
     */
    public DaoMaster getDaoMaster() {
        if (daoMaster == null) {
            if (helper == null) {
                helper = new DaoMaster.DevOpenHelper(context, DB_NAME, null);
            }
            daoMaster = new DaoMaster(helper.getWritableDatabase());
        }
        return daoMaster;
    }

    /**
     * 完成对数据库的增删改查只需要一个接口
     */
    public DaoSession getDaoSession() {
        if (session == null) {
            if (daoMaster == null) {
                daoMaster = getDaoMaster();
            }
            session = daoMaster.newSession();
        }
        return session;
    }

    /**
     * 设置打开日志，默认是关闭的
     */
    public void setDebug() {
        QueryBuilder.LOG_SQL = true;
        QueryBuilder.LOG_VALUES = true;
    }

    /**
     * 关闭所有数据库操作
     */
    public void closeConnection() {
        closeHelper();
        closeDaoSession();
    }

    public void closeHelper() {
        if (helper != null) {
            helper.close();
            helper = null;
        }
    }

    public void closeDaoSession() {
        if (session != null) {
            session.clear();
            session = null;
        }
    }
}
```
### 编写XxxDaoUtil，用于完成对某一张数据表的具体操作——ORM操作
```
/**
 * Created by niupule on 2017/12/30.
 */

public class BeanUtil {

    private static final String TAG = BeanUtil.class.getSimpleName();

    private DaoManager manager;

    public BeanUtil(Context context){
        manager = DaoManager.getInstance();
        manager.initContext(context);
    }

    /**
     * 插入，如果没有表，先创建表
     */
    public boolean insertMeizi(Bean meizi){
        return manager.getDaoSession().getBeanDao().insert(meizi) == -1 ? false:true;
    }

    /**
     * 插入多条数据，需要在多线程中完成
     */
    public boolean insertMultMeizi(final List<Bean> meizis){
        boolean tag = false;
        try {
            manager.getDaoSession().runInTx(new Runnable() {
                @Override
                public void run() {
                    for(Bean m: meizis){
                        manager.getDaoSession().insertOrReplace(m);
                    }
                }
            });
            tag = true;
        }catch (Exception e){
            e.printStackTrace();
        }
        return tag;
    }

    /**
     * 修改一条数据
     */
    public boolean updateMeizi(Bean meizi){
        boolean tag = false;
        try {
            manager.getDaoSession().update(meizi);
            tag = true;
        }catch (Exception e){
            e.printStackTrace();
        }
        return tag;
    }

    /**
     * 删除一条记录
     */
    public boolean deleteMeizi(Bean meizi){
        boolean tag = false;
        try {
            manager.getDaoSession().delete(meizi);
            tag = true;
        }catch (Exception e){
            e.printStackTrace();
        }
        return tag;
    }

    /**
     * 删除所有记录
     */
    public boolean deleteAll(){
        boolean tag = false;
        try {
            manager.getDaoSession().deleteAll(Bean.class);
            tag = true;
        }catch (Exception e){
            e.printStackTrace();
        }
        return tag;
    }

    /**
     * 查询所有记录
     */
    public List<Bean> queryAll(){
        return manager.getDaoSession().loadAll(Bean.class);
    }

    /**
     * 按照name查询
     */
    public Bean queryMeiziById(String name){
        return manager.getDaoSession().load(Bean.class,name);
    }

    /**
     * 使用native sql语句查询
     */
    public List<Bean> queryMeiziByNativeSql(String sql, String []conditions){
        return manager.getDaoSession().queryRaw(Bean.class,sql,conditions);
    }

    /**
     * 使用querybuilder查询
     */
    public List<Bean> queryMeiziByQueryBuilder(String name){
        QueryBuilder<Bean> builder = manager.getDaoSession().getBeanDao().queryBuilder();
        return builder.where(BeanDao.Properties.Name.eq(name)).list();
    }

}
```
### 单个插入
```
new BeanUtil(this).insertMeizi(new Bean("张三","123456"));
```
### 批量插入操作
```
List<Bean> beanlist = new ArrayList<>();
 beanlist.add(new Bean( "HuaWei",
         "http://7xi8d648096_n.jpg"));
 beanlist.add(new Bean( "Apple",
         "http://7xi8d648096_n.jpg"));
 beanlist.add(new Bean( "MIUI",
         "http://7xi8d648096_n.jpg"));
 new BeanUtil(this).insertMultMeizi(beanlist);
```
### 单个更改操作：（其中原有的数据都不会保存，如果新建的对象有属性没有设置，则会为空，不为空的字段没有设置，则报错
```
Bean bean = new Bean();
bean.setName("张三");
bean.setPassword("http://baidu.jpg");
new BeanUtil(this).updateMeizi(bean);
```
### 删除某条记录操作
```
Bean bean = new Bean();
bean.setname("张三");
new BeanUtil(this).deleteMeizi(bean);
```
### 删除所有记录操作
```
new BeanUtil(this).deleteAll();
```
### 查询
1. 查询所有记录
```
Log.d("greenDao",new BeanUtil.queryAll().size()+"");
```
2. 使用native sql进行条件查询
```
String sql = "where name like ?";
    String[] condition = new String[]{"张三"};
    List<Bean> list = new BeanUtil(this).queryMeiziByNativeSql(sql, condition);
    for (Bean bean : list) {
        Log.i(TAG, bean.toString());
    }
```
3. 使用queryBuilder查询
QueryBuilder能够让你在不涉及SQL语句的情况下查询实体。写SQL有几个缺点，首先是易错的，其次是要在运行时才知道有没有问题（假如属性名是pid，你写成了id，也要到运营时才会崩溃），QueryBuilder能够在编译时检查错误（如属性的引用是否错误）
```
List<Bean> list = new BeanUtil(this).queryMeiziByQueryBuilder("张三");
for (Bean bean : list) {
    Log.i(TAG, bean.toString());
}
```

暂时先写这么多，后面在修改


# 参考资料
[GreenDao3.2.2的使用](http://blog.csdn.net/bskfnvjtlyzmv867/article/details/71250101)