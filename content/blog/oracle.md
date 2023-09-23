---
title: Oracle
date: 2021-06-15 14:42:16.0
updated: 2022-04-01 16:02:35.407
url: /archives/oracle
categories: 
- 技术
tags: 
- SQL
- Oracle
---



Oracle是一个大型数据库，在各方面表现都算突出，但它是付费的，像一些大型公司为了降低成本都是二次开发MySQL进行使用。
<!--more-->

> 以下操作记录环境为本地Windows，改天再记录Linux服务器环境。


# SQL*Plus
Oracle自身提供的数据库管理工具，随数据库一起安装。

- 通过SQL*Plus可以完成以下操作
1. 对数据curd操作
2. 对查询出的结果格式化显示
3. 对数据库对象（用户，角色，表空间，数据表...）进行管理

## 用户登录

1. 本地管理员SYS,此用户只能在数据库本地登录，并且登录时不验证密码正确性。
以dba身份登录：`sqlplus sys/password as sysdba`

3. 网络用户SYSTEM，此用户可以远程登录，登录时校验密码正确性，

## 连接数据库

1. 默认数据库，用户在登录时没有指定数据库是会连接到默认数据库。
2. 登录时自定义连接数据库：`user/passwrod @SID`

## SQL*Plus 指令分类  
1. SQL指令，统一的结构化查询语言。
2. SQL*Plus指令，属于工具本身的指令，设置SQL*Plus管理工具的属性功能。

## SQL*Plus 连接问题  
[ORA-12560:TNS:协议适配器错误]
1. 检查监听程序正常运行
2. Oracle数据库实例正常运行
3. 数据库默认连接数据库是存在的，Windows可通过regedit修改注册表SID为存在的数据库。



# Oracle 支持数据类型

## 字符型，存储字符序列
|数据类型|数值范围|说明|
|:--:|:--:|:--:|
|varchr2|0~4000|储存变长字符串|
|nvarchar2|0~1000|储存Unicode的可变长字符串|
|char|0~2000|储存定长字符串|
|nchar|0~1000|储存Unicode定长字符串|
|long|0~2G|储存长度大于4000字节的可变长字符串|

> char(5)，在存入不足5字节时Oracle自动补充到5字节。

## 数值型，存储数字（整数小数）
| 数据类型 | 取值范围 | 说明 |
|:----:|:----:|:----:|
| number(m,n) | m最大值38(十进制位数） | 储存整数和小数，m为储存的十进制位数，n为储存的小数位数 |
| float | 126位二进制数据 |  储存小数（自动转换为二进制）  |


## 日期型，存储时间（年月日时分秒）

| 数据类型 | 说明 |
|:----:|:----:|
| date | 用来存储日期时间（公元前4712年1月1日~公元9999年12月31日 |  
| timestamp | 用来存储日期时间，比date更精确（date精确到秒，timestamp精确到小数秒），timestamp存储的日期时间能够显示上午/下午 |


## 大字段型，存储大数据及文件
| 数据类型 | 取值范围 | 说明 |
|:----:|:----:|:----:|
| blob | 0~4GB | 存储二进制数据 |
| clob | 0~4GB | 存储字符串数据 |
| bfile | 与操作系统相关 | 存储非结构化的二进制数据（磁盘文件） |


# 常见Oracle数据库对象
1. 用户 （当前Oracle账号信息）
2. 角色 （一组权限的集合）
3. 表空间 （数据库依托于表空间，表空间物理存储在文件中）
4. 数据表 （存储在表空间中，用来存储数据）
5. 索引 （提高查询速度）
6. 视图 （数据库表的一种抽象，可理解为表的映射指针，可通过视图操作实际的数据表）
7. 序列 （某列数据的自动增长）
8. 同义词 （起别名，效果为隐藏原表名称）
9. 存储过程 （类MySQL中的存储引
10. 函数  
11. 触发器 
12. 程序包 （存储过程，函数，触发器的集合）

# 创建数据表语法
```
CREATE TABLE table_name(
  column_name column_type [not null],
  column_name column_type [not null],
  ...,
  [constraint]
)
```
table_name：表名
column_name：表中列名（字段名）
column_type：数据类型
[not null]；约束，设置该列插入数据时不能为空（默认可为空）
[constraint] 约束，设置主键外键检查等

# ALTER 修改表

## ADD 增加列
```
ALTER TABLE table_name ADD column_name column_type
```

## MODIFY 修改列
```
ALTER TABLE table_name MODIFY column_name column_type
```

## DROP COLUMN 删除列
```
ALTER TABLE table_name DROP COLUMN column_name
```
## 添加约束 ADD CONSTRAINTES
```
ALTER TABLE table_name ADD CONSTRAINTS constraint_name PRIMARY KEY(column_name)

ALTER TABLE table_name ADD CONSTRAINTS constraint_name PRIMARY KEY(column_name,column_name)

```

## 添加外键约束
```
ALTER TABLE table_name ADD CONSTRAINTS constraint_name FOREIGN KEY(column_name) REFERENCES table_name(column_name) on delete cascade
```

## 添加CHECK约束
```
ALTER TABLE table_name ADD CONSTRAINTS constraint_name CHECK(约束条件)
```

## 删除约束
```
ALTER TABLE table_name DROP CONSTRAINTS constraint_name
```

# 删除表

```
DROP TABLE table_name
```

删除数据表及表内数据

# 约束

## 主键约束
```
CREATE TABLE table_name(
  column_name column_type [primary key],
  column_name column_type [not null],
  ...,
  [constraint]
)
CREATE TABLE table_name(
  column_name column_type,
  ...,
  [primary key(column_name)]
)
```
`primaey key`为主键约束，主键约束包含唯一性与不可为空。

## 联合主键
```
CREATE TABLE table_name(
  column_name column_type,
  ...,
  [primary key(column_name,column_name)]
)
```
## 外键约束
```
CREATE TABLE table_name(
  column_name column_type,
  ...,
  [constraint constraints_name foreign key(Acolumn_name) references table_name(Bcolumn_name) on delete cascade]
)
```
constraints_name：自定义外键名
Acolumn_name：关联表列名
table_name：被关联表名
Bcolumn_name：被关联表列名
on delete cascade：联级删除，被关联表字段删除时，同时删除关联表字段。


## CHECK 约束
```
CREATE TABLE table_name(
  ...,
  CONSTRAINT constraint_name CHECK(约束条件)
)
```
约束条件很自由，利用现有的条件符号就行限制，如：
`check(age between 6 and 40)`
`check(sex='男' or sex='女')`

> 唯一性约束 UNIQUE,与NOT NULL 较简单不做记录，照猫画虎即可。


# 插入数据 

## 通过insert添加数据
```
INSERT INTO table_name(column_name1,c_n2,..) values(data1,data2,...)
```
## 将select查询结果数据插入表。
```

```

# 创建表空间

```
create tablespace tablespacename datafile 'pathfile.dbf' size 100m autoextend on next 10m
```

# 创建用户
Oracle里用户是归属在表空间上的。
```
create user username identified by pawsd tablespace name
```

# 授权 
```
grant dba to username
```




# SQL命令分类

| 类型 | 命令 |
|:----:|:--|
| DDL | create：创建；drop：删除；alter：修改；rename：重命名；truncate：截断 |
| DML | insert：插入；delete：删除；update：更新；select：查询 | 
| DCL | grant：授权；revome：回收权限；commit：提交事务；rellback：回滚事务 |