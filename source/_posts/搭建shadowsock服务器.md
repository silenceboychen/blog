---
title: 搭建shadowsock服务器
toc: true
date: 2018-01-19 09:37:28
categories: shadowsock
tags: shadowsock
---

## 安装

Debian / Ubuntu:

```
$ apt-get install python-pip
$ pip install shadowsocks
```

CentOS:

```
$ yum install python-setuptools && easy_install pip
$ pip install shadowsocks
```

## 启动

> 有两种启动方式，建议使用配置文件的方式启动

直接启动：

```
ssserver -p 8388 -k password -m rc4-md5 -d start
```

使用配置文件启动：

执行``vim /etc/shadowsocks.json ``添加如下内容：

```
{
    "server":"0.0.0.0",
    "server_port":8388,
    "local_address": "127.0.0.1",
    "local_port":1080,
    "password":"mypassword",
    "timeout":300,
    "method":"rc4-md5"
}
```

多用户配置如下：

```
{  
 "server":"0.0.0.0"，  
 "local_address": "127.0.0.1",  
 "local_port":1080,  
  "port_password": {  
     "8388": "password",  
     "8387": "password",  
     "8386": "password",  
     "8385": "password"  
 },  
 "timeout":300,  
 "method":"rc4-md5",  
 "fast_open": false  
}  
```

然后通过执行一下命令启动：

> 如果要停止运行，将命令中的start改成stop。

```
$ ssserver -c /etc/shadowsocks.json -d start
```

TIPS: 加密方式推荐使用rc4-md5，因为 RC4 比 AES 速度快好几倍，如果用在路由器上会带来显著性能提升。旧的 RC4 加密之所以不安全是因为 Shadowsocks 在每个连接上重复使用 key，没有使用 IV。现在已经重新正确实现，可以放心使用。更多可以看 issue。

## 开机自启

编辑一下/etc/supervisord.conf文件，命令如下：

```
$ vim /etc/supervisord.conf
```

把下面的内容粘贴到文件尾部的空行处，然后保存：

```
[program:shadowsocks]
command=ssserver -c /etc/shadowsocks.json
autostart=true
autorestart=true
user=root
log_stderr=true
logfile=/var/log/shadowsocks.log
```

接下来需要编辑一下/etc/rc.local文件，请执行以下命令：

```
$ vi /etc/rc.local
```

请把以下内容粘贴到文件中部的空白处，然后保存

```
$ service supervisord start
```

完成以上步骤后，重启之后，shadowsock会自动运行。

## 问题

> 如果通过客户端始终代理失败，可以通过一下方法查找问题。

1.客户端通过 telnet ip port 确认 ss-server 是否正常开启，如果没有正常开启，有可能是设定的端口没有开放，

```
$ iptables -A INPUT -p tcp --dport 8388 -j ACCEPT
```

执行上述命令，将 8388 修改为你设定的端口即可。

2.如果第一步中连接正常，可以查看下 ss-server 的日志

```
$ ssserver -c /etc/shadowsocks.json --log-file /var/log/shadowsocks.log -d start
```

启动的时候添加 ``--log-file`` 参数，然后通过

```
$ tail -f /var/log/shadowsocks.log
```

查看实时日志，一般可以看出一点端倪。
