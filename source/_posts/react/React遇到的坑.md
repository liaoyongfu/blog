# React遇到的坑
---

1. 经常有编辑和查看的页面，编辑取消时需要调用redux-form的`resetForm()`方法。
2. 少用jquery！少用jquery！少用jquery！比如`$('input').prop('checked', true)`这样是不会触发`input`的`onChange`事件的，原因是？？？