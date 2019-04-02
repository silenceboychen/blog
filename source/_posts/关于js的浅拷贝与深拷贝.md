---
title: 关于js的浅拷贝与深拷贝
toc: true
date: 2019-04-02 11:10:59
categories: javascript
tags: javascript
---

浅拷贝和深拷贝只针对像`Object, Array`这样的复杂对象的.简单来说，浅拷贝只拷贝一层对象的属性，而深拷贝则递归拷贝了所有层级。

### 浅拷贝


#### 通过 `Object.assign` 来实现浅拷贝。

```
let a = {
    num: 1
}
let b = Object.assign({}, a)
a.num = 2
console.log(b.num) // 1
```

#### 通过展开运算符`(…)`来实现浅拷贝

```
let a = {
    num: 1
}
let b = {...a}
a.num = 2
console.log(b.num) // 1
```

#### 通过属性赋值来实现浅拷贝:

```
const obj = { a:1, arr: [2,3] };
const shallowObj = shallowCopy(obj);

function shallowCopy(src) {
  var dst = {};
  for (var prop in src) {
    if (src.hasOwnProperty(prop)) {
      dst[prop] = src[prop];
    }
  }
  return dst;
}
```

该方法体现了浅拷贝的问题．因为浅拷贝只会将对象的各个属性进行依次拷贝，并不会进行递归拷贝，而 JavaScript 存储对象都是存地址的，所以浅拷贝会导致 obj.arr 和 shallowObj.arr 指向同一块内存地址．

导致的结果就是：

```
shallowObj.arr[1] = 5;
obj.arr[1]   // = 5
```

这种情况就需要用到深拷贝了．

### 深拷贝

#### 通过JSON序列化实现深拷贝

{% blockquote 你不知道的JavaScript(上) %}
许多JavaScript框架都提出了自己的解决办法,但是Javascript应该采用那种方法作为标准呐? 在很长一段时间里,这个问题都没有明确的答案.对于JSON安全(也就是说可以被序列化为一个JSON字符串并且可以根据这个字符串解析出一个结构和值完全一样的对象)的对象来说,有一种巧妙的复制方法:

var newObj = JSON.parse(JSON.stringify(someObj));

当然,这种方法需要保证对象是JSON安全的,所以只适用于部分情况.
{% endblockquote %}

该方法的局限性:

1. 会忽略 undefined
2. 会忽略 symbol
3. 不能序列化函数
4. 不能解决循环引用的对象

#### 递归完成深拷贝

```
function deepCopy(obj){
    //判断是否是简单数据类型，
    if(typeof obj == "object"){
        //复杂数据类型
        var result = obj.constructor == Array ? [] : {};
        for(let i in obj){
            result[i] = typeof obj[i] == "object" ? deepCopy(obj[i]) : obj[i];
        }
    }else {
        //简单数据类型 直接 == 赋值
        var result = obj;
    }
    return result;
}
```
