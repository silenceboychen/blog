---
title: Docker启用TLS进行安全配置
toc: true
date: 2019-09-29 09:56:59
categories: docker
tags: docker
---

之前开启了docker的2375 Remote API，由于没有启用TLS，导致服务器被入侵，安装了挖矿程序。所以如果想开通docker远程访问，就必须做好安全验证。

> **文中出现的$HOST指的是主机ip， 实际执行时用主机ip替换即可。**



### 在Docker守护程序的主机上，生成CA私钥和公钥：

```
//  生成 CA 私钥
root@docker-manager:~# openssl genrsa -aes256 -out ca-key.pem 4096
Generating RSA private key, 4096 bit long modulus (2 primes)
.........................................................................++++
.............................................++++
e is 65537 (0x010001)
// 需要输入两次自定义密码
Enter pass phrase for ca-key.pem:
Verifying - Enter pass phrase for ca-key.pem:


// 生成 CA 公钥
root@docker-manager:~# openssl req -new -x509 -days 365 -key ca-key.pem -sha256 -out ca.pem
// 这里需要输入第一步设置的密码
Enter pass phrase for ca-key.pem:
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:CN
State or Province Name (full name) [Some-State]:zhejiang
Locality Name (eg, city) []:hangzhou
Organization Name (eg, company) [Internet Widgits Pty Ltd]:Docker Inc
Organizational Unit Name (eg, section) []:
Common Name (e.g. server FQDN or YOUR name) []: $HOST
Email Address []:
```

### 创建服务器密钥和证书签名请求(CSR)

```
root@docker-manager:~# openssl genrsa -out server-key.pem 4096
Generating RSA private key, 4096 bit long modulus (2 primes)
......................................................................................................++++
................++++
e is 65537 (0x010001)

root@docker-manager:~# openssl req -subj "/CN=$HOST" -sha256 -new -key server-key.pem -out server.csr
```

### 用CA签署公钥

由于可以通过IP地址和DNS名称建立TLS连接，因此在创建证书时需要指定IP地址。例如，允许使用10.10.10.20和进行连接127.0.0.1：

```
root@docker-manager:~# echo subjectAltName = DNS:$HOST,IP:172.16.132.200,IP:127.0.0.1 >> extfile.cnf
```

将Docker守护程序密钥的扩展用法属性设置为仅用于服务器身份验证：

```
root@docker-manager:~# echo extendedKeyUsage = serverAuth >> extfile.cnf
```

### 生成服务端签名证书

```
root@docker-manager:~# openssl x509 -req -days 365 -sha256 -in server.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out server-cert.pem -extfile extfile.cnf
Signature ok
subject=CN = 172.16.132.200
Getting CA Private Key
// 输入最开始设置的密码
Enter pass phrase for ca-key.pem:
```

### 创建客户端密钥和证书签名请求

```
root@docker-manager:~# openssl genrsa -out key.pem 4096
Generating RSA private key, 4096 bit long modulus (2 primes)
..++++
...............................................................................................................................................................................................................++++
e is 65537 (0x010001)

root@docker-manager:~# openssl req -subj '/CN=client' -new -key key.pem -out client.csr
```

为了使密钥适合客户端身份验证，请创建一个新的扩展配置文件：

```
root@docker-manager:~#  echo extendedKeyUsage = clientAuth > extfile-client.cnf
```

### 生成客户端签名证书

```
root@docker-manager:~# openssl x509 -req -days 365 -sha256 -in client.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out cert.pem -extfile extfile-client.cnf
Signature ok
subject=CN = client
Getting CA Private Key
// 输入最开始设置的密码
Enter pass phrase for ca-key.pem:

```

### 删除和修改文件权限

生成后cert.pem，server-cert.pem您可以安全地删除两个证书签名请求和扩展配置文件：

```
root@docker-manager:~# rm -v client.csr server.csr extfile.cnf extfile-client.cnf
removed 'client.csr'
removed 'server.csr'
removed 'extfile.cnf'
removed 'extfile-client.cnf'
```

为了保护您的钥匙免遭意外损坏，请删除其写权限。要使它们仅供您阅读，请按以下方式更改文件模式：

```
root@docker-manager:~# chmod -v 0400 ca-key.pem key.pem server-key.pem
mode of 'ca-key.pem' changed from 0600 (rw-------) to 0400 (r--------)
mode of 'key.pem' changed from 0600 (rw-------) to 0400 (r--------)
mode of 'server-key.pem' changed from 0600 (rw-------) to 0400 (r--------)
```

证书可以在世界范围内读取，但您可能希望删除写访问权限以防止意外损坏：

```
root@docker-manager:~# chmod -v 0444 ca.pem server-cert.pem cert.pem
mode of 'ca.pem' changed from 0644 (rw-r--r--) to 0444 (r--r--r--)
mode of 'server-cert.pem' changed from 0644 (rw-r--r--) to 0444 (r--r--r--)
mode of 'cert.pem' changed from 0644 (rw-r--r--) to 0444 (r--r--r--)
```

### 修改docker配置并重启docker

编辑docker配置文件（我的是ubuntu机器）：
`vim /lib/systemd/system/docker.service`

添加如下行：

```
ExecStart=/opt/kube/bin/dockerd  --tlsverify --tlscacert=/root/docker/ca.pem --tlscert=/root/docker/server-cert.pem --tlskey=/root/docker/server-key.pem -H unix:///var/run/docker.sock -H tcp://0.0.0.0:2375
```

重启docker服务：

```
systemctl daemon-reload
systemctl restart docker
```

查看端口号：

```
root@docker-manager:~# netstat -plnt
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp6       0      0 :::80                   :::*                    LISTEN      858/nginx: master p 
tcp6       0      0 :::25                   :::*                    LISTEN      1662/master         
tcp6       0      0 :::2375                 :::*                    LISTEN      25582/dockerd       
tcp6       0      0 :::2377                 :::*                    LISTEN      25582/dockerd       
```

### 客户端远程安全连接

将 `ca.pem cert.pem key.pem `三个文件通过 scp 下载到 客户端机器。

远程连接命令，路径根据实际情况填写：

```
docker --tlsverify \
  --tlscacert=/home/docker/ca.pem \ 
  --tlscert=/home/docker/cert.pem \
  --tlskey=/home/docker/key.pem \
  -H=172.16.132.200:2375 \
  info
```

把密钥放入 ~/.docker 文件夹中：

每次操作需要跟那么多参数，太麻烦了。我们可以把` ca.pem cert.pem key.pem `三个文件放入客户端` ~/.docker `中，然后配置环境变量就可以简化命令了。

```
$ export DOCKER_HOST=tcp://172.16.132.200:2375 DOCKER_TLS_VERIFY=1

$ docker info
```

