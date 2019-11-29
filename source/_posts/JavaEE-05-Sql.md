---
title: JavaEE 05 JavaWeb基础(MySql)
date: 2019-10-21 21:37:03
tags:
  - JavaEE
---

常用数据库操作语言

<!--more-->

# 数据库操作

1. 创建数据库
```
create database dbname;
```

2. 创建数据库判断不存在，再创建
```
create database if not exists dbname; 
```

3. 创建数据库并制定字符集
```
create database dbname default character set gbk;
```

4. 查看所有的数据库
```
show databases;
```

5. 查看某个数据库的定义信息
```
show create database dbname;
```

6. 修改数据库默认字符集
```
alter database dbname default character set gbk;
```

7. 删除数据库
```
drop database dbname;
```

8. 查看正在使用的数据库
```
select database();
```

9. 使用/切换数据库
```
use dbname;
```

# 表操作

常用数据类型

|类型|描述|
|---|---|
|int|整型|
|double|浮点型|
|varchar|字符串型|
|date|日期类型，格式为yyyy-MM-dd，只有年月日，没有时分秒|

1. 创建表
```
create table student{
    id int,---整数
    name varchar(20),---字符串
    birthday date,---生日，日期
    insert_time timestamp --插入时间，如果没有给他赋值则为null，默认使用当前系统时间来填充
}
```
2. 查询某个数据库中所有表
```
show tables;
```

3. 查看表结构
```
desc student;
```

4. 查看创建表的SQL语句
```
show create table student;
```

5. 删除表
```
drop table student;
```

6. 判断表是否存在，如果存在则删除表
```
drop table if exists `student`;
```

7. 添加表列
```
alter table student add age int;
```

8. 修改列类型
```
alter table student modify name varchar(100);
```

9. 修改列名
```
alter table student change name stuName varchar(50);
```

10. 删除列
```
alter table student drop stuName;
```

11. 修改表名
```
rename table student to students;
```

12. 修改字符集
```
alter table students character set utf8;
```

# 数据操作

1. 插入数据
所有字段名都写出来
```
insert into students (id,stuName,age,birthday,create_time) values (1001,'张三',18,生日,null);
```
不写字段名
```
insert into students values (1002,'李四',18,生日,null);
```
插入部分数据
```
insert into students (id,stuName,age) values (1001,'张三',18);
```

> insert注意事项
> 插入数据应与字段的数据类型相同
> 数据的大小应在列的规定范围内，如不能讲一个长度为80的字符创加入到长度是40的列中
> 在values中列出数据位置必须与被加入的列的排列位置相对应。
> 字符和日期类型数据窨井盖该包含在单引号中，MySql也可以使用双引号作为分隔符
> 不指定列或使用null，表示插入空值

2. 更新表数据
不带条件的修改数据
```
update students set name='无名氏';
```
带条件修改数据
```
update students set name='王五' where id = 1001;
```

3. 删除表数据
不带条件的删除
```
delete from students;
```
带条件的删除
```
delete from students where id=1002;
```
使用truncate删除表记录(相当于将当前表删除， 然后在创建一个新表)
```
truncate table students;
```

# 查询表数据
语法：
        select
			字段列表
		from
			表名列表
		where
			条件列表
		group by
			分组字段
		having
			分组之后的条件
		order by
			排序
		limit
			分页限定

            