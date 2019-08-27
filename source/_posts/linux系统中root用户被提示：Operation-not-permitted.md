---
title: linux系统中root用户被提示：Operation not permitted
toc: true
date: 2019-08-27 16:01:26
categories: linux
tags: linux
---

### 问题

在修改文件权限时遇到如下报错：

```
root@docker-manager:~/.ssh# chmod 600 authorized_keys 
chmod: changing permissions of 'authorized_keys': Operation not permitted
```


### 解决方法

这里涉及到`chattr`和`lsattr`的知识：

`chattr`是用来更改文件属性，`lsattr`可用来查看文件的属性，执行命令`lsattr authorized_keys`便可以看到当前文件的属性；

```
root@docker-manager:~/.ssh# lsattr authorized_keys 
----ia--------e--- authorized_keys
```

可以发现当前文件有个i属性，查阅命令帮助文档可以看到有i属性的文件是不能修改的，更不可被删除，即使是root用户也不可。

这里只需要去除i属性就可以修改文件权限。

```
root@docker-manager:~/.ssh# chattr -i authorized_keys 
root@docker-manager:~/.ssh# lsattr authorized_keys 
-----a--------e--- authorized_keys
```

### chattr命令

Linux chattr命令用于改变文件属性。

这项指令可改变存放在ext2文件系统上的文件或目录属性，这些属性共有以下8种模式：

```
a：让文件或目录仅供附加用途。
b：不更新文件或目录的最后存取时间。
c：将文件或目录压缩后存放。
d：将文件或目录排除在倾倒操作之外。
i：不得任意更动文件或目录。
s：保密性删除文件或目录。
S：即时更新文件或目录。
u：预防意外删除。
```

**参数**

```
-R 递归处理，将指定目录下的所有文件及子目录一并处理。

-v<版本编号> 设置文件或目录版本。

-V 显示指令执行过程。

+<属性> 开启文件或目录的该项属性。

-<属性> 关闭文件或目录的该项属性。

=<属性> 指定文件或目录的该项属性。
```