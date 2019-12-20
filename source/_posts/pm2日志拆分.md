---
title: pm2日志拆分
toc: true
date: 2019-12-20 11:41:48
categories: nodejs
tags: [nodejs, pm2]
---

pm2默认会将日志文件写入家目录下的 `.pm2/logs` 目录中，但是pm2的日志文件不能自动分割，这会导致一个文件不断变大，不但影响性能，查看这些日志也会带来麻烦。

### pm2的日志切割模块pm2-logrotate

* 安装`pm2-logrotate`：

```
$ pm2 install pm2-logrotate
```

* 设置切割规则

```
1. 设置文件大小为100M，大于等于开始切割
pm2 set pm2-logrotate:max_size 100M

2.设置文件切割的监控间，监控间隔比较大，可能会使切割出的文件大小和max_size有出入
pm2 set pm2-logrotate:workerInterval 1

3. 设置文件最多多少个，超过则删除
pm2 set pm2-logrotate:retain 10

4. 设置文件是否压缩
$ pm2 set pm2-logrotate:compress false

5. 设置文件命名格式
$ pm2 set pm2-logrotate:dateFormat YYYY-MM-DD_HH-mm-ss

```

* 更新pm2

执行一下命令使pm2配置生效

```
$ pm2 update
```
