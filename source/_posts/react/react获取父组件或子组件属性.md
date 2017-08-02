---
title: React获取父组件或子组件属性
date: 2017-08-02 14:09:44
tags:
    - react
---

## 获取子组件的方法

可以通过递归this.props.children中得到

<!-- more -->

## 获取父组件的方法

方法一：可以通过react内部私有函数this._reactInternalInstance._currentElement._owner._instance获取

```
var Parent = React.createClass({
  render() {
      return <Child v="test" />;
  },
  doAThing() {
    console.log("I'm the parent, doing a thing.", this.props.testing);
  }
});
var Child = React.createClass({
  render() {
    return <button onClick={this.onClick}>{this.props.v}</button>
  },
  onClick() {
    var parent = this._reactInternalInstance._currentElement._owner._instance;
    console.log("parent:", parent);
    parent.doAThing();
  }
});
ReactDOM.render(<Parent testing={true} />, container);
```

但是这种方法是不推荐的。

方法二：通过属性传递给子组件

```
class Parent extends React.Component {
    constructor(props) {
        super(props)
        this.fn = this.fn.bind(this)
    }
    fn() {
        console.log('parent')
    }
    render() {
        return <Child fn={this.fn} />
    }
}
const Child = ({ fn }) => <button onClick={fn}>Click me!</button>
```

但是这种只在Child组件在Parent组件中时才可以用。

方法三：使用上下文（没有直接的父/子关系）

```
class Parent extends React.Component {
    constructor(props) {
        super(props)
        this.fn = this.fn.bind(this)
    }
    getChildContext() {
        return {
            fn: this.fn,
        }
    }
    fn() {
        console.log('parent')
    }
    render() {
        return <Child fn={this.fn} />
    }
}
Parent.childContextTypes = {
    fn: React.PropTypes.func,
}
const Child = (props, { fn }) => <button onClick={fn}>Click me!</button>
Child.contextTypes = {
    fn: React.PropTypes.func,
}
```

## 给子组件（没有直接父/子关系）添加属性

```
return React.cloneElement(this.props.children, {/*要添加的属性*/})
```

## React关于子组件的API

### React.chlidren.map

```
React.Children.map(children, function[(thisArg)])
```

对包含在 children 中的每个直接子元素调用一个函数，使用 this 设置 thisArg 。 如果 children 是一个键片段（keyed fragment）或数组，它将被遍历：该函数永远不会传递容器对象（container objects）。 如果 children 为 null 或 undefined ，返回 null 或 undefined，而不是一个数组。

```
import React from 'react';
const Salmonize = ({ children }) => (
  <div>
    {React.Children.map(children, child => (
      React.cloneElement(child, {
        style: {
          backgroundColor: 'salmon',
          color: 'seagreen',
        }
      })
    ))}
  </div>
);
const SalmonBlog = ({ title, posts }) => (
  <div>
    <Salmonize>
      <NavBar title={title} />
    </Salmonize>
    {posts.map(post => (
      <Post key={post.id}>
        <Salmonize>
          <PostHeader title={post.title} />
        </Salmonize>
        <PostBody text={post.text} />
      </Post>
    ))}
  </div>
);
```

在上面这个例子中，Salmonize组件并不需要在乎谁是它的子组件，它通过遍历克隆每个子组件，通过React.cloneElement给子组件增加属性。

在React中编写真正可重复使用的组件肯定有点棘手，如果你遇到这样的麻烦，那么这种map和clone方法可以帮助到你。

### React.children.forEach

```
React.Children.forEach(children, function[(thisArg)])
```

类似React.children.map，但是没有返回值。

### React.Children.count

```
React.Children.count(children)
```

返回 children 中的组件总数，等于传递给 map 或 forEach 的回调将被调用的次数。

### React.Children.only

```
React.Children.only(children)
```

返回 children 中的唯一子集。否则抛出异常。当您想要确保组件只有一个子级时，这可能会派上用场，如果不满足此条件，则会抛出错误。

```
export default React.createClass({
  // ...
  render: function() {
    const {name, selectedValue, onChange, children} = this.props;
    const renderedChildren = children(radio(name, selectedValue, onChange));
    return renderedChildren && React.Children.only(renderedChildren);
  }
});
```

### React.Children.toArray

```
React.Children.toArray(children)
```

将 children 不透明数据结构作为一个平面数组返回，并且 key 分配给每个子集。 如果你想在渲染方法中操作children集合，特别是如果你想在传递它之前重新排序或切割 this.props.children ，这将非常有用。

该方法将children组件的支持转换为纯JavaScript数组，这可以使您比React.Children.map提供更多的灵活性。React.Children.toArray最近在我想要渲染一个分隔符的列表中时，它们之间散布着很方便。这导致我创建一个完成这个的组件。

```
import React from 'react';
const IntersperseDividers = ({ children }) => (
  <div>
    {React.Children.toArray(children).reduce((elements, child, i, array) => {
      elements.push(child);
      if (i < array.length - 1) {
        elements.push(<hr key={`${i}--divider`} />);
      }
      return elements;
    }, [])}
  </div>
);
const List = ({ data }) => (
  <IntersperseDividers>
    {data.map((item, i) => (
      <div key={i}>
        {item.value}
      </div>
    ))}
  </IntersperseDividers>
);
```

