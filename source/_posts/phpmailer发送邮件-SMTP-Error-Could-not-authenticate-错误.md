---
title: 'phpmailer发送邮件 SMTP Error: Could not authenticate 错误'
toc: true
date: 2015-12-24 10:40:37
categories: php
tags: php
---

今天在使用phpmailer发送smtp邮件时提示 SMTP Error: Could not authenticate 错误，其中密码帐号都是正确的，邮箱也设置开启了SMTP功能。

上谷歌百度了一遍，有的说是服务器禁用了端口，有的说把class.phpmailer.php中的

代码如下

    function IsSMTP() {
        $this->Mailer = 'smtp';
    }
    
    改为
    
    function IsSMTP() {
        $this->Mailer = 'SMTP';
    }
（我的问题通过以上修改解决）
如果解决不了还有一些解决方法可供参考：
这个错误说明虚拟主机不支持PHPMailer默认调用的fsockopen函数，找到class.smtp.php文件，搜索fsockopen，就找到了这样一段代码：

 代码如下

    // connect to the smtp server
    $this->smtp_conn = @fsockopen($host,// the host of the server
    $port,// the port to use
    $errno,   // error number if any
    $errstr,  // error message if any
    $tval);   // give up after ? secs

**方法1**：将fsockopen函数替换成pfsockopen函数

首先，在php.ini中去掉下面的两个分号

    ;extension=php_sockets.dll
    ;extension=php_openssl.dll

然后重启一下

因为pfsockopen的参数与fsockopen基本一致，所以只需要将@fsockopen替换成@pfsockopen就可以了。

**方法2**：使用stream_socket_client函数

一般fsockopen()被禁，pfsockopen也有可能被禁，所以这里介绍另一个函数stream_socket_client()。

stream_socket_client的参数与fsockopen有所不同，所以代码要修改为：

 代码如下

    $this->smtp_conn = stream_socket_client("tcp://".$host.":".$port, $errno,  $errstr,  $tval);

这样就可以了。


