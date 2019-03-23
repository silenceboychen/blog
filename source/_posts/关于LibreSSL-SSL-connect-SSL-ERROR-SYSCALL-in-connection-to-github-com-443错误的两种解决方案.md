---
title: >-
  关于LibreSSL SSL_connect: SSL_ERROR_SYSCALL in connection to
  github.com:443错误的两种解决方案
toc: true
date: 2019-03-23 12:54:30
categories: git
tags: [git, hexo]
---

### 错误来源

使用使用hexo部署博客是，遇到以下错误:

```
fatal: unable to access 'https://github.com/silenceboychen/silenceboychen.github.io.git/': LibreSSL SSL_connect: SSL_ERROR_SYSCALL in connection to github.com:443
FATAL Something's wrong. Maybe you can find the solution here: http://hexo.io/docs/troubleshooting.html
Error: Spawn failed
    at ChildProcess.<anonymous> (/Users/chenhao/new_start/nodejs/blog/node_modules/hexo-util/lib/spawn.js:52:19)
    at ChildProcess.emit (events.js:182:13)
    at Process.ChildProcess._handle.onexit (internal/child_process.js:239:12)
```

### 两种解决方案：

#### 方案一

取消http代理：

```
$ git config --global --unset http.proxy
$ git config --global --unset https.proxy
```

设置env ``GIT_SSL_NO_VERIFY``为``true``然后再次部署：

```
$ env GIT_SSL_NO_VERIFY=true hexo d
```

问题解决。

#### 方案二

在hexo项目的根目录下的_config.yml文件中把仓库链接地址由https修改为ssh的地址。