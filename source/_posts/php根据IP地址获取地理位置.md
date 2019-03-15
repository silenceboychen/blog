---
title: php根据IP地址获取地理位置
toc: true
date: 2016-01-24 22:35:15
categories: php
tags: php
---

```
<?php
header("Content-type: text/html; charset=utf-8"); 
//获取IP地址的方法
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

//通过php的file_get_contents()方法获取地理位置
//新浪接口根据ip查询所在区域信息

$res0 = file_get_contents("http://int.dpool.sina.com.cn/iplookup/iplookup.php?format=json&ip=$ip");
$res0 = json_decode($res0,true);
print_r($res0);
echo "<br/>";

//淘宝接口根据ip查询所在区域信息

$res1 = file_get_contents("http://ip.taobao.com/service/getIpInfo.php?ip=$ip");
$res1 = json_decode($res1,true);
print_r($res1);
echo "<br/>";


//通过php的curl获取地理位置
//新浪根据IP获取地理位置API

$url = 'http://int.dpool.sina.com.cn/iplookup/iplookup.php?format=json&ip=$ip'; 
$ch = curl_init($url); 
curl_setopt($ch,CURLOPT_ENCODING ,'utf8'); 
curl_setopt($ch, CURLOPT_TIMEOUT, 10); 
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true) ; // 获取数据返回 
$location = curl_exec($ch); 
$location = json_decode($location); 
print_r($location);
curl_close($ch); 
  
$loc = ""; 
if($location===FALSE) return ""; 
if (empty($location->desc)) { 
    $loc = $location->province.$location->city.$location->district.$location->isp; 
}else{ 
    $loc = $location->desc; 
} 
echo  $loc; 


//腾讯根据IP获取地理位置API

$url = 'http://ip.qq.com/cgi-bin/searchip?searchip1=$ip'; 
$ch = curl_init($url); 
curl_setopt($ch,CURLOPT_ENCODING ,'gb2312'); 
curl_setopt($ch, CURLOPT_TIMEOUT, 10); 
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true) ; // 获取数据返回 
$result = curl_exec($ch); 
$result = mb_convert_encoding($result, "utf-8", "gb2312"); // 编码转换，否则乱码 
curl_close($ch); 
preg_match("@<span>(.*)</span></p>@iU",$result,$ipArray); 
$loc = $ipArray[1]; 
echo $loc; 

```