---
title: 在docker中执行gitlab-runner
toc: true
date: 2018-05-20 11:45:45
categories: docker
tags: [docker, gitlab-runner]
---

> 环境:ubuntu 16.04 LTS
> 目的：使用Docker安装和配置GitLab Runner，搭建GitLab CI持续集成环境。

### 安装gitlab-runner

```
$ docker run -d --name gitlab-runner --restart always \
  -v /srv/gitlab-runner/config:/etc/gitlab-runner \
  -v /var/run/docker.sock:/var/run/docker.sock \
  gitlab/gitlab-runner:latest
```

参数说明：

* -d: 设置容器后台运行
* --name：容器名称
* -restart always：每次启动容器就重启 gitlab-runner
* -v: 共享目录挂载

安装好后，执行``$ docker ps `` 查看容器是否运行。

### 注册和初始化

```
$ docker exec -it gitlab-runner gitlab-ci-multi-runner register
```
``gitlab-runner register``是进入``gitlab-runner``容器的执行命令，用于注册和初始化``gitlab-runner``。
以下是我的配置：``注意``：docker image为满足你项目构建所需环境的镜像。
![](/../images/gitlab-runner.png)

我们也可以编辑``vim /srv/gitlab-runner/config/config.toml``，手动修改配置：
```
concurrent = 1
check_interval = 0

[[runners]]
  name = "test"
  url = "https://xxxx.oooo.com"
  token = "3894a417b64744e942008bcc51123a"
  executor = "docker"
  builds_dir = "/gitlab/runner-builds"
  cache_dir = "/gitlab/runner-cache"
  [runners.docker]
    tls_verify = false
    image = "node:latest"
    privileged = false
    disable_cache = false
    volumes = ["/data/gitlab-runner:/gitlab"]
    shm_size = 0
    pull_policy = "if-not-present"
  [runners.cache]
```
> ``gitlab-ci token``可以从gitlab上的项目的CI设置中获得。
> ``builds_dir`` 为文件存放位置
> ``volumes`` 挂载目录
> ``pull_policy`` 设置gitlab是否从远程拉去image,如果iamge是本地的需要配置该属性的值为: ***if-not-present*** 或者　***never***

#### 创建.gitlab-ci.yml文件
我的项目为nodejs项目，以下为测试配置。

```
stages:
  - install

cache:
  key: ${CI_BUILD_REF_NAME}
  paths:
    - node_modules/

job-install:
  stage: install
  script:
    - whoami
    - echo $SHELL
    - rm -rf node_modules/
    - pwd
    - source ~/.bashrc
    - nvm use 8
    - node -v
    - yarn
  only:
    - preview
  tags:
    - test
```
配置好gitlab-ci文件之后，提交修改，并将最新的修改推送到origin/preview分支，即可触发CI:
```
Running with gitlab-runner 10.2.0 (0a75cdd1)
  on test (3894a417)
Using Docker executor with image followme/node:v1 ...
Using docker image sha256:07e33b24b6a9bebc0e0d8ba24f15b4b3c0f6fcf321a3809371a6211ac1afc38e for predefined container...
Using locally found image version due to if-not-present pull policy
Using docker image followme/node:v1 ID=sha256:c99c549e8227e2323d1cebb6f988d5d8f6de7f77e1967fe0f02878b85cb72b0f for build container...
Running on runner-3894a417-project-643-concurrent-0 via 304e3efed168...
Cloning repository...
Cloning into '/gitlab/runner-builds/3894a417/0/Frontend/api-member'...
Checking out 311e85cb as preview...
Skipping Git submodules setup
Checking cache for preview...
Successfully extracted cache
$ whoami
root
$ echo $SHELL
/bin/bash
$ rm -rf node_modules/
$ pwd
/gitlab/runner-builds/3894a417/0/Frontend/api-member
$ source ~/.bashrc
$ nvm use 8
Now using node v8.3.0 (npm v5.3.0)
$ node -v
v8.3.0
$ yarn
yarn install v1.3.2
[1/4] Resolving packages...
[2/4] Fetching packages...
[3/4] Linking dependencies...
[4/4] Building fresh packages...
Done in 7.21s.
Creating cache preview...
node_modules/: found 5627 matching files           
Created cache
Job succeeded

```

> 注意：之前我是在Ubuntu14.04版本的系统上做这些配置，但是当执行CI的时候总会遇到以下报错:
> ```ERROR: Preparation failed: Error reading remote info: json: cannot unmarshal number into Go struct field Info.Debug of type bool```
> 
> 将系统升级为16.04后解决该问题