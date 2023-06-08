---
title: gomobile开发安卓应用环境搭建完整流程
toc: true
date: 2023-06-08 13:33:05
categories: go
tags: go
---

> go环境搭建不在这里赘述。
>
> 以下内容的执行环境为：
>
> 系统：ubuntu20.04
>
> go版本：v1.19

# 项目创建

执行以下命令创建一个go开发安卓应用的测试目录：

```bsah
$ mkdir $GOPATH/src/goapp && cd $GOPATH/src/goapp
$ go mod init
```

在该目录下执行以下命令获取官方提供的示例项目：

```bash
$ go get -d golang.org/x/mobile/example/basic
```

# 安装gomibile

```bash
$ go install golang.org/x/mobile/cmd/gomobile@latest
$ gomobile init
```

然后执行以下命令打包安卓应用：

```bash
$ gomobile build -target=android -androidapi 19 golang.org/x/mobile/example/basic
```

此时会发现以下相关错误：

    gomobile: could not locate Android SDK: stat /home/test/Android/Sdk: no such file or directory; Android SDK was not found at /home/test/Android/Sdk

    gomobile: no usable NDK in /home/test/Android/Sdk: open /home/test/Android/Sdk/ndk: no such file or directory, open /home/test/Android/Sdk/ndk-bundle/meta/platforms.json: no such file or directory

这是因为本地没有配置安卓开发环境导致的。

# 安卓开发环境搭建

## Android Studio安装

访问谷歌中国开发者网站下载 Android Studio 编辑器：<https://developer.android.google.cn/studio>

下载完成后执行以下操作：

```bash
# 将安装包移到/opt目录下，需要管理员权限
$ sudo mv android-studio-2022.2.1.20-linux.tar.gz /opt
# 进入/opt目录
$ cd /opt
# 解压文件，需要管理员权限
$ sudo tar -xzvf android-studio-2022.2.1.20-linux.tar.gz
# 运行Android Studio
$ ./android-studio/bin/studio.sh

==========================================================

# 如果想在任意位置打开android studio，可配置软连接
$ sudo ln -s /opt/android-studio/bin/studio.sh /usr/bin/android-studio

# 配置完成以后在任意位置执行android-studio即可打开应用
$ android-studio
```

第一次打开android-studio需要进行一些配置，一直选择下一步设置即可，其中有两个地方需要注意：

1.  选择自定义安装

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ff6726dec7c44037bcedd07064fa7050~tplv-k3u1fbpfcp-watermark.image?)

2.  插件安装，可以全选

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ecf62541fd1343e9a952444c91f418a2~tplv-k3u1fbpfcp-watermark.image?)

插件安装完成之后点击Finish即可打开应用。

## 安装NDK

Android studio安装完成后并没有万事大吉，默认并没安装NDK，需要自己手工再安装。

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b7f3ae9976b04abaad1491894d82d356~tplv-k3u1fbpfcp-watermark.image?)

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/040bf7ba8aa54a4285b2f2996774a5b0~tplv-k3u1fbpfcp-watermark.image?)

点击ok会自动下载选择的插件。

# 编译安卓应用

此时继续回到之前的项目目录，执行安卓构建命令

```bash
$ gomobile build -target=android -androidapi 19 golang.org/x/mobile/example/basic
```

这一次没有出现报错，并且目录下多了一个basic.apk文件，该文件即为打包成功的安卓应用，可以安装一个安卓模拟器进行测试了。

# 安装安卓模拟器

模拟器我选用了Anbox

```bash
$ sudo snap install --devmode --edge anbox
```

安装完成之后执行以下命令启动安卓模拟器：

```bash
$ anbox.appmgr
```

我比较顺利没有遇到报错，如果遇到模拟器启动报错，可以参考文章：<https://juejin.cn/post/7152407243974148127> 解决

打开后的界面如下：


![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/279fd239661844cd8ec94a9449b20cb1~tplv-k3u1fbpfcp-watermark.image?)

# 测试应用

安装安卓应用还需要adb命令：

```bash
$ sudo apt install android-tools-adb -y
```

然后在最开始的项目目录下执行以下命令安装应用，此时安卓模拟器必须是打开的状态：

```bahs
$ asb install ./basic.apk
# 或者
$ adb install /home/test/go/src/goapp/basic.apk
```

安装成功后即可在模拟器中看到该应用


![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b43efa40ce324a9caf8b08ae2759d9d5~tplv-k3u1fbpfcp-watermark.image?)

单击打开，运行效果如下：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/902dd6d8b0064000b929cde2f445f482~tplv-k3u1fbpfcp-watermark.image?)

此文章主要目的是为了帮助你了解如何使用golang开发安卓应用的流程，流程打通之后，可以结合自己的想法，做一些自己的应用。
