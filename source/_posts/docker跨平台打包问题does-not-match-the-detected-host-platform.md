---
title: docker跨平台打包问题does not match the detected host platform
toc: true
date: 2022-04-29 11:44:40
categories: docker
tags: [docker, mac, ubuntu]
---

### 问题

最近在mac M1上构建的docker镜像，发布到ubuntu20系统的服务器上后，一直运行失败。输出以下报错信息：

```
The requested image's platform (linux/arm64) does not match the detected host platform (linux/amd64) and no specific platform was requested
```

在使用docker logs 查看docker日志的时候提示：

```
standard_init_linux.go:228: exec user process caused: exec format error​
```

### 解决方案

1. mac M1上设置"experimental": true 

  ![docker.png](/../images/docker.png)

2. 实现跨平台打包

	docker buildx build --platform linux/amd64 -t name .  