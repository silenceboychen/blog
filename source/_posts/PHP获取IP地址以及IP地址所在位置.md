---
title: PHP获取IP地址以及IP地址所在位置
toc: true
date: 2015-12-22 10:29:45
categories: php
tags: php
---

获取IP地址：
```
function getIP(){
    if (isset($_SERVER)) {
        if (isset($_SERVER['HTTP_X_FORWARDED_FOR'])) {
            $realip = $_SERVER['HTTP_X_FORWARDED_FOR'];
        } elseif (isset($_SERVER['HTTP_CLIENT_IP'])) {
            $realip = $_SERVER['HTTP_CLIENT_IP'];
        } else {
            $realip = $_SERVER['REMOTE_ADDR'];
        }
    } else {
        if (getenv("HTTP_X_FORWARDED_FOR")) {
            $realip = getenv( "HTTP_X_FORWARDED_FOR");
        } elseif (getenv("HTTP_CLIENT_IP")) {
            $realip = getenv("HTTP_CLIENT_IP");
        } else {
            $realip = getenv("REMOTE_ADDR");
        }
    }
    return $realip;
}

echo $ip = getIP();
```
> 新浪接口根据ip查询所在区域信息

```
$res0 = file_get_contents("http://int.dpool.sina.com.cn/iplookup/iplookup.php?format=json&ip=$ip");
$res0 = json_decode($res0,true);
print_r($res0);
echo "<br/>";
```

> 淘宝接口根据ip查询所在区域信息

```
$res1 = file_get_contents("http://ip.taobao.com/service/getIpInfo.php?ip=$ip");
$res1 = json_decode($res1,true);
print_r($res1);
echo "<br/>";
```