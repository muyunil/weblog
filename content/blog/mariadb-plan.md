---
title: mariadb-plan
date: 2021-05-03 15:20:19.0
updated: 2022-07-31 02:00:35.623
url: /archives/mariadb-plan
categories: 
- 技术
tags: 
- Mariadb
- SQL
---



记录MariaDB-SQL的学习笔记
<!--more-->

# 数据库设计三范式

尊需三范式设计数据库表通常情况下可以得到较好的结果。

## 一范式（核心）
范式是最基础最核心的，所有表都需要满足。
i> 表必须有主键，每一个字段原子性不可在分。

一张数据表必须拥有主键，字段需要分的很细（不需要无意义的细分）。

下看一张演示表：

|学生编号|学生姓名|联系方式|
|--|--|--|
|1|lisi|lisi@163.com,13088888888|
|2|zhangsan|zs@163.com,13099999999|

可以看到这张表没有满足第一范式。
它没有主键，并且联系方式还可以在分。
按照第一范式进行优化

|学生编号(PK)|学生姓名|邮箱地址|联系电话|
|--|--|--|--|
|1|lisi|lisi@163.com|13088888888|
|2|zhangsan|zs@163.com|13099999999|

我们将编号设置为了主键，原来的联系方式细化为了邮箱地址和联系电话。
满足了第一范式！

----------

## 第二范式

i> 1.建立在第一范式基础之上。
2.要求非主键字段必须完全依赖主键，不要产生部分依赖。

看一张演示表

|学生编号|学生姓名|老师编号|老师姓名|
|--|--|--|--|
|1001|张三|001|王老师|
|1002|李四|002|赵老师|
|1003|王五|001|王老师|
|1001|张三|002|赵老师|

这张表没有满足第一范式
先让它满足第一范式

|PK(学生编号|老师编号)|学生姓名|老师姓名|
|--|--|--|--|
|1001|001|张三|王老师|
|1002|002|李四|赵老师|
|1003|001|王五|王老师|
|1001|002|张三|赵老师|
|--|--|--|--|

primary key(学生编号，老师编号）
将学生编号和老师编号联合做主键。
这样就满足了第一范式

第二范式要求非主键字段完全依赖主键，不要产生部分依赖。
这里显然产生了部分依赖，`张三：1001`，`王老师：001`，产生了数据冗余。
- 这种情况是多对多关系，需要拆分表以满足第二范式。

i> 多对对，三张表，关系表两外键。

将这张表拆分为三张表：学生表，老师表，两者关系表。
学生表：

|学生编号(PK)|学生姓名|
|--|--|
|1001|张三|
|1002|李四|
|1003|王五|

老师表:

|老师编号(PK)|老师姓名|
|--|--|
|001|王老师|
|002|赵老师|

关系表：

|id(PK)|学生编号(FK)|老师编号(FK)|
|--|--|
|1|1001|001|
|2|1002|002|
|3|1003|001|
|4|1001|002|

通过关系表外键储存学生：老师的关系。
满足了第二范式。

----------

## 第三范式
i> 1.建立在第二范式之上
2.要求所有非主键必须直接依赖主键，不要产生传递依赖。

来看一张表：

|学生编号（PK）|学生姓名|班级编号|班级名称|
|--|--|--|--|
|1001|张三|01|一年一班|
|1002|李四|02|一年二班|
|1003|王五|03|一年三班|
|1004|赵六|03|一年三班|

首先这张表满足第一范式，然后它是单一主键，非主键字段完全依赖主键，满足第二范式。
但发现班级名称字段发生了冗余，因为班级名称字段并没有直接依赖于主键，而是依赖于班级编号，而班级编辑依赖于显示编号，产生了传递依赖，不满足第三范式。

解决方法就是将冗余字段单独拿处理建表。

i> 一对多，两张表，多的表加外键。

班级表：
|班级编号(PK)|班级名称|
|--|--|
|01|一年级一班|
|02|一年级二班|
|03|一年级三班|

学生表：

|学生编号(PK)|学生姓名|班级编号(FK)|
|--|--|--|
|1001|张三|01|
|1002|李四|02|
|1003|王五|03|
|1004|赵六|03|

- 以上是典型的一对多，将一储存在一张表中，多储存在一张表中。多的表添加外键指向一的表主键。

----------

# 三范式总结

第一范式：有主键，具有原子性，字段不可分割
第二范式：完全依赖，没有部分依赖
第三范式：没有传递依赖

数据库设计尽量遵循三范式，但是还是根据实际情况进行取舍。
有时可能会拿冗余换速度，因为表连接查询时表越多效率越低（笛卡尔积），视情况满足需求即可。

- 多对多
三张表，关系表加两外键

- 一对多
两张表，多的表加外键。

- 一对一，有两种方案：
1. 主键共享
2. 外键唯一（分两表加外键（唯一性约束）