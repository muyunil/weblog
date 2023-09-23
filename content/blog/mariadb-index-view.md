---
title: Mariadb-index-view
date: 2021-05-03 15:18:15.0
updated: 2022-04-01 16:01:39.476
url: /archives/mariadb-index-view
categories: 
- 技术
tags: 
- Mariadb
- SQL
---



记录MariaDB-SQL索引学习过程

<!--more-->

# 索引概述

索引就像一本字典的目录，当查找某一个字时，通过目录可以知道大概在字典的多少页附近，跳转过去在翻几页就可以找到它，极大的减少了查询的时间。

可以往字段上添加索引，在查询时就可以通过索引定位大概位置，在进行局域性扫描就可以查到，而不需要把整个字段都遍历一遍。

i> 索引的目的就是减少扫描范围

```sql
SELECT id, name FROM t_test WHERE name = 'zhangsan';
```
如果没有给字段name添加索引，那么就会进行`全扫描`：遍历整个name字段所有的值，效率较低。

i> MySQL扫描就两种方式
1.全表扫描
2.根据索引检索

----------

# 索引原理

索引的原理图
只是简单的帮助理解认知，并不是真正的底层实现原理，实际上底层的实现要复杂许多。

![索引][1]

[1]: https://muio.net/data/MySQLindex.png

----------

# 选择性添加索引

索引适合在以下场景中发挥

1. 数据量较庞大，或者说数据库服务器处理较吃力时。
2. 该字段经常出现在`where`后做为查询条件，也就是说经常被扫描。
3. 该字段一般不会被`DML`语句`(增，删，改)`操作，因为`DML`之后该字段上的索引是需要重新排序的。
4. 该字段具有`unique`唯一性约束

i> 一般不建议随意添加索引，因为索引也是需要维护的，过多的话反而会影响性能。
建议通过主键查询，或者被unique约束的字段查询，效率是较高的。

----------

# 增加/删除索引

给用户表`name`字段增加索引
```sql
create index user_name_index on user(name);
```
给user表的name字段增加索引，起名叫`user_name_index`

----------

删除索引
```sql
delete index user_name_index on user;
```
从user表中删除名叫`user_name_index`的索引

----------

# 查询是否使用索引
`explain select ...`可以查询这条select语句是否使用了索引。

`explain select * from EMP where ename = 'KING';`

| id   | select_type | table | type | possible_keys | key  | key_len | ref  | rows | Extra       |
|--|---|--|--|--|--|--|--|--|--|
|    1 | SIMPLE      | EMP   | ALL  | NULL          | NULL | NULL    | NULL | 14   | Using where |

`select_type：ALL` 意为全扫描，一共扫描了14条记录。

----------

给name字段增加索引
`create index e_ename_index on EMP(ename);`
`Query OK, 0 rows affected (0.066 sec) Records: 0  Duplicates: 0  Warnings: 0`

----------

再次查询是否使用索引
`explain select * from EMP where ename = 'KING';`

| id   | select_type | table | type | possible_keys | key           | key_len | ref   | rows | Extra                 |
|--|--|--|--|--|--|--|--|--|--|
|    1 | SIMPLE      | EMP   | ref  | e_ename_index | e_ename_index | 13      | const | 1    | Using index condition |

`select_type : ref` 意为使用了索引，`rows`查询记录只查询了1次就查到了`KING`!

----------

# 索引失效

索引会在一下情况下失效。

----------

## 模糊查询中以`%`开始
```sql
explain SELECT * FROM emp WHERE ename like '%T';
```
| id   | select_type | table | type | possible_keys | key  | key_len | ref  | rows | Extra       |
|--|--|--|--|--|--|--|--|--|--|
|    1 | SIMPLE      | EMP   | ALL  | NULL          | NULL | NULL    | NULL | 14   | Using where |

在使用模糊查询有索引字段时尽量避免以`%`开始，因为不确定第一位字符时无法利用索引机制进行检索，只能进行全扫描。

----------

## 查询索引字段中使用`or`增加匹配一个没有索引的字段。
```sql
explain SELECT * FROM emp WHERE name = 'KING' or job = 'MANAGER';
```
| id   | select_type | table | type | possible_keys | key  | key_len | ref  | rows | Extra       |
|--|--|-|--|--|--|--|--|--|--|
|    1 | SIMPLE      | EMP   | ALL  | e_ename_index | NULL | NULL    | NULL | 14   | Using where |

使用 `or` 需要两边的字段都用索引，如果有一个字段没有索引那么两个字段都不会使用索引，会使用全扫描。

i> 建议使用union合并查询，union不会导致索引失效。

----------

## 不使用复合索引的左侧列查询

i> 什么是符合索引，多个字段联合添加一个索引叫符合索引。

创建复合索引：
```sql
CREATE INDEX  job_sal_index ON  emp(job,sal);
```
查询是否使用索引：

| id   | select_type | table | type | possible_keys | key           | key_len | ref   | rows | Extra                 |
|--|--|--|--|--|--|--|--|--|--|
|    1 | SIMPLE      | EMP   | ref  | job_sal_index | job_sal_index | 12      | const | 3    | Using index condition |

使用复合索引左列查询时使用了索引。

```sql
explain SELECT * FROM emp WHERE sal = 800;
```
查询是否使用索引：

| id   | select_type | table | type | possible_keys | key  | key_len | ref  | rows | Extra       |
|--|--|--|--|--|--|--|--|--|--|
|    1 | SIMPLE      | EMP   | ALL  | NULL          | NULL | NULL    | NULL | 14   | Using where |

使用复合索引右列查询时没有索引。

----------

## 索引字段进行了运算

添加索引

```sql
CREATE INDEX e_sal_index ON emp(sal);
```
查询是否使用索引：

```sql
explain SELECT * FROM emp WHERE sal -1 = 800;
```
| id   | select_type | table | type | possible_keys | key  | key_len | ref  | rows | Extra       |
|--|--|--|--|--|--|--|--|--|--|
|    1 | SIMPLE      | EMP   | ALL  | NULL          | NULL | NULL    | NULL | 14   | Using where |

当索引字段本身参与运算时索引失效

----------

## 索引字段使用了函数

```sql
explain SELECT * FROM emp WHERE lower(ename) = 'smith';
```
查询是否使用索引：
| id   | select_type | table | type | possible_keys | key  | key_len | ref  | rows | Extra       |
|--|--|--|--|--|--|--|--|--|--|
|    1 | SIMPLE      | EMP   | ALL  | NULL          | NULL | NULL    | NULL | 14   | Using where |

当索引字段使用函数时索引失效

----------

- 还有更多的索引失效，待更。

----------

# 索引的分类

i> 索引是数据库优化的重要手段！

1. 单一索引
一个字段添加索引
2. 复合索引
多个字段添加索引
3. 主键索引
主键上添加索引
4. 唯一性索引
`unique`约束字段上添加索引
....

!> 在唯一性较弱的字段上添加索引用处不大。

----------

# 视图

1. 视图是一种根据查询（也就是 SELECT 语句）定义的数据库对象，用于获取想要看到和使用的局部数据。
2. 视图有时也被成为“虚拟表”。
3. 视图可以被用来从常规表（称为“基表”）或其他视图中查询数据。
4. 相对于从基表中直接获取数据，视图有以下好处：
访问数据变得简单
可被用来对不同用户显示不同的表的内容
用来协助适配表的结构以适应前端现有的应用程序

## 创建视图 

```sql
CREATE 
    view v_dept_emp 
as 
    select 
        ename, dname,
        sal, e.deptno 
    from 
        emp e,dept d 
    where 
        e.deptno = d.deptno;
```

i> 视图只能使用DQL语句创建！

查询视图
`select * from v_dept_emp;`

| ename  | dname      | sal     | deptno |
|--|--|--|--|
| SMITH  | RESEARCH   |  800.00 |     20 |
| ALLEN  | SALES      | 1600.00 |     30 |
| WARD   | SALES      | 1250.00 |     30 |
| JONES  | RESEARCH   | 2975.00 |     20 |
| MARTIN | SALES      | 1250.00 |     30 |
| BLAKE  | SALES      | 2850.00 |     30 |
| CLARK  | ACCOUNTING | 2450.00 |     10 |
| SCOTT  | RESEARCH   | 3000.00 |     20 |
| KING   | ACCOUNTING | 5000.00 |     10 |
| TURNER | SALES      | 1500.00 |     30 |
| ADAMS  | RESEARCH   | 1100.00 |     20 |
| JAMES  | SALES      |  950.00 |     30 |
| FORD   | RESEARCH   | 3000.00 |     20 |
| MILLER | ACCOUNTING | 1300.00 |     10 |

可以看到视图（虚拟表）的内容就是`select`查询语句的结果。

i> 可以看为将查询结果抽象为一张表（虚拟表），这样做的好处就是以后有需要这条很长的select语句时可以直接操作视图，不需要再次使用很长的DQL语句查询。

----------

## 操作视图
使用DML语句操作视图字段等同于操作原表字段。

```sql
update v_dept_emp set sal = 9999 where ename = 'KING';
```

将视图表内 用户名叫`KING`的员工工资该为9999。

查询原表
`select * from emp;`

| EMPNO | ENAME  | JOB       | MGR  | HIREDATE   | SAL     | COMM    | DEPTNO |
|--|--|--|--|--|--|--|--|
|  7369 | SMITH  | CLERK     | 7902 | 1980-12-17 |  800.00 |    NULL |     20 |
|  7499 | ALLEN  | SALESMAN  | 7698 | 1981-02-20 | 1600.00 |  300.00 |     30 |
|  7521 | WARD   | SALESMAN  | 7698 | 1981-02-22 | 1250.00 |  500.00 |     30 |
|  7566 | JONES  | MANAGER   | 7839 | 1981-04-02 | 2975.00 |    NULL |     20 |
|  7654 | MARTIN | SALESMAN  | 7698 | 1981-09-28 | 1250.00 | 1400.00 |     30 |
|  7698 | BLAKE  | MANAGER   | 7839 | 1981-05-01 | 2850.00 |    NULL |     30 |
|  7782 | CLARK  | MANAGER   | 7839 | 1981-06-09 | 2450.00 |    NULL |     10 |
|  7788 | SCOTT  | ANALYST   | 7566 | 1987-04-19 | 3000.00 |    NULL |     20 |
|  7839 | KING   | PRESIDENT | NULL | 1981-11-17 | 9999.00 |    NULL |     10 |
|  7844 | TURNER | SALESMAN  | 7698 | 1981-09-08 | 1500.00 |    0.00 |     30 |
|  7876 | ADAMS  | CLERK     | 7788 | 1987-05-23 | 1100.00 |    NULL |     20 |
|  7900 | JAMES  | CLERK     | 7698 | 1981-12-03 |  950.00 |    NULL |     30 |
|  7902 | FORD   | ANALYST   | 7566 | 1981-12-03 | 3000.00 |    NULL |     20 |
|  7934 | MILLER | CLERK     | 7782 | 1982-01-23 | 1300.00 |    NULL |     10 |

原表内的`KING`工资被改变了！
所以可以通过视图操作原表的数据！

## 删除视图

```sql
drop view if exists v_dept_emp;
```
视图并不会因为重启关机丢失。