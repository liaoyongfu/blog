---
title: 2018年学习计划（babel篇）
date: 2018-9-4
tag: babel
---

本篇基于最新版的babel@7。详细请查看官网：https://babeljs.io/docs/en

## 使用指南

1. 安装必须的包：

````
yarn add @babel/core @babel/cli @babel/preset-env --dev
yarn add @babel/polyfill
````

2. 创建`babel.config.js`文件：

````
const presets = [
  ["@babel/env", {
    targets: {
      edge: "17",
      firefox: "60",
      chrome: "67",
      safari: "11.1"
    },
    useBuiltIns: "usage"
  }]
];

module.exports = { presets };
````

3. 运行编译命令：

````
./node_modules/.bin/babel src --out-dir lib
````

> 当然也可以使用`npx babel`命令

## polyfill

`@babel/polyfill`最新的选项`useBuiltIns`包括三个选项：

- `false`：不为每个文件自动添加polyfill文件或者转换`import '@babel/polyfill';

- `entry`：只会在入口引入`@babel/polyfll`将会自动转换成引用全部`core-js`模块；

````
// In
import "@babel/polyfill";

// Out
require("core-js/modules/es6.array.copy-within");
require("core-js/modules/es6.array.fill");
require("core-js/modules/es6.array.find");
...
````
- `usage`：代码中不需要引入`@babel/polyfll`，它会自动引入需要的`core-js`模块：

````
// In
new Promise(function (resolve) {
    setTimeout(function () {
      resolve();
    }, 2000);
  });
  
// Out
require("core-js/modules/es6.promise");

new Promise(function (resolve) {
    setTimeout(function () {
      resolve();
    }, 2000);
  });
````

## 预设

一下是比较常用的官方预设：

- @babel/preset-env

- @babel/preset-flow

- @babel/preset-react

- @babel/preset-typescript

## @babel/plugin-transform-runtime

- 当我们使用`gernerator/async`时自动引用`@babel/runtime/regenerator`

- 当需要使用垫片方法时，自动引用`core-js`中的相对应方法

- 自动移除babel所需的一些帮助方法，使用`@babel/runtime/helpers`替代，以防多次引用

Babel 转译后的代码要实现源代码同样的功能需要借助一些帮助函数，例如，{ [name]: 'JavaScript' } 转译后的代码如下所示：

````
'use strict';
function _defineProperty(obj, key, value) {
  if (key in obj) {
    Object.defineProperty(obj, key, {
      value: value,
      enumerable: true,
      configurable: true,
      writable: true
    });
  } else {
    obj[key] = value;
  }
  return obj;
}
var obj = _defineProperty({}, 'name', 'JavaScript');
````

类似上面的帮助函数 _defineProperty 可能会重复出现在一些模块里，导致编译后的代码体积变大。Babel 为了解决这个问题，提供了单独的包 babel-runtime 供编译模块复用工具函数。

启用插件 babel-plugin-transform-runtime 后，Babel 就会使用 babel-runtime 下的工具函数，转译代码如下：

````
'use strict';
// 之前的 _defineProperty 函数已经作为公共模块 `babel-runtime/helpers/defineProperty` 使用
var _defineProperty2 = require('babel-runtime/helpers/defineProperty');
var _defineProperty3 = _interopRequireDefault(_defineProperty2);
function _interopRequireDefault(obj) { return obj && obj.__esModule ? obj : { default: obj }; }
var obj = (0, _defineProperty3.default)({}, 'name', 'JavaScript');
````

除此之外，babel 还为源代码的非实例方法（Object.assign，实例方法是类似这样的 "foobar".includes("foo")）和 babel-runtime/helps 下的工具函数自动引用了 polyfill。这样可以避免污染全局命名空间，非常适合于 JavaScript 库和工具包的实现。例如 const obj = {}, Object.assign(obj, { age: 30 }); 转译后的代码如下所示：

````
'use strict';
// 使用了 core-js 提供的 assign
var _assign = require('babel-runtime/core-js/object/assign');
var _assign2 = _interopRequireDefault(_assign);
function _interopRequireDefault(obj) { return obj && obj.__esModule ? obj : { default: obj }; }
var obj = {};
(0, _assign2.default)(obj, {
  age: 30
});
````

思考：babel-runtime 为什么适合 JavaScript 库和工具包的实现？

- 避免 babel 编译的工具函数在每个模块里重复出现，减小库和工具包的体积；

- 在没有使用 babel-runtime 之前，库和工具包一般不会直接引入 polyfill。否则像 Promise 这样的全局对象会污染全局命名空间，这就要求库的使用者自己提供 polyfill。这些 polyfill 一般在库和工具的使用说明中会提到，比如很多库都会有要求提供 es5 的 polyfill。在使用 `@babel/runtime` 后，库和工具只要在 package.json 中增加依赖 babel-runtime，交给 babel-runtime 去引入 polyfill 就行了；

总结：

- 具体项目还是需要使用 babel-polyfill，只使用 babel-runtime 的话，实例方法不能正常工作（例如 "foobar".includes("foo")）；

- JavaScript 库和工具可以使用 babel-runtime，在实际项目中使用这些库和工具，需要该项目本身提供 polyfill；


## 参考网址

- https://survivejs.com/webpack/optimizing/minifying/

- http://2ality.com/2015/12/babel6-loose-mode.html