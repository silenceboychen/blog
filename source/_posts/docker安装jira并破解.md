---
title: docker安装jira并破解
toc: true
date: 2019-09-03 20:15:22
categories: docker
tags: [docker, jira]
---

### 下载镜像

```
$ docker pull cptactionhank/atlassian-jira
```

### 运行容器

```
$ docker volume create jira_home
$ docker run -d -p 8080:8080 --name jira --restart always -v jira_home:/var/atlassian/jira cptactionhank/atlassian-jira:latest
```

### 下载破解文件

```
$ wget https://github.com/silenceboychen/some-software/raw/master/Jira/mysql-connector-java-5.1.25-bin.jar
$ wget https://github.com/silenceboychen/some-software/raw/master/Jira/atlassian-universal-plugin-manager-plugin-2.22.4.jar
$ wget https://github.com/silenceboychen/some-software/raw/master/Jira/atlassian-extras-3.2.jar
```

### 添加mysql驱动程序

[mysql配置](https://confluence.atlassian.com/adminjiraserver/connecting-jira-applications-to-mysql-5-7-966063305.html)

```
$ docker cp mysql-connector-java-5.1.25-bin.jar jira:/opt/atlassian/jira/atlassian-jira/WEB-INF/lib/
$ docker restart jira
```

### nginx配置

```
upstream jira{
    server 127.0.0.1:8080;
}

# http配置
#server {
#    listen 80;
#    server_name  jira.domain.com;
#    access_log	 /var/log/nginx/jira.domain.com-access.log;
#    error_log	 /var/log/nginx/jira.domain.com-error.log;
#    location / {
#        proxy_pass_header Server;
#        proxy_set_header Host $http_host;
#        proxy_set_header X-Real-IP $remote_addr;
#        proxy_set_header X-Scheme $scheme;
#        proxy_pass http://jira;
#    }
#}

# https配置
server {
  #侦听443端口，这个是ssl访问端口
  listen    443;
  #定义使用 访问域名
  server_name  jira.iblackvip.com;

  access_log	 /var/log/nginx/jira.domain.com.access.log;
  error_log	 /var/log/nginx/jira.domain.com.error.log;

  ssl on;
  ssl_certificate /etc/nginx/cert/jira.domain.com.pem;
  ssl_certificate_key /etc/nginx/cert/jira.domain.com.key;
  ssl_session_timeout 5m;
  ssl_protocols TLSv1 TLSv1.1 TLSv1.2; #按照这个协议配置
  ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;#按照这个套件配置
  ssl_prefer_server_ciphers on;
  location / {
    proxy_pass_header Server;
    proxy_set_header Host $http_host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Scheme $scheme;
    proxy_pass http://jira;
  }
  gzip on;
  gzip_min_length 1k;
  gzip_buffers 4 16k;
  gzip_comp_level 5;
  gzip_types text/plain application/x-javascript text/css application/xml text/javascript application/x-httpd-php;
}

server {
  # 80端口是http正常访问的接口
  listen 80;
  server_name jira.domain.com;
  # 在这里，我做了https全加密处理，在访问http的时候自动跳转到https
  rewrite ^(.*) https://$host$1 permanent;
}
```

### Web设置

* 浏览器访问JiraWeb，语言可以设为中文，选择「我将设置它自己」——「下一步」

* 数据库设置，数据库类型选择「MySQL」，接着填入你的MySQL连接信息（需要你在你的MySQL数据库中创建数据库，数据库的字符类型必须是utf8），测试可以连接之后点击「下一步」

* 设置应用程序的属性——「下一步」

* 申请许可证关键字，点击「生成Jira试用许可证」

* 需要注册账号，注册完之后重新回到这个页面，选择相关信息，点击「Generate License」

* 点击「Yes」

* 页面就会带着你的许可证关键字回到Jira的设置页面，接着点击「下一步」

* 等待一会就进入设置管理员页面，填入一些信息即可，接着「下一步」

* 点击「完成」即完成设置

### 破解jira

拷贝文件到容器内：

```
$ docker cp atlassian-extras-3.2.jar jira:/opt/atlassian/jira/atlassian-jira/WEB-INF/lib/
$ docker cp atlassian-universal-plugin-manager-plugin-2.22.4.jar jira:/opt/atlassian/jira/atlassian-jira/WEB-INF/atlassian-bundled-plugins/
```

重启容器，破解结束：

```
$ docker restart jira
```

查看jira页面设置-》应用程序，我们可以很明显的看到jira我们可以使用到2033年。