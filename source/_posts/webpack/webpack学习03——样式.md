webpack学习03——样式
---

## 嵌入样式
	
	{ test: /\.css$/, loader: "style-loader!css-loader" }

这种情况下会在页面添加style标签式的样式

## 抽成样式文件

可以使用`extract-text-webpack-plugin`抽成样式文件。

	var ExtractTextPlugin = require("extract-text-webpack-plugin");
	...
	loaders: [
        // Extract css files
        {
            test: /\.css$/,
            loader: ExtractTextPlugin.extract("style-loader", "css-loader")
        },
        // Optionally extract less files
        // or any other compile-to-css language
        {
            test: /\.less$/,
            loader: ExtractTextPlugin.extract("style-loader", "css-loader!less-loader")
        }
        // You could also use other loaders the same way. I. e. the autoprefixer-loader
    ],
	plugins: [
		 new ExtractTextPlugin("[name].css")
	]


### 所有样式文件合并成一个样式文件
	
	plugins: [
	    new ExtractTextPlugin("style.css", {
	        allChunks: true
	    })
	]

### 公共样式

和`CommonsChunkPlugin`一起使用，commons块就会生成commons.css样文件。

	plugins: [
	    new webpack.optimize.CommonsChunkPlugin("commons", "commons.js"),
	    new ExtractTextPlugin("[name].css")
	]