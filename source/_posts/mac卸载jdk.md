---
title: mac卸载jdk
toc: true
date: 2022-04-29 17:25:19
categories: mac
tags: [mac, jdk]
---

1. 删除运行路径和运行环境等

```
sudo rm -fr /Library/Internet\ Plug-Ins/JavaAppletPlugin.plugin
sudo rm -fr /Library/PreferencesPanes/JavaControlPanel.prefPane
sudo rm -fr ~/Library/Application\ Support/Java
```

2. 删除当前版本的jdk

```
sudo rm -rf /Library/Java/JavaVirtualMachines/jdk1.8.0_301.jdk
```

3. 检查是否卸载成功

```
java -version
```