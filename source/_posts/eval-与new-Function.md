---
title: eval()与new Function()
toc: true
date: 2019-04-14 20:55:54
categories: javascript
tags: javascript
---

### eval

`eval`接受字符串参数，解析其中的js代码。如果编译失败，会抛出异常，否则执行其中的代码，计算返回值。

```
eval('2+2');  // 4
eval('console.log("ok")');  // ok
```
在实际应用中，通常这样转换JSON。

```
var jsonStr = '{ "age": 20, "name": "jack" }';
eval('(' + jsonStr + ')');
```
为什么要加括号呢？

> 因为js中{}通常是表示一个语句块，`eval`只会计算语句块内的值进行返回。加上括号就变成一个整体的表达式。

```
console.log( eval('{}') );      // undefind
console.log( eval('({})') );    // Object {}
```
使用`eval`需要注意执行作用域

```
var s = 1;
function a() {
    eval('var s=2');
    console.log(s);
}
a();                // 2
console.log(s);     // 1
```
在局部环境使用eval便会创建局部变量。可以显示指定`eval`调用者来改变上下文环境。

```
var s = 'global';
function a() {
    eval('var s = "local"');
    console.log(s);                 // local
    console.log(eval('s'));         // local
    console.log(window.eval('s'));  // global
}
```

### Function

在之前我对于`Function`的了解只限于“定义方法的一种非主流方式”。却忽略了`Function`与`eval`相同的字符串参数特性。

语法：`var func = new Function(arg1, arg2, ..., functionBody);`

实例：

```
var add = new Function('a', 'b', 'return a+b;');
console.log( add(2, 3) );    // 5
```
由于其形参使用字符串的方式表示，也可以使用1个字符串来描述多个形参。

```
var add = new Function('a, b', 'return a+b;');
console.log( add(2, 3) );    // 5
```
在转换JSON的实际应用中，只需要这么做。

```
var jsonStr = '{ "age": 20, "name": "jack" }',
    json = (new Function('return ' + jsonStr))();
```
`eval` 与 `Function` 都有着动态编译js代码的作用，但是在实际的编程中并不推荐使用。如果可以，请用更好的方法替代。

在一些特殊的运用场合，也有一些合理运用的实践。比如模板解析等。
