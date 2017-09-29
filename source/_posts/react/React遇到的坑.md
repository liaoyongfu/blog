# React遇到的坑
---

1. 经常有编辑和查看的页面，编辑取消时需要调用redux-form的`resetForm()`方法。
2. 少用jquery！少用jquery！少用jquery！比如`$('input').prop('checked', true)`这样是不会触发`input`的`onChange`事件的，原因是？？？
3. 当input框输入值时输入框自动被失去了焦点，原因是在render函数里又用了render。
参考网址：https://stackoverflow.com/questions/25385578/nested-react-input-element-loses-focus-on-typing