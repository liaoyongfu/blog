---
title: React遇到的坑.md
date: 2017-10-31 15:47:13
tags:
    - react
---


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

