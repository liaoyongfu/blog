# webpack学习06——怎么写loader

一个loader是一个node模块，导出一个函数。这个函数当资源被转换时会执行，这个loader有一个入参：待转换资源名称。可以在loader中通过`this`访问上下文。

一个同步的loader可以仅仅返回一个值。在其他情况下，loader可以通过`this.callback(err, values)`返回多个值。

一个loader被期望返回一个或两个值，第一个值返回字符串或buffer类型的javascript代码，第二个返回sourceMap。

在复杂的情况下，当多个loader链接时，仅仅只要最后一个loader返回资源文件，仅仅第一个loader期望返回一个或两个值（javascript代码或buffer）。

Example:

	module.exports = function(source){
		return source;
	};

//

	// Identity loader with SourceMap support
	module.exports = function(source, map) {
	  this.callback(null, source, map);
	};

## 指南

（按优先顺序排序，第一个应该得到最高优先级）

### 只做一个单一的任务

loader可以被链接，他们不应该转换为javascript代码，如果他们不需要的话。
例如：从模板中通过查询参数渲染html，我会编写一个从源代码中编译的loader，执行他并返回一个包含包含html的字符串，这是不好的。而是我应该编写装载程序在这个用例中的每一个任务，并将它们全部应用（流水线）：

- jade-loader: Convert template to a module that exports a function.
- apply-loader: Takes a function exporting module and returns raw result by applying  	query parameters.
- html-loader: Takes HTML and exports a string exporting module.

### generate modules that are modular

加载程序生成模块应尊重相同的设计原则，如正常模块。例子：这是一个糟糕的设计：（不模块化，全局状态，…）

	require("any-template-language-loader!./xyz.atl");
	
	var html = anyTemplateLanguage.render("xyz");

### 标志本身缓存如果可能的话

大多数装载机是可缓存的，所以他们应该标志本身作为缓存。只要在loader中调用`cacheable`。

	//Cacheable identity loader
	module.exports = function(source){
		this.cacheable();
		return source();
	};

### not keep state between runs and modules

一个加载程序应该独立于编译的其他模块（由装载程序发布的这些模块的期望）。一个程序应该独立于以前的编译的模块。

### 依赖

如果loader需要依赖第三方资源（如从系统中读取文件），他们必须要写清楚，此信息用于无效的缓存装载机和编译在观看模式。

	// Loader adding a header
	var path = require("path");
	module.exports = function(source) {
	    this.cacheable();
	    var callback = this.async();
	    var headerPath = path.resolve("header.js");
	    this.addDependency(headerPath);
	    fs.readFile(headerPath, "utf-8", function(err, header) {
	        if(err) return callback(err);
	        callback(null, header + "\n" + source);
	    });
	};


### 解决依赖关系

一些语言有自己的解决依赖图式，例如css的`@import`和`url(...)`。这些必须被模块系统解决。

有两个方法可以做到：

- 把他们转换成`require`；
- 使用`this.resolve`解析路径；

例子1：css-loader把依赖转换成require其他样式文件。
例子2：less-loader不转换为require，因为因为所有的less文件需要编译一次跟踪变量和混合，因此，less-loader扩展less编译器一个自定义的路径解决方法，该自定义逻辑使用this.resolve解决模块的系统配置文件（走样，自定义模块目录，等）。
If the language only accept relative urls (like css: url(file) always means ./file), there is the ~-convention to specify references to modules:

	url(file) -> require("./file")
	url(~module) -> require("module")

### 抽离公共代码

don’t generate much code that is common in every module processed by that loader. Create a (runtime) file in the loader and generate a require to that common code.

### 不应嵌入绝对路径

don’t put absolute paths in to the module code. They break hashing when the root for the project is moved. There is a method `stringifyRequest` in loader-utils which converts an absolute path to an relative one.

Example:

	var loaderUtils = require("loader-utils");
	return "var runtime = require(" +
	  loaderUtils.stringifyRequest(this, "!" + require.resolve("module/runtime")) +
	  ");";

### use a library as peerDependencies when they wrap it

using a peerDependency allows the application developer to specify the exact version in package.json if desired. The dependency should be relatively open to allow updating the library without needing to publish a new loader version.

	"peerDependencies": {
	    "library": "^1.3.5"
	}

### programmable objects as query-option

there are situations where your loader requires programmable objects with functions which cannot stringified as query-string. The less-loader, for example, provides the possibility to specify LESS-plugins. In these cases, a loader is allowed to extend webpack’s options-object to retrieve that specific option. In order to avoid name collisions, however, it is important that the option is namespaced under the loader’s camelCased npm-name.