---
title: nodejs通过later实现定时执行任务
toc: true
date: 2016-08-08 09:45:57
categories:
tags:
---

大多数情况我们都选用使用Linux的cron来控制定时执行的任务。当我们要维护多台计算机，几十个，几百个定时任务的时候，用cron会带来非常大的运维成本。可能写到程序中，就是一个不错的选择了。nodejs有一个later的插件可以简单实现该功能。
如果已经安装过npm，可以直接执行``npm install later``安装该插件。如果没有请先安装npm。

```
var later = require('later');
var basic = {h:[00],m:[00]};  //设置每天凌晨执行
var composite=[
    basic
];
var sched={
    schedules:composite
};
later.date.localTime();  //设置本地时区
//var occurrences = later.schedule(sched).next(10);
//for(var i=0;i<10;i++){
//    console.log(occurrences[i]);
//}
var t=later.setInterval(function(){
    console.log("asdasd");
},sched);
```

可根据自己的需求进行更改。
