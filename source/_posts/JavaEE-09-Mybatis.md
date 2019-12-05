---
title: JavaEE 09 MyBatis
date: 2019-11-25 29:08:22
tags:
  - JavaEE
---

MyBatis是一个优秀的基于java的持久层框架，它内部封装了JDBC，使开发者只需要关注sql语句本身，不需要花费精力去处理加载驱动，创建连接，创建statement等繁杂的过程。

<!--more-->

Mybatis通过xml或者注解的方式将要执行的各种statement配置起来，并通过java对象和statement中sql的动态参数进行映射生成最终执行的sql语句，最后由Mybatis框架执行sql并将结果映射为java对象并返回
采用ORM思想解决了实体和数据库映射问题，对JDBC进行封装，屏蔽了JDBC API底层访问细节，是我们不用与JDBC API打交道，就可以完成对数据库持久化操作

> 关于JDBC的操作我们都很熟悉了，这里就不再介绍

## Mybatis框架开发准备工作
首先我们需要通过Idea创建一个Maven工程，并在Maven工程中添加MyBatis的坐标
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.paulniu</groupId>
    <artifactId>mybatis01</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>

    <dependencies>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.6</version>
        </dependency>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.4.5</version>
        </dependency>
        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>1.2.12</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.10</version>
        </dependency>
    </dependencies>

</project>
```

紧接着我们编写一个User的实体类，并且让这个实体类实现Serializable接口
```
package com.paulniu.domain;

import java.io.Serializable;
import java.util.Date;

public class User implements Serializable {

    private Integer id;
    private String username;
    private String sex;
    private Date birthday;
    private String address;


    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getSex() {
        return sex;
    }

    public void setSex(String sex) {
        this.sex = sex;
    }

    public Date getBirthday() {
        return birthday;
    }

    public void setBirthday(Date birthday) {
        this.birthday = birthday;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", username='" + username + '\'' +
                ", sex='" + sex + '\'' +
                ", birthday=" + birthday +
                ", address='" + address + '\'' +
                '}';
    }
}
```

编写持久层接口IUserDao，在这个接口中我们只需要穿件一个方法findAll
```
package com.paulniu.dao;

import com.paulniu.domain.User;

import java.util.List;

public interface IUserDao {

    List<User> findAll();

}
```

紧接着我们需要创建一个持久层接口的映射文件IUserDao.xml
这里需要注意，我们创建的改文件的位置必须和IUserDao在同样的包名中，并且名称必须和持久层接口文件的命名相同，具体的结构如下所示
![创建持久层接口映射文件](/assets/JavaEE/mybatis-01.png)

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.paulniu.dao.IUserDao">
    <select id="findAll" resultType="com.paulniu.domain.User" >
        select * from user
    </select>
</mapper>
```

编写SQLMapConfig.xml配置文件，这个配置文件相当于是配置Mybatis的配置文件，包括了数据库连接的属性，线程池，以及映射关系等
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<!-- mybatis主配置文件 -->
<configuration>
    <!-- 配置环境 -->
    <environments default="mysql">
        <environment id="mysql">
            <transactionManager type="JDBC"></transactionManager>
            <dataSource type="POOLED">
                <!-- 配置连接基本信息 -->
                <property name="driver" value="com.mysql.jdbc.Driver" />
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis_db" />
                <property name="username" value="root" />
                <property name="password" value="root" />
            </dataSource>
        </environment>
    </environments>

    <!-- 指定映射配置文件位置，映射配置文件指的是每个独立到的配置文件 -->
    <mappers>
        <mapper resource="com/paulniu/dao/IUserDao.xml" />
    </mappers>
</configuration>
```

这样我们的Mybatis的最基础的使用方法已经完成，我们这里写一个方法测试一下
```
public class MyBatisTest {

    public static void main(String[] args) throws IOException {
        InputStream is = Resources.getResourceAsStream("SqlMapConfig.xml");
        SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
        SqlSessionFactory factory = builder.build(is);
        SqlSession sqlSession = factory.openSession();
        IUserDao userDao = sqlSession.getMapper(IUserDao.class);
        List<User> users = userDao.findAll();
        for (User user : users) {
            System.out.println(user);
        }
        sqlSession.close();
        is.close();
    }
}
```

运行之后的结果如图

![运行结果](/assets/JavaEE/mybatis-02.png)

### Mybatis的增删改查
IUserDao接口
```
package com.paulniu.mybatis_01;

import com.paulniu.domain.User;

import java.util.List;

public interface IUserDao {

    List<User> findAll();

    User findById(Integer id);

    int saveUser(User user);

    int updateUser(User user);

    int deleteUser(Integer id);

}
```

IUserDao.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.paulniu.mybatis_01.IUserDao">
    <select id="findAll" resultType="com.paulniu.domain.User">
        select * from user
    </select>
    <select id="findById" resultType="com.paulniu.domain.User" parameterType="int">
        select * from user where id = #{uid}
    </select>

    <insert id="saveUser" parameterType="com.paulniu.domain.User">
        insert into user (username,birthday,sex,address) values (#{username},#{birthday},#{sex},#{address})
    </insert>

    <update id="updateUser" parameterType="com.paulniu.domain.User">
        update user set username = #{username} ,birthday=#{birthday},sex=#{sex},address=#{address} where id = #{id}
    </update>

    <delete id="deleteUser" parameterType="Integer">
        delete from user where id = #{id}
    </delete>
</mapper>
```

测试文件
```
public class Test {
    private static InputStream is;

    public static void main(String[] args) throws IOException {
        is = Resources.getResourceAsStream("SQLMapConfig.xml");
        deleteUser();
        is.close();
    }

    /**
     * 查询所有用户
     */
    public static void findAll() {
        SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
        SqlSessionFactory factory = builder.build(is);
        SqlSession session = factory.openSession();
        IUserDao userDao = session.getMapper(IUserDao.class);
        List<User> users = userDao.findAll();
        for (User user : users) {
            System.out.println("user = " + user.toString());
        }
        session.close();

    }

    /**
     * 根据id查询用户
     */
    public static void findById(){
        SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
        SqlSessionFactory factory = builder.build(is);
        SqlSession session = factory.openSession();
        IUserDao userDao = session.getMapper(IUserDao.class);
        User user = userDao.findById(41);
        System.out.println(user);
        session.close();
    }

    /**
     * 保存新用户
     */
    public static void saveUser(){
        User user = new User();
        user.setUsername("哈哈");
        user.setSex("男");
        user.setBirthday(new Date());
        user.setAddress("北京丰台");
        SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
        SqlSessionFactory factory = builder.build(is);
        SqlSession session = factory.openSession();
        IUserDao userDao = session.getMapper(IUserDao.class);
        int count = userDao.saveUser(user);
        System.out.println(count);
        session.commit();
        session.close();
    }

    /**
     * 更新用户信息
     */
    public static void updateUser(){
        User user = new User();
        user.setUsername("哈哈");
        user.setSex("男");
        user.setBirthday(new Date());
        user.setAddress("北京丰台");
        user.setId(42);
        SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
        SqlSessionFactory factory = builder.build(is);
        SqlSession session = factory.openSession();
        IUserDao userDao = session.getMapper(IUserDao.class);
        int count = userDao.updateUser(user);
        System.out.println(count);
        session.commit();
        session.close();
    }

    /**
     * 删除用户
     */
    public static void deleteUser(){
        SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
        SqlSessionFactory factory = builder.build(is);
        SqlSession session = factory.openSession();
        IUserDao userDao = session.getMapper(IUserDao.class);
        int count = userDao.deleteUser(52);
        System.out.println(count);
        session.commit();
        session.close();
    }

}
```

如果在写入数据时出现乱码问题，则需要设置编码格式
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<!-- mybatis主配置文件 -->
<configuration>
    <!-- 配置环境 -->
    <environments default="mysql">
        <environment id="mysql">
            <transactionManager type="JDBC"></transactionManager>
            <dataSource type="POOLED">
                <!-- 配置连接基本信息 -->
                <property name="driver" value="com.mysql.jdbc.Driver" />
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis_db?characterEncoding=utf8" />
                <property name="username" value="root" />
                <property name="password" value="root" />
            </dataSource>
        </environment>
    </environments>

    <!-- 指定映射配置文件位置，映射配置文件指的是每个独立到的配置文件 -->
    <mappers>
        <mapper resource="com/paulniu/mybatis_01/IUserDao.xml" />
    </mappers>
</configuration>
```

![增加数据运行结果](/assets/JavaEE/mybatis-03.png)
![删除数据运行结果](/assets/JavaEE/mybatis-04.png)
![修改数据运行结果](/assets/JavaEE/mybatis-05.png)

### 查询使用聚合函数

```
 <select id="findTotal" resultType="Integer">
        select count(*) from user;
    </select>
```

```
 /**
     * 查询数据总数
     */
    public static void findTotal(){
        SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
        SqlSessionFactory factory = builder.build(is);
        SqlSession session = factory.openSession();
        IUserDao userDao = session.getMapper(IUserDao.class);
        int count = userDao.findTotal();
        System.out.println(count);
        session.close();
    }
```

![聚合函数查询结果](/assets/JavaEE/mybatis-06.png)

### Mybatis封装

#### 结果的封装
resultType配置类型结果
resultType属性可以指定结果集类型，它支持基本类型和试题类型

resultMap结果集
resultMap标签可以创建查询的列名和实体类的属性名称不一致时创建映射关系。在select标签中使用resultMap属性指定引用即可。同时resultMap属性可以实现将查询结果映射为复杂类的pojo。比如在查询结果映射对象中包括pojo和list实现实现一对一或一对多查询。

#### 参数
parameterType配置参数，我们在使用SQL语句传参时，使用标签parameterType属性来设定。该属性的取值可以是基本类型，引用类型，还可以是实体类类型(POJO类)。同时也可以使用实体类的包装类

> 注意：基本类型和String类型我们可以直接写类型名称，也可以使用包名.类名的方式。实体类类型只能使用全限定类名

传递POJO包装对象
开发中使用POJO传递查询条件，查询条件是综合的查询条件，不仅包括用户查询条件，还包括其他的查询条件(比如将用户购买商品信息也作为查询条件)，这是可以声依永包装对象输入参数

```
package com.paulniu.domain;

import java.io.Serializable;

public class QueryVo implements Serializable {

    private User user;

    public User getUser() {
        return user;
    }

    public void setUser(User user) {
        this.user = user;
    }
}
```

接口类
```
 //根据QueryVo条件查询
    List<User> findByVo(QueryVo queryVo);
```

映射
```
<select id="findByVo" parameterType="com.paulniu.domain.QueryVo" resultType="com.paulniu.domain.User" >
        select * from user where username like #{user.username}
    </select>
```

测试类
```
 /**
     * 使用pojo封装类
     */
    public static void findByVo(){
        SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
        SqlSessionFactory factory = builder.build(is);
        SqlSession session = factory.openSession();
        IUserDao userDao = session.getMapper(IUserDao.class);
        QueryVo vo = new QueryVo();
        User user = new User();
        user.setUsername("%小%");
        vo.setUser(user);
        List<User> users = userDao.findByVo(vo);
        for (User uu:users){
            System.out.println(uu.toString());
        }
        session.close();
    }
```

![POJO封装运行结果](/assets/JavaEE/mybatis-07.png)

### SqlMapConfig.xml配置文件
配置文件的额内容和顺序
```
-properties(属性)
    --property
-settings(全局配置参数)
    --setting
-typeAliases(类型别名)
    --typeAliase
    --package
-typeHandlers(类型处理器)
-objectFactory(对象工厂)
-plugins(插件)
-environments(环境集合属性对象)
    --environment(环境子属性对象)
        ---transactionManager(事务管理)
        ---dataSource(数据源)
-mappers(映射器)
    --mapper
    --package                    
```

properties(属性)
```
<properties>
    <property name="jdbc.driver" value="com.mysql.jdbc.Driver" />
    <property name="jdbc.url" value="jdbc:mysql://localhost:3306/eesy" />
    <property name="jdbc.username" value="root" />
    <property name="jdbc.password" value="123456">
</properties>
```

> 除了上面这样的方式之外我们可以定义db.properties文件

我们使用DataSource标签完成数据库的配置工作
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<!-- mybatis主配置文件 -->
<configuration>
    <!-- 配置环境 -->
    <environments default="mysql">
        <environment id="mysql">
            <transactionManager type="JDBC"></transactionManager>
            <dataSource type="POOLED">
                <!-- 配置连接基本信息 -->
                <property name="driver" value="com.mysql.jdbc.Driver" />
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis_db?characterEncoding=utf8" />
                <property name="username" value="root" />
                <property name="password" value="root" />
            </dataSource>
        </environment>
    </environments>

    <!-- 指定映射配置文件位置，映射配置文件指的是每个独立到的配置文件 -->
    <mappers>
        <mapper resource="com/paulniu/mybatis_01/IUserDao.xml" />
    </mappers>
</configuration>
```

## Mybatis连接池和事务
在我们之前的代码中，我们会发现如果我们执行插入操作需要调用SqlSession.commit()方法，其实这个就是将事务操作。

### 连接池
Mybatis中连接池分为三类，分别是UNPOOLED，POLLED，JNDI
配置操作如下
```
<dataSource type="POOLED">
                <!-- 配置连接基本信息 -->
                <property name="driver" value="com.mysql.jdbc.Driver" />
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis_db?characterEncoding=utf8" />
                <property name="username" value="root" />
                <property name="password" value="root" />
            </dataSource>
```

1. UNPOOLED:表示Mybatis会创建UnpooledDataSource实例
2. POOLED:表示Mybatis会创建PooledDataSource实例
3. JNDI:表示Mybatis会从JNDI服务上查找DataSource实例，然后返回使用

这三种数据源中我们一般采用POOLED数据源(就是我们所说的连接池技术)

Mybatis是通过工厂模式来创建数据源DataSource对象的，Mybatis定义了抽象的工厂接口:DataSourceFactory,通过getDataSource()方法返回数据源DataSource。在创建DataSource实例后，会将其放在Configuration对象的Environment对象中，供以后使用。

### 事务控制
在JDBC中农我们可以通过手动方式将事务的提交改为手动方式，通过setAutoCommit()方法就可以调整，因为Mybatis是对JDBC的封装，所以Mybatis框架的事务控制方式本身就是JDBC的setAutoCommit()方法来设置事务提交方式的。

其实我们在执行创建，更新，删除等Sql操作必须使用sqlSession.commit()提交事务，因为在连接池中获取连接，都会调用connection.setAutoCommit(false)方法，这样我们必须调用sqlSession.commit()方法。我们可以不进行手动提交，一样实现更新，删除，创建等操作
![自定提交事务设置](/assets/JavaEE/mybatis-08.png)

### Mybatis的动态Sql语句
我们根据实体类的不同取值，使用不同的SQL语句进行查询，例如在id如果不为空的情况下根据id查询， 如果username不为空还要加上用户名作为条件
```
List<User> findByUser(User user);
```
```
<select id="findByUser" resultType="com.paulniu.domain.User" parameterType="com.paulniu.domain.User">
        select * from user where 1 = 1
        <if test="username!=null and username != ' '.toString()">
            and username like #{username}
        </if>
        <if test="address != null">
            and address like #{address}
        </if>
    </select>
```
测试代码：
```
    public static void findByUser() {
        User user = new User();
        user.setUsername("%小%");
        user.setAddress("%门头沟%");
        SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
        SqlSessionFactory factory = builder.build(is);
        SqlSession session = factory.openSession(true);
        IUserDao dao = session.getMapper(IUserDao.class);
        List<User> users = dao.findByUser(user);
        for (User uu : users) {
            System.out.println(uu.toString());
        }
        session.close();
    }
```
运行结果：
![if语句动态查询结果](/assets/JavaEE/mybatis-09.png)

使用foreach标签
传入多个id查询用户信息，使用如下两个sql实现
```
select * from users where username like '%张%' and (id = 10 or id =89 or id = 16)
select * from users where username like '%张%' and id in (10,89,16)
```
这样我们在进行范围查询时，就将集合中的值作为参数动态添加进来

我们可以使用QueryVo加入一个List集合用于封装参数
创建一个新的类QueryVo2
```
public class QueryVo2 {

    private List<Integer> ids;

    public List<Integer> getIds() {
        return ids;
    }

    public void setIds(List<Integer> ids) {
        this.ids = ids;
    }
}
```
添加接口
```
List<User> findInIds(QueryVo2 queryVo);
```
修改IUserDao.xml文件
```
<select id="findInIds" resultType="com.paulniu.domain.User" parameterType="com.paulniu.domain.QueryVo2">
        <include refid="defaultUser" />
        <where>
            <if test="ids != null and ids.size() > 0">
                <foreach collection="ids" open="and id in ( " close=")" item="uid"  separator=",">
                    #{uid}
                </foreach>
            </if>
        </where>
    </select>
```
此处为了避免重复写太多的代码，我们写了一个include标签，标签的内容如下
```
<sql id="defaultUser">
        select * from user
    </sql>
```
测试代码：
```
    public static void findInIds() {
        QueryVo2 queryVo2 = new QueryVo2();
        List<Integer> ids = new ArrayList<Integer>();
        ids.add(40);
        ids.add(42);
        ids.add(47);
        queryVo2.setIds(ids);
        SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
        SqlSessionFactory factory = builder.build(is);
        SqlSession session = factory.openSession(true);
        IUserDao userDao = session.getMapper(IUserDao.class);
        List<User> users = userDao.findInIds(queryVo2);
        for (User user : users) {
            System.out.println(user.toString());
        }
    }
```
测试结果：
![测试结果](/assets/JavaEE/mybatis-10.png)

#### 多表查询一对多
我们使用的例子比较简单，就是用户和账户的模型来分析MyBatis中多表关系，用户为User表，账户为Account表，一个用户可以拥有多个账户。我们将查询所有账户信息，关联查询下单用户信息

> 注意，因为一个账户信息只能提供给某个用户使用，所以从查询账户信息出发关联查询用户信息是一对一查询，如果从用户信息出发查询用户下的账户信息为一对多查询

先编写Account.class类
```
public class Account implements Serializable {

    private Integer id;
    private Integer uid;
    private Double money;

    public Account(Integer id, Integer uid, Double money) {
        this.id = id;
        this.uid = uid;
        this.money = money;
    }

    public Account() {
    }

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public Integer getUid() {
        return uid;
    }

    public void setUid(Integer uid) {
        this.uid = uid;
    }

    public Double getMoney() {
        return money;
    }

    public void setMoney(Double money) {
        this.money = money;
    }

    @Override
    public String toString() {
        return "Account{" +
                "id=" + id +
                ", uid=" + uid +
                ", money=" + money +
                '}';
    }
}
```
正常情况下Sql语句应该是这样的
```
select account.*,user.name,user.address from account,user where account.uid = user.id;
```
查询结果如下所示
![多表查询](/assets/JavaEE/mybatis-11.png)

为了能够多表查询，我们创建一个新的类AccountUser类
```
public class AccountUser extends Account implements Serializable {

    private String username;
    private String address;

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }
}
```
新建一个接口IAccountUserDao
```
public interface IAccountUserDao {
    List<AccountUser> findAll();

}
```

创建IAccountUserDao.xml文件
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.paulniu.mybatis_01.IAccountUserDao">
    <select id="findAll" resultType="com.paulniu.domain.AccountUser">
        select account.*,user.username,user.address from account,user where account.uid = user.id
    </select>
</mapper>
```

测试类
```
public static void findAll2(){
        SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
        SqlSessionFactory factory = builder.build(is);
        SqlSession session = factory.openSession(true);
        IAccountUserDao dao = session.getMapper(IAccountUserDao.class);
        List<AccountUser> accountUsers = dao.findAll();
        for (AccountUser user : accountUsers){
            System.out.println(user.toString());
        }
    }
```

SQLMapConfig.xml文件中添加mapper标签
```
<!-- 指定映射配置文件位置，映射配置文件指的是每个独立到的配置文件 -->
    <mappers>
        <mapper resource="com/paulniu/mybatis_01/IUserDao.xml" />
        <mapper resource="com/paulniu/mybatis_01/IAccountUserDao.xml" />
    </mappers>
```

运行结果
![运行结果](/assets/JavaEE/mybatis-12.png)

> 作为21世纪祖国的花朵，我们岂能用这种方法，也太麻烦了，毕竟每次当我们执行一个新的多表查询的时候，都要创建一个新类，这样未免有些麻烦，所以我们还有另一种方法

我们需要做的就是在刚才创建的Account类中添加一个对象，叫做User，如果所示
```
public class Account implements Serializable {

    private Integer id;
    private Integer uid;
    private Double money;

    private User user;

    public User getUser() {
        return user;
    }

    public void setUser(User user) {
        this.user = user;
    }

    public Account(Integer id, Integer uid, Double money) {
        this.id = id;
        this.uid = uid;
        this.money = money;
    }

    public Account() {
    }

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public Integer getUid() {
        return uid;
    }

    public void setUid(Integer uid) {
        this.uid = uid;
    }

    public Double getMoney() {
        return money;
    }

    public void setMoney(Double money) {
        this.money = money;
    }

    @Override
    public String toString() {
        return "Account{" +
                "id=" + id +
                ", uid=" + uid +
                ", money=" + money +
                '}';
    }
}
```

在IAccountUserDao接口文件中添加如下代码
```
    List<AccountUser> findAll2();
```
在IAccountUserDao.xml文件中添加如下
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.paulniu.mybatis_01.IAccountUserDao">
    <!--简历对应关系-->
    <resultMap id="accountMap" type="com.paulniu.domain.Account">
        <id column="aid" property="id"/>
        <result column="uid" property="uid"/>
        <result column="money" property="money"/>   <!-- 它是用于指定从表方的引用实体属性的 -->
        <association property="user" javaType="com.paulniu.domain.User">
            <id column="id" property="id"/>
            <result column="username" property="username"/>
            <result column="sex" property="sex"/>
            <result column="birthday" property="birthday"/>
            <result column="address" property="address"/>
        </association>
    </resultMap>
    <select id="findAll" resultType="com.paulniu.domain.AccountUser">
        select account.*,user.username,user.address from account,user where account.uid = user.id
    </select>

    <select id="findAll2" resultType="com.paulniu.domain.Account">
        select u.*,a.id as aid,a.uid,a.money from account a,user u where a.uid =u.id
    </select>
</mapper>
```

## Mybatis延迟加载策略

> 延迟加载：在需要使用到数据的时候才会进行加载，不需要使用到数据时不加载数据;
> 好处：先从表单查询，需要时再关联表去关联查询，大大提高数据库性能，因为查询表单需要比关联查询多张标速度款
> 坏处：因为只有当需要用到数据时才会进行数据库查询，这样在大量数据批量次查询时，因为查询工作也会消耗时间，所以可能造成用户等待时间变长

我们可以做一个例子：查询账户信息，并且关联查询用户信息。


## Mybatis缓存
像大多数持久层框架一样，Mybatis也提供了缓存策略，通过缓存策略来减少数据库查询次数，从而提高性能
Mybatis中缓存分为一级缓存，二级缓存

> 以及缓存是SqlSession级别的缓存，只要SqlSession没有flush或者close，他就一直存在

一级缓存是SqlSession范围的缓存，当调用SqlSession的修改，添加，删除，commit，close等方法时，就会清空一级缓存

二级缓存是mapper映射级别的缓存，多个SqlSession去操作同一个Mapper映射的sql语句，多个SqlSession可以共用二级缓存，二级缓存是跨SqlSession的

