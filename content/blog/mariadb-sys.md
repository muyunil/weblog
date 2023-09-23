---
title: mariadb-sys
date: 2021-05-03 15:22:19.0
updated: 2022-07-31 01:58:56.466
url: /archives/mariadb-sys
categories: 
- 技术
tags: 
- Mariadb
- SQL
- Linux
---



记录数据库管理笔记

<!--more-->

# 数据库备份
数据的备份是非常重要的措施。
可以在数据意外丢失时起到恢复作用。

## mysqldump 
i> mysqldump属于逻辑备份，将数据库数据以sql语句的方式输出到.sql文件，恢复时执行语句相当于将数据重新插入了一遍。

!> mysqldump 是mysql提供的一个命令行工具，在shell使用，非mysql内部语句。

- 导出数据
```shell
mysqldump -uroot -p123 test > ./test.sql
mysqldump -uroot -p123 test t_class > ./test-t_class.sql
```
1. 将test数据库导出到当前目录下的test.sql文件内。
2. 将test数据库内t_class表导出到当前目录下的test-t_class.sql文件内。

- 导入数据
```shell
mysql -uroot -p123 test < ./test.sql
```
还可以使用内部命令source导入。

```sql
use test;
source ./test.sql;
```
### 参数 -B
- 导出数据
```shell
mysqldump -uroot -p123 -B test > ./test-B.sql
mysqldump -uroot -p123 -B test mdb > ./all-B.sql
```
- 导入数据
```shell
mysql -uroot -p123 < ./test-B.sql
mysql -uroot -p123 < ./all-B.sql
```
加了-B参数用于备份多个库到一个sql文件，为了区分所以增加了create和use语句，在没有数据库时会自动创建库再选择然后导入数据。
所以在恢复时可以不写数据库名就可以导入。
总的来说就算只备份一个库也建议使用。

### 压缩_gzip
可以在导出数据使用使用管道传给gzip压缩。
因为mysql导出数据全是文本的原因，压缩效率高，压缩后只有原体积50%左右。
- 导出数据
```shell
mysqldump -uroot -p123 -B test|gzip > ./test-B.sql.gz
```
----------

# grant/revoke

授权格式：
`GRANT 权限 ON 数据库/表 TO 用户;`
取消授权格式：
`REVOKE 权限 ON 数据库/表 FROM 用户`

查看 权限_数据库_用户 可选值
`show  privileges;`

| Privilege                | Context                               | Comment                                                            |
|--|--|--|
| Alter                    | Tables                                | To alter the table                                                 |
| Alter routine            | Functions,Procedures                  | To alter or drop stored functions/procedures                       |
| Create                   | Databases,Tables,Indexes              | To create new databases and tables                                 |
| Create routine           | Databases                             | To use CREATE FUNCTION/PROCEDURE                                   |
| Create temporary tables  | Databases                             | To use CREATE TEMPORARY TABLE                                      |
| Create view              | Tables                                | To create new views                                                |
| Create user              | Server Admin                          | To create new users                                                |
| Delete                   | Tables                                | To delete existing rows                                            |
| Delete history           | Tables                                | To delete versioning table historical rows                         |
| Drop                     | Databases,Tables                      | To drop databases, tables, and views                               |
| Event                    | Server Admin                          | To create, alter, drop and execute events                          |
| Execute                  | Functions,Procedures                  | To execute stored routines                                         |
| File                     | File access on server                 | To read and write files on the server                              |
| Grant option             | Databases,Tables,Functions,Procedures | To give to other users those privileges you possess                |
| Index                    | Tables                                | To create or drop indexes                                          |
| Insert                   | Tables                                | To insert data into tables                                         |
| Lock tables              | Databases                             | To use LOCK TABLES (together with SELECT privilege)                |
| Process                  | Server Admin                          | To view the plain text of currently executing queries              |
| Proxy                    | Server Admin                          | To make proxy user possible                                        |
| References               | Databases,Tables                      | To have references on tables                                       |
| Reload                   | Server Admin                          | To reload or refresh tables, logs and privileges                   |
| Binlog admin             | Server                                | To purge binary logs                                               |
| Binlog monitor           | Server                                | To use SHOW BINLOG STATUS and SHOW BINARY LOG                      |
| Replication master admin | Server                                | To monitor connected slaves                                        |
| Replication slave admin  | Server                                | To start/monitor/stop slave and apply binlog events                |
| Replication slave        | Server Admin                          | To read binary log events from the master                          |
| Select                   | Tables                                | To retrieve rows from table                                        |
| Show databases           | Server Admin                          | To see all databases with SHOW DATABASES                           |
| Show view                | Tables                                | To see views with SHOW CREATE VIEW                                 |
| Shutdown                 | Server Admin                          | To shut down the server                                            |
| Super                    | Server Admin                          | To use KILL thread, SET GLOBAL, CHANGE MASTER, etc.                |
| Trigger                  | Tables                                | To use triggers                                                    |
| Create tablespace        | Server Admin                          | To create/alter/drop tablespaces                                   |
| Update                   | Tables                                | To update existing rows                                            |
| Set user                 | Server                                | To create views and stored routines with a different definer       |
| Federated admin          | Server                                | To execute the CREATE SERVER, ALTER SERVER, DROP SERVER statements |
| Connection admin         | Server                                | To bypass connection limits and kill other users' connections      |
| Read_only admin          | Server                                | To perform write operations even if @@read_only=ON                 |
| Usage                    | Server Admin                          | No privileges - allow connect only                                 |
39 rows in set (0.002 sec)

可以及其细粒度的授权：

```sql
GRANT select(ename) ON mydb.EMP TO muyu;
```
授予用户muyu查询mydb数据库，emp表ename字段 查询的权限。

也就用户muyu只能使用select ename from emp; 只有查询mydb数据库emp表内这一个字段的权限。其他什么都干不了