---
title: 《svg精髓》学习笔记（常用形状绘制）
date: 2018-8-16 16:45:46
tag: svg
---

## 基本形状

### 线段

````
<line x1="start-x" y1="start-y" x2="end-x" y2="end-y" />
````

### 笔画特性

笔画的尺寸、颜色和风格都会影响线段的表现，这些都可以通过`style`属性来指定。

#### stroke-width

画布中的坐标系统网格线是无线细的，网格线位于画笔的正中心：

````
<line x1="0" y1="0" x2="100" y2="100" style="stroke-width: 10; stroke: black" />
````

> 大部分svg阅读器都会默认开启反锯齿功能，可以通过指定CSS属性shape-rendering的值来控制反锯齿特性，取值crispEdges（在元素上，或者整个svg上）会关闭反锯齿特性。

#### 笔画颜色

你可以通过以下几种方式指定画笔的颜色：

- 基本颜色

- 16进制

- rgb/rgba

- currentColor：当前元素应用的css属性的color值

````
<line x1="0" y1="0" x2="100" y2="100" style="stroke-width: 10; stroke: black" />
````

#### stroke-opacity

控制线条的透明度。

#### stroke-dasharray

如果你需要电线或者虚线，则需要使用这个属性，它的值由一系列数字构成，代表线的长度和空隙的长度，数字之间用逗号或空格隔开。数字的个数应为偶数。

````
// 9个像素的虚线，5个像素的间隙
stroke-dasharray: 9, 5
````

### 矩形

矩形只需要指定左上角坐标x和y，以及宽度width和高度height即可。如果不指定fill，则默认用黑色填充，需要指定stroke，不然它不会绘制：

````
<rect x="10" y="10" width="100" height="100" />
````

### 圆角矩形

圆角矩形可以通过指定x方向和y方向的圆角半径（rx和ry），rx的最大值是矩形宽度的一半，ry同理，如果只指定其中一个，则另外一个也相等。

### 圆和椭圆

圆使用circle元素，通过指定中心点(x，y)和半径(r)，椭圆使用ellipse需要另外同时指定x方向的半径和y方向的半径(rx, ry):

### 多边形

<polygon>元素可以用来画任意封闭图形，使用时只需为points属性指定一系列的x/y坐标点，并用逗号或空格隔开，指定坐标时不需要在最后指定返回起始坐标，因为图形总是封闭的，会自动回到起始点。

````
<polygon points="15, 10 55, 10 45, 20 5, 20" style="fill: red; stroke: black" />
````

### 填充边线交叉的多边形

可以使用fill-rule: [nonzero | evenodd]

> 参考网址：https://blog.csdn.net/wjnf012/article/details/72875739

### 折线

使用polyline元素，通过points属性指定点，但是和polygon不同的是，它不会自动封闭。

> 最好将polyline的fill设置为none，不然svg阅读器可能会自动填充黑色。

### 线帽和线连接

当使用line或polyline时，可以为stroke-linecap：butt、round、square指定。round和square在起始位置都超过了真实位置，默认值butt则精确和起止位置对齐。

你也可以通过`stroke-linejoin：miter（尖的）、round（圆的）、bevel（平的）`属性来指定线段在图形棱角处交叉时的效果。