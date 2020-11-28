---
title: ubuntu18.04安装jdk1.8
toc: true
date: 2020-11-28 14:30:31
categories: linux
tags: ['ubuntu', 'java', 'jdk']
---


### 下载jdk安装包
	
    https://www.oracle.com/java/technologies/javase/javase-jdk8-downloads.html
    
### 解压

```
tar -zxvf jdk-8u171-linux-x64.tar.gz
```

### 移动到自己想放的位置

```
##将文件从下载目录 挪到/usr/local下
sudo mv jdk1.8.0_171  /usr/local/jdk1.8
```

### 设置环境变量

* 设置全局生效
	
    修改全局配置文件，作用与所有用户： `vim /etc/profile`
    
    ```
    export JAVA_HOME=/usr/local/jdk1.8
	export JRE_HOME=${JAVA_HOME}/jre
	export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
	export PATH=.:${JAVA_HOME}/bin:$PATH
    ```
    
* 设置当前用户生效

	修改当前用户配置文件，只作用于当前用户：`vim ~/.bashrc`
    
    ```
    export JAVA_HOME=/usr/local/jdk1.8
	export JRE_HOME=${JAVA_HOME}/jre
	export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
	export PATH=.:${JAVA_HOME}/bin:$PATH
    ```

### 使修改的配置立刻生效

```
##对应方法一：
source /etc/profile 
##对应方法二：
source ~/.bashrc
```

### 检查是否安装成功

```
java -version
```