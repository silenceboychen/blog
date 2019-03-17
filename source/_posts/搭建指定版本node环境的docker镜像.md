---
title: 搭建指定版本node环境的docker镜像
toc: true
date: 2019-03-17 17:36:12
categories: docker
tags: [nodejs, docker]
---

> 基于ubuntu16.04的docker镜像去打包安装了nodejs环境的docker镜像

## 前置条件
1.获取ubuntu16.04镜像
```
# docker pull ubuntu:16.04
```
2.基于ubuntu16.04镜像启动容器
```
# docker run -ti --name ubuntu ubuntu:16.04 /bin/bash
```

## 从源代码安装Node.JS

> 安装node过程均在容器内进行

1.更新源并安装必要工具
```
# apt-get update
# apt-get install git wget python gcc make g++
```
2.获取指定版本的node源代码
> 这里我们使用v8.9.0版,目前为长期支持版,可以使用两中获取源码的方式.
```
# wget http://nodejs.org/dist/v8.9.0/node-v8.9.0.tar.gz
# tar zxvf node-v8.9.0.tar.gz
# mv node-v8.9.0 node
```
or
```
# git clone -b v8.9.0 git@github.com:nodejs/node.git
```
3.修改目录权限
```
# chmod -R 755 node
```
4.编译安装node
```
# cd node
# ./configure
# make
# make install
```
5.查看node版本
```
# node --version
v8.9.0
```
> 安装完成后退出镜像

## 利用包管理器安装Node.JS

> 安装在镜像内进行

1.更新源并安装必要工具
> setup_8.x为安装8.x版本,若安装9.x版本为:setup_9.x
```
# apt-get update
# apt-get install curl
# curl -sL https://deb.nodesource.com/setup_8.x | bash -
```
2.安装nodejs
```
# apt-get install -y nodejs

```
3.查看node版本
```
# node --version
v8.9.0
```
> 安装完成后退出镜像

## 从容器创建一个新的镜像

> 注意: 在上一步已经退出容器,下面的操作是在本机上进行的.

1.执行`` docker ps -a`` 查看name为ubuntu的ID
![clipboard.jpeg](/../images/clipboard.jpeg)

2.创建新的镜像
```
$ docker commit -a "author" -m "commit message" b0084b239645 xxx/node8.9:v1
sha256:bc03d86ef63bab18deafe643f99b2aa1da5697860e1432102dbbcbb281fdf335
```
* -a: 作者信息
* -m: 提交信息
* b0084b239645: ``docker ps -a``中查看的ID
* xxx/node8.9:v1: 新的镜像名称

3.上传到镜像仓库

镜像制作完成可以将镜像上传到镜像仓库,便于以后使用,可以指定仓库地址,也可以使用官方的仓库.
```
$ docker push xxx/node8.9:v1
```