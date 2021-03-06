---
title: Linux Shell >/dev/null 2>&1 &含义
toc: true
date: 2019-04-01 18:04:28
categories: shell
tags: [linux, shell]
---

shell中可能经常能看到：`echo log > /dev/null 2>&1 &!`, 但具体代表什么意思，很多人都不理解，下面对此进行一一讲解：

* `>`: 代表重定向到哪里，例如：`echo "123" > /home/123.txt`, 表示将字符串`'123'`写入文件`/home/123.txt`中,

* `/dev/null`: 代表空设备文件.

* `%>`: 用来定义输出形式，其中`'%'`有几种可选值:

  * 0: 标准输入
  * 1: 表示`stdout`标准输出，系统默认值是1，所以`">/dev/null"`等同于`"1>/dev/null`"
  * 2: 表示`stderr`标准错误
  * &: 表示等同于的意思，`2>&1`，表示2的输出重定向等同于1

> 所以`> /dev/null 2>&1`可以分两步理解为:
>  1. `1 > /dev/null`: 首先表示标准输出重定向到空设备文件，也就是不输出任何信息到终端，说白了就是不显示任何信息。
>  2. `2>&1`: 接着，标准错误输出重定向（等同于）标准输出，因为之前标准输出已经重定向到了空设备文件，所以标准错误输出也重定向到空设备文件。

  
* 最后一个`&!` ， 是让该命令在后台执行，并且关闭终端后不退出。
