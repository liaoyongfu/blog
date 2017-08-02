---
title: 关于同一元素下dbClick触发两次click的问题
date: 2017-8-2 14:31:15
tags: 
    - js
---

## 问题

将处理程序绑定到相同元素的click和dblclick事件是不合适的。

触发的事件顺序因浏览器而异，有些在dblclick之前接收到两个点击事件，而其他事件只有一个。

双击灵敏度（双击检测到的点击之间的最大时间）可能因操作系统和浏览器而异，并且通常是用户可配置的。

所以最好不要在同一个元素下绑定click和dbclick事件。

<!-- more -->

## 解决方法

方法一：

```
var v_Result;  
function OneClick(event) {  
    console.log("detail",event.detail);  
    //if (event.detail == 2)   
    //  return ;   
    v_Result = false;  
    window.setTimeout(check, 300);  
    function check() {  
        if (v_Result != false) return;  
        console.log("单击");  
    }  
}  
function TwoClick() {  
    v_Result = true;  
    console.log("双击");  
}
```

方法二：

```
var clickTimer = null;  
function _click() {  
    if (clickTimer) {  
        console.log("clearTimeout", clickTimer);  
        window.clearTimeout(clickTimer);  
        clickTimer = null;  
    }  
  
    clickTimer = window.setTimeout(function() {  
        // your click process code here  
        console.log("你单击了我");  
    },  
    300);  
    console.log("setTimeout", clickTimer);  
}  
  
function _dblclick() {  
    console.log("dblclick");  
    if (clickTimer) {  
        console.log("=clearTimeout", clickTimer);  
        window.clearTimeout(clickTimer);  
        clickTimer = null;  
    }  
  
    // your click process code here  
    console.log("你双击了我");  
}
```