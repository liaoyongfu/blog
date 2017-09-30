---
title: React 16新增功能
date: 2017-9-29 15:19:23
tags:
    - react
---

## 官网介绍

https://facebook.github.io/react/blog/2017/09/26/react-v16.0.html

## render函数不必在包一层元素

```
render() {
  // No need to wrap list items in an extra element!
  return [
    // Don't forget the keys :)
    <li key="A">First item</li>,
    <li key="B">Second item</li>,
    <li key="C">Third item</li>,
  ];
}
```

## 更好的错误处理

以前的版本，如果子组件有报错整个组件本身直接不渲染，而现在可以通过`componentDidCatch`捕获错误信息：

```
class ErrorBoundary extends React.Component {
    constructor(props) {
        super(props);
        this.state = { error: null, errorInfo: null };
    }

    componentDidCatch(error, errorInfo) {
        // Catch errors in any components below and re-render with error message
        this.setState({
            error: error,
            errorInfo: errorInfo
        })
        // You can also log error messages to an error reporting service here
    }

    render() {
        if (this.state.errorInfo) {
            // Error path
            return (
                <div>
                    <h2>Something went wrong.</h2>
                    <details style={{ whiteSpace: 'pre-wrap' }}>
                        {this.state.error && this.state.error.toString()}
                        <br />
                        {this.state.errorInfo.componentStack}
                    </details>
                </div>
            );
        }
        // Normally, just render children
        return this.props.children;
    }
}
```

## Portals

现在可以通过`React.createPortal(<SomeComponent/>, document.getElementById('root'))`的形式在react应用容器之外修改或增加DOM：

```
render() {
  // React does *not* create a new div. It renders the children into `domNode`.
  // `domNode` is any valid DOM node, regardless of its location in the DOM.
  return ReactDOM.createPortal(
    this.props.children,
    domNode,
  );
}
```

这样对于像要创建模态框就很容易了：

```
class Overlay extends React.Component{
    constructor(){
        super();

        this.overlayContainer = document.createElement('div');
        document.body.appendChild(this.overlayContainer);
    }
    componentWillUnmount(){
        document.body.removeChild(this.overlayContainer);
    }

    render(){
        return ReactDOM.createPortal(
            <div className="overlay" style={{position: 'absolute', top: 0, right: 0, bottom: 0, left: 0, background: 'rgba(0,0,0, 0.5)', color: '#fff'}}>
                {this.props.children}
            </div>,
            this.overlayContainer
        )
    }
}

class Demo extends React.Component{
    constructor(){
        super();

        this.state = {
            overlayActive: true
        };
    }
    render(){
        return (
            <div>
                <button type="button" onClick={() => this.setState({
                    user: null
                })}>Update</button>
                {this.state.overlayActive && (
                    <Overlay onClose={() => this.setState({overlayActive: false})}>
                        hello modal!!!
                        <button type="button" onClick={() => this.setState({overlayActive: false})}>Close</button>
                    </Overlay>
                )}
            </div>
        );
    }
}
```

## 更好的服务端渲染

省略。。。

## 支持自定义DOM属性

以前是直接忽视无法识别的属性，而现在是直接渲染给DOM，这也直接节省了很多代码。

具体参考：https://facebook.github.io/react/blog/2017/09/08/dom-attributes-in-react-16.html

## 文件减少

- react is 5.3 kb (2.2 kb gzipped), down from 20.7 kb (6.9 kb gzipped).

- react-dom is 103.7 kb (32.6 kb gzipped), down from 141 kb (42.9 kb gzipped).

- react + react-dom is 109 kb (34.8 kb gzipped), down from 161.7 kb (49.8 kb gzipped).

## MIT协议

妥协了...

## 新的核心架构Fiber