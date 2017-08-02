---
title: react文档-css动画
date: 2017-8-2 14:38:52
tags: 
    - react
---

TransitionGroup和CSSTransitionGroup已移动到react-transition-group有社区维护。它的1.x分支与现有插件的API是完全兼容的。

TransitionGroup是一个具有低级API的动画组件，而CSSTransitionGroup是一个基于css的animation和transition更容易被使用的组件。

<!-- more -->

## 高级API：CSSTransitionGroup

```
import { CSSTransitionGroup } from 'react-transition-group' // ES6
var CSSTransitionGroup = require('react-transition-group/CSSTransitionGroup') // ES5 with npm

class TodoList extends React.Component{
    constructor(){
        super();
        
        this.state = {
            items: [
                'hello',
                'world',
                'click',
                'me'
            ]
        };
        this.handleAdd = this.handleAdd.bind(this);
    }
    
    handleAdd() {
        const newItems = this.state.items.concat([
          prompt('Enter some text')
        ]);
        this.setState({items: newItems});
    }
    
    handleRemove(i) {
        let newItems = this.state.items.slice();
        newItems.splice(i, 1);
        this.setState({items: newItems});
    }
    
    render(){
        const items = this.state.items.map((item, i) => (
          <div key={i} onClick={() => this.handleRemove(i)}>
            {item}
          </div>
        ));
        return (
            <div>
                <button type="button" onClick={this.handleAdd}>Add Item</button>
                <CSSTransitionGroup
                    transitionName="example"
                    transitionEnterTimeout={500}
                    transitionLeaveTimeout={300}
                >
                    {items}
                </CSSTransitionGroup>
            </div>
        );
    }
}
```

<script src="https://jsfiddle.net/liaoyf/f4dxmnbr/2/embedded/js,html,css,result/dark/"></script>

在这个组件中，当一个新的Item被添加时，ReactCSSTransitionGroup将获得一个example-enter的类和example-enter-active的类，这些类名是基于trnasitionName的值。

您可以使用这些类来触发css动画或转换。例如：尝试添加如下样式：

```
.example-enter{
    opacity: 0.01;
}
.example-enter.example-enter-active{
    opacity: 1;
    transition: opacity 500ms ease-in;
}
.example-leave{
    opacity: 1;
}
.example-leavel.example-leavel-active{
    opacity: 0.01;
    transition: opacity 300ms ease-in;
}
```

### 初始化加载动画

ReactCSSTransitionGroup提供一个可选的属性transitionAppear，以在组件的初始渲染时添加额外的过渡阶段。默认为false：

```
render() {
  return (
    <CSSTransitionGroup
      transitionName="example"
      transitionAppear={true}
      transitionAppearTimeout={500}
      transitionEnter={false}
      transitionLeave={false}>
      <h1>Fading at Initial Mount</h1>
    </CSSTransitionGroup>
  );
}
```

在初始化渲染ReactCSSTransitionGroup过程中，example-appear类和example-appear-active类将被添加。

```
.example-appear {
  opacity: 0.01;
}
.example-appear.example-appear-active {
  opacity: 1;
  transition: opacity .5s ease-in;
}
```

在初始化渲染阶段，所有ReactCSSTransitionGroup的孩子节点将会appear而不是enter，然而，所有孩子稍后被渲染后将会触发enter而不是appear。

注意：

属性transitionAppear在0.13版本中才被添加进入ReactCSSTransitionGroup中，为了向后兼容，默认值为false。

然而，transitionEnter和transtionLeave默认为true，所以你默认只要指定transitionEnterTimeout和transitionLeaveTimeout。

### 自定义类名

当然，我们也可以使用自定义类名代替每一步默认的类名。你可以通过传递一个包含enter或leave或appear等等的对象给transitionName而不是字符串。

如果未提供active时，默认会为添加-active后的类名：

```
<CSSTransitionGroup
    transitionName={
        {
            enter: 'enter',
            enterActive: 'enterActive',
            leave: 'leave',
            leaveActive: 'leaveActive',
            appear: 'appear',
            appearActive: 'appearActive'
        }
    }
>
{item}
</CSSTransitionGroup>
<CSSTransitionGroup
  transitionName={ {
    enter: 'enter',
    leave: 'leave',
    appear: 'appear'
  } }>
  {item2}
</CSSTransitionGroup>
```

### 动画组必须已经被渲染到DOM

为了时其子节点的动画能够生效，ReactCSSTransitionGroup必须已经被加载进DOM中，或者transitionAppear必须为true。

下面的例子将不会正常运行，因为ReactCSSTransitionGroup被和新的item一起渲染，而不是新的item渲染进其子节点：

```
render() {
  const items = this.state.items.map((item, i) => (
    <div key={item} onClick={() => this.handleRemove(i)}>
      <CSSTransitionGroup transitionName="example">
        {item}
      </CSSTransitionGroup>
    </div>
  ));
  return (
    <div>
      <button onClick={this.handleAdd}>Add Item</button>
      {items}
    </div>
  );
}
```

### 渲染一个或零个子节点

在上面的例子中，我们渲染了一个列表节点，当然，ReactCSSTransitionGroup也可以渲染一个或零个节点：

```
import ReactCSSTransitionGroup from 'react-addons-css-transition-group';
function ImageCarousel(props) {
  return (
    <div>
      <CSSTransitionGroup
        transitionName="carousel"
        transitionEnterTimeout={300}
        transitionLeaveTimeout={300}>
        <img src={props.imageSrc} key={props.imageSrc} />
      </CSSTransitionGroup>
    </div>
  );
}
```

### 禁用动画

可以通过设置false来禁用指定阶段，如：transitionEnter为false禁用进入后的动画效果。

注意：使用ReactCSSTransitionGroup你没办法知道一个transition已经结束或其他更详细的细节，如果你需要，则得使用低级APIReactTransitionGroup，它提供更多的钩子让你能够监听。

## 低级API：TransitionGroup

```
import TransitionGroup from 'react-transition-group/TransitionGroup' // ES6
var TransitionGroup = require('react-transition-group/TransitionGroup') // ES5 with npm
```

ReactTransitionGroup是动画的基础。当子节点添加或删除，下面这些钩子函数将会被调用：

- componentWillAppear(callback)

这个方法将在渲染到节点时调用，它将阻塞其他当前正在发生的动画，知道callback被调用。这个方法仅仅在初始化渲染TransitionGroup时被调用。

- componentDidAppear()

这个方法将在componentWillAppear的callback被调用后被调用。

- componentWillEnter(callback)

当有新元素被添加到TransitionGroup时调用，它将会阻塞其他动画知道callback调用。

- componentDidEnter()

在componentWillEnter的callback调用后触发。

- componentWillLeave(callback)

当有元素被移除时触发，尽管元素已经被删除，但它还是还将保留在DOM中，直到callback被调用。

- componentDidLeave()

在componentWillLeave的callback被调用时触发（和componentWillUnmount()同时触发）。

### 渲染其他组件

TransitionGroup默认渲染为一个span，你通过增加一个compoennt属性改变默认行为：

```
<TransitionGroup component="ul">
    {/*...*/}
</TranstionGroup>
```

另外，用户自定义的属性也会对应增加到组件的属性，如下例子，ul也将会得到className属性：

```
<TransitionGroup component="ul" className="animated-list">
  {/* ... */}
</TransitionGroup>
```

所有react可以渲染的组件都可以成为component，它不是必须为一个DOM节点，如：component={CustomList}，那么，TransitionGroup的子节点传递给CustomList组件的属性this.props.children。

### 渲染单独的子节点

我们经常会用TransitionGroup来作为单独子节点的渲染和移除时的效果，诸如手风琴效果的面板。

正常情况下，TransitionGroup把其子节点包裹在一个span（或者自定义的component中），这是因为react的render函数必须返回单独子节点，但是TransitionGroup并不需要这样的规则。

然而，如果你需要只渲染一个单独子节点，你可以通过定义一个只渲染第一个子节点的组件：

```
function FirstChild(props) {
  const childrenArray = React.Children.toArray(props.children);
  return childrenArray[0] || null;
}
```

现在你可以指定TransitionGroup的component属性为FirstChild来避免包裹外层组件：

```
<TransitionGroup component={FirstChild}>
  {someCondition ? <MyComponent /> : null}
</TransitionGroup>
```

这仅仅在你只想要渲染一个子节点时有用，在需要渲染多个节点时没有效果。


