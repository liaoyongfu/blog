---
title: lerna在项目中的运用
date: 2018-10-26
tags:
    - lerna
    
    - npm
---

## 背景

公司自己编写的库越来越多，而且之间的关联性比较强，如果某个包升级了版本，其他包也得相对应升级，这样非常的繁琐。

## 什么是 monorepo 和 multirepo

在讲lerna之前，我们要先了解一下两个不同的代码管理组织方式 monorepo 和 multirepo：

- `monorepo`：把所有的相关项目放在同一个仓库下，有点类似`yarn workspaces`

- `multirepo`：每个子项目拥有自己的 repo

## 什么是lerna

[lerna](https://github.com/lerna/lerna)是一个管理多个javascript包的工具。它属于`monorepo`类型，当你的项目有相关联时最好使用`monorepo`方式进行管理。

## 以一个简单的示例进行说明

新建一个`testLerna`项目：

````
$ npx lerna init
````

初始化一个lerna命令，生成以下目录结构：

````
testLerna/
    packages/
    lerna.json
    package.json
````

`pcakges`目录主要放置包，`lerna.json`为配置文件。

## 两种模式

- Fixed/Locked 模式 (默认)

通过在`lerna.json`中指定`version`，当你运行`lerna publish`时，如果其中某一个包至上次发布以来有更新，则将会更新`version`。

这个模式是babel目前使用的模式，当你想把所有包关联在一起的时候可以考虑使用这种模式，需要注意的是，当其中某个包升级了大版本之后，其他的每个包都会自动升级大版本。

- Independent mode

````
lerna init --independent
````

独立模式允许您更具体地更新每个包的版本，并对一组组件有意义。跟[semantic-release](https://github.com/semantic-release/semantic-release)等工具配合使用会更好。

> 已经有结合使用的[示例](https://github.com/atlassian/lerna-semantic-release)

## 发布

假设以下项目：

````
packages
    shareui
    shareui-html
    shareui-font
````
shareui依赖shareui-html，shareui-html依赖shareui-font，可以命令添加：

````
$ lerna add shareui-html --scope=shareui
$ lerna add shareui-font --scope=shareui-html
````
> 前提是要关联git，修改registry到公司私库

git提交到暂存区域，之后通过`lerna publish`发布：

````
$ lerna publish
````

以下情况需要考虑：

- shareui-font升级了，相对应的shareui-html和shareui也要升级？

    运行`lerna bootstrap`升级对应依赖

## 参考网址

- https://github.com/lerna/lerna
- https://zhuanlan.zhihu.com/p/33858131
- https://zhuanlan.zhihu.com/p/35237759