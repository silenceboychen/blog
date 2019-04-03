---
title: 使用git进行远程分支同步
toc: true
date: 2019-04-03 10:48:12
categories: git
tags: git
---

在git项目目录下执行`git remote show origin`命令：

```
➜  api git:(419788c) ✗ git remote show origin
* remote origin
  Fetch URL: git@github.com:silenceboychen/blog.git
  Push  URL: git@github.com:silenceboychen/blog.git
  HEAD branch: master
  Remote branches:
    alibeta                                      tracked
    dev                                          tracked
    master                                       tracked
    production                                   tracked
    refs/remotes/origin/feat-10.9                stale (use 'git remote prune' to remove)
    refs/remotes/origin/feat-docker              stale (use 'git remote prune' to remove)
    refs/remotes/origin/feat_avatar              stale (use 'git remote prune' to remove) 
```

可以看到有一些状态为`stale`的分支，这些分支都已经被删除了，但是我们本机上还有记录，这些记录不会通过`git pull`自动清除。

为了删除这些分支，实现和远程分支的同步，可以执行`git remote prune origin`

```
➜  api git:(419788c) ✗ git remote prune origin
Pruning origin
URL: git@github.com:silenceboychen/blog.git
 * [pruned] origin/feat-10.9
 * [pruned] origin/feat-docker
 * [pruned] origin/feat_avatar
```

再次查看，发现那些无效分支已经在本机被删除：

```
➜  api git:(419788c) ✗ git remote show origin
* remote origin
  Fetch URL: git@github.com:silenceboychen/blog.git
  Push  URL: git@github.com:silenceboychen/blog.git
  HEAD branch: master
  Remote branches:
    alibeta                     tracked
    dev                         tracked
    master                      tracked
    production                  tracked
  Local branch configured for 'git pull':
    master merges with remote master
  Local ref configured for 'git push':
    master pushes to master (local out of date)
```