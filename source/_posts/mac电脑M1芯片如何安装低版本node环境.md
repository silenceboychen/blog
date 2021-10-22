---
title: mac电脑M1芯片如何安装低版本node环境
toc: true
date: 2021-10-22 17:15:17
categories: nodejs
tags: [nodejs, mac, M1]
---

在mac M1上安装v14 及以下的老版本 Node会出现闪退问题，究其原因还是因为低版本的 node 并不是基于 arm64 架构的，所以不适配 M1 芯片。在这里教大家两个方法，就能成功安装上低版本 Node。

### 方法一

在终端中，输入：

```
arch -x86_64 zsh
```

通过这个命令可以让 shell 运行在Rosetta2下。
之后你可以通过 `nvm install v14` 来安装低版本 Node。
在此之后，您可以不用在 Rosetta2 中就可以使用安装的可执行文件，也就是说，您可以将 Node v15与其他节点版本互换使用。

### 方法二

方法二就是通过 Rosetta2 来启动终端，这样通过 Rosetta2 转译到 x86 架构中执行安装，也一样可以安装成功。

1. 在 finder 中，点击应用程序，并在实用工具中找到终端 (Terminal)
2. 右键终端，点击获取信息
3. 勾选 Open using Rosetta
4. 重启终端，并执行 `nvm install v14` 命令