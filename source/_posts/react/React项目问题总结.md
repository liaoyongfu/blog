---
title: React项目问题总结
date: 2017-11-1 10:17:10
tags:
    - react
---

## 目前出现的问题

1. 需要定义很多初始化state变量？

2. 表单编辑后点取消，重新从后台拿数据如果是null的时候它不会自动更新state？

3. 组件复用：未编写PropTypes，如果写了有时候返回的字段是null的，怎么写PropTypes？

4. 组件复用：如果组件自己处理内部逻辑，那外面的组件如何拿到组件里面的数据？

5. 页面需要记住类似查询状态，如何简单有效的实现？使用高阶组件实现？如何做到不冲突？

6. 提示框太难用了？

7. 需要封装常用的对象或数组操作工具方法

8. 表单使用体验太差：需要编写一大推标签；需要手动编写value和onChange；表单校验需要改善

9. npm 包依赖如何做到同步更新？npm安装依赖包经常报错？npm打包太慢，启动太慢？打包后的文件不会自动加入svn版本控制？

npm安装依赖包报：
    `npm ERR! unlink`的错误，解决方法：
    
    ```
    rm (-rf) node_modules
    rm package-lock.json yarn.lock
    npm cache clear --force
    npm i -g npm # 5.4.1, as for now
    npm install --no-optional
    ```

10. 如何做到公共thunk的合并？

11. front编译如何做到自动化？

12. process.env.NODE_ENV在编译后变成`I.env.NODE_ENV`？

13. 如何require或import远程地址的文件？

14. 脚手架运行时最好能判断当前的版本是不是最新的，如果不是就报错，这样能确保安装最新的脚手架？

15. 热加载更新？

16. js生成sourcemap方便调试？