---
title: webpack学习——loose模式
date: 2018-5-29 11:54:51
tags: 

  - webpack

  - babel
---

简要介绍Babel6的loose mode。

## 简介

babel的松散模式将ES6代码转换为不遵循ES6语义的ES5代码。

## 两种模式

babel中的许多插件有两种模式：

- 正常模式尽可能地遵循ECMAScript 6的语义。

- 松散模式产生更简单的ES5代码。

通常，建议不要使用松散模式。优点和缺点是：

- 优点：生成的代码可能更快，并且与旧引擎兼容。它也趋于更清洁，更“ES5式”。

- 缺点：当你从ES6转换到ES6时，你可能会遇到问题。这很少是值得冒险的。

### 打开松散模式

[es2015-loose](https://github.com/bkonkle/babel-preset-es2015-loose)预设是标准ES6预设的松散版本。它提供了一个概观如何打开某个插件的松散模式：

```
module.exports = {
  plugins: [
    ···
    [require("babel-plugin-transform-es2015-classes"), {loose: true}],
    require("babel-plugin-transform-es2015-object-super"),
    ···
  ]
};
```

## 示例：松散模式和正常模式输出区别

让我们看看模式的区别如何影响到以下代码的输出：

```
class Point {
    constructor(x, y) {
        this.x = x;
        this.y = y;
    }
    toString() {
        return `(${this.x}, ${this.y})`;
    }
}
```

### 正常模式

正常模式下，类的属性通过`Object.defineProperty`：

```
"use strict";

var _createClass = (function () {
    function defineProperties(target, props) {
        for (var i = 0; i < props.length; i++) {
            var descriptor = props[i];
            descriptor.enumerable = descriptor.enumerable || false;
            descriptor.configurable = true;
            if ("value" in descriptor) descriptor.writable = true;
            Object.defineProperty(target, descriptor.key, descriptor); // (A)
        }
    }
    return function (Constructor, protoProps, staticProps) {
        if (protoProps) defineProperties(Constructor.prototype, protoProps);
        if (staticProps) defineProperties(Constructor, staticProps);
        return Constructor;
    };
})();

function _classCallCheck(instance, Constructor) {
    if (!(instance instanceof Constructor)) {
        throw new TypeError("Cannot call a class as a function");
    }
}

var Point = (function () {
    function Point(x, y) {
        _classCallCheck(this, Point);

        this.x = x;
        this.y = y;
    }

    _createClass(Point, [{
        key: "toString",
        value: function toString() {
            return "(" + this.x + ", " + this.y + ")";
        }
    }]);

    return Point;
})();
```

### 松散模式

松散模式下，通过正常添加方法方式，更像es5写法：

```
"use strict";

function _classCallCheck(instance, Constructor) { ··· }

var Point = (function () {
    function Point(x, y) {
        _classCallCheck(this, Point);

        this.x = x;
        this.y = y;
    }

    Point.prototype.toString = function toString() { // (A)
        return "(" + this.x + ", " + this.y + ")";
    };

    return Point;
})();
```