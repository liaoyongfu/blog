---

title: create-react-app读后感

date: 2018-3-30 16:48:34

tags: 

  - react

---

## 项目github地址

https://github.com/facebook/create-react-app

## 一些概念

### npx

 npm 5.2+多出一个工具，npx 会帮你执行依赖包里的二进制文件。
 
 旨在提高从npm注册表使用软件包的体验 ，npm使得它非常容易地安装和管理托管在注册表上的依赖项，npx使得使用CLI工具和其他托管在注册表。

 举例来说，之前我们可能会写这样的命令：

 ```
npm i -D webpack
./node_modules/.bin/webpack -v
 ```

 有了 npx，你只需要这样：

 ```
npm i -D webpack
npx webpack -v
 ```

 也就是说 npx 会自动查找当前依赖包中的可执行文件，如果找不到，就会去 PATH 里找。如果依然找不到，就会帮你安装！

 npx 甚至支持运行远程仓库的可执行文件，如:

 ```
$ npx github:piuccio/cowsay hello
npx: 1 安装成功，用时 1.663 秒
 _______
< hello >
 -------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
 ```

 再比如 npx http-server 可以一句话帮你开启一个静态服务器！（第一次运行会稍微慢一些）：

 ```
$ npx http-server
npx: 23 安装成功，用时 48.633 秒
Starting up http-server, serving ./
Available on:
  http://127.0.0.1:8080
  http://192.168.5.14:8080
Hit CTRL-C to stop the server
 ```


 ### react-scripts

 bin目录下主要文件：

 ```
#!/usr/bin/env node

"use strict";

const crypto = require('crypto');
const path = require('path');
// 跨平台调用系统命令
const spawn = require('cross-spawn');

const script = process.argv[2];
const args = process.argv.slice(3);

switch (script) {
  case 'build':
  case 'start':
  case 'upload':
  case 'test': {
    const result = spawn.sync(
      'node',
      [require.resolve(path.join('../scripts', script))].concat(args),
      { stdio: 'inherit' }
    );
    process.exit(result.status);
    break;
  }
  case 'pwhash': {
    let stdin = process.openStdin();
    let data = "";
    stdin.on('data', function(chunk) {
      data += chunk;
    });

    stdin.on('end', function() {
      let hash = crypto.createHash('md5').update(data).digest('hex');
      console.log(hash);
    });
    break;
  }
  default:
    console.log(`Unknown script "${script}".`);
    break;
}
 ```

- !/usr/bin/node是告诉操作系统执行这个脚本的时候，调用/usr/bin下的node解释器；

- !/usr/bin/env node这种用法是为了防止操作系统用户没有将node装在默认的/usr/bin路径里。当系统看到这一行的时候，首先会到env设置里查找node的安装路径，再调用对应路径下的解释器程序完成操作。

 ## 创建项目

 > node版本必须>=6

 运行创建命令：

 ```
npx create-react-app my-app
 ```

 目录结构大致如下：

 ```
my-app
├── README.md
├── node_modules
├── package.json
├── .gitignore
├── public
│   └── favicon.ico
│   └── index.html
│   └── manifest.json
└── src
    └── App.css
    └── App.js
    └── App.test.js
    └── index.css
    └── index.js
    └── logo.svg
    └── registerServiceWorker.js
 ```

 ## 运行`npm start`或`yarn start`开启项目

 ## 运行`npm test`或`yarn test`测试

 ## 运行`npm run build`或者`yarn run build`打包项目

未完待续...