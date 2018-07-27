---
title: React遇到的坑.md
date: 2018-3-27 20:55:47
updated: 2018-6-7 15:15:08
tags:
    - react
---

## webpack编译后，代码中判断子组件名称功能失效。

因为压缩后函数名称混淆，所以不能通过`typeof child.type === 'function' && child.type.name === 'FormControl'`，而是应该通过`child.type === FormControl`


## 当input框输入值时输入框自动被失去了焦点，原因是在render函数里又用了render。
 
 参考网址：https://stackoverflow.com/questions/25385578/nested-react-input-element-loses-focus-on-typing

## input值不会随着属性改变而改变

```
<Parent>
    <Child inputVal={this.state.inputVal}/>
</Parent>
```

当`Child`组件的`value`为`null`后input的值不会更改。所以一般要判断是否存在`null`值：

```
<FormControl value={this.props.inputVal || ''}/>
```

## UglifyJs编辑后有些arguments无效

升级uglifyjs-webpack-plugin和uglify-js：

```
"uglify-js": "^3.1.6",
"uglifyjs-webpack-plugin": "^1.0.1",
```

## 上传文件不能上传同一张图片

在上传完成后，设置`$('input').val('')`即可。

## react中`ref`传递给页面组件时失效

比如如下代码：

```
let Component = require(`./${item.componentName}/index.js`).default;
return <Component ref={c => this.a = c} key={i} {...props} updateCzsj={updateCzsj}/>;
```

这时，使用`ref`是无效的，这时因为包装了`withRouter`导致ref失效，应该使用`wrappedComponentRef`，获取时使用如下语句获取：

```
this.a
```

## 在react组件中引入样式文件导致echarts宽度计算失败的bug

比如下面：

```
<div className="box">
    <div className="left">
        <Echart ...>
    </div>
    <div className="right">
        <Echart ...>
    </div>
</div>
```

`box`中的`left`、`right`各占`50%`（样式写在样式文件内），这时候渲染echarts的时候，因为样式文件还未生效，所以Echarts读取的`left`和`right`宽度是`100%`的。

***这个是为什么呢？***

## 修改react子组件

```
return React.Children.map(this.props.children, child => {
    if (!child) return child;
    if (typeof child.type === 'function' && child.type.name === 'Tr') {
        // 这边要修改children属性而不是直接返回它的children
        return React.cloneElement(child, {
            children: (
                React.Children.map(child.props.children, subChild => {
                    if (typeof subChild.type === 'function' && subChild.type.name === 'Label') {
                        return React.cloneElement(subChild, {
                            required: true
                        })
                    } else if (typeof subChild.type === 'function' && subChild.type.name === 'Content') {
                        return React.cloneElement(subChild, {
                            children: [...subChild.props.children, textTip],
                            validationState: validationState
                        })
                    }
                })
            )
        })
    } else {
        return child;
    }
});
```

## 使用react-css-modules后的问题

使用react-css-modules后会导致antd的Picker不能滑动选择（setState会一直触发componentWillReceiveProps）。所以改成`babel-plugin-react-css-modules`，配置如下：

````
plugins: [
    [
        'react-css-modules', {
            exclude: 'node_modules',
            filetypes: {
                '.scss': {
                    syntax: 'postcss-scss'
                }
            },
            handleMissingStyleName: 'ignore',
            // 这个必须和css-loader配置的名称一致，不然会导致生成的类名不相对应
            generateScopedName: '[local]___[hash:base64:5]'
        }
    ]
]
````
