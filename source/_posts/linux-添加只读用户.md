---
title: linux 添加只读用户
toc: true
date: 2019-10-16 16:31:43
categories: linux
tags: linux
---


> 这里未使用rbash新建用户，使用rbash新建只读用户不能使用cd等内置命令。


*  添加用户

```
# useradd -m test
```

* 设置密码

```
# passwd test
```

* 修改用户的`shell`配置文件

```
# chown root. /home/test/.bash_profile
# chmod 755 /home/test/.bash_profile
```

* 修改`/home/test/.bash_profile`配置文件

> 将`PATH`改为`$HOME/.bin`

```
# .bash_profile

# Get the aliases and functions
if [ -f ~/.bashrc ]; then
	. ~/.bashrc
fi

# User specific environment and startup programs

# PATH=$PATH:$HOME/.local/bin:$HOME/bin
PATH=$HOME/.bin
export PATH
```

* 创建用户`.bin`目录

```
# mkdir /home/test/.bin
```

* 将允许执行的命令链接到`/home/test/.bin`目录

```
ln -s /usr/bin/wc /home/test/.bin/wc
ln -s /usr/bin/tail /home/test/.bin/tail
ln -s  /usr/bin/more /home/test/.bin/more
ln -s  /usr/bin/cat /home/test/.bin/cat
ln -s  /usr/bin/grep /home/test/.bin/grep
ln -s  /usr/bin/find /home/test/.bin/find
ln -s  /usr/bin/pwd /home/test/.bin/pwd
ln -s  /usr/bin/ls /home/test/.bin/ls
ln -s  /usr/bin/less /home/test/.bin/less
```

之后使用创建的用户登录系统，用户只拥有只读权限，只能使用软连接的命令。