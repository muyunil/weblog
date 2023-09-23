---
title: Mariadb-transaction
date: 2021-05-03 15:11:57.0
updated: 2022-07-31 02:01:30.918
url: /archives/mariadb-transaction
categories: 
- 技术
tags: 
- Mariadb
- SQL
---



记录MariaDB-SQL语句学习过程
<!--more-->

# 事务简介-transaction

MySQL默认的InnoDB储存引擎是支持事务的，什么是事务？
事务就是一个完整的业务流程/逻辑，最小不可再分。

事务只和 `instert`, `delete`, `update` DML语句有关，因为只有DML语句才会对数据数据增删改。

> 只要你的操作涉及到增，删，改，就一定需要考虑数据安全。

因为一个业务流程/逻辑普遍不能使用一条`DML`语句就完成，经常需要多条语句配合，所以事务就存在了

本质上事务就是多条`DML`语句一起成功或者一起失败。

> InnoDB储存引擎：提供一组用来记录事务性活动的日志文件

```sql
start transaction;  (事务开启)
update    ...
update    ...
inster    ...
inster    ...
delete    ...
delete    ...
   ...    ...
commit; / rollback;  (事务结束)
````
在事务执行过程中，每一个`DML`语句操作都会记录在`事务性活动日志文件`中

在事务执行中可以提交事务也可以回滚事务。
提交事务 `commit` (事务执行全部成功）：
- 清空`事务性活动日志文件`，将事务执行过程中的操作全部持久化到数据库表，事务结束（全部成功的结束）。

回滚事务 `rollback` (事务执行全部失败）：
- 清空`事务性活动日志文件`，将事务执行过程中的操作全部撤销/恢复，事务结束（全部失败的结束）。

----------

# 提交/回滚事务
`commit;` 提交事务
`rollback;` 回滚事务

!> 在MySQL中是默认自动提交的，你每执行一次DML语句就自动提交一次，所以默认情况下是无法使用提交回滚达到目的的。

`start transaction;` 关闭自动提交。
在关闭自动提交后我们就可以手动管理事务提交和回滚了。

创建表测试
```sql
drop table if exists t_tran;

CREATE TABLE t_tran(
    id int unique auto_increment,
    name varchar(255)
);
```

插入数据
```sql
INSERT INTO t_tran(name) values 
    ('a'),
    ('b'),
    ('c'),
    ('d');
```
查询 `select * from t_tran;`
| id | name |
|--|-|
|  1 | a    |
|  2 | b    |
|  3 | c    |
|  4 | d    |
4 rows in set (0.004 sec)

尝试回滚到没插入前

执行回滚：`rollback;`
`Query OK, 0 rows affected (0.006 sec)`

然后查询： `select * from t_tran;`

| id | name |
|--|--|
|  1 | a    |
|  2 | b    |
|  3 | c    |
|  4 | d    |
4 rows in set (0.001 sec)

发现并没有回滚到我们插入数据前的状态。
这就是MySQL默认的自动提交事务，执行一条DML语句就自动提交事务完成数据固化。

i> 自动提交是不符合实际使用的，因为一个业务逻辑通常需要多条DML语句共同完成，我们需要在它们全部成功执行后再提交事务。

!> 在实际业务中执行一条DML就提交一次事务数据是非常不安全的。

关闭自动事务提交，或者说启动正常的事务，不使用默认的事务机制

`start transaction;`
`Query OK, 0 rows affected (0.004 sec)`
成功。

我们删除表内全部数据

`delete from t_tran;`

`Query OK, 4 rows affected (0.056 sec)`
成功删除。

查看：`select * from t_tran;`
`Empty set (0.013 sec)`
提示无数据。

回滚：`rollback;`
`Query OK, 0 rows affected (0.005 sec)`
回滚成功。
再次查看数据：`select * from t_tran;`

| id | name |
|--|--|
|  1 | a    |
|  2 | b    |
|  3 | c    |
|  4 | d    |
4 rows in set (0.005 sec)
数据恢复！ 
我们选择了回滚事务，将事务期间的操作全部撤回/取消，所以数据又回来了。

如果我们选择提交事务，会将操作更新固化到数据表数据就会真的删除。

----------

# 事务四特性
- A- 原子性
事务为最小业务逻辑。
- C-一致性
事务要求一段业务DML语句不是全部成功就是全部失败，保证数据一致性。
- I-隔离性
a事务与b事务之间有一定的隔离
可设置隔离级别。
- D-持久性
事务最终结束的保障，事务提交，将操作数据固化到数据表（没有储存到硬盘上的数据储存到硬盘上）。

----------

# 事务隔离

事务与事务之间有隔离性，隔离等级的不同隔离效果也不同。

有以下四种隔离1~4档，隔离效果依次增加。

- 读未提交-1  `read uncommitted`

不提交就可读到。
问题：会造成`脏读`
事务a可以读到事务b还没有提交的数据，这被认为是`脏读` 意为读到了脏数据，b未提交的数据被a视为脏数据
此隔离级别只是理论上的，大多数数据库最少都是`读已提交`级别的隔离

- 读已提交-2  `read committed`

提交之后可以读到。
解决了`脏读`
新问题：`不可重复读`
事务a可以读到事务b以及提交的数据，但b事务在不断提交，a每次读取都是不一样的条数，所以不可重复读取。
这种隔离级别下读取的数据绝对真实。
在Oracle中是默认隔离级别

- 可重复读-2  `repeatable read`

提交之后也读不到。
解决了`不可重复读`
新问题：`幻读`
事务a开启后，不管开启多久，每次从事务b中读取的数据都是一样的，即使事务b将数据又修改并且提交了，事务a读取到的数据还是没有发生改变，这就是`可重复读`，但读到的都是幻象。
这种隔离级别下的数据不够绝对真实。
在MySQL中是默认隔离级别。

- 序列化/串号化-3 `serializable`

这是事务隔离最高级别，效率最低，解决了所有问题。
就是事务排队，类`锁`的概念，同一时刻只有一个事务执行，不可以并发。

----------

## 隔离演示_read-uncommitted

查看当前隔离级别。
```sql
select @@tx_isolation;
```
| @@tx_isolation  |
|--|
| REPEATABLE-READ |
当前为MySQL的默认事务隔离 `repeatable-read` : 重复读取

- 设置为 `read uncommitted ` : 读未提交。  
需要退出重进才可刷新改变。
```sql
set global transaction isolation level read uncommitted;

exit;
```
不得不说这命令又臭又长。

重新连接数据库之后再次查看当前隔离。
`select @@tx_isolation;`

| @@tx_isolation   |
|--|
| READ-UNCOMMITTED |

设置成功，现在为读未提交。

测试数据脏读。
创建一个测试表
`create table t_ru(name varchar(255));`

----------

开两个窗口模拟两个事务执行。
事务A：
`start transacition;`
事务B：
`start transacition;`

----------

事务B查看数据：
`select * from t_ru;`
`Empty set (0.019 sec)`
B事务没有查询到数据。

----------

事务A插入数据：
`INSERT INTO t_ru(name) values ('zhangsan');`
查看数据
`select * from t_ru;`

| name     |
|--|
| zhangsan |

1 row in set (0.003 sec)
不结束提交事务。

----------

事务B查看数据：
`select * from t_ru;`

| name     |
|--|
| zhangsan |

事务B查询到了事务A还没有提交的数据！
这就是`read uncommitted` 造成的脏读现象。

事务A回滚数据：
`rollback;`
事务B再次查询：
`select * from t_ru;`
`Empty set (0.001 sec)`
事务B也查询不到数据了 

i> 这种管理级别造成的`墙`几乎等于透明，一般没有数据库会采用。

----------

## 隔离演示_read-committed
更改事务级别：
```sql
set global transaction isolation level read committed;
exit;
```
开两个窗口连接到数据库模拟两个事务执行。
事务A：
`start transacition;`
事务B：
`start transacition;`

----------

事务A插入数据：
`INSERT INTO  t_ru(name) values ('lisi')`
查询数据：
`select * from t_ru;`

| name |
|--|
| lisi |
暂不提交事务

----------

事务B查看数据：
`select * from t_ru;`
`Empty set (0.001 sec)`
事务B没有读到事务A还没有提交的数据。

----------

事务A提交事务，完成数据固化。
`commit;` 

----------

事务B再次查询：
`select * from t_ru;`

| name |
|--|
| lisi |
事务B查询到了事务A提交的数据！

i> 这种隔离级别下读到的数据绝对真实！是Oracle数据库的默认隔离级别。

----------

## 隔离演示_repeatable-read
设置隔离级别
```sql
set global transaction isolation level repeatable read;
exit;
```
开两个窗口连接到数据库模拟两个事务执行。
事务A：
`start transacition;`
事务B：
`start transacition;`

----------

事务A查询数据：
`select * from t_ru;`

| name |
|--|
| lisi |

----------

事务B插入数据并且`提交`：
```sql
INSERT INTO  t_ru(name) values 
    ('wangwu'),
    ('jack'),
    ('zhaoluu'),
    ('zhangwei');

commit;
```
然后查询：
`select * from t_ru;`

| name     |
|--|
| lisi     |
| wangwu   |
| jack     |
| zhaoluu  |
| zhangwei |

----------

事务A再次查询：
`select * from t_ru;`

| name |
|--|
| lisi |
1 row in set (0.003 sec)

事务A并没有读到事务B提交的数据。

----------

事务B删除全表数据
`delete from t_ru`
查询：
`select * from t_ru;`
`Empty set (0.002 sec)`

----------

事务A再次查询：
`select * from t_ru;`

| name |
|--|
| lisi |
1 row in set (0.001 sec)
还是没有变化！

这就是可重复读事务隔离的效果。
事务只能读到它开启的那一刻的数据，在没有结束事务前它每次读取都是一样的，是`幻读` 哪怕数据已经被其他事务修改删除！
MySQL默认隔离就是可重复读 `repeatable read`

----------

## 隔离演示_serializable 
设置隔离级别
```sql
set global transaction isolation level serializable;
exit;
```
开两个窗口连接到数据库模拟两个事务执行。
事务A：
`start transacition;`
事务B：
`start transacition;`

----------

事务A插入数据：
`insert into t_ru(name) values ('lisi');`
暂不提交

----------

事务B查询数据：
`select * from t_ru;`
| <光标会一直卡在这里！

----------

事务A提交结束
`commit;`

----------

事务B的查询在事务A提交的一瞬间查询出来了。
| name |
|--|
| lisi |

这就是`serializable`：序列化/串行化管理的效果。
一堆事务进行排队执行，在上一个事务没有提交事务前，后的事务都在暂停等待，不允许两个事务同时执行！
这是最为严格的事务隔离！

----------