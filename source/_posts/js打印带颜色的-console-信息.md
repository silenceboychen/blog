---
title: js打印带颜色的 console 信息
toc: true
date: 2019-07-12 14:30:24
categories: javascript
tags: javascript
---

打印红色的hello world:

```
console.log(`\x1b[31mhello world\x1b[31m`)
```

以下是可以使用的文本命令的参考：

前景色（文字颜色）：

```
\x1b[30m = 黑色
\x1b[31m = 红色
\x1b[32m = 绿色
\x1b[33m = 黄色
\x1b[34m = 蓝色
\x1b[35m = 洋红色
\x1b[36m = 青色
\x1b[37m = 白色
```

背景色：

```
\x1b[40m = 黑色
\x1b[41m = 红色
\x1b[42m = 绿色
\x1b[43m = 黄色
\x1b[44m = 蓝色
\x1b[45m = 洋红色
\x1b[46m = 青色
\x1b[47m = 白色
```

其他：

```
\x1b[0m = 清除样式
\x1b[1m = 加粗
\x1b[2m = 半透明
\x1b[4m = 下划线
\x1b[5m = 闪动
\x1b[7m = 取反：背景色变前景色 前景色变背景色
\x1b[8m = 看不见 但位置还留着
```