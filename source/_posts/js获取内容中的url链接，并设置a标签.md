---
title: js获取内容中的url链接，并设置a标签
toc: true
date: 2016-05-24 09:39:27
categories: javascript
tags: [javascript, 正则表达式]
---


```
var regexp = /(http:\/\/|https:\/\/)((\w|=|\?|\.|\/|\&|-)+)/g;
content = content.replace(regexp, function($url){
 return "<a href='" + $url + "' target='_blank'>" + $url + "</a>";
});
console.log(content);
```