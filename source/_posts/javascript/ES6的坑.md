---
title: ES6的坑
date: 2017-8-2 14:31:15
tags: 
    - js
---

1. IE8下用babel转换会报错：
	
	function _interopRequireDefault(obj) { return obj && obj.__esModule ? obj : { default: obj }; }

	解决方法：
	
		$ npm install --save-dev babel-plugin-transform-es2015-modules-simple-commonjs
	
	配置：
	
		//webpack.config.js
		"plugins": ["transform-es2015-modules-simple-commonjs"]

2. ES6 + angular1 + webpack，遇到controller文件里的constructor运行2次？

	那是因为声明了2次`controller`，在配置中配了`app.controller('MyController')`，然后又在页面中使用了`ng-controller`，导致运行了2次，坑爹~

3. Babel转ES5后IE8下的兼容性解决方法。

1)webpack配置文件，增加插件transform-es3-property-literals和transform-es3-member-expression-literals

	const webpackdevConfig = {
	  entry: entry,
	  output: {
	    path: path.join(__dirname, 'dist/js'),
	    filename: '[name].js',
	    publicPath: '/static/'
	  },
	  plugins: [
	    new webpack.NoErrorsPlugin(),
	  ],
	  module: {
	    loaders: [
	      {
	        test: /\.js$/, loader: ['babel'], include: [path.join(new_dir, 'src')],
	        query:{
	          "presets": ["es2015", "stage-0"],
	          "plugins" : [
	            "transform-es3-property-literals",
	            "transform-es3-member-expression-literals",
	          ]
	        }
	      },
	      {test: /\.scss$/, loaders: ['style', 'css', 'sass'], include: path.join(new_dir, 'src/style')},
	      {test: /\.(jpg|png)$/, loader: 'url-loader?limit=8192', include: path.join(new_dir, 'src/img')}
	    ]
	  }
	}

2)模块导出不能使用 export default ，改为export { xxx }

3)模块引入使用 import  { } from 'xxx'

4)引入es5-shim.min.js和es5-sham.min.js
