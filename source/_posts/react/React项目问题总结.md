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

    ```
    /**
     *  * Created by liaoyf on 2017/11/1 0001.
     */
    import React from 'react';
    import 'bootstrap';
    import { render } from 'react-dom';
    import v4 from 'uuid';
    
    class ModalTool{
        constructor(options){
            this.options = {
                okText: '确定',
                cancelText: '取消',
                title: '提示框',
                okAutoClose: true,
                cancelAutoClose: true,
                bsStyle: null,
                bsSize: '',
                subContent: '',
                customContent: '',
                closeBtn: true,
                backdrop: true,
                ...options
            };
    
            this.init();
        }
        bindEvent(){
            let { onOk, onCancel } = this.options;
    
            $('#' + this.modalId + ' .btn-ok').unbind('click').on('click', function(){
                onOk && onOk();
            });
            $('#' + this.modalId + ' .btn-cancel').unbind('click').on('click', function(){
                onCancel && onCancel();
            });
        }
        closeModal(){
            $('#' + this.modalId ).modal('hide');
        }
        init(){
            // $('.modalBox').remove();
            $('.modal-backdrop').remove();
            let { title, content, okText, closeBtn, cancelText, okAutoClose, cancelAutoClose, bsStyle, bsSize, subContent, customContent, backdrop } = this.options;
            if(bsStyle){
                content = (
                    <dl className={`modal-state-show ${bsStyle === 'warning' ? 'state-error' : bsStyle === 'success' ? 'state-success' : ''}`}>
                        <dt className="state-ico"><i className="fa"></i></dt>
                        <dd className="state-text">{content}</dd>
                        {subContent && <dd className="state-info">{subContent}</dd>}
                        {customContent && <dd><p>{customContent}</p></dd>}
                    </dl>
                );
            }else if(!React.isValidElement(content)){
                content = (
                    <div className="text-vertical-wrap">
                        <p className="text-center text-vertical">{content}</p>
                    </div>
                );
            }
            let modalId = `modal_${v4.v4()}`;
            let html =
                `
    <div class="modal fade modalBox" id="${modalId}" tabIndex="-1" role="dialog" aria-labelledby="myModalLabel">
        <div class="${'modal-dialog ' + (bsSize ? 'modal-' + bsSize : (bsStyle ? 'modal-sm' : ''))}" role="document">
            <div class="modal-content">
                <div class="modal-header">
                    ${closeBtn ? `<button type="button" class="close" data-dismiss="modal" aria-label="Close"><span aria-hidden="true">&times;</span></button>` : ''}
                    <h4 class="modal-title" id="myModalLabel">${title}</h4>
                </div>
                <div class="body-inset modal-body" id="modalBoxBody">
                    
                </div>
                <div class="modal-footer">
                    ${okText ? `<button type="button" class="btn btn-primary btn-ok" ${okAutoClose && `data-dismiss="modal"`}>${okText}</button>` : ''}
                    ${cancelText ? `<button type="button" class="btn btn-default btn-cancel" ${cancelAutoClose && `data-dismiss="modal"`}>${cancelText}</button>` : ''}
                </div>
            </div>
        </div>
    </div>
                `;
            $('body').append(html);
            this.modalId = modalId;
            render(
                content,
                $(`#${modalId} #modalBoxBody`)[0],
                () => {
                    $(`#${modalId}`).modal({
                        backdrop: backdrop
                    });
                    this.bindEvent();
                }
            );
            return $(`#${modalId}`);
        }
    }
    
    export default ModalTool;
    ```

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
    最好打包时自动运行`npm update`操作：
    
    ```
    //package.json
    "script": {
        "updater": "npm update",
        "build": "npm run updater&&webpack --env=prod --progress --profile --colors"
    }
    ```

10. 如何做到公共thunk的合并？

11. front编译如何做到自动化？

12. process.env.NODE_ENV在编译后变成`I.env.NODE_ENV`？

13. 如何require或import远程地址的文件？

14. 脚手架运行时最好能判断当前的版本是不是最新的，如果不是就报错，这样能确保安装最新的脚手架？

15. 热加载更新？

    ```
    import Container from './router';
    
    const render = (Component) => {
        ReactDOM.render(
            <Component/>,
            document.getElementById('root')
        )   
    };
    
    render(Container);
    
    if (module.hot) {
        module.hot.accept('./router.js', () => {
            const NextRootContainer = require('./router').default;
            render(NextRootContainer);
        });
    }
    ```

16. js生成sourcemap方便调试？

17. 组件间的样式在打包后会被相互影响？

    使用`css module`

18. 如何解决ulynlist问题：basePath问题（貌似可以自己引）？

19. 打包时如果output.publicPath为空字符串时会找不到字体图标的bug？

    配置：

    ```
    {
        test: /\.scss$/,
        use: ExtractTextPlugin.extract({
            //主要是加这一句
            publicPath: '../',
            fallback: 'style-loader',
            use: [
                {
                    loader: 'css-loader?sourceMap'
                },
                {
                    loader: 'resolve-url-loader'
                },
                {
                    loader: 'sass-loader?sourceMap'
                }
            ]
        })
    }
    ```

20. 样式想要抽成公用的，有两种方法：

    如果只想在入口引用一个，则必须写到entry中；
    如果在多个文件中引用？