---
title: css-transforms-3d
date: 2018-6-21 09:38:21
tags: 
    - css
---

## 浏览器支持情况

### 2D Transforms

Chrome	Safari	Firefox	Opera	IE	Android	iOS
Any	    3.1+	  3.5+	  10.5+	9+	4.1+	  At least 4

### 3D Transforms

Chrome	Safari	Firefox	Opera	IE	Android	iOS
10+	    4+	    12+	    none	10+	4.1+	  5+

## 2D Transforms

- scale(): 缩放元素，包括：`font-size`,`padding`,`height`,`width`。它还提供`scaleX`和`scaleY`速记函数。

- skewX() and skewY(): 元素向左或者向右倾斜，这个没有`skew`函数。

- translate(): 位移元素

- rotate(): 顺时针翻转元素

- matrix(): 矩阵，这个函数可能不是专门用手写的，但将所有转换合并为一个。

- perspective(): 不会影响元素本身，但会影响后代元素3D变换的变换，从而使它们都具有一致的深度透视图。

## 3D Transforms

### Perspective

要激活3D空间，元素必须要有透视。可以有两种方式使用：使用`transform: perspective(600px)`或者`perspective: 600px`。

perspective决定3D效果的强度的值。把它看作从观察者到物体的距离。值越大，距离越远，视觉效果越不强烈。perspective: 2000px;产生微妙的3D效果，就像我们通过双筒望远镜从远处观看物体一样。perspective: 100px;产生巨大的3D效果，就像一只看到巨大物体的小昆虫。

可以在子元素上使用perspective或者父级元素，但是，当用于多个元素时，转换后的元素不会按预期排列。如果跨不同位置的元素使用相同的变换，则每个元素都有自己的视点。为了解决这个问题，请使用perspective父元素的属性，以便每个孩子可以共享相同的3D空间。参考https://desandro.github.io/3dtransforms/examples/perspective-02-children.html

<p data-height="265" data-theme-id="0" data-slug-hash="xjGKMW" data-default-tab="css,result" data-user="liaoyf" data-embed-version="2" data-pen-title="css-3d-transforms-perspective" class="codepen">See the Pen <a href="https://codepen.io/liaoyf/pen/xjGKMW/">css-3d-transforms-perspective</a> by liaoyf (<a href="https://codepen.io/liaoyf">@liaoyf</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

默认情况下，3D空间的视点位于中心。你可以用perspective-origin属性来改变视点的位置。

```
perspective-origin: 25% 75%;
```

### 3D Transform functions

- rotateX( angle )
- rotateY( angle )
- rotateZ( angle )
- translateZ( tz )
- scaleZ( sz )

还有几个速记变换函数需要所有三个维度的值：

- translate3d( tx, ty, tz )
- scale3d( sx, sy, sz )
- rotate3d( rx, ry, rz, angle )

> 这些foo3d()转换函数还具有在Safari中触发硬件加速的好处，如果您正在编写适用于iOS或Safari的生产准备CSS，请务必使用这些foo3d()函数以获得最佳的渲染性能。