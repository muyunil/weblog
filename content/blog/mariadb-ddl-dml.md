---
title: Mariadb-DDL-DML
date: 2021-05-03 15:07:29.0
updated: 2022-04-01 16:02:19.488
url: /archives/mariadb-ddl-dml
categories: 
- 技术
tags: 
- Mariadb
- SQL
---


记录MariaDB-SQL语句学习过程

<!--more-->

>标准SQL语句可分为以下五类。

- DQL ：查询语言，用于查询库，表，数据等等，关键字：`select` ，凡拥有关键字的都是DQL。
- DML ：数据操作语言，用于操作数据表内数据，关键字：`insert`（增），`delete`（删），`update`（改），凡拥有关键字的都是DML。
- DDL ：数据定义语言，用于操作数据表本身结构等，关键字：`create`（增）， `drop`（删），`alter`（改），凡拥有关键字的都是DDL。
- TCL ：事务控制语言
提交事务：`commit`，回滚事物：`rollback`
- DCL ：数据控制管理
授权 ：`grant`，撤销授权：`revoke` 等等

----------

# 建表  (DDL)
建表格式
```sql
create table(
    ename varchar,
    sal int,
    age int
);
```
建表常用类型

```sql
varchar 
//可变长字符串,长度限制为255
//动态分配长度，节省空间。

char    
//定长字符串，长度限制为255
//分配长度固定，不需要动态判断，性能高。

int    
//数字整型，长度限制为11。

bigint
//数字长整型，暂未知具体长度。

float
//单精度浮点型

double
//双精度浮点型

date
//短日期类型

datetime
//长日期类型

clob
//字符大对象，Character Large Object
//最多可存储4G的字符串

blob
//二进制大对象，Binary Large Object
//用于存储二进制数据
//如：图片，视频，等等
//存储手段只能使用io流进行存数据。
```

```sql
CREATE TABLE t_student(
    no int,
    name varchar(32),
    sex char(1) default '?',
    age int(3),
    email varchar(255)
);
```
i> 可以使用default设置默认值，即无数据时使用默认值而不是null

建表完成
!> varchar(32),为建议长度为32。（非字节长度）

----------

## 表复制

```
CREATE  TABLE 
    t_cp as select * from t_student;
```
将t_student表查询结果储存为t_cp，达到复制目的。
查询结果
select * from t_student;

| no   | name     | sex  | age  | email         |
|------|----------|------|------|---------------|
|    5 | zhangsan | m    |   18 | z@test.com    |
|    6 | lisi     | f    |   22 | lisi@test.com |
|    7 | wangwu   | m    |   21 | w@test.com    |

3 rows in set (0.003 sec)

select * from t_cp;

| no   | name     | sex  | age  | email         |
|------|----------|------|------|---------------|
|    5 | zhangsan | m    |   18 | z@test.com    |
|    6 | lisi     | f    |   22 | lisi@test.com |
|    7 | wangwu   | m    |   21 | w@test.com    |
3 rows in set (0.002 sec)

两张表结构及数据完全一致，（包括字段的一些设置如sex字段的默认值'?'）

----------

# 删表  (DDL)
关键字： `drop`
```sql
drop table t_student;

drop table if exists t_student;
```
第一种语句为简单删除，表不存在时会报错。
第二种语句加了if判断，判断表是否存在，存在时将其删除。表不存在时也不会报错。

----------

# 修改表结构
使用 alter 
....待更

----------

# 插入数据_insert  (DML)
```sql
insert into t_student(
    no,
    name,
    sex,
    age,
    email
)values(
    001,
    'root',
    '?',
    100,
    'muyu@daomail.net'
);

insert into t_student(name) values('tmp');

insert into t_student values(3, 'tmp2', '?', 10, 'test@test.com');
```
`insert into tableName(字段名1，字段名2) values(值1，值2)`
字段名与值按顺序对应。

i> 没有赋予值的字段默认为null
insert语句只要执行成功就必然会多一条记录。
可以省略字段名，即为全写，所以后面的值需要按顺序全写上，否则报错。

表内数据：  

| no   | name | sex  | age  | email            |
|------|------|------|------|------------------|
|    1 | root | ?    |  100 | muyu@daomail.net |
| NULL | tmp  | NULL | NULL | NULL             |
|    3 | tmp2 | ?    |   10 | test@test.com    |
3 rows in set (0.002 sec)

----------

## 一次插入多条数据
```sql
INSERT INTO t_student(no,name,sex,age,email) 
values
    (5,'zhangsan','m',18,'z@test.com'), 
    (6,'lisi','f',22,'lisi@test.com'), 
    (7,'wangwu','m',21,'w@test.com');
```
在values后可以继续追加，用英文逗号分隔就行。

----------

## 将查询结果插入表

与表上面的表复制相同，可以使用select子查询将查询结果当作数据插入表。

```sql
INSERT INTO t_student() select * from t_student;
```
将查询结果插入到表中，这里没好的表就自己插入自己的数据了。
这样插入数据能看得出来两张表需要结构相同才可以插入。
插入结果

| no   | name     | sex  | age  | email         |
|------|----------|------|------|---------------|
|    5 | zhangsan | m    |   18 | z@test.com    |
|    6 | lisi     | f    |   22 | lisi@test.com |
|    7 | wangwu   | m    |   21 | w@test.com    |
|    5 | zhangsan | m    |   18 | z@test.com    |
|    6 | lisi     | f    |   22 | lisi@test.com |
|    7 | wangwu   | m    |   21 | w@test.com    |
6 rows in set (0.002 sec)

----------

## 插入日期
```sql
CREATE TABLE t_user(
    id int,
    name varchar(32),
    birth date
);

insert into t_user(id,name,birth) values(0,'zhangsan',str_to_date('27-08-2000','%d-%m-%Y'));

insert into t_user(id,name,birth) values(1,'lisi','1999-9-9');
```
创建了个有`date`日期类型字段的表
日期在插入时使用`str_to_date`函数
`str_to_date('日期字符串','日期格式')`
可用日期格式：`%Y`(年），`%m`(月)，`%d`(日)，`%h`(时), `%i`(分)，`%s`(秒)
- 当你的字符串日期属于`%Y-%m-%d`时可以不写`str_to_date`函数

查询结果
MariaDB [mdb]> select * from t_user;

| id   | name     | birth      |
|--|--|--|
|    0 | zhangsan | 2000-08-27 |
|    1 | lisi     | 1999-09-09 |
2 rows in set (0.003 sec)

i> 此处查询显示结果中MySQL自动对birth进行了格式化，将date转为varchar，格式为MySQL默认格式 ：%Y%m%d

----------

## 格式化日期
MySQL会默认格式化date字段，但如果想自定义其他格式可以使用date_format函数
```sql
SELECT
    id, name,
    date_format(birth,'%Y_%m_%d') as birth
FROM 
    t_user;
```
将birth的date格式化为制定格式的varchar字符串

查询结果  
| id   | name     | birth      |
|------|----------|------------|
|    0 | zhangsan | 2000_08_27 |
|    1 | lisi     | 1999_09_09 |
|------|----------|------------|

----------

## 长日期-Data_Time
与date不同的是datetime可以存储时分秒

删表重新创建
```sql
drop table if exists t_user;
CREATE TABLE t_user(
    id int,
    name varchar(32),
    birth date,
    create_time datetime
);
```

- 插入数据

```sql
INSERT INTO t_user(id,name,birth,create_time)
values(0,'lisi','1999-09-09','2021-03-17 20:45:59');

INSERT INTO t_user(id,name,birth,create_time)
values(1,'zhangsan','1999-09-09',now());

SELECT * FROM t_user;
```
datetime可以储存更详细的数据。
i> datetime的默认格式为 %Y-%m-%d %h:%i:%s
使用now()函数可以获取当前系统时间。

查询结果    

| id   | name     | birth      | create_time         |
|------|----------|------------|---------------------|
|    0 | lisi     | 1999-09-09 | 2021-03-17 20:45:59 |
|    1 | zhangsan | 1999-09-09 | 2021-03-17 20:46:32 |

----------

# 更新数据_update  (DML)
> 语法：UPDATE table_name SET 字段1  = 值1, 字段2 = 值2 WHERE id = 1;

```sql
UPDATE 
    t_user 
SET 
    name = 'root', 
    birth = '9999-9-9', 
    create_time = now() 
WHERE 
    id = 0;
```
更新结果    
| id   | name     | birth      | create_time         |
|------|----------|------------|---------------------|
|    0 | root     | 9999-09-09 | 2021-03-17 21:06:55 |
|    1 | zhangsan | 1999-09-09 | 2021-03-17 20:46:32 |

```sql
UPDATE t_user SET name = 'test';
```

!> 更新字段值时如果不设置where条件会导致所有的记录的该字段被修改。

更新结果  

| id   | name | birth      | create_time         |
|------|------|------------|---------------------|
|    0 | test | 9999-09-09 | 2021-03-17 21:06:55 |
|    1 | test | 1999-09-09 | 2021-03-17 20:46:32 |

----------

# 删除数据_delete  (DML)
语法： `delete from table_name where id = 1`

delete属于逻辑删除，类似于linux上的之删除文件链接。但文件本身还存在于硬盘上。

!> 删除效率较低但支持回滚。

```sql
DELETE  
FROM
    t_user
WHERE
    id = 0;

SELECT * FROM t_user;
```
查看删除结果  
| id   | name | birth      | create_time         |
|------|------|------------|---------------------|
|    1 | test | 1999-09-09 | 2021-03-17 20:46:32 |

----------

```sql
INSERT INTO t_user(id) values(3);
INSERT INTO t_user(id) values(4);
INSERT INTO t_user(id) values(5);
SELECT * FROM t_user;

DELETE  
FROM
    t_user;

SELECT * FROM t_user;
```
!> 同样，如果不设置where条件，delete语句会删除这张表内所有数据。

增加数据在无条件删除
MariaDB [mdb]> SELECT * FROM t_user;

| id   | name | birth      | create_time         |
|------|------|------------|---------------------|
|    1 | test | 1999-09-09 | 2021-03-17 20:46:32 |
|    3 | NULL | NULL       | NULL                |
|    4 | NULL | NULL       | NULL                |
|    5 | NULL | NULL       | NULL                |
4 rows in set (0.001 sec)

MariaDB [mdb]> DELETE  FROM  t_user;
Query OK, 4 rows affected (0.002 sec)

MariaDB [mdb]> SELECT * FROM t_user;
Empty set (0.003 sec)

最后查询不到任何数据。

----------

# 删除全表数据_truncate  (DDL)

与delete不同，truncate属于DDL，删除效果不属于逻辑删除，会真实删除数据。

!> 删除效率较高，但不支持回滚，使用时需要警告。
只能用于删除整张表内的数据，不能只删除一条。

```sql
truncate table t_user;
```