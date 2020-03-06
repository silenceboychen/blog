---
title: zsh在scp时不能使用通配符的原因和解决方案
toc: true
date: 2020-03-06 15:14:33
categories: shell
tags: [scp, zsh]
---

## 问题

`scp`是经常使用的一个本地与远程服务器相互拷贝数据的命令，`zsh`是我最喜欢的`shell`，但是在`zsh`下使用`scp`来拷贝远程服务器的文件时，却出现这样的错误。

```
$ scp -r test-server:/etc/nginx/conf.d/* .
zsh: no matches found: test-server:/etc/nginx/conf.d/*
```

同样地命令，在`bash`下确实可以执行的，这个原因是什么呢？

由于`zsh`不会按照远程地址上的文件去扩展参数，当你使用`test-server:/etc/nginx/conf.d/*`，因为本地当前目录中，是不存在`test-server:/etc/nginx/conf.d/*`，所以匹配失败。默认情况下，`bash` 在匹配失败时就使用原来的内容，`zsh` 则报告一个`no matches`的错误。


## 解决方案

在`zsh`中执行`setopt nonomatch`，告诉它不要报告`no matches`的错误，而是当匹配失败时直接使用原来的内容。

实际上，不管是 `bash` 还是 `zsh`，不管设置了什么选项，只要把`test-server:/etc/nginx/conf.d/*`加上引号，如`"test-server:/etc/nginx/conf.d/*"`，就可解决问题。

当然根本的解决办法还是告诉`zsh`不要报告`no matches`错误。

执行下面的命令可以一劳永逸：

```
$ echo "setopt nonomatch" >> ~/.zshrc
或
$ echo "set -o nonomatch" >> ~/.zshrc
```