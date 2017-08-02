webpack学习05——长期缓存
---

有两种级别的添加hash方法：

- Compute a hash of all chunks and add it.
- Compute a hash per chunk and add it.

## 方法一：只有一个hash
	
	{
	    output: {
	        path: path.join(__dirname, "assets", "[hash]"),
	        publicPath: "assets/[hash]/",
	        filename: "output.[hash].bundle.js",
	        chunkFilename: "[id].[hash].bundle.js"
	    }
	}

## 方法二：每个块都有一个hash

	output: { chunkFilename: "[chunkhash].bundle.js" }

Note that you need to reference the entry chunk with its hash in your HTML. You may want to extract the hash or the filename from the stats.

In combination with Hot Code Replacement you must use option 1, but not on the publicPath config option.

## 从文件名获取状态

You probably want to access the final filename of the asset to embed it into your HTML. This information is available in the webpack stats. If you are using the CLI you can run it with --json to get the stats as JSON to stdout.

You can add a plugin such as assets-webpack-plugin to your configuration which allows you to access the stats object. Here is an example how to write it into a file:
	
	plugins: [
	  function() {
	    this.plugin("done", function(stats) {
	      require("fs").writeFileSync(
	        path.join(__dirname, "..", "stats.json"),
	        JSON.stringify(stats.toJson()));
	    });
	  }
	]

The stats JSON contains a useful property assetsByChunkName which is a object containing chunk name as key and filename(s) as value.

> Note: It’s an array if you are emitting multiple assets per chunk. I. e. a JavaScript file and a SourceMap. The first one is your JavaScript source.