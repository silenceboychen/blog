---
title: macbook如何卸载天空卫士
toc: true
date: 2022-05-20 14:07:27
categories: mac
tags: [mac, 天空卫士]
---

1. 首先找到安装天空卫士的目录：``/Library/Application Support/SkyGuard``

2. 电脑关机，长按开机键直到进入recovery模式, 点自己的用户，输入密码，点下一步，然后到左上方找到终端打开

3. diskutil list 查看目前的磁盘, 找到标记有``synthesized``的磁盘，并找到对应的数据盘（有data关键字），这里假设为：``/dev/disk3s3``

4. 执行``diskutil mount /dev/disk3s3``, 可能会报 ``this is an encrypted and locked APFS Volume``的错，根据提示执行：``diskutil apfs unlockVolume /dev/disk3s3``，然后输入密码。再次执行``diskutil mount /dev/disk3s3``即可。

5. 进入``/Volumes/Macintosh HD/Library/Application Support``目录，可以看到有``SkyGuard``文件。

6. 执行``rm -rf SkyGuard``删除天空位置文件目录

7. ``reboot``重启电脑，发现天空卫士已成功卸载
