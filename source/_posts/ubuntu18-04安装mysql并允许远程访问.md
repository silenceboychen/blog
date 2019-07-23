---
title: ubuntu18.04安装mysql并允许远程访问
toc: true
date: 2019-07-23 18:29:37
categories:
tags:
---

### 安装

```
$ apt-get install mysql-server mysql-client
```

### 字符集修改utf8

进入mysql命令终端：（默认root密码为空）:

```
$ mysql -u root -p
```

```
mysql> show variables like 'char%';
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
| character_set_database   | latin1                     |
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
| character_set_server     | latin1                     |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
```

```
mysql> show variables like 'collation%';
+----------------------+-------------------+
| Variable_name        | Value             |
+----------------------+-------------------+
| collation_connection | utf8_general_ci   |
| collation_database   | latin1_swedish_ci |
| collation_server     | latin1_swedish_ci |
+----------------------+-------------------+
```

修改字符集：

```
$ vim /etc/mysql/mysql.conf.d/mysqld.cnf

// 在文件末尾添加以下内容：
collation-server = utf8_unicode_ci
init-connect='SET NAMES utf8'
character-set-server = utf8
```

重启后字符集修改为utf8： 

```
$ service mysql restart
```

### 修改端口号

修改mysql配置文件，然后重启即可生效：

```
$ vim /etc/mysql/mysql.conf.d/mysqld.cnf

// 修改以下内容：
port = 6033
```

### 登录权限问题

查看当前用户：

```
mysql> SELECT User,Host FROM mysql.user;
+------------------+-----------+
| User             | Host      |
+------------------+-----------+
| debian-sys-maint | localhost |
| mysql.session    | localhost |
| mysql.sys        | localhost |
| root             | localhost |
+------------------+-----------+
4 rows in set (0.00 sec)
```

删除root账号：

```
mysql> DROP USER 'root'@'localhost';
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT User,Host FROM mysql.user;
+------------------+-----------+
| User             | Host      |
+------------------+-----------+
| debian-sys-maint | localhost |
| mysql.session    | localhost |
| mysql.sys        | localhost |
+------------------+-----------+
3 rows in set (0.00 sec)
```

重新创建root：

```
mysql> CREATE USER 'root'@'%' IDENTIFIED BY '123456';
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT User,Host FROM mysql.user;
+------------------+-----------+
| User             | Host      |
+------------------+-----------+
| root             | %         |
| debian-sys-maint | localhost |
| mysql.session    | localhost |
| mysql.sys        | localhost |
+------------------+-----------+
4 rows in set (0.00 sec)
```

授权：

```
mysql> GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION;
Query OK, 0 rows affected (0.01 sec)

mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.00 sec)
```

退出mysql，修改配置文件：

```
$ vim /etc/mysql/mysql.conf.d/mysqld.cnf
注释这一行：bind-address:127.0.0.1
```

重新启动mysql:

```
$ service mysql restart
```