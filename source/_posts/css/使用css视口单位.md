---
title: 使用css视口单位
date: 2016-8-2 14:38:52
tags: 
    - css
---

css提供了许多单位用来规定元素，最熟悉的莫属`px`、`%`、`pt`、`em`以及最近比较火的`rem`，还有其他两个`vw`、`vh`，它们是相对单位，但是不同于`em`和`rem`那样相对于当前元素或相对于根元素，他们是相对于视口，一个视口单位等于1%的视口的宽度（`vw`）或高度（`vh`）。

这是很有用的。`vw`单位可以用于一些有大小的规则（如：`font-size`、或者一个`div`的高度），下面是一些使用案例：

## 使标题固定

如果你想让一个标题占满横向屏幕且让他固定在一行，你可以使用`vw`单位，这实际上是用原生的方法实现了jquery插件[FitText](http://fittextjs.com/)的功能，但是相对于使用`vw`，FitText需要手动管理大小，使用检查工具，是一个快速的方法以确定适当的值。

	//html
    <h1>Always a great fit!</h1>
	//css
    h1{
		font-size: 12vw;
		text-align: center;
	}

## Infinite Lines

Whilst building the falcon633 WordPress theme (used on this site), I needed to create an angled background that would appear to continue indefinitely. This is achievable by 1) making sure that the angle in the centre stays the same regardless of window size and 2) setting the height to be relative to the viewport width. I used an SVG background for the overall cut-out then set the height based on the width using vw units.

## 简单的视频包装

Let’s say you want to set the proportions of an element, an `iframe`, to stay at a fixed aspect ratio. You previously might have chosen to create a relative div filling the required space, then set carefully selected padding values with iframe inside absolutely positioned to cling to div on all sides (e.g. the approach demonstrated here).

A better solution could be to use the vw and vh units. This way you can set your height and width directly on the element in question, whilst also keeping the ‘layers-for-layout’ number down.

## 全屏的[`hero Image`](https://en.wikipedia.org/wiki/Hero_image "Hero Image")

你要做的只是在`body`和`html`上运用`height: 100%`，然后在元素上简单地使用`width: 100vw; heihgt: 100vh`就行了。

## `div`居中

一个常见的需求是要让一个`div`在页面中居中，这时候只要设置`margin: 20vh 20vw; width: 60vw, height: 60vh; padding: 10vh 10vw`即可。

## 浏览器支持情况

IE9及以上