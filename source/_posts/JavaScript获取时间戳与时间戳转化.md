---
title: JavaScript获取时间戳与时间戳转化
toc: true
date: 2016-08-03 09:57:19
categories: javascript
tags: javascript
---

### Javascript 获取当前时间戳（毫秒级别）：

* 第一种方法：

```
var timestamp1 = Date.parse( new Date());
```

结果：1470220594000

* 第二种方法：

```
var timestamp2 = ( new Date()).valueOf();
```

结果：1470220608533

* 第三种方法：

```
var timestamp3 = new Date().getTime();
```

结果：1470220608533

> 第一种获取的时间戳是精确到秒，第二种和第三种是获取的时间戳精确到毫秒。

### 获取指定时间的时间戳：

```
new Date("2016-08-03 00:00:00").getTime;
```

### 时间戳转化成时间：

```
function timetrans(date){
    var date = new Date(date*1000);//如果date为13位不需要乘1000
    var Y = date.getFullYear() + '-';
    var M = (date.getMonth()+1 < 10 ? '0'+(date.getMonth()+1) : date.getMonth()+1) + '-';
    var D = (date.getDate() < 10 ? '0' + (date.getDate()) : date.getDate()) + ' ';
    var h = (date.getHours() < 10 ? '0' + date.getHours() : date.getHours()) + ':';
    var m = (date.getMinutes() <10 ? '0' + date.getMinutes() : date.getMinutes()) + ':';
    var s = (date.getSeconds() <10 ? '0' + date.getSeconds() : date.getSeconds());
    return Y+M+D+h+m+s;
}
```