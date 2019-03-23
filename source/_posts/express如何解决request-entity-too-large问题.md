---
title: express如何解决request entity too large问题
toc: true
date: 2016-05-30 11:21:33
categories: nodejs
tags: nodejs
---

通过js向后台post一些文件信息时，会出现如下图所示的错误。这是express框架的问题，默认的很小，可以通过设置：app.use(express.json({limit: '5mb'}));解决该问题。

![entity_too_large.jpeg](/../images/entity_too_large.jpeg)