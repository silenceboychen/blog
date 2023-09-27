---
title: shell脚本里的#*@是什么意思
toc: true
date: 2023-09-27 17:22:45
categories: shell
tags: shell
---

在Shell脚本中，${variable#pattern} 是一种字符串处理方式，其中 # 后跟着一个模式（pattern）。这个语法的作用是从字符串变量 variable 的开头删除匹配 pattern 的最短子串，并返回删除后的结果。${variable#*@} 的含义是：

● variable 是一个字符串变量，通常是一个包含文本的字符串。

● # 表示从字符串开头开始匹配。

● *@ 是一个通配符模式，它匹配字符串中的任意字符序列，直到第一个 @ 字符。

所以，${audioplay#*@} 的作用是从变量 audioplay 的开头删除匹配 *@ 模式的最短子串，并返回删除后的结果。通常，这种操作用于处理文本或字符串，以过滤掉或提取感兴趣的部分。
例如，如果 audioplay 的值是 "user@example.com"，那么 ${audioplay#*@} 的结果将是 "example.com"，因为它删除了字符串中第一个 "@" 符号及其之前的部分。