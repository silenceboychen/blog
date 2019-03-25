---
title: nodejs读写excel内容
toc: true
date: 2016-07-16 09:47:27
categories: nodejs
tags: [nodejs, excel]
---

支持读写Excel的node.js模块

* node-xlsx: 基于Node.js解析excel文件数据及生成excel文件，仅支持xlsx格式文件；
* excel-parser: 基于Node.js解析excel文件数据，支持xls及xlsx格式文件；
* excel-export : 基于Node.js将数据生成导出excel文件，生成文件格式为xlsx；
* node-xlrd: 基于node.js从excel文件中提取数据，仅支持xls格式文件。

我将展示通过node-xlsx提取上传上来的excel文件里的数据，以及生成新的excel文件。代码如下：

```
var xlsx = require('node-xlsx');
var fs = require('fs');
//读取文件内容
var obj = xlsx.parse(__dirname+'/test.xlsx');
var excelObj=obj[0].data;
console.log(excelObj);

var data = [];
for(var i in excelObj){
    var arr=[];
    var value=excelObj[i];
    for(var j in value){
        arr.push(value[j]);
    }
    data.push(arr);
}
var buffer = xlsx.build([
    {
        name:'sheet1',
        data:data
    }        
]);

//将文件内容插入新的文件中
fs.writeFileSync('test1.xlsx',buffer,{'flag':'w'});
```