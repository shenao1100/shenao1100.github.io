---
title: 近期使用MySQL踩过的坑
description: MySQL入门操作
author: ShenNya
date: 2025-03-18 14:56:00 +0800
categories: [MySQL]
tags: [MySQL]
#pin: true
math: true
mermaid: true
---

## 背景

近期使用Docker封装了一个环境，为了方便迁移，我想将我电脑的测试环境的数据库导出，之后在服务器上导入

主机环境：

- 运行在Windows上的 `MariaDB 10.5.10`

- 运行在WSL中Docker的服务器

服务器环境：

- 运行在CentOS上的`MySQL 8.0.41`

- 运行在CentOS中Docker的服务器

## 登录

首先遇到的就是登陆问题，MySQL除了用户名和密码，对`登录IP`也有限制

在服务器放行`3306`端口后，使用Windows连接服务器的MySQL报错：

```
1130 - Host XXX.XXX.XXX.XXX is not allowed to connect to this MySQL server
```

长话短说，`MySQL 8.0`默认使用了`caching_sha2_password`认证插件，所以需要创建远程访问用户

> 其他版本可能不同

1. 在服务器上连接到MySQL

```bash
mysql -u root -p
```
输入密码后即可链接

2. 创建用户并允许访问

```sql
CREATE USER 'root'@'XXX.XXX.XXX.XXX' IDENTIFIED BY 'your_password';
GRANT ALL PRIVILEGES ON *.* TO 'root'@'XXX.XXX.XXX.XXX' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

`XXX.XXX.XXX.XXX`: 连接到MySQL的IP

`your_password`: MySQL密码

3. 在服务器中开启服务端测试

又遇到了服务端报错：

```
1130 - Host 10.88.0.1 is not allowed to connect to this MySQL server
```

这次是因为`Docker`默认分配的子网IP是`10.88.0.X`

于是放行整个`Docker子网`

```sql
CREATE USER 'root'@'10.88.0.%' IDENTIFIED BY 'your_password';
GRANT ALL PRIVILEGES ON *.* TO 'root'@'10.88.0.%' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

`%`在这里位任意匹配`*`


## 数据库大小和行数不正确

我使用`HeidiSQL`导出了主机中大小为44mb的数据库为sql文件，这个文件也有40多MB大

但是当我使用`HeidiSQL`连接到服务器后，发现服务器的数据库却只有1.3MB

许多表显示行数为0，然而当我查看这些表的详细数据，却发现其中有数据，和导出的数据库一样

### 检查数据库存储引擎

那么首先先检查是不是存储引擎不兼容，在主机和服务器的数据库中运行

```sql
SELECT table_name, engine FROM information_schema.tables WHERE table_schema = 'your_database_name';
```
`your_database_name` 为表名

![alt text](../imgs/2025-03-18-mysql-guide/image.png)

发现全为`Inno DB`，没有什么问题

### 导入是否有报错？

检查MySQL的日志，一般在这里

```
/var/log/mysql/error.log
```

看看有没有类似这样的报错

```
ERROR 1062 (23000): Duplicate entry
ERROR 1452 (23000): Cannot add or update a child row
```
然而并没有，导入很顺利

### 索引或者统计信息没有更新？

可能是因为导入后数据库没有更新，在MySQL中输入

```sql
ANALYZE TABLE your_table;
OPTIMIZE TABLE your_table;
```

即可强制更新

运行之后发现统计信息都出来了，确实是没有更新

但是数据库中表太多了，手动更新很麻烦，有没有方法批量更新？

有的兄弟，有的

可以让MySQL批量生成命令，之后执行

```sql
SELECT CONCAT('ANALYZE TABLE ', table_schema, '.', table_name, ';') 
FROM information_schema.tables 
WHERE table_schema = 'your_database_name';

SELECT CONCAT('OPTIMIZE TABLE ', table_schema, '.', table_name, ';') 
FROM information_schema.tables 
WHERE table_schema = 'your_database_name';
```
`your_database_name` 为数据库名

执行之后复制生成的SQL语句再执行就可以了


## 大小写敏感

叒启动服务端，这次报错表不存在

```
java.sql.SQLSyntaxErrorException: Table 'guizhou_basis_frame.QRTZ_JOB_DETAILS' doesn't exist
```

然而这个表很明显是是存在的，只不过叫

```
qrtz_job_details
```

是否是数据库开启了大小写敏感？

输入下列命令查看：

```sql
SHOW VARIABLES LIKE 'lower_case_table_names';
```

如果返回`0`，则表名区分大小写

如果返回`1`，则表名不区分大小写

很不幸的，是`0`

之后我觉得只要改改配置文件应该就可以了

于是在MySQL的配置文件中加上

```ini
[mysqld]
lower_case_table_names=1
```

重启MySQL服务

报错了？

```
Job for mysqld.service failed because the control process exited with error code.
See "systemctl status mysqld.service" and "journalctl -xe" for details.
```

之后我了解到

### *lower_case_table_names只能在初始化数据库之前设置，数据库里有东西是不能修改的*

唯一的解决方法是备份数据库重来

于是关闭数据库，删除MySQL数据目录

```bash
systemctl stop mysql
rm -rf /var/lib/mysql
```

在数据库配置文件中添加

```ini
[mysqld]
lower_case_table_names=1
```

重新初始化MySQL

> 这里的`--user=mysql`是因为在 Linux 环境下，MySQL 通常会使用一个专门的 `mysql` 用户账户运行，而不是 `root`，这样可以提高安全性，防止 MySQL 进程具有过高权限

```bash
mysqld --initialize --user=mysql
```

重新初始化之后，会生成随机的root密码

使用

```bash
grep 'temporary password' /var/log/mysqld.log
```

获取这个随机密码

进入MySQL后修改密码

```sql
ALTER USER 'root'@'localhost' IDENTIFIED BY 'NewStrongPassword';
```

之后就是把之前的全部重来了，这下服务端终于能跑起来了