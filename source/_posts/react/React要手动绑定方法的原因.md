---
title: React要手动绑定方法的原因
date: 2017-08-02 14:09:44
tags:
    - react
---

## 我们从javascript开始吧

在js中，函数的上下文是指函数调用的时候，而不是定义的时候。

有以下四中调用函数的模式：

- 函数调用模式

- 方法调用模式

- 构造函数调用模式

- 应用调用模式

所有这些使用的模式都不同地定义函数上下文。接下来我们看看各种模式的区别。

<!-- more -->

### 函数调用模式

定义：如果在调用时没有.操作，那么上下文可能为window。

调用函数最直接的方法就是直接调用它：

```
var func = function(){
    //...
};
func();
```

这时的上下文（this）将会设置成javascript操作环境的全局变量，在浏览器中，它是window变量。

我们再来看另一个例子：

```
var unicorns = {
  func: function() { // ... }
};
var fun = unicorns.func;
fun();
```

你认为在func中的上下文为uniconrns对象？那是错误的，由于上下文时通过调用此函数时确定的，所以这里的上下文还是window。

### 方法调用模式

定义：如果函数调用中有点操作，则上下文将会是一序列点中最右边的那个变量。

如上面的例子中，如果我们直接调用unicorns.func()，上下文会是unicorns对象。

```
var frog = {
  RUN_SOUND: "POP!!",
  run: function() {
    return this.RUN_SOUND;
  }
};
frog.run(); // returns "POP!!" since this points to the `frog` object.
var runningFun = frog.run;
runningFun(); // returns "undefined" since this points to the window
```

### 构造函数模式

定义：每次看到一个new函数名后，你this将指向一个新创建的新对象。

```
function Wizard() {
  this.castSpell = function() { return "KABOOM"; }
}
```

直接调用它将会是window（因为它是一个函数调用），但是如果通过new来调用：

```
function Wizard() {
  this.castSpell = function() { return "KABOOM"; };
}
var merlin = new Wizard(); // this is set to an empty object {}. Returns `this` implicitly.
merlin.castSpell() // returns "KABOOM";
```

这将会发生两件事：

- 函数将会有一个指向当前对象的上下文this。

- 如果没有指定return或者这个函数返回一个非对象值，this将从这个函数返回。

### 应用调用模式

当你对函数有引用的时候，你可以通过两种方法来手动提供上下文：

- call

- apply

```
function addAndSetX(a, b) {
  this.x += a + b;
}
var obj1 = { x: 1, y: 2 };
addAndSetX.call(obj1, 1, 1); // this = obj1, obj1 after call = { x: 3, y : 2 }
// It is the same as:
// addAndSetX.apply(obj1, [1, 1]);
```

如果您需要调用从某个其他地方传递的函数（例如，作为参数到函数中）与某个上下文对象，这是非常方便的。它不是非常可用于异步回调，因为绑定与一个函数调用相结合。

要使用回调设置正确的上下文，您可能需要另一种方便的技术 - 您可以从中创建有界函数。

### 绑定功能

有界函数是一个限定给定上下文的函数，这意味着无论你怎么调用它，它的上下文都是不变的。唯一例外是总是返回一个新上下文的new运算符。

要是普通函数变成有界函数，应该使用bind方法，bind方法将您要将函数绑定到的上下文作为第一个参数。其余的参数是将始终传递给这样的函数的参数。

结果返回有界函数。我们来看一个例子：

```
function add(x, y) {
  this.result += x + y;
}
var computation1 = { result: 0 };
var boundedAdd = add.bind(computation1);
boundedAdd(1, 2); // `this` is set to `computation1`.
                  //  computation1 after call: { result: 3 }
var boundedAddPlusTwo = add.bind(computation1, 2);
boundedAddPlusTwo(4); // `this` is set to `computation1`.
                      // computation1 after call: { result: 9 }
```

被绑定了的函数甚至不能在通过call或apply改变上下文：

```
var obj = { boundedPlusTwo: boundedAddPlusTwo };
obj.boundedPlusTwo(4); // `this` is set to `computation1`.
                       // even though method is called on `obj`.
                       // computation1 after call: { result: 15 }
var computation2 = { result: 0 };
boundedAdd.call(computation2, 1, 2); // `this` is set to `computation1`.
                                     // even though context passed to call is
                                     // `computation2`
                                     // computation1 after call: { result: 18 }
```

您现在已经掌握了关于JavaScript的知识，现在让我们来看react中的情况。

## 怎么绑定以及绑定什么

ECMAScript 2015（ECMAScript 6）引入了一种新的类语法，可用于创建React组件类。实际上，这个类语法是面向对象JavaScript 的旧的原型系统的语法糖。

这意味着ES2015类中的函数上下文调用遵循与其余JavaScript相同的原则。

```
class Foo {
  constructor() {
    this.x = 2;
    this.y = 4;
  }
  bar() {
    // ...
  }
  baz() {
    // ...
  }
}
```

与以下大致相同：

```
function Foo() {
  this.x = 2;
  this.y = 4;
  this.bar = function() { // ... };
  this.baz = function() { // ... };
}
```

记住这只是一个简化。在确定函数上下文调用的情况下，这个更复杂的逻辑遵循与上面的代码片段相同的原理。

### React.createClass

在这个语法下，绑定问题是不存在的，在传递给对象的对象中定义的所有方法React.createClass将自动绑定到组件实例。这意味着你可以随时使用setState，访问props和state等等这些方法。

尽管在99％的情况下可能完全可以接受，但它限制了您对任意设置上下文的能力 - 这可能是更复杂的代码库中的一个大问题。

### ECMAScript 2015 classes

在ECMAScript 2015 classes写法中，你需要手动绑定方法。

以下是React库中是可以识别为方法调用模式执行调用：

- 组件生命周期方法。它仅仅通过component.componentDidUpdate(...)方式调用（因此，this已经正确绑定到组件实例本身）。

- render方法。它也是被识别为方法调用模式执行调用。大多数的非事件处理函数在render方法中调用，它已经被自动绑定到组件实例，所以你可以放心使用。

但是，传递给事件处理属性的函数可能有许多来源，甚至可能通过顶级组件的属性从非React级别传递给他们。

在React.createClassReact假定它们来自您的组件并自动绑定它们。但是在ES2015 classes中你有自由。在引擎中，它们被以函数调用模式调用。

这意味这，在默认情况下，你无法在事件处理程序中读取组件属性、状态和组件的方法，为此，你需要明确地绑定他们。

绑定事件处理程序的最佳位置是构造函数：

```
class InputExample extends React.Component {
  constructor(props) {
    super(props);
    this.state = { text: '' };
    this.change = this.change.bind(this);
  }
  change(ev) {
    this.setState({ text: ev.target.value });
  }
  render() {
    let { text } = this.state;
    return (<input type="text" value={text} onChange={this.change} />);
  }
}
```

这样你的事件处理程序的上下文将会绑定到组件实例中。

### 类属性

有一个实验功能，称为类属性，可以帮助您明确避免绑定方法。它是用于在构造函数中定义字段和函数的语法糖。看起来像这样：

```
class InputExample extends React.Component {
  state = { text: '' };
  // ...
}
```

并编译成以下：

```
class InputExample extends React.Component {
  constructor(...arguments) {
    super(...arguments);
    this.state = { text: '' };
  }
  // ...
}
```

那么怎么定义一个方法呢？

```
class InputExample extends React.Component {
  state = { text: '' };
  change = function(ev) {
    this.setState({ text: ev.target.value });
  };
  // ...
}
```

所以现在，你得到一个等同于以下类：

```
class InputExample extends React.Component {
  constructor(...arguments) {
    super(...arguments);
    this.state = { text: '' };
    this.change = function(ev) {
      this.setState({ text: ev.target.value });
    };
  }
  // ...
}
```

但是这样有一个问题，this.change函数上下文还是错误的，所以我们要结合箭头函数：

```
class InputExample extends React.Component {
  state = { text: '' };
  change = ev => this.setState({text: ev.target.value});
  render() {
    let {text} = this.state;
    return (<input type="text" value={text} onChange={this.change} />);
  }
}
```

该解决方案的缺点是类属性仍处于实验阶段。这意味着此功能可以在ECMAScript 2016（也称为ECMAScript 7或ES7）的后续迭代中被删除，而不会发出警告。

### createClass以及class语法编译完的不同

我们先来看类写法：

```
class Todo extends Component{
    handleClick(){
        console.info(this);
    }
    method(){
        console.info(this);
    }
    render(){
        this.method();
        return (
            <div>
                <p onClick={this.handleClick}>Hello</p>
            </div>
        )
    }
}
```

编译完：

```
var Todo = function (_Component) {
    _inherits(Todo, _Component);
    function Todo() {
        _classCallCheck(this, Todo);
        return _possibleConstructorReturn(this, _Component.apply(this, arguments));
    }
    Todo.prototype.handleClick = function handleClick() {
        console.info(this);
    };
    Todo.prototype.method = function method() {
        console.info(this);
    };
    Todo.prototype.render = function render() {
        this.method();
        return __WEBPACK_IMPORTED_MODULE_0_react___default.a.createElement(
            'div',
            null,
            __WEBPACK_IMPORTED_MODULE_0_react___default.a.createElement(
                'p',
                { onClick: this.handleClick },
                'Hello'
            )
        );
    };
    return Todo;
}(__WEBPACK_IMPORTED_MODULE_0_react__["Component"]);
```

this.handleClick被放在{onClick: this.handleClick}中，所以当被调用的时候会被识别为函数调用模式，所以这时的上下文是null（为什么不是window或其他的？？？）