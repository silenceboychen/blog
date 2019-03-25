---
title: js利用clipboardData在网页中实现截屏粘贴的功能
toc: true
date: 2016-07-23 09:54:44
categories: javascript
tags: javascript
---

最近在做一个将屏幕截图直接粘贴发送的功能，于是对此做了一些研究，下面是具体的实现代码：
html代码如下，在这里只是简单的做了一个textare框用作演示

```
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width">
  <title>截屏粘贴</title>
</head>
<body>
  <textarea onpaste ="paste()">
  </textarea>
</body>
</html>
```

具体实现在JavaScript中：

```
function paste(event){
  var clipboardData = event.clipboardData;
  console.log(clipboardData);
  var items,item,types;
  if( clipboardData ){
    items = clipboardData.items;
    if( !items ){
      return;
    }
    // 保存在剪贴板中的数据类型
    types = clipboardData.types || [];
    for(var i=0 ; i < types.length; i++ ){
      if( types[i] === 'Files' ){
        item = items[i];
        break;
      }
    }
    // 判断是否为图片数据
    if( item && item.kind === 'file' && item.type.match(/^image\//i) ){
      // 读取该图片
      var file = item.getAsFile(),
          reader = new FileReader();
      reader.readAsDataURL(file);
      console.log(reader);
      //下面是讲粘贴的图片内容传送到后端进行处理，如果直接前端处理可以不要后边的代码
      var xhr = new XMLHttpRequest();
      xhr.open('post', '/pasteImage',true);
      xhr.setRequestHeader('Content-Type', 'application/json');
      reader.onload = function(){
        console.log(reader.result);
        xhr.send(JSON.stringify({
          file: reader.result
        }));
      };
      //接收返回数据
      xhr.onload = function(){
        var response = JSON.parse(xhr.responseText);
        if(response.code == 200){
        //
        }else{
        //
        }
      }
    }
  }
}
```