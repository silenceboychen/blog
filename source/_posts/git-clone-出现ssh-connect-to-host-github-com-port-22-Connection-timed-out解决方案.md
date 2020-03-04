---
title: 'git clone 出现ssh: connect to host github.com port 22: Connection timed out解决方案'
toc: true
date: 2020-03-04 13:07:40
categories: git
tags: git
---

物理机服务器，安装`git`之后，想从`github`上`clone`自己的项目运行，`ssh key`已经配置过, 但在执行`git clone`命令时出现了如下报错：

```
ssh: connect to host github.com port 22: Connection timed out
fatal: 无法读取远程仓库。

请确认您有正确的访问权限并且仓库存在。
```

### 解决方案

在系统`~/.ssh`目录下执行`touch config`命令新建`config`文件，并修改文件权限：

```
sudo chmod 600 config
```

然后在`config`文件中添加如下内容：

```
Host github.com
User email@qq.com  // 替换成自己的github登录邮箱
Hostname ssh.github.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/id_rsa
Port 443
```

然后设置：

```
git config --global user.name "XXX"
git config --global user.email XXX@xx.com
```

此时再去执行`git clone`命令，一切正常。