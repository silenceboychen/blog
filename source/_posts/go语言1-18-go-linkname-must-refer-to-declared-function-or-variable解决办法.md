---
title: 'go语言1.18 go:linkname must refer to declared function or variable解决办法'
toc: true
date: 2022-07-12 14:13:58
categories: go
tags: go
---

在macos环境中，go1.18刚刚部署后，会报错如下：

```
golang.org/x/sys/unix
# golang.org/x/sys/unix
vendor/golang.org/x/sys/unix/syscall_darwin.1_13.go:29:3: //go:linkname must refer to declared function or variable
vendor/golang.org/x/sys/unix/zsyscall_darwin_amd64.1_13.go:27:3: //go:linkname must refer to declared function or variable
vendor/golang.org/x/sys/unix/zsyscall_darwin_amd64.1_13.go:40:3: //go:linkname must refer to declared function or variable
vendor/golang.org/x/sys/unix/zsyscall_darwin_amd64.go:28:3: //go:linkname must refer to declared function or variable
vendor/golang.org/x/sys/unix/zsyscall_darwin_amd64.go:43:3: //go:linkname must refer to declared function or variable
vendor/golang.org/x/sys/unix/zsyscall_darwin_amd64.go:59:3: //go:linkname must refer to declared function or variable
vendor/golang.org/x/sys/unix/zsyscall_darwin_amd64.go:75:3: //go:linkname must refer to declared function or variable
vendor/golang.org/x/sys/unix/zsyscall_darwin_amd64.go:90:3: //go:linkname must refer to declared function or variable
vendor/golang.org/x/sys/unix/zsyscall_darwin_amd64.go:105:3: //go:linkname must refer to declared function or variable
vendor/golang.org/x/sys/unix/zsyscall_darwin_amd64.go:121:3: //go:linkname must refer to declared function or variable
vendor/golang.org/x/sys/unix/zsyscall_darwin_amd64.go:121:3: too many errors
```

解决办法如下：

1.运行如下命令：

```
go get -u golang.org/x/sys
```

2.运行：

```
go mod vendor
```