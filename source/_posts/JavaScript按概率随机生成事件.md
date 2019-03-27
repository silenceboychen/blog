---
title: JavaScript按概率随机生成事件
toc: true
date: 2017-08-02 12:20:03
categories: 算法
tags: [算法, javascript]
---

```
/*
*在抽奖的活动中经常会用到这个算法，不同奖项的获取概率不同，要按概率去随机生成对应的奖品
*
*/
function random(arr1, arr2) {
    var sum = 0,
        factor = 0,
        random = Math.random();

    for(var i = arr2.length - 1; i >= 0; i--) {
        sum += arr2[i]; // 统计概率总和
    };
    random *= sum; // 生成概率随机数
    for(var i = arr2.length - 1; i >= 0; i--) {
        factor += arr2[i];
        if(random <= factor) 
          return arr1[i];
    };
    return null;
};

// test
var a = ['mac', 'iphone', 'vivo', 'OPPO'];
var b = [0.1, 0.2, 0.3, 0.4];
console.log(random(a, b));
```