# webpack学习01——简介
---

webpack是一个模块打包工具，

- webpack特性：
	- 插件：webpack有大量的插件，大多数的功能是使用这个接口的内部插件，这使得webpack很很大的**灵活性**;
	- 性能：webpack使用异步I/O和多级缓存机制，使得webpack有更快的速度，而且编辑速度超快；
	- 加载器(loaders)：webpack通过定义loaders预处理文件，这允许你打包**任何资源**而不仅仅是javascript，你也可以自己写适合自己的加载器;
	- 模块化：webpack支持AMD和CommonJs模块化规范，它能够聪明地分析你的代码，甚至有一个评估引擎来评估简单的表达式，这个允许你**支持大多数现存模块化规范**
	- 代码分割：webpack允许你把代码分割成不同的块`thunk`，块是按需加载的，这个可以减少初始加载所发费的时间。
	- 代码压缩：webpack可以压缩你的代码，它当然也支持通过哈希缓存；
	- 开发工具：WebPACK支持调试简单sourceurls和sourcemaps。还可以通过`development middleware`和`development server`实现**自动刷新**功能;
	- 多个目标：webpack的首要目标是`web`，但是它也支持在WebWorkers、Node.js中使用；
- 加载器：
	- 转换文件；
	- 加载器可以通过管道被链接，最后一个加载器则返回javascript，其他的加载器则可以返回任意的格式代码；
	- 加载器可以是同步或异步的；
	- 加载器在`Node.js`环境中运行，所以你可以做任何你想要的事情；
	- 加载器支持查询参数；
	- 加载器可以在配置中绑定到扩展名或正则表达式的监听；
	- 加载器可以被发布到`npm`上或从`npm`中安装下来；
	- Normal modules can export a loader in addition to the normal main via package.json loader.
	- 加载器可以访问配置；
	- 插件可以向加载器提供更多的特性和功能；
	- 加载器可以调用额外的任意文件；
	- 。。。
- 加载器解决方案

	加载器类似模块，一个加载器导出一个兼容于Node的javascript函数，一般情况下，你通过`npm`管理你的加载器，当然你也可以作为文件在你的项目中使用

	- 引用加载器：加载器一般以`xxx-loader`命名，`xxx`是其名字，例如：`json-loader`；
	- 你可以通过全名`xxx-loader`或者短名`xxx`来使用加载器；
	- 加载器的命名惯例和优先顺序是由`resolveLoader.moduleTemplates`配置决定的；
	- 加载器的命名惯例一些情况下是很有用的，特别是在`require()`里面引用加载器的时候；
	
	加载器的使用情况：

	- 在`require()`语句内；
	
			//使用文件loader.js来转换file.txt文件
			require('./loader!./dir/file.txt');

			//使用jade-loader，如果配置文件中已经有绑定到此文件的加载器，他们仍然会运行的
			require('jade!./template.jade');

			//由less-loader->css-loader->style-loader转换顺序，由于有前缀`!`，所以如果在配置文件中已经
			//有绑定到此文件的加载器，他们将不会运行
			require('!style!css!less!bootstrap/less/bootstrap.less');	

	- 在配置文件中配置；（推荐这种方式）

			{
				module: {
					loaders: [
						{test: /\.jade$/, loader: 'jade'},
						{test: /\.css$/, loader: 'style!css'},
						{test: /\.css$/, loaders: ['style', 'css']}
					]
				}
			}		

	- 在`CLI`命令行中配置；

			webpack --module-bind jade --module-bind 'css=style!css'

- 加载器的查询参数

	可以在加载器后面加上`?`添加多个查询参数，例如：`url-loader?mimetype=image/png`.大多数的加载器支持正常的格式`?key=value&key2=value2`和JSON对象格式`?{key: "value", key2: "value2"}`

	- 在`require()`语句中：

			require('url-loader?mimetype=image/png!./file.png');

	- 在配置中：

			{test: /\.png$/, loader: 'url-loader?mimetype=image/png'}

		或者

			{
				test: /\.png$/,
				loader: 'url-loader',
				query: {
					mimetype: 'image/png'
				}
			}

	- 在CLI中：

			webpack --module-bind "png=url-loader?mimetype=image/png"