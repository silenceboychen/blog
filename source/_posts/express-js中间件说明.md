---
title: express.js中间件说明
toc: true
date: 2019-03-19 09:39:06
categories: nodejs
tags: [express.js, middleware]
---

express的新开发人员往往对路由处理程序和中间件之间的区别感到困惑。因此他们也对``app.use()``,``app.all()``,``app.get()``,``app.post()``,``app.delete()``和``app.put()``方法的区别感到困惑。

在本文中,我将解释中间件和路由处理程序之间的区别。以及如何正确使用``app.use()``,``app.all()``,``app.get()``,``app.post()``,``app.delete()``和``app.put()``方法。

### 路由处理

``app.use()``,``app.all()``,``app.get()``,``app.post()``,``app.delete()``和``app.put()``全部是用来定义路由的。这些方法都用于定义路由。路由用于处理HTTP请求。路由是路径和回调的组合，在请求的路径匹配时执行。回调被称为路由处理程序。

它们之间的区别是处理不同类型的HTTP请求。例如： ``app.get()``方法仅仅处理get请求，而``app.all()``处理GET、POST等请求。


> 下面是一个例子,如何定义一个路由：

```
var app = require("express")();

app.get("/", function(req, res, next){
    res.send("Hello World!!!!");
});

app.listen(8080);
```

每个路由处理程序都获得对当前正在提供的HTTP请求的请求和响应对象的引用。

> 可以为单个HTTP请求执行多个路由处理程序。这是一个例子：

```
var app = require("express")();

app.get("/", function(req, res, next){
    res.write("Hello");
    next();
});

app.get("/", function(req, res, next){
    res.write(" World !!!");
    res.end();
});

app.listen(8080);

```

这里第一个句柄写入一些响应，然后调用``next()``。 ``next()``方法用于调用与路径路径匹配的下一个路由处理程序。

路由处理程序必须结束请求或调用下一个路由处理程序。

我们还可以将多个路由处理程序传递给``app.all()``，``app.get()``，``app.post()``，``app.delete()``和``app.put()``方法。
> 这是一个证明这一点的例子：

```
var app = require("express")();

app.get("/", function(req, res, next){
    res.write("Hello");
    next();
}, function(req, res, next){
    res.write(" World !!!");
    res.end();
});

app.listen(8080);
```

### 中间件

中间件是一个位于实际请求处理程序之上的回调。它采用与路由处理程序相同的参数。

要了解中间件，我们来看一个带有``dashboard``和``profile``页面的示例站点。要访问这些页面，用户必须登录。还会记录对这些页面的请求。
> 以下是这些页面的路由处理程序的代码：

```
var app = require("express")();

function checkLogin(){
    return false;
}

function logRequest(){
    console.log("New request");
}

app.get("/dashboard", function(req, res, next){

    logRequest();

    if(checkLogin()){
        res.send("This is the dashboard page");
    }
    else{
        res.send("You are not logged in!!!");
    }
});

app.get("/profile", function(req, res, next){

    logRequest();

    if(checkLogin()){
        res.send("This is the dashboard page");
    }
    else{
        res.send("You are not logged in!!!");
    }
});

app.listen(8080);

```

这里的问题是有很多重复的代码，即我们不得不多次使用``logRequest()``和``checkLogin()``函数。这也使得更新代码变得困难。因此，为了解决这个问题，我们可以为这两条路径编写一条通用路径。

> 这是重写的代码：

```
var app = require("express")();

function checkLogin(){
    return false;
}

function logRequest(){
    console.log("New request");
}

app.get("/*", function(req, res, next){
    logRequest();
    next();
})

app.get("/*", function(req, res, next){
    if(checkLogin()){
        next();
    }
    else{
        res("You are not logged in!!!");
    }
})

app.get("/dashboard", function(req, res, next){
    res.send("This is the dashboard page");
});

app.get("/profile", function(req, res, next){
    res.send("This is the dashboard page");
});

app.listen(8080);
```

这里的代码看起来更清晰，更易于维护和更新。这里将前两个定义的路由处理程序称为中间件，因为它们不处理请求，而是负责预处理请求。

Express为我们提供了``app.use()``方法，该方法专门用于定义中间件。 ``app.use()``方法可能看起来与``app.all()``类似，但它们之间存在很多差异，这使得``app.use()``非常适合于声明中间件。让我们看看``app.use()``方法是如何工作的：

#### ``app.use()`` 和 ``app.all()`` 的不同:

##### CALLBACK


``app.use()``只需要一个回调，而``app.all()``可以进行多次回调。

##### PATH

``app.use()``只查看url是否以指定路径开头,``app.all()``匹配完整路径。

> 这里有一个例子来说明:

```
app.use( "/product" , mymiddleware);
// will match /product
// will match /product/cool
// will match /product/foo

app.all( "/product" , handler);
// will match /product
// won't match /product/cool   <-- important
// won't match /product/foo    <-- important

app.all( "/product/*" , handler);
// won't match /product        <-- Important
// will match /product/cool
// will match /product/foo

```

##### NEXT()


中间件内的``next()``调用下一个中间件或路由处理程序，具体取决于接下来声明的那个。但是路由处理程序中的``next()``仅调用下一个路由处理程序。如果接下来有中间件，则跳过它。因此，必须在所有路由处理程序之前声明中间件。

> 这里有一个例子来说明:

```
var express = require('express');
var app = express();

app.use(function frontControllerMiddlewareExecuted(req, res, next){
  console.log('(1) this frontControllerMiddlewareExecuted is executed');
  next();
});

app.all('*', function(req, res, next){
  console.log('(2) route middleware for all method and path pattern "*", executed first and can do stuff before going next');
  next();
});

app.all('/hello', function(req, res, next){
  console.log('(3) route middleware for all method and path pattern "/hello", executed second and can do stuff before going next');
  next();
});

app.use(function frontControllerMiddlewareNotExecuted(req, res, next){
  console.log('(4) this frontControllerMiddlewareNotExecuted is not executed');
  next();
});

app.get('/hello', function(req, res){
  console.log('(5) route middleware for method GET and path patter "/hello", executed last and I do my stuff sending response');
  res.send('Hello World');
});

app.listen(80);
```

现在我们看到了``app.use()``方法的唯一性以及它用于声明中间件的原因。

> 让我们重写我们的示例站点代码：

```
var app = require("express")();

function checkLogin(){
    return false;
}

function logRequest(){
    console.log("New request");
}

app.use(function(req, res, next){
    logRequest();
    next();
})

app.use(function(req, res, next){

    if(checkLogin()){
        next();
    }
    else{
        res.send("You are not logged in!!!");
    }
})

app.get("/dashboard", function(req, res, next){
    res.send("This is the dashboard page");
});

app.get("/profile", function(req, res, next){
    res.send("This is the dashboard page");
});

app.listen(8080);
```