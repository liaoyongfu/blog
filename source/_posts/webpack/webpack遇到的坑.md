# webpack的坑
---

1. 引用jQuery插件时会报"jQuery is not defined"，解决方法：
	
	**(1). Prefer unminified CommonJS/AMD over dist**

	Most modules link the dist version in the main field of their package.json. While this is useful for most developers, for webpack it is better to alias the src version because this way webpack is able to optimize dependencies better (e.g. when using the DedupePlugin).

		// webpack.config.js
		module.exports = {
		    ...
		    resolve: {
		        alias: {
		            jquery: "jquery/src/jquery"
		        }
		    }
		};

		However, in most cases the dist version works just fine as well.

	**(2). Use the ProvidePlugin to inject implicit globals**

	Most legacy modules rely on the presence of specific globals, like jQuery plugins do on $ or jQuery. In this scenario you can configure webpack, to prepend var $ = require("jquery") everytime it encounters the global $ identifier.

		var webpack = require("webpack");
		    ...
		    plugins: [
		        new webpack.ProvidePlugin({
		            $: "jquery",
		            jQuery: "jquery"
		        })
		    ]
	**(3). Use the imports-loader to configure this**

	Some legacy modules rely on this being the window object. This becomes a problem when the module is executed in a CommonJS context where this equals module.exports. In this case you can override this with the imports-loader.

	Run `npm i imports-loader --save-dev` and then

		module: {
		    loaders: [
		        {
		            test: /[\/\\]node_modules[\/\\]some-module[\/\\]index\.js$/,
		            loader: "imports?this=>window"
		        }
		    ]
		}

	The imports-loader can also be used to manually inject variables of all kinds. But most of the time the ProvidePlugin is more useful when it comes to implicit globals.

	**(4). Use the imports-loader to disable AMD**

	There are modules that support different module styles, like AMD, CommonJS and legacy. However, most of the time they first check for define and then use some quirky code to export properties. In these cases, it could help to force the CommonJS path by setting define = false.

		module: {
		    loaders: [
		        {
		            test: /[\/\\]node_modules[\/\\]some-module[\/\\]index\.js$/,
		            loader: "imports?define=>false"
		        }
		    ]
		}

	**(5). Use the script-loader to globally import scripts**

	If you don't care about global variables and just want legacy scripts to work, you can also use the script-loader. It executes the module in a global context, just as if you had included them via the `<script>` tag.

	**(6). Use noParse to include large dists**

	When there is no AMD/CommonJS version of the module and you want to include the dist, you can flag this module as noParse. Then webpack will just include the module without parsing it, which can be used to improve the build time. This means that any feature requiring the AST, like the ProvidePlugin, will not work.

		module: {
		    noParse: [
		        /[\/\\]node_modules[\/\\]angular[\/\\]angular\.js$/
		    ]
		}

2. 常见的loader
	
		{
                test: /\.js/,
                loader: "babel-loader",
                query: {
                    "presets": ["es2015", 'stage-0'],
                    plugins: []
                },
                exclude: /(node_modules)/
            },
            {
                test: /\.css$/,
				//注意：此处不能有autoprefix-loader
                loader: ExtractText.extract('style-loader', 'css-loader')
            },
            {
                test: /\.(png|gif|jpg|jpeg)$/,
                loader: "url?name=img/[hash:8].[ext]"
            },
            {
                test: /\.woff(\?v=\d+\.\d+\.\d+)?$/,
                loader: 'url?name=font/[name].[ext]&limit=10000&minetype=application/font-woff'
            },
            {
                test: /\.woff2(\?v=\d+\.\d+\.\d+)?$/,
                loader: 'url?name=font/[name].[ext]&limit=10&minetype=application/font-woff'
            },
            {
                test: /\.ttf(\?v=\d+\.\d+\.\d+)?$/,
                loader: 'url?name=font/[name].[ext]&limit=10&minetype=application/octet-stream'
            },
            {
                test: /\.eot(\?v=\d+\.\d+\.\d+)?$/,
                loader: 'file'
            },
            {
                test: /\.svg(\?v=\d+\.\d+\.\d+)?$/,
                loader: 'url?limit=10&minetype=image/svg+xml'
            }

3. 样式的loader
	
		(1)style-loader|css-loader is the way to do it just with css
		(2)style-loader|css-loader|postcss-loader is the way to post-process css
		(3)style-loader|css-loader|less-loader is the way to do it if you want to use less
		(4)style-loader|css-loader|postcss-loader|less-loader is the way to post-process the compiled less (css)

4. ES6引用`art-template`,报错：`Module not found: Error: Cannot resolve module 'fs'`，解决方法：
	
		//webpack.config.js
		module.exports={
			node: {
				fs: "empty"
			}
		};

	[参考链接](https://github.com/pugjs/pug-loader/issues/8)

5. webpack开发时打包第三方库都比较大，可以通过配置`alias`指向压缩版本：

		resolve: {
	        alias: {
	            modernizr$: path.resolve(__dirname, "./.modernizrrc"),
	            bootstrap: path.join(__dirname, "./node_modules/bootstrap/dist/js/bootstrap.min.js"),
	            bootstrapCss: path.join(__dirname, "./node_modules/bootstrap/dist/css/bootstrap.min.css"),
	            fontAwesomeCss: path.join(__dirname, "./node_modules/font-awesome/css/font-awesome.min.css")
	        }
   		 }

6. 引用第三方插件如：`ulynlist`，需要配置别名：

		alias: {
            'ulynlist.table': path.join(__dirname, './src/sslib/ulynlist/ulynlist.table.js'),
            'ulynlist.pager': path.join(__dirname, './src/sslib/ulynlist/ulynlist.pager.js'),
            artTemplate: path.join(__dirname, './node_modules/art-template')
        }

7. import样式文件页面会有闪烁现象，这是可以通过`extract-text-webpack-plugin`抽取样式文件，就不会有这个问题了
8. 使用ES6 + webpack + angular教程[参考链接](http://angular-tips.com/blog/2015/06/using-angular-1-dot-x-with-es6-and-webpack/)
9. 合并jquery和第三方插件时，外面是读取不到`$`和`jQuery`的，所以我们可以通过`expose-loader`把`jQuery`对象导出到全局:

	You can either do this when you require it:

		require("expose?$!jquery");

	or you can do this in your config:

		loaders: [
		    { test: require.resolve('jquery'), loader: 'expose?jQuery!expose?$' }
		]

	相同的道理，如果插件里有this，则我们可以通过`imports-loader`把this当成window处理：

		{ 
			test: require.resolve('respond.js'), 
			loader: 'imports?this=>window' 
		}

10. 使用第三方插件，如果其没有判断commonjs这一层，则我们可以配合exports-loader和imports-loader使用，如eos3还有eos服务，eos3需要导出eos对象，eos服务的js需要导入eos这个对象：

		import 'exports?eos!./lib/eos3/eos3';
		//这里define设为false，防止组件判断为AMD模块
		import 'imports?define=>false,this=>window!./lib/auth/dmService';

11. webpack-dev-server默认是localhost访问，不能通过ip访问，我们可以配置如下：

	webpack-dev-server --host 0.0.0.0