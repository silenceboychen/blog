---
title: docker搭建私有仓库、自签发证书、登录认证
toc: true
date: 2018-03-28 09:24:21
categories: docker
tags: docker
---

docker官方文档对如何搭建私有仓库说的已经很详细了，我在这里主要介绍一下使用自签发证书如何搭建私有仓库并认证成功。

>假设registry的域名为：registry.domain.com。

### 生成自签发证书。

```
> mkdir -p certs
> openssl req -newkey rsa:2048 -nodes -sha256 -keyout certs/domain.key -x509 -days 365 -out certs/domain.crt
```
执行以上命令，生成证书，Common Name那里要输入我们registry的域名，生成的证书只对该域名有效。其他的可以任意填。生成后可以在certs目录下查看到证书。

![](/../images/openssl.png)

### 生成鉴权密码文件

> 注意使用时username替换为你自己的用户名，password替换为你自己的密码。
```
$ mkdir auth
$ docker run --entrypoint htpasswd registry:2 -Bbn username password  > auth/htpasswd
$ ls auth
```
### 启动Registry

```
docker run -d -p 5000:5000 --restart=always --name registry \
   -v `pwd`/auth:/auth \
   -e "REGISTRY_AUTH=htpasswd" \
   -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \
   -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd \
   -v `pwd`/data:/var/lib/registry \
   -v `pwd`/certs:/certs \
   -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt \
   -e REGISTRY_HTTP_TLS_KEY=/certs/domain.key \
   registry:2
199ad0b3591fb9613b21b1c96f017267f3c39661a7025d30df636c6805e7ab50
```
如果没有registry镜像会自动下载然后启动，可以使用阿里云提供的加速器。
现在完成了registry服务器的搭建，可以尝试pull image到registry：
```
$ docker pull busybox  // 从官方拉去镜像作为我们的测试镜像
$ docker tag busybox:latest registry.domain.com:5000/busybox:latest // 为busybox打tag,tag的前缀一定要为我们registry服务器的域名。
$ docker push registry.domain.com:5000/busybox:latest  // 将镜像推送到我们的registry服务器
```
如果直接这样去push，会失败，并且出现 ``no basic auth credentials``的错误，这是因为我们没有进行登录认证。
```
$ docker login registry.domain.com:5000
$ Username: username
$ Password: password
WARNING: login credentials saved in ~/.docker/config.json
Login Succeeded
```
登录成功后再次执行push操作，会出现``x509: certificate signed by unknown authority``的报错。这是因为docker client认为server传输过来的证书的签署方是一个unknown authority（未知的CA），因此验证失败。我们需要让docker client安装我们的CA证书：
```
$ sudo mkdir -p /etc/docker/certs.d/registry.domain.com:5000
$ sudo cp certs/domain.crt /etc/docker/certs.d/registry.domain.com:5000/ca.crt
$ sudo service docker restart //安装证书后，重启Docker Daemon
```
再次执行push操作，成功推送：
```
$ docker push registry.domain.com:5000/busybox:latest
The push refers to a repository [registry.domain.com:5000/busybox]
0271b8eebde3: Pushed
latest: digest: sha256:3571ca1b0e90e159de4fc07b3bf94ef189a0645314704f629204adb7035ecf45 size: 527
```
> 这里需要注意：如果使用自签署的证书，那么所有要与Registry交互的Docker主机都需要安装registry.domain.com的ca.crt(domain.crt)。但如果你使用知名CA，这一步也就可以忽略。如果是MacOS版的docker也是一样的操作，尽管它连/etc/docker都不存在,一样去创建目录就好了。只是macOS的用户想要认证生效需要执行额外的命令：
>```
$ sudo security add-trusted-cert -d -r trustRoot -k /Library/Keychains/System.keychain /etc/docker/certs.d/registry.domain.com:5000/ca.crt
```
> 此时在mac上执行docker login才可成功。