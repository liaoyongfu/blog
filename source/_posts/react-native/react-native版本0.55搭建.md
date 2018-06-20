---
title: react-native版本0.55搭建
date: 2018-06-20
tag: react-native
---

## `react-native run-ios`报错

报错如下：

```
An error was encountered processing the command (domain=NSPOSIXErrorDomain, code=2):
Failed to install the requested application
An application bundle was not found at the provided path.
Provide a valid path to the desired application bundle.
Print: Entry, ":CFBundleIdentifier", Does Not Exist
/development/misc/react/AwesomeProject/node_modules/promise/lib/done.js:10
      throw err;
      ^

Error: Command failed: /usr/libexec/PlistBuddy -c Print:CFBundleIdentifier build/Build/Products/Debug-iphonesimulator/AwesomeProject.app/Info.plist
Print: Entry, ":CFBundleIdentifier", Does Not Exist
```

泪崩。。。试用遍了网上大多数的解决方法均不行，浪费了好多宝贵时间，后来发现是xcode的版本问题，0.55版本要求xcode的版本要`>= 9`。

所以要更新xcode版本即可解决，记得更新完后重新`react-native init`项目。

## 结合`antd-mobile`使用

安装antd-mobile-rn:

```
yarn add antd-mobile-rn
```

使用：

```
import React, { Component } from 'react';
import { AppRegistry } from 'react-native';
import Button from 'antd-mobile-rn/lib/button';

class HelloWorldApp extends Component {
  render() {
    return <Button>Start</Button>;
  }
}

AppRegistry.registerComponent('HelloWorldApp', () => HelloWorldApp);
```

更多坑请[点击查看](https://liaoyongfu.github.io/2017/12/21/react-native/react-native%E4%B8%AD%E9%81%87%E5%88%B0%E7%9A%84%E5%9D%91/)