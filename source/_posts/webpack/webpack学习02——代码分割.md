# webpack学习02——代码分割

## 定义一个代码分割点

### commonjs: `require.ensure`

	require.ensure(dependencies, callback)

example:

	//require.ensure仅仅加载而不执行模块
	require.ensure(["module-a", "module-b"], function(require) {
		var a = require("module-a");
		// ...
	});

### AMD: `require`

	require(dependencies, callback)

example:

	//AMD的require加载并会执行模块
	require(["module-a", "module-b"], function(a, b) {
		// ...
	});

### ES6模块

**webpack1是不支持原生es6模块的，可以通过babal转换**

## 块的内容

所有分隔的文件会成为一个块，这个块是由其依赖递归加进去的。

## 块压缩

如果两个块包含同一个模块，他们会被合并成一个。这可能造成块有相同的父级。
如果一个模块在一个块的所有父级中是可获取的，这个模块将会在这个块中删除。
如果一个块包含另一个块的所有模块，则存储这个块，它实现多个块。

## 块加载

根据配置`target`目标，将将块加载的运行时逻辑添加到包中。例如：`web`目标块通过jsonp加载。只有一个块被加载一次，并行请求被合并成一个。运行时检查加载的块是否完成多个块。

## 块的类型

### 入口块

一个入口块包含了请求时加载的模块。如果这个块包含模块`0`则加载并执行它，如果没有，它等待有请求模块`0`的块。

### 正常块

一个正常的块不会在运行时加载，它仅仅包含一些模块，这个结构依赖于块加载算法，例如：如果目标是jsonp，则这些模块会包含一个jsonp回调函数，这个块当然还包含它负责的一些块id列表。

### 初始化块（非入口）

一个初始化块是一个正常的块，不同的仅仅是压缩工具视它更重要，因为它计算向初始加载时间（像入口块），这个块类型可以结合`CommonsChunkPlugin`发生。

### 分隔app和vendor代码

	var webpack = require("webpack");
	
	module.exports = {
	  entry: {
	    app: "./app.js",
	    vendor: ["jquery", "underscore", ...],
	  },
	  output: {
	    filename: "bundle.js"
	  },
	  plugins: [
	    new webpack.optimize.CommonsChunkPlugin(/* chunkName= */"vendor", /* filename= */"vendor.bundle.js")
	  ]
	};

vendor块将会从移除所有在app块中的模块，使得bundle块仅仅包含你的代码而不包含其依赖。
	
	<script src="vendor.bundle.js"></script>
	<script src="bundle.js"></script>

### 多入口块

#### 加载多入口

通过`CommonsChukPlugin`运行时被移动到公共块，入口文件变成初始化块。只有当初始块可以被加载时，其他入口块才可以被加载。

	var webpack = require("webpack");
	module.exports = {
	    entry: { a: "./a", b: "./b" },
	    output: { filename: "[name].js" },
	    plugins: [ new webpack.optimize.CommonsChunkPlugin("init.js") ]
	}

	<script src="init.js"></script>
	<script src="a.js"></script>
	<script src="b.js"></script>

### 公共块

`CommonsChunkPlugin`可以移动发生在多个入口的模块到一个新的块（公共块），运行时也移动到公共块，这意味着旧的入口块已经变成初始化块。

### 压缩优化插件

- LimitChunkCountPlugin
- MinChunkSizePlugin
- AggressiveMergingPlugin

### 命名块

The require.ensure function accepts an additional 3rd parameter. This must be a string. If two split point pass the same string they use the same chunk.
	
#### require.include

	require.include(request)

require.include is a webpack specific function that adds a module to the current chunk, but doesn’t evaluate it (The statement is removed from the bundle).

Example:

	require.ensure(["./file"], function(require) {
	  require("./file2");
	});
	
	// is equal to
	
	require.ensure([], function(require) {
	  require.include("./file");
	  require("./file2");
	});

require.include can be useful if a module is in multiple child chunks. A require.include in the parent would include the module and the instances of the modules in the child chunks would disappear.