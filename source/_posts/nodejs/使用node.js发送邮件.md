---
title: 使用node.js发送邮件
date: 2017-12-13 15:20:57
tags: 
    - express
    
    - nodejs
---

## 使用nodemailler发送邮件

这里我们以使用qq邮箱发送邮件为例。

1. 安装nodemailer

```
npm install --save nodemailer
```

2. 使用

```
var nodemailer = require('nodemailer');

var mailTransport = nodemailer.createTransport({
    host: 'smtp.qq.com',  //服务器地址
    port: 465,  //端口
    auth: {
      user: 'xxx@qq.com',
      pass: '授权码'  //注：这里需要是授权码（在设置->账户->POP3/IMAP/SMTP/Exchange/CardDAV/CalDAV服务下“生成授权码”），而不是登录密码
    }
  });
  mailTransport.sendMail({
    from: '"your name" <xxx@qq.com>',
    to: 'xxx@gmail.com',  //收件人地址
    subject: '测试邮件111',
    text: '测试内容哦11111！！！！！！！'
  }, function(err){
    if(err) console.error('发送失败:', err);
    console.info('发送成功！');
  })
```