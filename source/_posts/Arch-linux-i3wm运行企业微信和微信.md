---
title: Arch linux i3wm运行企业微信和微信
toc: true
date: 2021-05-26 22:05:02
categories: linux
tags: [linux, i3wm, i3]
---

当你运行在i3wm环境下时，运行通过deepin-wine安装的企业微信和微信时，打开软件时会出现闪退的现象，启动不了软件。或直接在命令行里执行命令也启动不了程序，会出现以下提示：

```
X Error of failed request:  BadWindow (invalid Window parameter)
  Major opcode of failed request:  20 (X_GetProperty)
  Resource id in failed request:  0x0
  Serial number of failed request:  10
  Current serial number in output stream:  10
```

**解决方案**

这个问题其实和 KDE 无关, 应该是 deepin 在打包 deepin-wine 的过程中有意或者无意加入了 GNOME 依赖。

执行 ``/usr/lib/gnome-settings-daemon/gsd-xsettings`` 即可.
或者后台运行：

```
nohup /usr/lib/gnome-settings-daemon/gsd-xsettings > /dev/null 2>&1 &
```

如果 GNOME 的版本较低(比如Debian 9), 没有单独的 ``gsd-xsettings`` 可执行文件, 则执行 ``gnome-settings-daemon``.

然后切换到对应目录 ``cd /opt/deepinwine/apps/Deepin-WXWork`` 或者 ``/opt/deepinwine/apps/Deepin-WeChat``
运行 ``./run.sh``即可启动软件。


由于每次都执行上边的命令很繁琐，可以将其加入i3的启动项，每次开机制动设置即可。

