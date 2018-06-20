---
title: react学习总结

time: 2018-4-2 16:55:58

tag: 

    - react
---

## 协调（Reconciliation）

当你使用React，在单一时间点你可以考虑render()函数作为创建React元素的树。在下一次状态或属性更新，render()函数将返回一个不同的React元素的树。React需要算出如何高效更新UI以匹配最新的树。

有一些解决将一棵树转换为另一棵树的最小操作数算法问题的通用方案。然而，树中元素个数为n，最先进的算法的时间复杂度为O(n 3 ) 。

若我们在React中使用，展示1000个元素则需要进行10亿次的比较。这操作太过昂贵，相反，React基于两点假设，实现了一个启发的O(n)算法：

1. 两个不同类型的元素将产生不同的树。

2. 通过渲染器附带key属性，开发者可以示意哪些子元素可能是稳定的。

实践中，上述假设适用于大部分应用场景。

<!-- more -->

### 对比算法

当对比两棵树时，React首先比较两个根节点。根节点的type不同，其行为也不同。

#### 不同类型的元素

每当根元素有不同类型，React将卸载旧树并重新构建新树。

当树被卸载，旧的DOM节点将被销毁。组件实例会调用componentWillUnmount()。当构建一棵新树，新的DOM节点被插入到DOM中。组件实例将依次调用componentWillMount()和componentDidMount()。任何与旧树有关的状态都将丢弃。

这个根节点下所有的组件都将会被卸载，同时他们的状态将被销毁。例如，以下节点对比之后：

```
<div>
  <Counter />
</div>

<span>
  <Counter />
</span>
```

这将会销毁旧的Counter并重装新的Counter。

#### 相同类型的DOM元素

当比较两个相同类型的React DOM元素时，React则会观察二者的属性，保持相同的底层DOM节点，并仅更新变化的属性。例如：

```
<div className="before" title="stuff" />

<div className="after" title="stuff" />
```

通过比较两个元素，React知道仅更改底层DOM元素的className。

当更新style时，React同样知道仅更新变更的属性。例如：

```
<div style={{color: 'red', fontWeight: 'bold'}} />

<div style={{color: 'green', fontWeight: 'bold'}} />
```

当在调整两个元素时，React知道仅改变color样式而不是fontWeight。

在处理完DOM元素后，React递归其子元素。

#### 相同类型的组件元素

当组件更新时，实例仍保持一致，以让状态能够在渲染之间保留。React通过更新底层组件实例的props来产生新元素，并在底层实例上依次调用componentWillReceiveProps()和componentWillUpdate()方法。

接下来，render()方法被调用，同时对比算法会递归处理之前的结果和新的结果。

### 权衡

牢记协调算法的实现细节非常重要。React可能会在每次操作时渲染整个应用；而结果仍是相同的。为保证大多数场景效率能更快，我们通常提炼启发式的算法。

在目前实现中，可以表明一个事实，即子树在其兄弟节点中移动，但你无法告知其移动到哪。该算法会重渲整个子树。

由于React依赖于该启发式算法，若其背后的假设没得到满足，则其性能将会受到影响：

- 算法无法尝试匹配不同组件类型的子元素。若你发现两个输出非常相似的组件类型交替出现，你可能希望使其成为相同类型。实践中，我们并非发现这是一个问题。

- Keys应该是稳定的，可预测的，且唯一的。不稳定的key（类似由Math.random()生成的）将使得大量组件实例和DOM节点进行不必要的重建，使得性能下降并丢失子组件的状态。


## Context（React v16.3.0）

### 创建生产者

```
React.createContext(/* some value */)
```

### 消费者

```
<Consumer>
  {value => /* render something based on the context value */}
</Consumer>
```

### 示例

```
// Create a theme context, defaulting to light theme
const ThemeContext = React.createContext('light');

function ThemedButton(props) {
  // The ThemedButton receives the theme from context
  return (
    <ThemeContext.Consumer>
      {theme => <Button {...props} theme={theme} />}
    </ThemeContext.Consumer>
  );
}

// An intermediate component
function Toolbar(props) {
  return (
    <div>
      <ThemedButton />
    </div>
  );
}

class App extends React.Component {
  render() {
    return (
      <ThemeContext.Provider value="dark">
        <Toolbar />
      </ThemeContext.Provider>
    );
  }
}
```

在生命周期中，可以通过`this.props`访问：

```
class Button extends React.Component {
  componentDidMount() {
    // ThemeContext value is this.props.theme
  }

  componentDidUpdate(prevProps, prevState) {
    // Previous ThemeContext value is prevProps.theme
    // New ThemeContext value is this.props.theme
  }

  render() {
    const {theme, children} = this.props;
    return (
      <button className={theme ? 'dark' : 'light'}>
        {children}
      </button>
    );
  }
}

export default props => (
  <ThemeContext.Consumer>
    {theme => <Button {...props} theme={theme} />}
  </ThemeContext.Consumer>
);
```

### 创建消费HOC组件

```
const ThemeContext = React.createContext('light');

// This function takes a component...
export function withTheme(Component) {
  // ...and returns another component...
  return function ThemedComponent(props) {
    // ... and renders the wrapped component with the context theme!
    // Notice that we pass through any additional props as well
    return (
      <ThemeContext.Consumer>
        {theme => <Component {...props} theme={theme} />}
      </ThemeContext.Consumer>
    );
  };
}
```

现在就可以简单用了：

```
function Button({theme, ...rest}) {
  return <button className={theme} {...rest} />;
}

const ThemedButton = withTheme(Button);
```

### ref引用到被包装组件（React v16.3.0）

`v16.3.0`引入`React.createRef`和`React.forwardRef`新语法：

```
// fancy-button.js
class FancyButton extends React.Component {
  focus() {
    // ...
  }

  // ...
}

// Use context to pass the current "theme" to FancyButton.
// Use forwardRef to pass refs to FancyButton as well.
export default React.forwardRef((props, ref) => (
  <ThemeContext.Consumer>
    {theme => (
      <FancyButton {...props} theme={theme} ref={ref} />
    )}
  </ThemeContext.Consumer>
));
```

```
// app.js
import FancyButton from './fancy-button';

const ref = React.createRef();

// Our ref will point to the FancyButton component,
// And not the ThemeContext.Consumer that wraps it.
// This means we can call FancyButton methods like ref.current.focus()
<FancyButton ref={ref} onClick={handleClick}>
  Click me!
</FancyButton>;
```

## Fragments

Fragments 看起来像空的JSX 标签：

```
render() {
  return (
    <>
      <ChildA />
      <ChildB />
      <ChildC />
    </>
  );
}
```

另一种使用片段的方式是使用React.Fragment组件，React.Fragment组件可以在React对象上使用。这可能是必要的，如果你的工具还不支持JSX片段。注意在React中，<></>是<React.Fragment/>的语法糖。

```
class Columns extends React.Component {
  render() {
    return (
      <React.Fragment>
        <td>Hello</td>
        <td>World</td>
      </React.Fragment>
    );
  }
}
```

<></> 语法不能接受键值或属性。

如果你需要一个带key的片段，你可以直接使用<React.Fragment />。一个使用场景是映射一个集合为一个片段数组—例如：创建一个描述列表：

```
function Glossary(props) {
  return (
    <dl>
      {props.items.map(item => (
        // 没有`key`，将会触发一个key警告
        <React.Fragment key={item.id}>
          <dt>{item.term}</dt>
          <dd>{item.description}</dd>
        </React.Fragment>
      ))}
    </dl>
  );
}
```

## Portals

Portals 提供了一种很好的将子节点渲染到父组件以外的DOM 节点的方式：

```
ReactDOM.createPortal(child, container)
```

第一个参数（child）是任何可渲染的React子元素，例如一个元素，字符串或碎片。第二个参数（container）则是一个DOM元素。

> 对于portal的一个典型用例是当父组件有overflow: hidden或z-index样式，但你需要子组件能够在视觉上“跳出（break out）”其容器。例如，对话框、hovercards以及提示框。

尽管portal可以被放置在DOM树的任何地方，但在其他方面其行为和普通的React子节点行为一致。如上下文特性依然能够如之前一样正确地工作，无论其子节点是否是portal，由于portal仍存在于React树中，而不用考虑其在DOM树中的位置。

这包含事件冒泡。一个从portal内部会触发的事件会一直冒泡至包含React树的祖先。假设如下HTML结构：

```
<html>
  <body>
    <div id="app-root"></div>
    <div id="modal-root"></div>
  </body>
</html>
```

在#app-root里的Parent组件能够捕获到未被捕获的从兄弟节点#modal-root冒泡上来的事件。

```
// These two containers are siblings in the DOM
const appRoot = document.getElementById('app-root');
const modalRoot = document.getElementById('modal-root');

class Modal extends React.Component {
  constructor(props) {
    super(props);
    this.el = document.createElement('div');
  }

  componentDidMount() {
    modalRoot.appendChild(this.el);
  }

  componentWillUnmount() {
    modalRoot.removeChild(this.el);
  }

  render() {
    return ReactDOM.createPortal(
      this.props.children,
      this.el,
    );
  }
}

class Parent extends React.Component {
  constructor(props) {
    super(props);
    this.state = {clicks: 0};
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    // This will fire when the button in Child is clicked,
    // updating Parent's state, even though button
    // is not direct descendant in the DOM.
    this.setState(prevState => ({
      clicks: prevState.clicks + 1
    }));
  }

  render() {
    return (
      <div onClick={this.handleClick}>
        <p>Number of clicks: {this.state.clicks}</p>
        <p>
          Open up the browser DevTools
          to observe that the button
          is not a child of the div
          with the onClick handler.
        </p>
        <Modal>
          <Child />
        </Modal>
      </div>
    );
  }
}

function Child() {
  // The click event on this button will bubble up to parent,
  // because there is no 'onClick' attribute defined
  return (
    <div className="modal">
      <button>Click</button>
    </div>
  );
}

ReactDOM.render(<Parent />, appRoot);
```

在父组件里捕获一个来自portal的事件冒泡能够在开发时具有不完全依赖于portal的更为灵活的抽象。例如，若你在渲染一个<Modal />组件，父组件能够捕获其事件而无论其是否采用portal实现。

## Error Boundaries

错误边界是用于捕获其子组件树JavaScript异常，记录错误并展示一个回退的UI的React组件，而不是整个组件树的异常。错误组件在渲染期间，生命周期方法内，以及整个组件树构造函数内捕获错误。

如果一个类组件定义了一个名为componentDidCatch(error, info):的新方法，则其成为一个错误边界：

```
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  componentDidCatch(error, info) {
    // Display fallback UI
    this.setState({ hasError: true });
    // You can also log the error to an error reporting service
    logErrorToMyService(error, info);
  }

  render() {
    if (this.state.hasError) {
      // You can render any custom fallback UI
      return <h1>Something went wrong.</h1>;
    }
    return this.props.children;
  }
}
```

而后你可以像一个普通的组件一样使用：

```
<ErrorBoundary>
  <MyWidget />
</ErrorBoundary>
```

componentDidCatch()方法机制类似于JavaScript catch {}，但是针对组件。仅有类组件可以成为错误边界。实际上，大多数时间你仅想要定义一个错误边界组件并在你的整个应用中使用。

注意错误边界仅可以捕获其子组件的错误。错误边界无法捕获其自身的错误。如果一个错误边界无法渲染错误信息，则错误会向上冒泡至最接近的错误边界。这也类似于JavaScript中catch {}的工作机制。

#### componentDidCatch 参数

error 是被抛出的错误。

info是一个含有componentStack属性的对象。这一属性包含了错误期间关于组件的堆栈信息。

```
//...
componentDidCatch(error, info) {
  
  /* Example stack information:
     in ComponentThatThrows (created by App)
     in ErrorBoundary (created by App)
     in div (created by App)
     in App
  */
  logComponentStackToMyService(info.componentStack);
}

//...
```

### Test Utilities


## 未来计划

- 16.3：介绍别名为不安全的生命周期，UNSAFE_componentWillMount，UNSAFE_componentWillReceiveProps，和UNSAFE_componentWillUpdate。（旧的生命周期名称和新的别名都可以在此版本中使用。）

- 未来的16.x版本：启用弃用警告componentWillMount，componentWillReceiveProps和componentWillUpdate。（旧的生命周期名称和新的别名都可以在此版本中使用，但旧名称会记录DEV模式警告。）

- 17.0：删除componentWillMount，componentWillReceiveProps和componentWillUpdate。（从现在开始，只有新的“UNSAFE_”生命周期名称将起作用。）