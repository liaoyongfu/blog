---
title: 创建一个React项目
date: 2017-10-31 15:47:13
tags:
    - react
---

React由Facebook所写，由虚拟Dom、组件化获得广大前端开发者的青睐，下面我们通过一个示例来演示创建React项目的步骤：

## 为什么用`React`

React解决了创建大型项目性能以及复用性问题，React可以有两种写法：

*使用React.createClass语法*

    var HelloComponent = React.createClass({
  		render: function() {
	    	return (
		      	<div className="hello">
		       	 Hello, world!
		      	</div>
	    	);
  		}
	});

*使用ES6语法*

	class HelloComponent extends React.Component {
	  render() {
	    return (
	      <div className="hello">
	        Hello, world!
	      </div>
	    );
	  }
	}

## 创建React项目

1. 安装`Node.js`和`NPM`，然后运行`npm init`创建一个`package.json`文件.
2. 在控制台中运行：`npm install react react-dom babel-core babel-loader babel-preset-es2015 babel-preset-react webpack webpack-dev-server --save`
3. 创建`webpack.config.js`配置文件，这个的作用是帮我们打包资源，转换JSX为JS文件、合并、压缩、编译等等等。。。

一个简单的webpack.config.js大致如下：

	var debug = process.env.NODE_ENV !== "production";
	var webpack = require('webpack');
	var path = require('path');
	
	module.exports = {
	  context: path.join(__dirname, "src"),
	  devtool: debug ? "inline-sourcemap" : null,
	  entry: "./js/App.js",
	  devServer: {
	    inline: true,
	    port: 3333
	  },
	  module: {
	    loaders: [
	      {
	        test: /\.jsx?$/,
	        exclude: /(node_modules|bower_components)/,
	        loader: 'babel-loader',
	        query: {
	          presets: ['react', 'es2015'],
	          plugins: ['react-html-attrs', 'transform-class-properties', 'transform-decorators-legacy'],
	        }
	      }
	    ]
	  },
	  output: {
	    path: __dirname + "/src/",
	    filename: "bundle.min.js"
	  },
	  plugins: debug ? [] : [
	    new webpack.optimize.DedupePlugin(),
	    new webpack.optimize.OccurenceOrderPlugin(),
	    new webpack.optimize.UglifyJsPlugin({ mangle: false, sourcemap: false }),
	  ],
	};

程序的入口通过`entry`设置，即页面第一次加载运行的文件，Webpack将把所有的JS和JSX文件到文件的输出对象，通过`devServer`设置webpack开发服务器为内联，并设置端口为`3333`，在`module`配置中，我们配置`babel`转换规则：使用`react`和`es2015`,plugins增加了类的属性和装饰器的功能。

### 热加载

首先安装热加载模块：

    npm install --save-dev babel-preset-react-hmre

然后加到配置中：

	....
	query: {
	  presets: ['react', 'es2015', 'react-hmre'],
	  plugins: ['react-html-attrs', 'transform-class-properties', 'transform-decorators-legacy'],
	}

另一个选择是安装`react-hot-loader`然后添加`react-hot`到`webpack.config.js`配置中：
	
	...
	loader: ['babel-loader', 'react-hot']
	...

为了运行项目更简单，我们一般会使用package.json的命令：

	{
	  "scripts": {
	    "start": "node_modules/.bin/webpack-dev-server --progress --inline --hot",
	  }
	}

注意：我们命令中添加了`--hot`，这个启动了热加载模式.

## 路由

路由是一个应用非常重要的一部分，在React中比较受欢迎的莫属`React Router`了，事实上，很多开发者认为它就是React官方版的路由，当然，你得先安装它：

    npm install --save react-router

一个简单的示例看起来是这样子的：

	import React from 'react';
	import { render } from 'react-dom';
	import { browserHistory, Router, Route, IndexRoute } from 'react-router'
	
	import App from '../components/App'
	import Home from '../components/Home'
	import About from '../components/About'
	import Features from '../components/Features'
	
	render(
	  <Router history={browserHistory}>
	    <Route path='/' component={App}>
	      <IndexRoute component={Home} />
	      <Route path='about' component={About} />
	      <Route path='features' component={Features} />
	    </Route>
	  </Router>,
	  document.getElementById('app')
	)

## 国际化(I18N)

通过`react-intl`你可以很轻松地实现国际化，它支持超过150中不同语言，默认是英文，呃~

## 权限认证

Authentication is an important part of any application. The best way to do user authentication for single page apps is via JSON Web Tokens (JWT). A typical authentication flow is this:

- A user signs up/logs in, generate JWT token and return it to the client
- Store the JWT token on the client and send it via headers/query parameters for future requests

A comprehensive example of adding authentication to a ReactJS app is here. Using Redux? Here is a good example of setting up authentication in your ReactJS application.

## 数据持久化

Without a backend, you can persist data in your Single Page App by using Firebase. In a Reactjs app, all you simply need is ReactFire. It is a ReactJS mixin for easy Firebase integration. With ReactFire, it only takes a few lines of JavaScript to integrate Firebase data into React apps via the ReactFireMixin

    npm install --save reactfire react firebase

TodoList Example

	import React from 'react';
	
	class TodoList extends React.Component {
	  render() {
	    var _this = this;
	    var createItem = (item, index) => {
	      return (
	        <li key={ index }>
	          { item.text }
	          <span onClick={ _this.props.removeItem.bind(null, item['.key']) }
	                style=>
	            X
	          </span>
	        </li>
	      );
	    };
	
	    return <ul>{ this.props.items.map(createItem) }</ul>;
	  }
	}
	
	class TodoApp extends React.Component {
	  getInitialState() {
	    return {
	      items: [],
	      text: ''
	    };
	  }
	
	  componentWillMount() {
	    this.firebaseRef = firebase.database().ref('todoApp/items');
	    this.firebaseRef.limitToLast(25).on('value', function(dataSnapshot) {
	      var items = [];
	      dataSnapshot.forEach(childSnapshot => {
	        const item = childSnapshot.val();
	        item['.key'] = childSnapshot.key;
	        items.push(item);
	      });
	
	      this.setState({
	        items: items
	      });
	    }.bind(this));
	  }
	
	  componentWillUnmount() {
	    this.firebaseRef.off();
	  }
	
	  onChange(e) {
	    this.setState({text: e.target.value});
	  }
	
	  removeItem(key) {
	    var firebaseRef = firebase.database().ref('todoApp/items');;
	    firebaseRef.child(key).remove();
	  }
	
	  handleSubmit(e) {
	    e.preventDefault();
	    if (this.state.text && this.state.text.trim().length !== 0) {
	      this.firebaseRef.push({
	        text: this.state.text
	      });
	      this.setState({
	        text: ''
	      });
	    }
	  }
	
	  render() {
	    return (
	      <div>
	        <TodoList items={ this.state.items } removeItem={ this.removeItem } />
	        <form onSubmit={ this.handleSubmit }>
	          <input onChange={ this.onChange } value={ this.state.text } />
	          <button>{ 'Add #' + (this.state.items.length + 1) }</button>
	        </form>
	      </div>
	    );
	  }
	}
	
	ReactDOM.render(<TodoApp />, document.getElementById('todoApp'));

More information about persisting your data using ReactFire [here](https://github.com/firebase/reactfire/blob/master/docs/quickstart.md).

## 测试

Most projects become a mountain of spaghetti code at some point during development due to lack of solid tests or no tests at all. ReactJS apps are no different, but this can be avoided if you know some core principles. When writing tests for ReactJS code, it is helpful to pull out any functionality that doesn't have to do with any UI components into separate modules, so that they can be tested separately. Tools for unit testing those functionalities are mocha, expect, chai, jasmine.

Testing becomes tricky in a ReactJS application when you have to deal with components. How do you test stateless components? How do you test components with state? Now, ReactJS provides a nice set of test utilities that allow us to inspect and examine the components we build. A particular concept worthy of mention is Shallow Rendering. Instead of rendering into a DOM the idea of shallow rendering is to instantiate a component and get the result of its render method. You can also check its props and children and verify they work as expected. More information here.

Facebook uses Jest to test React applications. AirBnB uses Enzyme. When bootstrapping your ReactJS application, you can set up any of these awesome tools to implement testing.

## 脚手架和模板

A lot of tools have been mentioned in this post in relation to setting up different parts of a ReactJS app. If you don't intend writing your app from scratch, there are lots of generators and boilerplates that tie all these tools together to give you a great starting point for your app. One fantastic example is React Starter Kit. It has a Yeoman generator. It's an isomorphic web app boilerplate that contains almost everything you need to build a ReactJS app. Another boilerplate is React Static boilerplate, it helps you build a web app that can be hosted directly from CDNs like Firebase and Github Pages. Other alternatives are React redux starter kit and React webpack generator. Recently, a nice and effective tool called create-react-app was released by the guys at Facebook. It's a CLI tool that helps create React apps with no build configuration!

## 结语

There are several tools that will help bootstrap your React app, we looked at a couple that we consider quite good and will have your application up and running in no time. But feel free to search for your own tools, and if you think that we are missing something, let us know in the comments. Setting up a React project should be painless!

"Setting up a React project should be painless!"