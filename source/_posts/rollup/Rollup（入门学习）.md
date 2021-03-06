---
title: Rollup（入门学习）
date: 2018-08-01
tag: rollup
---

## 概述

Rollup 是一个 JavaScript 模块打包器，Rollup 对代码模块使用新的标准化格式，这些标准都包含在 JavaScript 的 ES6 版本中，而不是以前的特殊解决方案，如 CommonJS 和 AMD。ES6 模块可以使你自由、无缝地使用你最喜爱的 library 中那些最有用独立函数，而你的项目不必携带其他未使用的代码。ES6 模块最终还是要由浏览器原生实现，但当前 Rollup 可以使你提前体验。

<!-- more -->

## 安装

可以使用全局安装`rollup`：

````
npm install --global rollup
````

## 摇树优化（Tree-Shaking）

在使用CommonJS时，必须导入完整的库对象：

````
var utils = require('utils');
var query = 'Rollup';

utils.ajax('...').then(...);
````

但在使用ES6模块时，无需导入整个utils对象，我们只要导入我们所需的ajax即可：

````
import { ajax } from 'utils';
var query = 'Rollup';

utils.ajax('...').then(...);
````

因为 Rollup 只引入最基本最精简代码，所以可以生成轻量、快速，以及低复杂度的 library 和应用程序。因为这种基于显式的 import 和 export 语句的方式，它远比「在编译后的输出代码中，简单地运行自动 minifier 检测未使用的变量」更有效。

> js各种模块分类：
> - CommonJS：一个单独的文件就是一个模块，每个模块都是一个单独的作用域，在该模块内部定义的变量，无法被其他模块读取，除非定义为global对象的属性。模块只有一个出口，exports对象。
> - CommonJS2：commonjs 规范只定义了exports，而 module.exports是nodejs对commonjs的实现，实现往往会在满足规范前提下作些扩展，我们这里把这种实现称为了commonjs2，所以CommonJS2只是添加了module.exports而已。
> - AMD：需要用到对应的库函数，比如RequireJS。多个js文件可能有依赖关系，通过define定义模块，require加载模块
> - CMD：推崇一个模块一个文件，使用define定义模块，一个文件一个模块，所以经常就用文件名作为模块id；推崇依赖就近，所以一般不在define的参数中写依赖，在factory中写
> - ES：跟CommonJS类似，通过export命令显式指定输出的代码，再通过import命令输入，es module输出的是值的引用，而commonJS模块输出的是值的拷贝，不存在动态更新
> - UMD：在执行UMD规范时，会优先判断是当前环境是否支持AMD环境，然后再检验是否支持CommonJS环境，否则认为当前环境为浏览器环境
> ````
> (function (root, factory) {
>     if (typeof define === 'function' && define.amd) {
>         // AMD
>         define(['jquery'], factory);
>     } else if (typeof exports === 'object') {
>         // Node, CommonJS-like
>         module.exports = factory(require('jquery'));
>     } else {
>         // Browser globals (root is window)
>         root.returnExports = factory(root.jQuery);
>     }
> }(this, function ($) {
>     //    methods
>     function myFunc(){};
> 
>     //    exposed public method
>     return myFunc;
> }));
> ````
> 
> 参考网址：
> - https://www.cnblogs.com/chengdabelief/p/6547847.html
> - https://www.cnblogs.com/cag2050/p/7562550.html

## 兼容性

### 导入CommJS

Rollup 可以通过[rollup-plugin-commonjs](https://github.com/rollup/rollup-plugin-commonjs)导入已存在的 CommonJS 模块，它会将 CommonJS 转换成 ES2015 模块的。

### 发布 ES6 模块

为了确保你的 ES6 模块可以直接与「运行在 CommonJS（例如 Node.js 和 webpack）中的工具(tool)」使用，你可以使用 Rollup 编译为 UMD 或 CommonJS 格式，然后在 package.json 文件的 main 属性中指向当前编译的版本。如果你的 package.json 也具有 module 字段，像 Rollup 和 webpack 2 这样的 ES6 感知工具(ES6-aware tools)将会直接[导入 ES6 模块版本](https://github.com/rollup/rollup/wiki/pkg.module)。

> Rollup可以通过`pkg.module`字段感知ES模块：
> 
> ````
> {
>   "name": "my-package",
>   "version": "0.1.0",
>   "main": "dist/my-package.umd.js",
>   "module": "dist/my-package.esm.js"
> }
> ````
> 
> 当打包工具遇到我们的模块时：
> 
> - 如果它已经支持 pkg.module 字段则会优先使用 ES6 模块规范的版本，这样可以启用 Tree Shaking 机制。
> - 如果它还不识别 pkg.module 字段则会使用我们已经编译成 CommonJS 规范的版本，也不会阻碍打包流程。
>
> 参考网址：https://loveky.github.io/2018/02/26/tree-shaking-and-pkg.module/

## 常见问题

### Rollup 是用来构建库还是应用程序？

Rollup 已被许多主流的 JavaScript 库使用，也可用于构建绝大多数应用程序。但是 Rollup 还不支持一些特定的高级功能，尤其是用在构建一些应用程序的时候，特别是代码拆分和运行时态的动态导入 dynamic imports at runtime. 如果你的项目中更需要这些功能，那使用 Webpack可能更符合你的需求。

> 针对app级别的应该使用Webpack，针对js库级别的应用应该使用Rollup

### 给npm包作者的建议：请使用pkg.module!

在很长一段时间，使用JavaScript库就意味着你要冒很大的风险，这是因为你和库作者使用的模块系统可能不一样，所以这就需要你和库作者在模块系统的选择上必须达成一致意见。举个例子，假设你使用的是Browserify打包工具，但是库作者更喜欢AMD模块系统，所以在你构建之前，你必须把库作者的模块系统替换成自己项目所使用的模块系统。虽说Universal Module Definition(UMD)模块系统可以稍稍解决上述问题，不过，由于UMD模块系统不强制要求你使用什么的模块系统，（这也就意味着），你永远不知道你下一步使用的模块系统是哪一种。

ES2015模块颠覆了这一切，这是因为import以及export命令已经变成JavaScript语言的一部分啦。在不久的将来，模块系统的选择将会变得更加明确，不再模棱两可啦，而且所有的JavaScript代码都能够无缝对接。不幸的是，由于浏览器（大多数）以及Node还不支持import以及export命令，所以我们仍然需要对js文件使用UMD模块系统（如果你构建的文件只是用于Node，或许可以考虑CommonJS）。

通过给你项目的package.json文件增加（针对模块）入口"module": "dist/my-library.es.js"配置，让你的项目同时支持UMD以及ES2015模块系统（的想法）变成了可能。这一点很重要，这是因为，Webpack和Rollup都会使用pkg.module来尽可能构建出高效代码。在某些情况下，Webpack以及Rollup甚至都能利用tree-shake特性来剔除项目中未使用的代码。

## 创建第一个bundle

````
// src/main.js
import foo from './foo.js';
export default function () {
  console.log(foo);
}

// foo.js
export default 'hello world!';
````

现在可以创建bundle了：

````
rollup src/main.js -f cjs
````

-f 选项（--output.format 的缩写）指定了所创建 bundle 的类型——这里是 CommonJS（在 Node.js 中运行）。由于没有指定输出文件，所以会直接打印在 stdout 中：

````
'use strict';

var foo = 'hello world!';

var main = function () {
  console.log(foo);
};

module.exports = main;
````

也可以像下面一样将 bundle 保存为文件：

````
rollup src/main.js -o bundle.js -f cjs
````

（你也可以用 rollup src/main.js -f cjs > bundle.js，但是我们之后会提到，这种方法在生成 sourcemap 时灵活性不高。）

试着运行下面的代码：

````
node
> var myBundle = require('./bundle.js');
> myBundle();
'hello world!'
````

恭喜，你已经用 Rollup 完成了第一个 bundle。

## 使用配置文件

类似webpack.config.js，rollup使用`rollup.config.js`：

````
// rollup.config.js
export default {
  input: 'src/main.js',
  output: {
    file: 'bundle.js',
    format: 'cjs'
  }
};
````

我们用 --config 或 -c 来使用配置文件：

````
rm bundle.js # so we can check the command works!
rollup -c
````

同样的命令行选项将会覆盖配置文件中的选项：

````
rollup -c -o bundle-2.js # `-o` is short for `--output.file`
````

（注意 Rollup 本身会处理配置文件，所以可以使用 export default 语法——代码不会经过 Babel 等类似工具编译，所以只能使用所用 Node.js 版本支持的 ES2015 语法。）

如果愿意的话，也可以指定与默认 rollup.config.js 文件不同的配置文件：

````
rollup --config rollup.config.dev.js
rollup --config rollup.config.prod.js
````

配置项预览：

````
// rollup.config.js
export default {
  // 核心选项
  input,     // 必须
  external,
  plugins,

  // 额外选项
  onwarn,

  // danger zone
  acorn,
  context,
  moduleContext,
  legacy

  output: {  // 必须 (如果要输出多个，可以是一个数组)
    // 核心选项
    file,    // 必须
    format,  // 必须
    name,
    globals,

    // 额外选项
    paths,
    banner,
    footer,
    intro,
    outro,
    sourcemap,
    sourcemapFile,
    interop,

    // 高危选项
    exports,
    amd,
    indent
    strict
  },
};
````

你必须使用配置文件才能执行以下操作：

- 把一个项目打包，然后输出多个文件
- 使用Rollup插件, 例如 rollup-plugin-node-resolve 和 rollup-plugin-commonjs 。这两个插件可以让你加载Node.js里面的CommonJS模块

如果你想使用Rollup的配置文件，记得在命令行里加上--config或者-c @@2

## 使用插件

类似webpack插件，比如要支持读取JSON格式文件，增加插件支持：

````
// rollup.config.js
import json from 'rollup-plugin-json';

export default {
  input: 'src/main.js',
  output: {
    file: 'bundle.js',
    format: 'cjs'
  },
  plugins: [ json() ]
};
````

npm run build 执行 Rollup。结果如下：

````
'use strict';

var version = "1.0.0";

var main = function () {
  console.log('version ' + version);
};

module.exports = main;
````

## 使用命令行

命令行的参数：

````
-i, --input                 要打包的文件（必须）
-o, --output.file           输出的文件 (如果没有这个参数，则直接输出到控制台)
-f, --output.format [es]    输出的文件类型 (amd, cjs, es, iife, umd)
-e, --external              将模块ID的逗号分隔列表排除
-g, --globals               以`module ID:Global` 键值对的形式，用逗号分隔开 
                              任何定义在这里模块ID定义添加到外部依赖
-n, --name                  生成UMD模块的名字
-m, --sourcemap             生成 sourcemap (`-m inline` for inline map)
--amd.id                    AMD模块的ID，默认是个匿名函数
--amd.define                使用Function来代替`define`
--no-strict                 在生成的包中省略`"use strict";`
--no-conflict               对于UMD模块来说，给全局变量生成一个无冲突的方法
--intro                     在打包好的文件的块的内部(wrapper内部)的最顶部插入一段内容
--outro                     在打包好的文件的块的内部(wrapper内部)的最底部插入一段内容
--banner                    在打包好的文件的块的外部(wrapper外部)的最顶部插入一段内容
--footer                    在打包好的文件的块的外部(wrapper外部)的最底部插入一段内容
--interop                   包含公共的模块（这个选项是默认添加的）
````

此外，还可以使用以下参数：
````
-h/--help
````
打印帮助文档。
````
-v/--version
````
打印已安装的Rollup版本号。
````
-w/--watch
````
监听源文件是否有改动，如果有改动，重新打包
````
--silent
````
不要将警告打印到控制台。

## Javascript API

Rollup 提供 JavaScript 接口那样可以通过 Node.js 来使用。你可以很少使用，而且很可能使用命令行接口，除非你想扩展 Rollup 本身，或者用于一些难懂的任务，例如用代码把文件束生成出来。

### rollup.rollup

The rollup.rollup 函数返回一个 Promise，它解析了一个 bundle 对象，此对象带有不同的属性及方法，如下：

````
const rollup = require('rollup');

// see below for details on the options
const inputOptions = {...};
const outputOptions = {...};

async function build() {
  // create a bundle
  const bundle = await rollup.rollup(inputOptions);

  console.log(bundle.imports); // an array of external dependencies
  console.log(bundle.exports); // an array of names exported by the entry point
  console.log(bundle.modules); // an array of module objects

  // generate code and a sourcemap
  const { code, map } = await bundle.generate(outputOptions);

  // or write the bundle to disk
  await bundle.write(outputOptions);
}

build();
````

### 输入参数

inputOptions 对象包含下列属性 (查看big list of options 以获得这些参数更详细的资料):

````
const inputOptions = {
  // 核心参数
  input, // 唯一必填参数
  external,
  plugins,

  // 高级参数
  onwarn,
  cache,

  // 危险参数
  acorn,
  context,
  moduleContext,
  legacy
};
````

### 输出参数

outputOptions 对象包括下列属性 (查看 big list of options 以获得这些参数更详细的资料):

````
const outputOptions = {
  // 核心参数
  file,   // 若有bundle.write，必填
  format, // 必填
  name,
  globals,

  // 高级参数
  paths,
  banner,
  footer,
  intro,
  outro,
  sourcemap,
  sourcemapFile,
  interop,

  // 危险区域
  exports,
  amd,
  indent
  strict
};
````

### rollup.watch

Rollup 也提供了 rollup.watch 函数，当它检测到磁盘上单个模块已经改变，它会重新构建你的文件束。 当你通过命令行运行 Rollup，并带上 --watch 标记时，此函数会被内部使用。

````
const rollup = require('rollup');

const watchOptions = {...};
const watcher = rollup.watch(watchOptions);

watcher.on('event', event => {
  // event.code 会是下面其中一个：
  //   START        — 监听器正在启动（重启）
  //   BUNDLE_START — 构建单个文件束
  //   BUNDLE_END   — 完成文件束构建
  //   END          — 完成所有文件束构建
  //   ERROR        — 构建时遇到错误
  //   FATAL        — 遇到无可修复的错误
});

// 停止监听
watcher.close();
````

### 监听参数

watchOptions 参数是一个你会从一个配置文件中导出的配置 (或一个配置数据)。

````
const watchOptions = {
  ...inputOptions,
  output: [outputOptions],
  watch: {
    chokidar,
    include,
    exclude
  }
};
````

查看以上文档知道更多 inputOptions 和 outputOptions 的细节, 或查询 [big list of options](https://www.rollupjs.com/guide/zh#big-list-of-options) 关 chokidar, include 和 exclude 的资料。

## Rollup与其他工具的集成

### npm依赖包

Rollup并不像webpack那样，它不知道如何打破常规去处理这些依赖，因此我们需要一些配置：

````
npm install --save-dev rollup-plugin-node-resolve
````

````
// rollup.config.js
import resolve from 'rollup-plugin-node-resolve';

export default {
  input: 'src/main.js',
  output: {
    file: 'bundle.js',
    format: 'cjs'
  },
  plugins: [ resolve() ]
};
````

### Babel

````
npm i -D rollup-plugin-babel
````

````
// rollup.config.js
import resolve from 'rollup-plugin-node-resolve';
import babel from 'rollup-plugin-babel';

export default {
  input: 'src/main.js',
  output: {
    file: 'bundle.js',
    format: 'cjs'
  },
  plugins: [
    resolve(),
    babel({
      exclude: 'node_modules/**' // 只编译我们的源代码
    })
  ]
};
````

在Babel实际编译代码之前，需要进行配置。 创建一个新文件src/.babelrc：

````
{
  "presets": [
    ["latest", {
      "es2015": {
        "modules": false
      }
    }]
  ],
  "plugins": ["external-helpers"]
}
````

这个设置有一些不寻常的地方。首先，我们设置"modules": false，否则 Babel 会在 Rollup 有机会做处理之前，将我们的模块转成 CommonJS，导致 Rollup 的一些处理失败。

第三，我们将.babelrc文件放在src中，而不是根目录下。 这允许我们对于不同的任务有不同的.babelrc配置，比如像测试，如果我们以后需要的话 - 通常为单独的任务单独配置会更好。

## ES模块语法

### 导入

导入的值不能重新分配，尽管导入的对象和数组可以被修改（导出模块，以及任何其他的导入，都将受到该修改的影响）。在这种情况下，它们的行为与const声明类似。

#### 命名导入

````
import { something } from './module.js';
````

````
import { something as somethingElse } from './module.js';
````

#### 命名空间导入

````
import * as module from './module.js'
````

#### 默认导入

````
import something from './module.js';
````

#### 空导入

````
import './module.js';
````

这对于polyfills是有用的，或者当导入的代码的主要目的是与原型有关的时候。

### 导出

#### 命名导出

导出以前声明的值：

````
var something = true;
export { something };
````

在导出时重命名：

````
export { something as somethingElse };
````

声明后立即导出：

````
// 这可以与 `var`, `let`, `const`, `class`, and `function` 配合使用
export var something = true;
````

#### 默认导出

````
export default something;
````

仅当源模块只有一个导出时，才建议使用此做法。

将默认和命名导出组合在同一模块中是不好的做法，尽管它是规范允许的。

### 绑定是如何工作的

ES模块导出实时绑定，而不是值，所以值可以在最初根据这个示例导入后更改：

````
// incrementer.js
export let count = 0;

export function increment() {
  count += 1;
}
````

````
// main.js
import { count, increment } from './incrementer.js';

console.log(count); // 0
increment();
console.log(count); // 1

count += 1; // Error — 只有 incrementer.js 可以改变这个值。
````

## 大选项列表（配置项）

### 核心功能

#### 输入(input -i/--input)

String 这个包的入口点 (例如：你的 main.js 或者 app.js 或者 index.js)

#### 文件(file -o/--output.file)

String 要写入的文件。也可用于生成 sourcemaps，如果适用

#### 格式(format -f/--output.format)

String 生成包的格式。 下列之一:

- amd – 异步模块定义，用于像RequireJS这样的模块加载器
- cjs – CommonJS，适用于 Node 和 Browserify/Webpack
- es – 将软件包保存为ES模块文件
- iife – 一个自动执行的功能，适合作为<script>标签。（如果要为应用程序创建一个捆绑包，您可能想要使用它，因为它会使文件大小变小。）
- umd – 通用模块定义，以amd，cjs 和 iife 为一体

#### 生成包名称(name -n/--name)

String 变量名，代表你的 iife/umd 包，同一页上的其他脚本可以访问它。

````
// rollup.config.js
export default {
  ...,
  output: {
    file: 'bundle.js',
    format: 'iife',
    name: 'MyBundle'
  }
};

// -> var MyBundle = (function () {...
````

#### 插件(plugins)

插件对象 数组 Array (或一个插件对象) – 有关详细信息请参阅 插件入门。记住要调用导入的插件函数(即 commonjs(), 而不是 commonjs).

````
// rollup.config.js
import resolve from 'rollup-plugin-node-resolve';
import commonjs from 'rollup-plugin-commonjs';

export default {
  entry: 'main.js',
  plugins: [
    resolve(),
    commonjs()
  ]
};
````

#### 外链(external -e/--external)

两者任一 Function 需要一个 id 并返回 true（外部引用）或 false（不是外部的引用）， 或者 Array 应该保留在bundle的外部引用的模块ID。ID应该是：

- 外部依赖的名称
- 一个已被找到路径的ID（像文件的绝对路径）

````
// rollup.config.js
import path from 'path';

export default {
  ...,
  external: [
    'some-externally-required-library',
    path.resolve( './src/some-local-file-that-should-not-be-bundled.js' )
  ]
};
````

当作为命令行参数给出时，它应该是以逗号分隔的ID列表：

````
rollup -i src/main.js ... -e foo,bar,baz
````

#### 全局模块(globals -g/--globals)

Object 形式的 id: name 键值对，用于umd/iife包。例如：在这样的情况下...

````
// rollup.config.js
export default {
  ...,
  format: 'iife',
  name: 'MyBundle',
  globals: {
    jquery: '$'
  }
};

/*
var MyBundle = (function ($) {
  // 代码到这里
}(window.jQuery));
*/.
````

### 高级功能(Advanced functionality)

#### 路径(paths)

Function，它获取一个ID并返回一个路径，或者id：path对的Object。在提供的位置，这些路径将被用于生成的包而不是模块ID，从而允许您（例如）从CDN加载依赖关系：

````
// app.js
import { selectAll } from 'd3';
selectAll('p').style('color', 'purple');
// ...

// rollup.config.js
export default {
  input: 'app.js',
  external: ['d3'],
  output: {
    file: 'bundle.js',
    format: 'amd',
    paths: {
      d3: 'https://d3js.org/d3.v4.min'
    }
  }
};

// bundle.js
define(['https://d3js.org/d3.v4.min'], function (d3) {

  d3.selectAll('p').style('color', 'purple');
  // ...

});
````

#### banner/footer

String 字符串以 前置/追加 到文件束(bundle)。(注意:“banner”和“footer”选项不会破坏sourcemaps)

````
// rollup.config.js
export default {
  ...,
  banner: '/* my-library version ' + version + ' */',
  footer: '/* follow me on Twitter! @rich_harris */'
};
````

#### intro/outro

````
export default {
  ...,
  intro: 'var ENVIRONMENT = "production";'
};
````

### 缓存(cache)

Object 以前生成的包。使用它来加速后续的构建——Rollup只会重新分析已经更改的模块。

#### onwarn

Function 将拦截警告信息。如果没有提供，警告将被复制并打印到控制台。

警告是至少有一个code 和 message属性的对象，这意味着您可以控制如何处理不同类型的警告：

````
onwarn (warning) {
  // 跳过某些警告
  if (warning.code === 'UNUSED_EXTERNAL_IMPORT') return;

  // 抛出异常
  if (warning.code === 'NON_EXISTENT_EXPORT') throw new Error(warning.message);

  // 控制台打印一切警告
  console.warn(warning.message);
}
````

#### sourcemap -m/--sourcemap

如果 true，将创建一个单独的sourcemap文件。如果 inline，sourcemap将作为数据URI附加到生成的output文件中。

#### sourcemapFile

String生成的包的位置。如果这是一个绝对路径，sourcemap中的所有源代码路径都将相对于它。 map.file属性是sourcemapFile的基本名称(basename)，因为sourcemap的位置被假定为与bundle相邻

如果指定 output，sourcemapFile 不是必需的，在这种情况下，将通过给bundle输出文件添加 “.map” 后缀来推断输出文件名。

#### interop

Boolean 是否添加'interop块'。默认情况下（interop：true），为了安全起见，如果需要区分默认和命名导出，则Rollup会将任何外部依赖项“default”导出到一个单独的变量。这通常只适用于您的外部依赖关系（例如与Babel）（如果您确定不需要它），则可以使用“interop：false”来节省几个字节。

### 危险区域(Danger zone)

你可能不需要使用这些选项，除非你知道你在做什么!

#### treeshake

是否应用tree-shaking。建议您省略此选项（默认为treeshake：true），除非您发现由tree-shaking算法引起的bug，在这种情况下，请使用“treeshake：false”，一旦您提交了问题！

#### acorn

任何应该传递给Acorn的选项，例如allowReserved：true。

#### context

默认情况下，模块的上下文 - 即顶级的this的值为undefined。在极少数情况下，您可能需要将其更改为其他内容，如 'window'。

#### moduleContext

和options.context一样，但是每个模块可以是id: context对的对象，也可以是id => context函数。

#### legacy

为了增加对诸如IE8之类的旧版环境的支持，通过剥离更多可能无法正常工作的现代化的代码，其代价是偏离ES6模块环境所需的精确规范。

#### exports

String 使用什么导出模式。默认为auto，它根据entry模块导出的内容猜测你的意图：

- default – 如果你使用 export default ... 仅仅导出一个东西，那适合用这个
- named – 如果你导出多个东西，适合用这个
- none – 如果你不导出任何内容 (例如，你正在构建应用程序，而不是库)，则适合用这个
- default 和 named之间的区别会影响其他人如何使用文件束(bundle)。如果您使用default，则CommonJS用户可以执行此操作，例如

var yourLib = require( 'your-lib' );

使用 named，用户可以这样做：

var yourMethod = require( 'your-lib' ).yourMethod;

有点波折就是如果你使用named导出，但是同时也有一个default导出，用户必须这样做才能使用默认的导出：

var yourMethod = require( 'your-lib' ).yourMethod;
var yourLib = require( 'your-lib' )['default'];
amd --amd.id and --amd.define

Object 可以包含以下属性：

amd.id String 用于 AMD/UMD 软件包的ID：

// rollup.config.js
export default {
  ...,
  format: 'amd',
  amd: {
    id: 'my-bundle'
  }
};

// -> define('my-bundle', ['dependency'], ...
amd.define String 要使用的函数名称，而不是 define:

// rollup.config.js
export default {
  ...,
  format: 'amd',
  amd: {
    define: 'def'
  }
};

// -> def(['dependency'],...

#### indent

String 是要使用的缩进字符串，对于需要缩进代码的格式（amd，iife，umd）。也可以是false（无缩进）或true（默认 - 自动缩进）

// rollup.config.js
export default {
  ...,
  indent: false
};

#### strict

true或false（默认为true） - 是否在生成的非ES6软件包的顶部包含'use strict'pragma。严格来说（geddit？），ES6模块始终都是严格模式，所以你应该没有很好的理由来禁用它。

### Watch options

这些选项仅在运行 Rollup 时使用 --watch 标志或使用 rollup.watch 时生效。

#### watch.chokidar

一个 Boolean 值表示应该使用 chokidar 而不是内置的 fs.watch，或者是一个传递给 chokidar 的选项对象。

如果你希望使用它，你必须单独安装chokidar。

#### watch.include

限制文件监控至某些文件：

// rollup.config.js
export default {
  ...,
  watch: {
    include: 'src/**'
  }
};

#### watch.exclude

防止文件被监控：

// rollup.config.js
export default {
  ...,
  watch: {
    exclude: 'node_modules/**'
  }
};