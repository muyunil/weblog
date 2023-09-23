---
title: Mariadb-constraint
date: 2021-05-03 15:17:00.0
updated: 2022-07-31 02:01:06.342
url: /archives/mariadb-constraint
categories: 
- 技术
tags: 
- Mariadb
- SQL
---



记录MariaDB-SQL语句学习过程
<!--more-->

表在创建时可以设置约束（constraint）。
用于保证数据的完整性，可用性，有效性。

√> 约束就是为了保证数据的有效性！

有一下几种约束

非空约束： not null
唯一性约束：unique
主键约束：primary key
外键约束：foreign key
检查约束：check

但在MySQL中不支持检查约束，Oracle是支持的。

# 非空约束_not null

创建一个拥有非空约束字段的表

```sql
CREATE TABLE t_vip(
    id int,
    name varchar(32) not null
);
```
表结构如下
`MariaDB [test]> desc t_vip;`

| Field | Type        | Null | Key | Default | Extra |
|--|--|--|--|--|--|
| id    | int(11)     | YES  |     | NULL    |       |
| name  | varchar(32) | NO   |     | NULL    |       |

我们发现在name字段的null属性上显示为no，意为不允许空值。

我们往t_vip表中存储数据。

```sql
INSERT INTO t_vip(id,name) values (1,'lisi');
INSERT INTO t_vip(id,name) values (2,'zhangsan');

INSERT INTO t_vip(id) values (3);
```
先执行前两条语句  
`select * from  t_vip;`

| id   | name     |
|--|--|
|    1 | lisi     |
|    2 | zhangsan |

2 rows in set (0.004 sec)

数据插入正常，在试试第三条只插入了id值的语句
`INSERT INTO t_vip(id) values (3);`
会发现报错
ERROR 1364 (HY000): Field 'name' doesn't have a default value
插入失败，提示为name字段没有默认值。
这就是非空约束，让字段必须有值，不可以空值null。

----------

# 唯一性约束_unique

创建一个拥有唯一性约束的表

```sql
drop table if exists t_vip;

CREATE TABLE t_vip(
    id int not null,
    name varchar(32) unique,
    tmp char
);
```
查看表结构
`desc t_vip;`

| Field | Type        | Null | Key | Default | Extra |
|--|--|--|--|--|--|
| id    | int(11)     | NO   |     | NULL    |       |
| name  | varchar(32) | YES  | UNI | NULL    |       |
| tmp   | char(1)     | YES  |     | NULL    |       |

3 rows in set (0.007 sec)

i> 唯一性约束可以和非空约束一起使用即不可为空的同时不可重复，<name varchar (255) not null unique>

----------

# 表级约束（联合约束）
给多个字段联合起来添加某个约束时需要使用表级约束

i> 在字段后面添加的约束较列级约束，与字段同级添加的约束叫表级约束。

我们创建一个表，其中name与email字段联合起来具有唯一性。
```
drop table if exists t_vip;

CREATE TABLE t_vip(
    id int not null,
    name varchar(32),
    email varchar(255),
    unique(name,email)
);
```
查看表结构
`desc t_vip;`

| Field | Type         | Null | Key | Default | Extra |
|--|--|--|--|--|--|
| id    | int(11)      | NO   |     | NULL    |       |
| name  | varchar(32)  | YES  | MUL | NULL    |       |
| email | varchar(255) | YES  |     | NULL    |       |
3 rows in set (0.005 sec)

插入数据
```sql
INSERT INTO t_vip(id,name,email) values 
    (1,'lisi','lisi@123.com'),
    (2,'zhangsan','zhangsan@123.com'),
    (3,'zhangsan','zhangsan@qq.com');

INSERT INTO t_vip(id,name,email) values (4,'lisi','lisi@123.com');
```
第2条与第3条数据的name字段重合也可以插入，因为他们的name与email联合起来是不一样的，但第四条数据name与email联合起来与第一条数据重复，受到约束所以不能插入。
提示重复：`ERROR 1062 (23000): Duplicate entry 'lisi-lisi@123.com' for key 'name'`

----------

# 主键约束

i> 添加了主键约束的字段叫主键字段，字段值叫主键值，主键值具有not null和unique这两个效果的约束，主键值具有此条数据的唯一标识符作用（似身份证号码）。

!> 在MySQL里会将同时设置了not null 和 unique 这两个约束的字段视为主键。但在Oracle中不是。

实际使用中主键值类型建议使用定长类型如：int, bigint,char ，不建议使用varchar。

## 单一主键
i> 一个字段做主键叫单一主键。

!> 一张表只能有一个主键约束！不可以添加第二个主键约束，主键只能有一个。

创建一个拥有主键的表

```
drop table if exists t_vip;

CREATE TABLE t_vip(
    id int primary key,
    name varchar(32),
    email varchar(255)
);
```
查看表结构：`desc t_vip;`

| Field | Type         | Null | Key | Default | Extra |
|--|--|--|--|--|--|
| id    | int(11)      | NO   | PRI | NULL    |       |
| name  | varchar(32)  | YES  |     | NULL    |       |
| email | varchar(255) | YES  |     | NULL    |       |

可以看到 `id` 字段的key为PIR(primary key)意为主键。
此时向表内插入数据时主键字段的值必须不为空（not null）和不重复（unique）。

```sql
INSERT INTO t_vip(id,name,email) values 
    (1,'lisi','lisi@123.com'),
    (2,'zhangsan','zhangsan@123.com');
```
语句正常插入
效果 `select * from t_vip;`

| id | name     | email            |
|--|--|--|
|  1 | lisi     | lisi@123.com     |
|  2 | zhangsan | zhangsan@123.com |

2 rows in set (0.005 sec)

尝试插入这两条数据
```sql
INSERT INTO t_vip(id,name,email) values (2,'wangwu','wangwu@123.com');

INSERT INTO t_vip(name,email) values ('wangwu','wangwu@123.com');
```
第一条提示错误：
`ERROR 1062 (23000): Duplicate entry '2' for key 'PRIMARY'`
意为值重复，受到主键`primary`(主键不可重复）约束无法插入数据。

第二条提示错误：
`ERROR 1364 (HY000): Field 'id' doesn't have a default value`
意为id字段没有默认值。受到not null 约束无法插入数据。

----------

## 复合主键

i> 多个字段联合起来做主键为符合主键（使用表级约束）

!> 符合主键是多字段联合起来添加一个主键约束，并不是给多个字段全部添加主键约束，主键只能有一个！

!> 在实际使用中建议使用单一主键，因为主键的意义就是这行数据记录的唯一标识，单一主键即可做到，符合主键比单一主键复杂。

创建一个复合主键表
```sql
drop table if exists t_vip;

CREATE TABLE t_vip(
    id int,
    name varchar(32),
    email varchar(255),
    primary key(id,name)
);
```
查看表结构 `desc t_vip;`

| Field | Type         | Null | Key | Default | Extra |
|--|--|--|--|--|--|
| id    | int(11)      | NO   | PRI | NULL    |       |
| name  | varchar(32)  | NO   | PRI | NULL    |       |
| email | varchar(255) | YES  |     | NULL    |       |

可见id与name字段key属性值都为PRI(primary key)。

插入数据
```sql
INSERT INTO t_vip(id,name,email) values
    (1,'lisi','lisi@123.com'),
    (1,'zhangsan','zhangsan@123.com');
```
成功插入数据 查询 `select * from t_vip;`

| id | name     | email            |
|--|--|--|
|  1 | lisi     | lisi@123.com     |
|  1 | zhangsan | zhangsan@123.com |

2 rows in set (0.006 sec)
虽然李四与张三的id值重复了但它们的name没有重复。联合起来就是不重复。

在尝试插入一条数据
```sql
INSERT INTO t_vip(id,name,email) values (1,'zhangsan','zhangsan@qq.com');
```
执行报错：`ERROR 1062 (23000): Duplicate entry '1-zhangsan' for key 'PRIMARY'`
id与name字段值联合起来于现有的数据重复了，因为主键约束（primary)不能插入数据。

----------

# 外键约束
外键约束常使用在本表字段对应另一张表字段时。

i> 外键字段的值可为null
外键字段所引用的父表字段至少需unique约束（不一定主键）


例如设计一个学生表，具有姓名，学校，班级名字等段。
```sql
drop table if exists t_student;

CREATE TABLE t_student(
    no int primary key auto_increment,
    name varchar(255),
    class varchar(255)
);

INSERT INTO t_student(name,class) values 
    ('lisi','兰州市第一中学七年级一班'),
    ('zhangsan','兰州市第一中学七年级一班'),
    ('wangwu','兰州市第一中学七年级一班'),
    ('zhaoluu','兰州市第一中学七年级二班'),
    ('jack','兰州市第一中学七年级二班');
```

把no做为主键，设置自增属性`auto_increment` ，这样这个no字段就是自然主键，不需要每次插入数据都给值，本身会自动将数字自增以完成主键唯一性。

查询 `select * from t_sutdent;`

| no | name     | class                                |
|--|--|--|
|  1 | lisi     | 兰州市第一中学七年级一班 |
|  2 | zhangsan | 兰州市第一中学七年级一班 |
|  3 | wangwu   | 兰州市第一中学七年级一班 |
|  4 | zhaoluu  | 兰州市第一中学七年级二班 |
|  5 | jack     | 兰州市第一中学七年级二班 |
5 rows in set (0.005 sec)

会发现一个问题，发生数据冗余，空间浪费。
数据越来越多的时候class字段的值高度重复，储存资源浪费。

所以这张表的设计不合理。

所以将学生表内学校班级这种高度重复的字段分为另一张分表。
t_student（学生表）与t_class（学校班级表）使用`班级编号`字段关联匹配。

但为了保证这个`班级编号`的真实有效性，给t_student表的cno字段设置外键约束，让它与t_class表的classno字段关联，效果即为t_student.cno内的值只能是t_class.classno字段内的值。
这样就保证了学生表的班级编号值真实存在于学校班级表的编号字段内。

i> a表中的某个字段外键约束了b表的某个字段时，即b表为a表的父表，a表就是b表的子表。

因父子关系创建及删除时都需要一定的顺序。
创建表时：父表先创建 （没有父表时子表无法约束）
删除表时：子表先删除 （子表有引用父表值的关系，所以先删除子表）
删除数据时：先删除子表（类删表关系）
插入数据时：父表先插入（类创建关系）

设计表：
```sql
drop table if exists t_class;
drop table if exists t_student;

CREATE TABLE t_class(
    class_no int primary key auto_increment,
    class_name varchar(255)
);

CREATE TABLE t_student(
    no int primary key auto_increment,
    name varchar(255),
    cno int,
    foreign key(cno) references t_class(class_no)
);
```
`foreign key(cno) references t_class(class_no)` ： 给cno设置外间约束，约束对象为t_class表的class_no字段，让cno的值只能引用class_no的值不可以使用class_no字段内不存在的值。

然后插入数据
```sql
INSERT INTO t_class(class_no,class_name) values 
    (1,'兰州市第一中学七年纪一班'),
    (2,'兰州市第一中学七年纪二班'),
    (3,'兰州市第一中学七年纪三班');

INSERT INTO t_student(name,cno) values 
    ('lisi',1),
    ('zhangsan',1),
    ('wangwu',2),
    ('zhaoluu',2),
    ('jack',3),
    ('shidifu',3);
```
因为父子关系，先给父表插入数据，然后是子表。
查看数据
 `select * from t_class;`

| class_no | class_name                           |
|--|--|
|        1 | 兰州市第一中学七年纪一班 |
|        2 | 兰州市第一中学七年纪二班 |
|        3 | 兰州市第一中学七年纪三班 |

3 rows in set (0.006 sec)

`select * from t_student;`

| no | name     | cno  |
|--|--|--|
|  1 | lisi     |    1 |
|  2 | zhangsan |    1 |
|  3 | wangwu   |    2 |
|  4 | zhaoluu  |    2 |
|  5 | jack     |    3 |
|  6 | shidifu  |    3 |
6 rows in set (0.003 sec)

如果向t_student表cno内插入一个t_class表class_no内不存在的学校班级编号会发生什么？

```sql
INSERT INTO t_student(name,cno) values ('cuihua',4);
```
发生报错
```ERROR 1452 (23000): Cannot add or update a child row: a foreign key constraint fails (`test`.`t_student`, CONSTRAINT `t_student_ibfk_1` FOREIGN KEY (`cno`) REFERENCES `t_class` (`class_no`))```
因为4这个编号不存在于学校班级表的现有编号，因为外键约束不可以插入数据。

----------

# 自然主键与业务主键
主键还可以分为自然主键与业务主键

自然主键就是使用一个自然数做主键，与业务无关。
业务主键就是用业务数据字段做主键，与业务紧密关联。

在使用中建议使用自然主键，因为业务内容发生变化时不影响主键。
如果使用业务主键（如银行卡号做主键），在有需要改变时就会非常麻烦。