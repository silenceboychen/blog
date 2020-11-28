---
title: alsamixer控制音量
toc: true
date: 2020-11-28 13:51:40
categories: linux
tags: linux
---

## 解除各声道的静音

目前版本的 ALSA 安装后，所有声道默认是静音的，必须手动解除。

使用 `alsamixer` 的 `ncurses` 界面，配置十分简单：

```
$ alsamixer
```

此外，还可以在命令行下使用 `amixer`：

```
$ amixer sset Master unmute
```

在` alsamixer` 中，下方标有 `MM` 的声道是静音的，而标有 `00` 的通道已经启用。

使用 `←` 和 `→` 方向键，选中 `Master` 和 `PCM` 声道。按下 `m` 键解除静音。使用 `↑` 方向键增加音量，直到增益值为`0`。该值显示在左上方` Item:` 字段后。过高的增益值会导致声音失真。

要启用麦克风，切换至 Capture 选项卡，按下 `F4`，按下 `空格` 启用其中一个声道即可。

按下 `Esc` 键退出 `alsamixer`。

## alsamixer 终端交互式设置音量

```
F6 选择网卡
F2 显示系统信息，可以看到系统中已有网卡信息
Esc 后退
```

```
M 静音状态切换
Q,W,E 增大 左,右,通道 的音量
Z,X,C 减小 左,右,通道 的音量
```

## amixer 命令行控制系统声音

```
cat /proc/asound/cards  # 查看系统声卡
```

输出如下：

```
0 [PCH            ]: HDA-Intel - HDA Intel PCH
                      HDA Intel PCH at 0xe1340000 irq 130
```

设置声音

```
amixer -c 1 -q set Master 2dB+ unmute
```

```
-c 制定声卡id, 默认为0
-q 安静模式，不输出结果
```