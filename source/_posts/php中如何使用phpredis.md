---
title: php中如何使用phpredis
toc: true
date: 2015-12-24 10:34:10
categories: php
tags: [php, redis]
---
安装redis服务：
下载地址：http://redis.io/download，下载最新文档版本。
本教程使用的最新文档版本为 2.8.17，下载并安装：

    $ wget http://download.redis.io/releases/redis-2.8.17.tar.gz
    $ tar xzf redis-2.8.17.tar.gz
    $ cd redis-2.8.17
    $ make

make完后 redis-2.8.17目录下会出现编译后的redis服务程序redis-server,还有用于测试的客户端程序redis-cli,两个程序位于安装目录 src 目录下：
下面启动redis服务.

    $ cd src
    $ ./redis-server

注意这种方式启动redis 使用的是默认配置。也可以通过启动参数告诉redis使用指定配置文件使用下面命令启动。

    $ cd src
    $ ./redis-server redis.conf

redis.conf是一个默认的配置文件。我们可以根据需要使用自己的配置文件。
启动redis服务进程后，就可以使用测试客户端程序redis-cli和redis服务交互了。 比如：

    $ cd src
    $ ./redis-cli
    redis> set foo bar
    OK
    redis> get foo
    "bar"

安装PHP redis 驱动
装 PHP redis 驱动：下载地址为:https://github.com/nicolasff/phpredis。
首先git clone 项目到本地，切换到phpredis目录下
在shell中输入 phpize 然后 ./configure 进行配置（ps:可能找不到phpize，phpize是属于php-devel的内容，因此在centos中只要运行如下命令：yum install php-devel 然后就会安装上phpize了。）
接下来就是最后的make 和make install了，make 之后记得跑一下 make test，在make install中遇到点权限问题，所以要加上sudo
这样就完成了phpredis的编译工作，接下来我们需要来配置了。

然后，在PHP.INI 配置文件中添加一条extension = redis.so 就OK

对了，别忘了重启Apache