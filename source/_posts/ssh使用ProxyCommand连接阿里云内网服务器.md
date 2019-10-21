---
title: ssh使用ProxyCommand连接阿里云内网服务器
toc: true
date: 2019-10-21 13:44:57
categories: ssh
tags: ssh
---

在没有发现proxyCommand命令的好处之前，本地连接想要访问内网服务器，需要先ssh连接开放外网ip并且与我们要访问的目标主机在同一个内网环境的esc服务器，然后将该服务器作为跳板机，在该服务器上ssh连接内网服务器。该操作非常麻烦。使用proxyCommand能够很方便的解决该问题。

### proxyCommand配置

修改`~/.ssh/config`：

```
Host tiaoban
Hostname 跳板机的ip
Port 跳板机的端口(如果是非22的需要填写)
User root(如果非root,换成跳板机的用户)
Host target
Hostname 目标机的IP
Port 跳板机的端口(如果是非22的需要填写)
User root(如果非root,换成跳板机的用户)
ProxyCommand ssh -q -x -W %h:%p tiaoban
```

这儿的`%h`表示要连接的目标机,也就是`Hostname`指定的ip或者主机名,`%p`表示要连接到目标机的端口.这儿可以直接写死固定值,但是使用`%h`和`%p`可以保证在`Hostname`和`Port`变化的情况下`ProxyCommand`这行不用跟着变化.

然后我们直接ssh target,可以看到直接就连接上了.