webpack学习04——压缩优化
---

## 压缩

为了压缩你的脚本（和你的样式，如果你用`css-loader`的话），webpack支持下面两个途径：

	--optimize-minimize 或者 new webpack.optimize.UglifyJsPlugin()

webpack给你的模块和块赋予了id来区别他们，webpack可以为经常用到的id通过下面途径得到最小id长度：

	--optimize-occurrence-order resp. new webpack.optimize.OccurrenceOrderPlugin()

## 去重

如果你使用第三方库有相同依赖时，会重复引用相同的文件，webpack可以找到并去重，默认是不开启的，可以使用一下方法开启：

	--optimize-dedupe resp. new webpack.optimize.DedupePlugin()

## 块优化

限制快的最大大小 --optimize-max-chunks 15 new webpack.optimize.LimitChunkCountPlugin({maxChunks: 15})
限制块的最小大小 --optimize-min-chunk-size 10000 new webpack.optimize.MinChunkSizePlugin({minChunkSize: 10000})

Webpack会照顾它通过合并块（它会合并块，有重复的模块）。不会有东西合并到入口块，所以不会影响初始页面加载时间。

### 单页应用

A Single-Page-App is the type of web app webpack is designed and optimized for.

You may have split the app into multiple chunks, which are loaded at your router. The entry chunk only contains the router and some libraries, but no content. This works great while your user is navigating through your app, but for initial page load you need 2 round trips: One for the router and one for the current content page.

If you use the HTML5 History API to reflect the current content page in the URL, your server can know which content page will be requested by the client code. To save round trips to the server you can include the content chunk in the response: This is possible by just adding it as script tag. The browser will load both chunks parallel.

	<script src="entry-chunk.js" type="text/javascript" charset="utf-8"></script>
	<script src="3.chunk.js" type="text/javascript" charset="utf-8"></script>

You can extract the chunk filename from the stats. (stats-webpack-plugin could be used for exports the build stats)

### 多页应用

	var CommonsChunkPlugin = require("webpack/lib/optimize/CommonsChunkPlugin");
	module.exports = {
	    entry: {
	        p1: "./page1",
	        p2: "./page2",
	        p3: "./page3",
	        ap1: "./admin/page1",
	        ap2: "./admin/page2"
	    },
	    output: {
	        filename: "[name].js"
	    },
	    plugins: [
	        new CommonsChunkPlugin("admin-commons.js", ["ap1", "ap2"]),
	        new CommonsChunkPlugin("commons.js", ["p1", "p2", "admin-commons.js"])
	    ]
	};
	// <script>s required:
	// page1.html: commons.js, p1.js
	// page2.html: commons.js, p2.js
	// page3.html: p3.js
	// admin-page1.html: commons.js, admin-commons.js, ap1.js
	// admin-page2.html: commons.js, admin-commons.js, ap2.js

Advanced hint: You can run code inside the commons chunk:

	var CommonsChunkPlugin = require("webpack/lib/optimize/CommonsChunkPlugin");
	module.exports = {
	    entry: {
	        p1: "./page1",
	        p2: "./page2",
	        commons: "./entry-for-the-commons-chunk"
	    },
	    plugins: [
	        new CommonsChunkPlugin("commons", "commons.js")
	    ]
	};

See also multiple-entry-points example and advanced multiple-commons-chunks example.