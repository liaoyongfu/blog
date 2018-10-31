---
title: koa2学习：路由
date: 2018-9-5
tag: koa2
---

参考网址：https://chenshenhai.github.io/koa2-note/

## 原生koa2路由

````
const Koa = require('koa');
const app = new Koa();
const fs = require('fs');

async function route(url) {
    let view = '404.html';
    switch (url) {
        case '/':
            view = 'index.html';
            break;
        case '/index':
            view = 'index.html';
            break;
        case '/todo':
            view = 'todo.html';
            break;
        case '/404':
            view = '404.html';
            break;
        default:
            break;
    }

    return await render(view);
}

async function render(page) {
    return new Promise((resolve, reject) => {
        fs.readFile(`./views/${page}`, 'utf8', (err, data) => {
            if (err) {
                reject(err);
            }else{
                resolve(data);
            }
        });
    })
}

app.use(async (ctx) => {
    const {url} = ctx.request;
    ctx.body = await route(url);
});

app.listen(3000);
console.log('[demo] start-quick is starting at port 3000');
````

这里`fs.readFile(..., 'utf8', ...)`第二个参数是文件编码，如果不指定默认为`raw buffer`，刷新页面会变成下载。

## koa-router中间件

````
const Router = require('koa-router');

// 主页路由
let home = new Router();

home.get('/', async(ctx) => {
    ctx.body = `
    <ul>
      <li><a href="/page/helloworld">/page/helloworld</a></li>
      <li><a href="/page/404">/page/404</a></li>
    </ul>
  `;
});

// 子页路由
let page = new Router();

page.get('/404', async(ctx) => {
    ctx.body = '404 page';
}).get('/helloworld', async(ctx) => {
    ctx.body = 'helloworld page';
});

// 主路由
let router = new Router();
router.use('/', home.routes(), home.allowedMethods());
router.use('/page', page.routes(), page.allowedMethods());

app.use(router.routes()).use(router.allowedMethods());
````

官方文档：https://github.com/alexmingoia/koa-router

## 获取请求参数

### GET请求参数获取

在koa中，获取GET请求数据源头是koa中request对象中的query方法或querystring方法，query返回是格式化好的参数对象，querystring返回的是请求字符串，由于ctx对request的API有直接引用的方式，所以获取GET请求数据有两个途径。

### POST请求参数获取

对于POST请求的处理，koa2没有封装获取参数的方法，需要通过解析上下文context中的原生node.js请求对象req，将POST表单数据解析成query string（例如：a=1&b=2&c=3），再将query string 解析成JSON格式（例如：{"a":"1", "b":"2", "c":"3"}）

````
function parsePostData(ctx) {
    return new Promise((resolve, reject) => {
        try {
            let postdata = '';
            ctx.req.addListener('data', function (data) {
                postdata += data;
            });
            ctx.req.addListener('end', function () {
                let parseData = parseQueryStr(postdata);

                resolve(parseData);
            });
        }catch(err){
            reject(err);
        }
    });
}

// 将POST请求参数字符串解析成JSON
function parseQueryStr(queryStr) {
    let queryData = {}
    let queryStrList = queryStr.split('&')
    for (let [index, queryStr] of queryStrList.entries()) {
        let itemList = queryStr.split('=')
        queryData[itemList[0]] = decodeURIComponent(itemList[1])
    }
    return queryData
}

app.use(async (ctx) => {
    if (ctx.url === '/' && ctx.method === 'GET') {
        ctx.body = `
          <h1>koa2 request post demo</h1>
          <form method="POST" action="/">
            <p>userName</p>
            <input name="userName" /><br/>
            <p>nickName</p>
            <input name="nickName" /><br/>
            <p>email</p>
            <input name="email" /><br/>
            <button type="submit">submit</button>
          </form>
        `;
    } else if (ctx.url === '/' && ctx.method === 'POST') {
        ctx.body = await parsePostData(ctx);
    } else {
        // 其他请求显示404
        ctx.body = '<h1>404！！！ o(╯□╰)o</h1>'
    }
});
````

## koa-bodyparser中间件

````
var Koa = require('koa');
var bodyParser = require('koa-bodyparser');

var app = new Koa();
app.use(bodyParser());

app.use(async ctx => {
  // the parsed body will store in ctx.request.body
  // if nothing was parsed, body will be an empty object {}
  ctx.body = ctx.request.body;
});
````

## 静态资源加载

````
const serve = require('koa-static');
const Koa = require('koa');
const app = new Koa();

// $ GET /package.json
app.use(serve('.'));

// $ GET /hello.txt
app.use(serve('test/fixtures'));

// or use absolute paths
app.use(serve(__dirname + '/test/fixtures'));

app.listen(3000);

console.log('listening on port 3000');
````
## session操作

koa2原生功能只提供了cookie的操作，但是没有提供session操作。session就只用自己实现或者通过第三方中间件实现。在koa2中实现session的方案有一下几种

- 如果session数据量很小，可以直接存在内存中
- 如果session数据量很大，则需要存储介质存放session数据

数据库存储方案：

- 将session存放在MySQL数据库中

    需要用到中间件：
    
    - koa-session-minimal 适用于koa2 的session中间件，提供存储介质的读写接口 。
    - koa-mysql-session 为koa-session-minimal中间件提供MySQL数据库的session数据读写操作。
    - 将sessionId和对于的数据存到数据库
- 将数据库的存储的sessionId存到页面的cookie中
- 根据cookie的sessionId去获取对于的session信息
