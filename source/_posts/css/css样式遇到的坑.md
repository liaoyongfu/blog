---
title: css样式遇到的坑
date: 2017-10-31 15:38:57
tags:
    - css
---

## bootstrap-datetime-picker在模态框中会随着页面滚动而错位

解决方法：在`options`中增加`container: '#modalId .modal-dialog'`，默认的`container`是`body`，所以要改成`.modal-dialog`。
