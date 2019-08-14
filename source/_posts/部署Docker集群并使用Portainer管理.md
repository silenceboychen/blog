---
title: 部署Docker集群并使用Portainer管理
toc: true
date: 2019-08-14 12:34:06
categories: docker
tags: [docker, portainer]
---

> 在有多台Docker的情况下，进行集群管理就十分重要了，Portainer也支持集群管理，Portainer可以和Swarm一起来进行集群管理操作。

### 环境要求

* 需要提前安装docker环境，[docker安装教程](https://docs.docker.com/install/linux/docker-ce/ubuntu/)。
* 使用docker安装portainer， [安装教程](https://www.portainer.io/installation/)

### 搭建Swarm集群环境

**基本环境**

用两台机器来搭建(都是ubuntu18.04系统)

```
172.16.132.200    docker-manager
172.16.132.201    dcoker-worker01
```

**修改两台机器的主机名并做hosts**

* `172.16.132.200`机器
```
# hostnamectl set-hostname docker-manager
# echo "docker-manager" > /etc/hostname
# vim /etc/hosts
172.16.132.200    docker-manager
172.16.132.201    dcoker-worker01
```

* `172.16.132.201`机器
```
# hostnamectl set-hostname docker-worker01
# echo "docker-worker01" > /etc/hostname
# vim /etc/hosts
172.16.132.200    docker-manager
172.16.132.201    dcoker-worker01
```

### 开通对外2375端口（方便portainer管理）

```
// 先做备份
# cp  /lib/systemd/system/docker.service /lib/systemd/system/docker.service.bak 
# vim /lib/systemd/system/docker.service     
    找到ExecStart行改成这样的：  ExecStart=/usr/bin/dockerd -H fd:// -H tcp://0.0.0.0:2375
# systemctl daemon-reload
# systemctl restart docker     重启docker服务，使用service docker restart也可以
# netstat -plnt   查看端口号使用
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      20917/nginx: master 
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      742/sshd            
tcp6       0      0 :::2375                 :::*                    LISTEN      5836/dockerd        
tcp6       0      0 :::7946                 :::*                    LISTEN      5836/dockerd        
tcp6       0      0 :::80                   :::*                    LISTEN      20917/nginx: master
```

### Swarm集群创建

**初始化Swarm**

```
# docker swarm init --advertise-addr 172.16.132.200
Swarm initialized: current node (7ggeai3dlqn0j8gkxjs46y250) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-0n04bao3bkte48prmcf1xfmlfrk9zh19b9u16ysb63yhvgjiyi-3w0rd58keboh52zul8xcjfrof 172.16.132.200:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

上面命令执行后，该机器自动加入到swarm集群。这个会创建一个集群token，获取全球唯一的 token，作为集群唯一标识。
后续将其他节点加入集群都会用到这个token值。

```
--advertise-addr 指定与其他 node 通信的地址。

docker swarm init 输出告诉我们：

① swarm 创建成功，swarm-manager 成为 manager node。

② 添加 worker node 需要执行的命令。

③ 添加 manager node 需要执行的命令。
```

**添加集群节点**

在`docker-worker01`机器上执行以下添加集群节点的操作命令:

```
# docker swarm join --token SWMTKN-1-0n04bao3bkte48prmcf1xfmlfrk9zh19b9u16ysb63yhvgjiyi-3w0rd58keboh52zul8xcjfrof 172.16.132.200:2377
```

如后续要加入其他更多的节点,添加操作也是执行这个命令.

**查看集群节点**

在`docker-manager`机器上执行查看,因为此时它是`swarm`集群的`leader`节点:

```
root@docker-manager:~# docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
7ggeai3dlqn0j8gkxjs46y250 *   docker-manager      Ready               Active              Leader              18.09.7
1v09483w8dczd36bmtruzm2ix     docker-node01       Ready               Active                                  19.03.1
```

**最后查看下两个机器上的2375端口是否都已经开启了**

```
[root@docker-manager ~]# lsof -i:2375
COMMAND   PID USER   FD   TYPE  DEVICE SIZE/OFF NODE NAME
dockerd 13785 root    5u  IPv6 4518841      0t0  TCP *:2375 (LISTEN)

[root@docker-woeker01 ~]# lsof -i:2375
COMMAND    PID USER   FD   TYPE  DEVICE SIZE/OFF NODE NAME
dockerd-c 2966 root    5u  IPv6 3602947      0t0  TCP *:2375 (LISTEN)
```

### 部署Portainer

```
$ docker volume create portainer_data
$ docker run -d -p 8000:8000 -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer
$ docker ps
CONTAINER ID        IMAGE                     COMMAND             CREATED             STATUS                  PORTS                                                               NAMES
9b051147a4c2        portainer/portainer       "/portainer"        20 hours ago        Up 18 hours             0.0.0.0:9000->9000/tcp                                              portainer
```

访问http://172.16.132.200:9000,   同样首次登陆需要注册用户，给admin用户设置密码：
![](/../images/portainer-login.png)

集群模式, 这样一定要选择Remote, 输入docker-worker01的ip，然后点击Connect。
![](/../images/portainer-remote.png)

同样点击左边栏的"Endpoints" - "+add endpoint", 添加集群节点:
![](/../images/portainer-endpoints.png)

添加之后,点击左边栏的"Home", 右边就可以看到节点信息了,可以进行切换操作.
![](/../images/portainer-home.png)