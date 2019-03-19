---
title: express简单测试连接mysql
toc: true
date: 2016-01-06 22:11:16
categories: nodejs
tags: [nodejs, mysql, express.js]
---

使用express应用生成器生成express模板后，先写package.json

    {
      "name": "mysql-test",
      "version": "0.0.1",
      "private": true,
      "scripts": {
        "start": "node ./bin/www"
      },
      "dependencies": {
        "body-parser": "~1.13.2",
        "cookie-parser": "~1.3.5",
        "debug": "~2.2.0",
        "ejs": "~2.3.3",
        "express": "~4.13.1",
        "morgan": "~1.6.1",
        "serve-favicon": "~2.3.0",
    	"mysql":"*"
      }
    }

npm install安装依赖

新建立两个文件夹，models和config

写一个config配置文件，去连接mysql的:

    module.exports = {
        mysql_dev: {
            host: 'localhost',
            user: 'user',
            password: 'your password',
            database: 'your db name',
            connectionLimit: 10,
            supportBigNumbers: true
        }
    };

再写上一个database.js文件：

    var mysql = require('mysql');
    var config = require('../config/config');
    
    var pool = mysql.createPool(config.mysql_dev);
    
    exports.pool = pool;

在models里建立一个User.js文件作为model：

        var db = require('./database');
        
        var User = function() {};
        
        User.prototype.find = function(id, callback) {
            var sql = "SELECT * FROM users WHERE id =?";
            // get a connection from the pool
            db.pool.getConnection(function(err, connection) {
                if (err) {
                    callback(true);
                    return;
                }
                // make the query
                connection.query(sql, [id], function(err, results) {
                    if (err) {
                        callback(true);
                        return;
                    }
                    callback(false, results);
                });
            });
        };
    
    module.exports = User;

最后在app.js里引入，再调用：

    var User = require('./models/User');
    
    //.......
    
    app.get('/users/:userid',function(req,res){
        var userid = req.params.userid;
        var user = new User();
        user.find(userid,function(err,result){
            if(err){
                res.send('not found');
            }
            res.send(result.length === 1 ? result[0]:result);
        });
    
    });

这样就简单地完成一个后端的node.js分级结构，前端提供rest请求。