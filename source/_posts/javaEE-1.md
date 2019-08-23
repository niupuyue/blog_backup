---
title: JavaEE mysql
date: 2016-12-07 20:37:03
tags:
  - java
  - mysql
---

Mac下使用mysql笔记
<!--more-->
## 连接mysql
打开mysql服务，然后在终端中输入
```
mysql -u root -p 123456
```
或者
```
mysql --user=root --host=3306 --password=123456
```

## SQl语句
SQL语句是结构化查询语句
### SQL语句分类
1. 数据定义语句   DDL用来定义数据库对象，一般操作数据库，标，视图等，关键字create alter drop
2. 数据操作语句   DML对数据库中表数据进行操作，增删改查，关键字insert delete update 
3. 数据查询语句   DQL查询数据表中的记录，关键字select where from
4. 数据控制语句   DCL用来定义数据库访问权限和安全级别，以及创建用户，关键字grant

### 数据库操作 DataBase
#### 创建数据库
```
create database 数据库名
create database 数据库名 character set 字符集
```
#### 查看数据库
```
show databases 
//查看数据库时的语句
show create database 数据库名
```
#### 删除数据库
```
drop database 数据库名
```
#### 其他命令
```
//切换数据库
use 数据库名
//查看当前正在使用的数据库
select database()
```
### 表操作
#### 创建表
```
create table 表名 (
    字段名  变量  [约束],
    字段名  变量  [约束]
)
```
> 字符类型 varchar(n)
表单约束

|约束类型 | 描述 |
|-------|------|
|主键约束 | primary key 被修饰的字段，唯一，非空 |
|唯一约束 | unique 被修饰的字段，唯一 |
|非空约束 | not null 被修饰的字段不为空 |

#### 查看表
```
//查看数据库中的所有表
show tables
//查看表结构
desc 表名
```
#### 删除表
```
drop table 表名
```
#### 修改表
1. alter table 表名 add 列名  类型  约束   ----添加列
2. alter table 表名 modify 列名  类型  约束  ---修改列类型或者长度
3. alter table 表名 change 旧列名  新列名  类型  约束  ---修改列名
4. alter table 表名 drop 列名  ---删除列
5. rename table 表名 to 新表明  ---修改表名
6. alter table 表名 character set 字符集  ---修改表编码

### 字段类型

| 分类 | 类型名称 | 说明 | 存储需求 |
|----|----|----|----|
|整型| int(integer) |普通大小的整型 | 4个字节 |
|小数 | double(m,d) (m,数字长度,d，小数位数) | 双精度浮点 | 8个字节 |
|日期类型|datetime | YYYY-MM-DD HH:MM:SS |8个字节|
|日期类型|timestamp|YYYY-MM-DD HH:MM:SS |8个字节 |
|文本，二进制类型|varchar(n)|

### 插入记录
```
insert into 表名 (列名1 列名2 列名3) values (value1 value2 value3)
insert into 表名 values (value1 value2 value3...)   // 向表中所有的列插入
```
> 注意
> 1. 列名数和values后面的值的数量保持一致
> 2. 列顺序和插入值顺序一致
> 3. 列类型和插入值的顺序一致
> 4. 插入值不能超过最大长度
> 5. 值如果是字符串或日期，需要添加''符号

### 更新记录
```
update 表名 set 字段名=值，字段名=值，字段名=值 
update 表名 set 字段名=值，字段名=值，字段名=值 where 条件
```
> 注意
> 1. 列明类型和修改值一致
> 2. 修改值是不得超过最大长度
> 3. 值如果是字符串或日期，添加''符号

### 删除记录
```
delete from 表名 where 条件
```

### 查询记录
#### 简单查询
1. 查询所有商品 select * from product
2. 查询商品和价格 select pname,price from product
3. 别名查询 select * from product as p    select price as money from product
4. 去掉重复值 select distinct price from product

#### 条件查询
1. 查询商品名称是"海尔"的商品 select * from product where pname = '海尔'
2. 查询商品价格大于60的  select * from product where price>60
在where中可以使用运算符

| 运算符 | 标识 | 描述 |
|----|----|----|
| 比较运算符 | > < >= <= <>  | 无 |
|比较运算符 | between ... and .. | 在某一个区间的值 包含|
| 比较运算符 | in(set)| 显示在in列表中的值 |
|比较运算符 | like '张 pattern' | 模糊查询 %标识零个或多个任意字符  _表示一个字符 |
| 比较运算符 | is null | 判断是否为空 |
| 逻辑运算符 | and | 多个条件同时成立 |
| 逻辑运算符 | or| 多个条件成立一个即可 |
|逻辑运算符 | not | 不成立 |

#### 排序
1. asc  升序
2. desc  降序

#### 聚合
常用的聚合函数有 sum(),avg(),max(),min()

#### 分组
1. 根据cid分组，并且计算每组的个数  select cid,count(*) from product group by cid;
2. 根据cid分组，并且计算每组的平均价格，并且打印出大于60的数据 select cid,avg(price) from product group by cid having avg(price) > 60

> 查询语句总结:
> select distince * |字段... from 表 where 查询条件 group by 分组字段 having 分组条件 order by 升序|降序

#### 实现外键联机删除
创建两个数据表persons和orders
```
create table persons (
  p_id int primary key,
  p_name varchar(10),
  order_id int
);
```
```
create table orders(
  order_id int primary key,
  order_name varchar(20)
);
```
如上所示两个表，其中persons中的order_id是外键，orders中的order_id是主键，我们的要求是如果删除orders中的一个数据，那么受外键影响的对象也要被删除，需要在外键的表中添加如下的代码
```
alter table persons add constraint fk_persons foreign key (order_id) references orders (order_id) ON DELETE CASCADE;
```

### sql语句练习



