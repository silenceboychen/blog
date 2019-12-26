---
title: mysql5.7 ibtmp1文件过大
toc: true
date: 2019-12-26 16:29:14
categories: mysql
tags: mysql
---


服务器上的磁盘被占满，通过一下命令查找服务器上的大文件：

```
$ sudo find / -type f -size +800M  -print0 | xargs -0 du -h | sort -nr
```

经过排查后发现，`/var/lib/mysql`目录下面有一个`ibtmp1`的文件特别大，有64G 。

ibtmp1是个什么东西呢？查看官方文档后发现这是非压缩的innodb临时表的独立表空间。通过`innodb_temp_data_file_path`参数指定文件的路径，文件名和大小，默认配置为`ibtmp1:12M:autoextend`，也就是说在支持大文件的系统这个文件大小是可以无限增长的。

**解决办法：**

* 修改my.cnf配置文件：

```
innodb_temp_data_file_path = ibtmp1:12M:autoextend:max:5G
```

* 设置innodb_fast_shutdown参数

```
SET GLOBAL innodb_fast_shutdown = 0;  #InnoDB does a slow shutdown, a full purge and a change buffer merge before shutting down
```

* 关闭mysql服务

```
systemctl stop mysqld.service
```

* 删除ibtmp1文件

* 启动mysql服务

```
systemctl start mysqld.service
```
