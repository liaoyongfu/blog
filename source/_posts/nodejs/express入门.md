---
title: express入门
date: 2017-10-12 11:18:46
tags: 
    - express
    
    - nodejs
---

## 配置handlebars

```
//须安装express-handlebars
var exhbs = require('express-handlebars');
var index = require('./routes/index');
var users = require('./routes/users');

var app = express();

// view engine setup，这边名称如果为.hbs，则文件命名结尾也是.hbs（extname无效？）
app.engine('.hbs', exhbs({
    extname: '.hbs'
}));
app.set('views', path.join(__dirname, 'views'));
app.set('view engine', '.hbs');
```

## 使用supervisor进行热部署

安装：

```
npm install -g supervisor
```

如果你使用的是 Linux 或 Mac，直接键入上面的命令很可能会有权限错误。原因是 npm需要把 supervisor 安装到系统目录，需要管理员授权，可以使用 sudo npm install -g supervisor 命令来安装。

接下来，更改pcakge.json中的`start`字段：

```
"start": "supervisor bin/www"
```

之后运行`npm start`既可，命令行窗口会显示启动成功信息，即开启了代码监听。

## 开放API

```
var express = require('express');
var router = express.Router();
var mysql = require('mysql');
var conncection = mysql.createConnection({
    host: 'localhost',
    user: 'root',
    password: '*****',
    database: 'test',
    port: 3306
});
conncection.connect();

router.get('/getFamilyList', function(req, res, next){
    conncection.query('select * from family', function(err, results, fields){
        if(err) throw err;
        res.json(results);
    });
});

router.get('/getFamilyById', function (req, res, next) {
    conncection.query('select * from family where id=?', [req.query.id], function(err, results, fields){
        if(err) throw err;
        res.json(results[0]);
    });
});

router.post('/updateFamilyById', function (req, res, next) {
    conncection.query('update family set name=?,age=? where id=?', [req.body.name, req.body.age, req.body.id], function(err, results, fields){
        if(err) throw err;
        res.json({
            status: true
        });
    });
});

module.exports = router;
```
  