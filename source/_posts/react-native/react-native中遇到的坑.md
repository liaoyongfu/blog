---
title: react-native中遇到的坑
date: 2017-12-21 15:37:56
tags:
    - react-native
---

## Android中TextInput有下边框

添加`transparent`即可解决

```
<TextInput underlineColorAndroid="transparent"/>
```

## Android中borderRadius和borderWidth一起使用时效果会出错

看issue好像还没有解决：

https://github.com/facebook/react-native/issues/11042