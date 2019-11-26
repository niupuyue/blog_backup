---
title: JavaEE 09 MyBatis
date: 2019-11-25 29:08:22
tags:
  - JavaEE
---

MyBatis是一个优秀的基于java的持久层框架，它内部封装了JDBC，使开发者只需要关注sql语句本身，不需要花费精力去处理加载驱动，创建连接，创建statement等繁杂的过程。
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