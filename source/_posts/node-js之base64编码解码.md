---
title: node.js之base64编码解码
toc: true
date: 2016-09-16 12:15:18
categories: nodejs
tags: [nodejs, base64]
---

利用buffer来进行编解码：

```
> var a = new Buffer('key1=value1&key2=value2').toString('base64');
undefined
> a
'a2V5MT12YWx1ZTEma2V5Mj12YWx1ZTI='
> new Buffer(a, 'base64').toString()
'key1=value1&key2=value2'
```

可以在终端中执行以下命令查看解码后的内容：

```
echo a2V5MT12YWx1ZTEma2V5Mj12YWx1ZTI= | base64 -D
```
