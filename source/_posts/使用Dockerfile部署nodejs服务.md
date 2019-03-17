---
title: 使用Dockerfile部署nodejs服务
toc: true
date: 2017-12-04 17:49:49
categories: nodejs
tags: [nodejs, docker, dockerfile]
---

## 初始化Dockerfile

假设我们的项目名为``express``,在``express``项目中创建编辑``Dockerfile``文件：
```
$ vim Dockerfile

FROM node:latest

RUN mkdir -p /home/www/express
WORKDIR /home/www/express

COPY . /home/www/express

RUN npm install

EXPOSE 3000

ENTRYPOINT ["npm", "run"]
CMD ["start"]
```
这个文件包含了以下命令：

* ``FROM node:latest`` - 指定使用最新版本的node基础镜像
* ``RUN mkdir -p /home/www/express`` - 在容器内创建/home/www/express目录
* ``WORKDIR /home/www/express`` - 将容器内工作目录设置为/home/www/express
* ``COPY . /home/www/express`` - 将宿主机当前目录下内容复制到镜像/home/www/express目录下
* ``RUN npm install`` - npm install安装应用所需的NPM包
* ``EXPOSE 3000`` - 对外开放容器的3000端口
* ``ENTRYPOINT ["npm", "run"]`` - 容器启动后执行的命令。不可被``docker run``提供的参数覆盖
* ``CMD ["start"]`` - 在容器启动时，执行的命令，可被``docker run``提供的参数覆盖

## 构建镜像

编写完Dockerfile文件后，就可以通过docker build命令来构建镜像：
```
$ sudo docker build -t test/express .
```
> 我们通过-t参数，将镜像命名为test/express。构建过程类似如下：
```
Sending build context to Docker daemon  29.7 kB
Step 1/8 : FROM registry.src.followme.com:5000/node:v1
 ---> c99c549e8227
Step 2/8 : RUN mkdir -p /home/www/express-app
 ---> Running in 8be9a90629b0
 ---> b9f584851225
Removing intermediate container 8be9a90629b0
Step 3/8 : WORKDIR /home/www/express-app
 ---> 5072c31f9dd9
Removing intermediate container e9dbf4ce3d8b
Step 4/8 : COPY . /home/www/express-app
 ---> a4d1725f15ed
Removing intermediate container 30aa49765015
Step 5/8 : RUN yarn
 ---> Running in f181c243deaa
yarn install v1.3.2
[1/4] Resolving packages...
[2/4] Fetching packages...
[3/4] Linking dependencies...
[4/4] Building fresh packages...
Done in 9.46s.
 ---> d390931d73e6
Removing intermediate container f181c243deaa
Step 6/8 : EXPOSE 3000
 ---> Running in 94101ab38864
 ---> 43199a8a5a90
Removing intermediate container 94101ab38864
Step 7/8 : ENTRYPOINT npm run
 ---> Running in 80b1318962cf
 ---> 6b203c50e855
Removing intermediate container 80b1318962cf
Step 8/8 : CMD start
 ---> Running in a9909e537f59
 ---> d56eae48377c
Removing intermediate container a9909e537f59
Successfully built d56eae48377c
```
## 运行容器

镜像构建完成后，可以通过所构建的镜像创建/运行容器，从而实现express应用的 Docker 化部暑。

使用tets/express镜像运行一个容器：
```
$ sudo docker run -d --name experss-app -p 3000:3000 test/express
```
在以上操作中，我们通过``test/express``镜像运行了容器，并将容器命名为``experss-app``。运行容器，我们还指定了``-d``参数，该参数使容器以后台的方式运行。而``-p``参数将宿主机的3000端口映射到了容器的3000端口。运行容器后，可以通过docker ps命令看到运行中的容器。此时可通过``localhost:3000``访问服务。