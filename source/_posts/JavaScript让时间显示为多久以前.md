---
title: JavaScript让时间显示为多久以前
toc: true
date: 2016-06-02 14:41:28
categories: javascript
tags: javascript
---

在做论坛的功能时，要求帖子的发帖时间显示几秒前，几分钟前，几小时前。。。这种功能，于是就把获取到的发帖时间做了如下处理：

```
function gettime(createtime){
  var now=Date.parse(new Date())/1000;
  var limit=now-createtime;
  var content="";
  if(limit<60){
    content="刚刚";
  }else if(limit>=60 && limit<3600){
    content=Math.floor(limit/60)+"分钟前";
  }else if(limit>=3600 && limit<86400){
    content=Math.floor(limit/3600)+"小时前";
  }else if(limit>=86400 && limit<2592000){
    content=Math.floor(limit/86400)+"天前";
  }else if(limit>=2592000 && limit<31104000){
    content=Math.floor(limit/2592000)+"个月前";
  }else{
    content="很久前";
  }
  return content;
}
```