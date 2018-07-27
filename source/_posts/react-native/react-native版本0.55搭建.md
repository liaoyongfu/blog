---
title: react-native版本0.55搭建
date: 2018-06-20
tag: react-native
---

## react-native开发流程

### create-react-native-app方式

使用此方式无需安装编译器、xcode或者Android Studio等。

```
npm install -g create-react-nateive-app
create-react-native-app AwesomeProject
cd AwesomeProject
npm start
```

启动后，控制台会输出一个二维码，接下来，你需要使用手机安装`Expo`应用程序，然后使用二维码登录即可在访问，如果你修改代码，应该也会热更新。

### 用native code构建

在IOS上，需要安装一些必须软件：

```
brew install node
brew install watchman
npm install -g react-native-cli
react-native init AwesomeProject
cd AwesomeProject
react-native run-ios
```

使用`Command + R`刷新。

> 注意：Xcode版本必须要`>=8`

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

泪崩。。。试用遍了网上大多数的解决方法均不行，浪费了好多宝贵时间，后来发现是xcode的版本问题，0.55版本要求xcode的版本要`>= 8`。

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

## 调用原生模块

```
import { NativeModules } from 'react-native';
var CalendarManager = NativeModules.CalendarManager;
CalendarManager.addEvent('Birthday Party', '4 Privet Drive, Surrey');
```

详细介绍请[点击查看](https://facebook.github.io/react-native/docs/native-modules-ios.html)

## 开发后安装问题

在真机上测试iOS应用需要一台Mac电脑，同时还需要注册一个Apple ID。如果你需要把应用发布到App Store，那么你还需要去苹果开发者网站购买一个开发者账户（在自己手机上测试则不用）

### 在真机上访问开发服务器（packager）

你可以在真机上访问开发服务器以快速测试和迭代。首先需要确保设备已使用usb连接至电脑，同时和电脑处在同一wifi网络内，然后在Xcode中选择你的设备作为编译目标（左上角运行按钮的右边），然后点击运行按钮即可。
如果你需要在真机上启用调试功能，则需要打开RCTWebSocketExecutor.m文件，然后将其中的"localhost"改为你的电脑的IP地址，最后启用开发者菜单中的"Debug JS Remotely"选项。

开发步骤：

1. 使用xcode打开项目

2. 使用USB链接手机，并和电脑处于统一wifi信号下

3. 选择右上角为你的手机，然后点击三角形运行

4. 输入电脑登录密码，之后软件会安装在你的手机上

5. 由于软件不被信任，所以要在`通用->设备管理->点击信任`

> 提示：摇晃设备就可以打开开发者菜单。

### 发布应用

当你使用React Native做好一个漂亮的应用之后，一定跃跃欲试想要在App Store上发布了吧。发布的流程和其他iOS原生应用完全一样，除了以下一些注意事项。
在App Store上发布应用首先需要编译一个“发布版本”(release)的应用。具体的做法是在Xcode中选择Product -> Scheme -> Edit Scheme (cmd + <)，然后选择Run选项卡，将Build Configuration设置为release。 Release版本的应用会自动禁用开发者菜单，同时也会将js文件和静态图片打包压缩后内置到包中，这样应用可以在本地读取而无需访问开发服务器（同时这样一来你也无法再调试，需要调试请将Buiid Configuration再改为debug）。
由于发布版本已经内置了js文件，因而也无法再通过开发服务器来实时更新。面向用户的热更新，请使用专门的热更新服务。
编译完成后，你就可以打包提交到TestFlight进行内测，或是提交到App Store进行发布。相关流程较为复杂，不熟悉原生应用发布流程的同学请自行搜索学习。

### App Transport Security

App Transport Security(简称ATS)是iOS 9中新增的一项安全特性。在默认设置下，只允许HTTPS的请求，而所有HTTP的请求都会被拒绝。详情可参考[这篇帖子](https://segmentfault.com/a/1190000002933776)。


## 使用react-native开发的可行性

根据Airbnb提供，具体参考：https://zhuanlan.zhihu.com/p/38288285?utm_source=com.alibaba.android.rimet&utm_medium=social：

### 优点

- 跨平台 （只有 0.2% 的平台特定代码）
- 统一的设计语言，同时还能为不同平台提供不同设计
- React 的 scale 很好，生命周期比原生简单，声明式很好
- 迭代速度快（主要是 hot reloading 很快）
- 大量基础设施的投入值得（网络、国际化、复杂动画、设备信息、用户信息等等都是通过一个桥把原生 api 暴露给 RN 的。）
- 同时他们在这里也指出：他们并不相信在一个已有 app 上集成 RN 是一件简单事儿，必须要大量且持续地投入基础设施才行（说好的「满意的地方」呢）
- 性能 （尽管大家都担心但是其实基本没有问题）
- 不过首次渲染比较慢，导致不适合用作启动屏、deeplink，也增加了可交互时间（TTI），另外掉帧不好 debug（说好的「满意的地方」呢）
- Redux（好用，虽然废话太多）
- 背后是原生，一些曾经不确定能不能做的功能（Shared element transitions、动画库 Lottie、网络层、核心基础设施）发现都能做
- 静态分析（eslint，prettier，一些性能检测）
- 动画
- JS/React 的开源生态
- Flexbox （via Yoga）
- 有时候可以加上 Web 跨三端

### 缺点

- RN 太不成熟
- 需要 fork RN
- JS 不行 （JS 没有类型不 scale，flow 不好用，TS 不好集成到 babel 和 metro）
- 不好重构（JS 没有类型无法静态分析，重构引起的错误不能在编译时被捕捉到）
- 咳，用我 FB 的新编译到 JS 语言大 ReasonML 啊……静态强类型 + 类型推断 + 自带不可变数据结构 + JS 友好语法 + 官方 React 支持，绝对 scale（咳扯远了
- JSCore 在 iOS / Android 上不一致 （Android 上是 RN 自己 bundle 的），很难 debug 这种坑
- RN 的开源库质量不行（因为太少人能精通所有平台了）
- 做功能时要回去搞基础设施（因为有的基础设施可能还没暴露给 RN）
- 奔溃监控（业内没方案，只能自己搞）
- 原生桥太难写，另外 JS 的类型太难预料（和强类型语言 interop 时）
-RN 运行时的初始化太慢
- 首次渲染时间慢（需要从 主线程 -> JS -> Yoga -> 主线程）
- 应用体积
- 64 位 （因为 RN 不兼容的 issue 导致他们至今没法在 android 发布 64 位应用）
- 手势（iOS 和 Android 的手势不好统一，虽然他们搞了 react-native-gesture-handler）
- 长列表
- 升级 RN 有的时候非常麻烦
- Accessibility （RN 的有 bug，又要 fork）
- 稀奇古怪的奔溃
- 安卓上的应用实例序列化问题

### 个人观点

如果不需要开发那种非常复杂的功能，react-native社区提供的第三方库基本能满足开发需求；