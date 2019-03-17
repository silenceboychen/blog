---
title: mysql，sql server等数据库连接集成库
toc: true
date: 2018-04-08 17:33:08
categories: nodejs
tags: [mysql, sql server, nodejs]
---

> 地址: https://github.com/fmfe/lib-sql  
### Installation

```
$ npm install @fmfe/lib-sql

```

or

```
$ yarn add @fmfe/lib-sql
```

### Usage

有两种传入配置信息的方式:

##### 使用[config](https://github.com/lorenwest/node-config)来管理我们的配置文件.

假设我们的项目目录下有一个config目录,config目录里有一个dev.json文件.

> config/dev.json

```

{
    "mysql": {
        "host": "127.0.0.1",
        "port": 3306,
        "database": "test",
        "user": "root",
        "password": "123456"
    },
    "mssql": {
        "user": "sa",
        "password": "123456",
        "server": "127.0.0.1",
        "database": "test",
        "port": 1433,
        "pool": {
            "min": 0,
            "max": 10,
            "idleTimeoutMillis": 3000
        }
    }
}

```

> mysql.js

```
const { mysql } = require('@fmfe/lib-sql');

const mysqlPool = mysql.init();

const _getNewSqlParamEntity = mysql._getNewSqlParamEntity;

// 执行单条sql语句  
// mysql.exec(mysqlPool, sql, params);
async function exec() {
    const sql1 = 'select * from ?? limit 2';
    const data = await mysql.exec(mysqlPool, sql1, ['tbl_user']);
}

// 执行mysql事务,可以传入多条增/删/改sql语句 
// mysql.exectrans(mysqlpool, sqlParamsEntity);
async function execTrans() {
    const sqlParamsEntity = [];
    const sql1 = 'insert into ?? (name, age, sex) values (?, ?, ?)';
    const param1 = ['tbl_user', 'aaa', 20, 1];
    sqlParamsEntity.push(_getNewSqlParamEntity(sql1, param1));

    const sql2 = 'insert into ?? (name, age, sex) values (?, ?, ?)';
    const param2 = ['tbl_user', 'bbb', 22, 0];
    sqlParamsEntity.push(_getNewSqlParamEntity(sql2, param2));

    const sql3 = 'update ?? set age = ? where id = ?';
    const param3 = ['tbl_user', 10, 1];
    sqlParamsEntity.push(_getNewSqlParamEntity(sql3, param3));
    // ....
    const data = await mysql.execTrans(mysqlPool, sqlParamsEntity);  
}
```

> mssql.js

```
const { mssql } = require('@fmfe/lib-sql');
const _getNewSqlParamEntity = mssql._getNewSqlParamEntity;

// 执行单条语句  
// mssql.exec(sql)
async function exec() {
    const sql1 = 'select Top 3 name, age, sex from tbl_user order by age desc';
    const data = await mssql.exec(sql1);  
}

// 执行sql server 事务, 最好执行增/删/改语句,这里只是用select演示使用方法
// mssql.exectrans(sqlParamsEntity); 
async function exectrans() {
  
    const sqlParamsEntity = [];
    const sql1 = 'select * from tbl_user where id = 1';
    sqlParamsEntity.push(_getNewSqlParamEntity(sql1));

    const sql2 = 'select * from tbl_user where id = 2';
    sqlParamsEntity.push(_getNewSqlParamEntity(sql2));
    // ...
    const data = await mssql.execTrans(sqlParamsEntity);  
}

```

由于使用``config``管理配置文件, 运行项目时通过使用命令: `` NODE_ENV=dev node ...``, ``@fmfe/lib-sql``即可自动获取到数据库相关配置.

##### 通过传入配置文件来调用库

我们引用上边的代码示例,只需做一点改动:

> mysql.js

只需在初始化时传入mysql数据库配置就好.

```
const { mysql } = require('@fmfe/lib-sql');

const mysqlPool = mysql.init({
    host: '127.0.0.1',
    port: 3306,
    database: 'test',
    user: 'root',
    password: '123456'
});

......

```

> mssql.js

在每次调用方法时传入配置

```
const { mssql } = require('@fmfe/lib-sql');
const config = {
    user: 'sa',
    password: '123456',
    server: '127.0.0.1',
    database: 'test',
    port: 1433,
    pool: {
        min: 0,
        max: 10,
        idleTimeoutMillis: 3000
    }  
}

async function exec() {
    ......
    const data = await mssql.exec(sql1, config);  
}

async function exectrans() {
  
    ......
    const data = await mssql.execTrans(sqlParamsEntity, config);  
}

```
