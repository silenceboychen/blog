---
title: JavaScript数组随机排序
toc: true
date: 2017-08-08 12:21:39
categories: 算法
tags: [算法, javascript]
---

```
//不断从原数组中随机取一个元素放进新数组，同时删除原数组中该值，递归重复至全部取出。

function randomSort(arr, newArr) {
    var newArr = newArr || []
    if (arr.length == 1) {
        newArr.push(arr[0])
        return newArr; // 相当于递归退出
    }
 
    var random = Math.ceil(Math.random() * arr.length) - 1
    newArr.push(arr[random])
    arr.splice(random, 1)
    return randomSort(arr, newArr)
}
randomSort([1, 2, 3, 4, 5, 6, 7]); //[2, 3, 1, 5, 6, 7, 4]
randomSort([1, 2, 3, 4, 5, 6, 7]); //[3, 4, 2, 5, 1, 6, 7]
```