---
title: CSSDOM简介
date: 2018-12-13 17:39:57
tag: css
---

## 什么是CSSOM

> CSS对象模型是一组允许从JavaScript处理CSS的API。它很像DOM，但对于CSS而不是HTML。它允许用户动态地读取和修改CSS样式。

## 通过element.style获取和设置内联样式

使用JavaScript操作或访问CSS属性和值的最基本方法是通过元素的`style`属性进行操作：

````
document.body.style.background = 'lightblue';
document.body.style.width = '888px';
document.body.style.cssFloat = 'left';
````

这是使用JavaScript操作或获取CSS属性和值的简单方法。但是style以这种方式使用属性需要注意：***这只适用于元素的内联样式***。

如果元素没有定义内联样式，即时有一个外部样式文件，获取不存在的样式属性时将得不到任何东西：

````
console.log(document.body.cssFloat) // ""
````

这显然有一些显着的局限性，所以让我们看一些使用JavaScript阅读和操作样式的更有用的技术。

## 获取计算样式

可以使用以下`window.getComputedStyle()`方法读取元素上任何CSS属性的计算CSS值：

````
widnow.getComputedStyle(document.body).background
// "rgba(0, 0, 0, 0) none repeat scroll 0% 0% / auto padding-box border-box"
getComputedStyle(document.body).width
// "134px"
````

计算属性将会得到最终的样式结果，包括浏览器默认的样式，这可能会得到太多东西了，(⊙o⊙)…！

有几种不同的方法可以使用`window.getComputedStyle()`：

````
// dot notation, same as above
window.getComputedStyle(el).backgroundColor;

// square bracket notation
window.getComputedStyle(el)['background-color'];

// using getPropertyValue()
window.getComputedStyle(el).getPropertyValue('background-color');
````

## 获取伪元素的计算样式

一个鲜为人知的小问题`window.getComputedStyle()`是，它允许您检索伪元素的样式信息。你会经常看到这样的`window.getComputedStyle()`声明：

````
window.getComputedStyle(document.body, ":before").width;
````

第二个可选参数允许我指定我正在访问伪元素的计算CSS。

## CSSStyleDeclaration

不管是通过`style`还是`window.getComputedStyle`，它返回的都是`CSSStyleDeclaration`对象，后者是只读的。

## setProperty(), getPropertyValue() 和 item()

考虑如下代码：

````
let box = document.querySelector('.box');

box.style.setProperty('color', 'orange');
box.style.setProperty('font-family', 'Georgia, serif');
op.innerHTML = box.style.getPropertyValue('color');
op2.innerHTML = `${box.style.item(0)}, ${box.style.item(1)}`;
````

- `setProperty(css属性名，css属性值)`

- `getPropertyValue(css属性名)`

- `item(index)`：上图`item(0)`为`color`、`item(1)`为`font-family`，`item(2)`如果没有的话返回""

## removeProperty()

````
box.style.setProperty('font-size', '1.5em');
box.style.item(0) // "font-size"

document.body.style.removeProperty('font-size');
document.body.style.item(0); // ""
````

## 获取和设置属性的优先级

````
box.style.setProperty('font-family', 'Georgia, serif', 'important');
box.style.setProperty('font-size', '1.5em');

box.style.getPropertyPriority('font-family'); // important
op2.innerHTML = box.style.getPropertyPriority('font-size'); // ""
````

当`setProperty`增加第三个参数`important`时，`font-family`将会有`!important`权重。

我要强调的是，这些方法可以与已经直接放在HTML元素style属性上的任何内联样式一起使用。所以，如果我有以下HTML：

````
<div class="box" style="border: solid 1px red !important;">
````

其他子属性也将会有`!important`权重。

````
// These all return "important"
box.style.getPropertyPriority('border'));
box.style.getPropertyPriority('border-top-width'));
box.style.getPropertyPriority('border-bottom-width'));
box.style.getPropertyPriority('border-color'));
box.style.getPropertyPriority('border-style'));
````

## CSSStyleSheet接口

从样式表访问信息的最简单方法是使用`document`开放的`styleSheets`属性，例如，下面的行使用该length属性来查看当前页面具有多少样式表：

````
document.styleSheets.length; // 1
````

我可以使用从零开始的索引来引用任何文档的样式表：

````
document.styleSheets[0];
````

它将返回`CSSStyleSheet`对象，里面最有用的是`cssRules`属性，此属性提供该样式表中包含的所有CSS规则（包括声明块，规则，​​媒体规则等）的列表。在下面的部分，我们将详细介绍该API的操作。

> cssRules（CSSRuleList） -> (CSSStyleRule) -> type, cssText, selectorText, style(CSSStyleDeclaration),...

## 使用样式表对象

下面使用样式表对象遍历选择器：

````
let myRules = document.styleSheets[0].cssRules,
    p = document.querySelector('p');

for (i of myRules) {
  if (i.type === 1) {
    p.innerHTML += `<c​ode>${i.selectorText}</c​ode><br>`;
  }
}
````

注意：cssRules是对象。

`type`类型：

- `1`: STYLE_RULE
- `3`: IMPORT_RULE
- `4`: MEDIA_RULE
- `5`: KEYFRAMES_RULE

> 完整类型请参考：https://developer.mozilla.org/en-US/docs/Web/API/CSSRule#Type_constants

其中`selectorText`是可写属性，所以我们可以动态修改：

````
for (i of myRules) {
    if (i.type === 1) {
      if (i.selectorText === 'a:hover') {
        i.selectorText = 'a:hover, a:active';
      }
    }
  }
````

## 使用CSSOM访问@media规则

假如有如下属性：

````
@media (max-width: 800px) {
  body {
    line-height: 1.2;
  }

  .component {
    float: none;
    margin: 0;
  }
}
````

可以通过`conditionText`获取：

````
let myRules = document.styleSheets[0].cssRules,
    p = document.querySelector('.output');

for (i of myRules) {
  if (i.type === 4) {
    p.innerHTML += `<c​ode>${i.conditionText}</c​ode><br>`;
    // (max-width: 800px) 
  }
}
````

## 使用CSSOM访问@keyframes规则

假如有如下属性：

````
@keyframes exampleAnimation {
  from {
    color: blue;
  }

  20% {
    color: orange;
  }

  to {
    color: green;
  }
}
````

可以通过`keyText`获取：

````
for (i of myRules) {
  if (i.type === 7) {
    for (j of i.cssRules) {
     p.innerHTML += `<code>${j.keyText}</code><br>`;
    }
  }
}

// 0% 20% 100%
````

您会注意到我的原始CSS使用from并to作为第一个和最后一个关键帧，但keyText属性计算这些0%和100%。keyText也可以设置值。在我的示例样式表中，我可以像这样硬编码：

````
// Read the current value (0%)
document.styleSheets[0].cssRules[6].cssRules[0].keyText;

// Change the value to 10%
document.styleSheets[0].cssRules[6].cssRules[0].keyText = '10%'

// Read the new value (10%)
document.styleSheets[0].cssRules[6].cssRules[0].keyText;
````

访问@keyframes规则时可用的另一个属性是name：

````
let myRules = document.styleSheets[0].cssRules,
    p = document.querySelector('.output');

for (i of myRules) {
  if (i.type === 7) {
    p.innerHTML += `<c​ode>${i.name}</c​ode><br>`;
  }
}

// exampleAnimation
````

我在这里要提到的最后一件事是能够获取单个关键帧内的特定样式。这是一些带有演示的示例代码：

````
let myRules = document.styleSheets[0].cssRules,
    p = document.querySelector('.output');

for (i of myRules) {
  if (i.type === 7) {
    for (j of i.cssRules) {
      p.innerHTML += `<c​ode>${j.style.color}</c​ode><br>`;
    }
  }
}
````

## 增加或删除CSSDeclarations规则

通过`insertRule()`增加规则：

````
let myStylesheet = document.styleSheets[0];
console.log(myStylesheet.cssRules.length); // 8

document.styleSheets[0].insertRule('article { line-height: 1.5; font-size: 1.5em; }', myStylesheet.cssRules.length);
console.log(document.styleSheets[0].cssRules.length); // 9
````

`insertRule()`第一个参数是样式字符串，第二个参数是表示您希望插入规则的位置或索引。如果未包括此项，则默认为0（意味着规则将插入规则集合的开头）。如果索引恰好大于rules对象的长度，则会抛出错误。

通过`deleteRule()`删除规则：

````
let myStylesheet = document.styleSheets[0];
console.log(myStylesheet.cssRules.length); // 8

myStylesheet.deleteRule(3);
console.log(myStylesheet.cssRules.length); // 7
````

该方法接受一个参数，该参数表示我要删除的规则的索引，参数传入的索引值必须小于cssRules对象的长度，否则将引发错误。

## 重新访问CSSStyleDeclaration API

记住，`CSSRule`下的`style`属性返回`CSSStyleDeclaration`属性：

````
// Grab the style rules for the body and main elements
let myBodyRule = document.styleSheets[0].cssRules[1].style,
    myMainRule = document.styleSheets[0].cssRules[2].style;

// Set the bg color on the body
myBodyRule.setProperty('background-color', 'peachpuff');

// Get the font size of the body
myBodyRule.getPropertyValue('font-size');

// Get the 5th item in the body's style rule
myBodyRule.item(5);

// Log the current length of the body style rule (8)
myBodyRule.length;

// Remove the line height
myBodyRule.removeProperty('line-height');

// log the length again (7)
myBodyRule.length;

// Check priority of font-family (empty string)
myBodyRule.getPropertyPriority('font-family');

// Check priority of margin in the "main" style rule (!important)
myMainRule.getPropertyPriority('margin');
````

## CSSDOM的未来？

在我在本文中考虑过的所有内容之后，我不得不打破这样的消息，即有一天我们所知道的CSSOM可能已经过时了。

这是因为称为CSS Typed OM的东西，它是Houdini项目的一部分。虽然有些人已经注意到新的Typed OM与目前的CSSOM相比更加冗长，但Eric Bidelman在本文中概述的好处包括：

- 更少的错误
- 算术运算和单位转换
- 更好的性能
- 错误处理
- CSS属性名称始终是字符串

有关这些功能的完整详细信息和语法的一瞥，请务必查看[完整的文章](https://developers.google.com/web/updates/2018/03/cssom)。

在撰写本文时，[CSS Typed OM](https://drafts.css-houdini.org/css-typed-om/)仅在Chrome中受支持。