---
title: HTML不同空格的特性和表现研究
date: 2016-8-2 14:38:52
tags: 
    - css
---

## Unicode编码空格

- `&nbsp;`: **不换行空格**,用于不会被浏览器判断为可以在中间打断，比如：`There is&nbsp;Space`，如果会换行，只会在`There`和`is`之间换行，而不会在`is`和`Space`之间换行。
- 跟随字体大小产生相应空白的空格：`&ensp;`(1/2em)，`&emsp;`(1em)，`&thinsp;`(1/6em)，这类就很适合坐表单`label`的排版了，坑爹，终于找到好的解决方法了