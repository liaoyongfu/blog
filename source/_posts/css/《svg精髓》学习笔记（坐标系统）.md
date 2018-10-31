---
title: 《svg精髓》学习笔记（坐标系统）
date: 2018-8-15 14:55:55
tag: svg
---

## 在网页中使用svg

- 将svg作为图像：使用img元素包含svg文件

- 将svg作为css背景：使用background-image

- 将svg作为对象

- 内联svg

## 坐标系统

### 视口

文档打算使用的画布区域被称为视口。我们可以使用svg元素上的`width`和`height`属性确定视口的大小。属性值如果没有单位默认是`px`。

### 默认用户坐标

原点(x=0, y=0)在阅读器的左上角

### 为视口指定用户坐标

没有单位的数值都被视为像素，但是有时候我们并不想这样。为了实现这一效果，我们可以在svg元素上设置viewBox属性，这个属性的值由4个数值组成，他们分别代表想要叠加在视口上的用户坐标系统的最小x坐标、最小y坐标、宽度、高度。

因此，要在4厘米*5厘米的图纸上设置一个每厘米16个单位的坐标系统，要使用这个开始标记：

````
<svg width="4cm" height="5cm" viewBox="0 0 64 80">
````

## 保留宽高比

如果viewBox和视口宽高比不一样，svg可以做三件事：

- 按较小的尺寸等比例缩放

- 按较大的尺寸等比例缩放并裁剪掉超出视口的部分

- 拉伸或挤压绘图以恰好填充新的视口（不保留宽高比）

svg元素的`preserveAspectRatio="align [meet | slice:裁剪]""`属性允许我们指定被缩放的图像相对视口的对齐方式，以及它适配边缘还是要裁剪。

默认值：`xMidYMid meet`，它会缩小图像以适配可用的空间，并且使它水平和垂直居中。

<?xml>
  <svg width="45" height="135" viewBox="0 0 90 90">
    <rect x="0" y="0" width="45" height="135" style="stroke: black; fill: none"/>
    <!-- 猫 -->
    <circle r="50" cx="70" cy="95" style="stroke: black;fill: none"></circle>
    <circle r="5" cx="55" cy="80" stroke="black" fill="#339933"/>
    <circle r="5" cx="85" cy="80" stroke="black" fill="#339933"/>
    <g id="whiskers">
        <line x1="75" y1="95" x2="135" y2="85" style="stroke: black" />
        <line x1="75" y1="95" x2="135" y2="105" style="stroke: black"/>
    </g>
    <use xlink:href="#whiskers" transform="scale(-1 1) translate(-140 0)"/>
    <polyline points="108 62, 90 10, 70 45, 50 10, 32 62" style="stroke: black;fill: none;"/>
    <path d="M 75 90 L 65 90 A 5 10 0 0 0 75 90" style="stroke: black;fill: #ffcccc"/>
    <polyline points="35 110, 45 120, 95 120, 105 110" style="stroke: black;fill:none"/>
    <text x="60" y="165" style="font-family: sans-serif;font-size: 14px; stroke: none;fill: black">Cat</text>
  </svg>
</xml>

如果`preserveAspectRatio`值为`none`，则不保留宽高比。

## 嵌套坐标系统

可以在文档中嵌套svg元素。默认新坐标是0,0，当然也可以通过x和y属性指定新原点。

---

参考网址：https://segmentfault.com/a/1190000007143300