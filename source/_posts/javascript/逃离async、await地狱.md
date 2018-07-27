---

title: 逃离async、await地狱

date: 2018-4-16 09:32:49

tags: 

  - es6

---

async/await已经帮助我们逃离了回调函数的地狱，但是我们又陷入了async/await地狱。

## 什么是async/await地狱

当我们处理异步函数调用时，我们习惯在调用前添加一个await,然后一行接着一行，以同步的形式书写，但是这样就造成了我们的语句并不依赖前一个，你必须等待前一个语句的完成。

## 一个async/await地狱的示例

考虑你要订购pizza和drink，代码如下：

```
(async () => {
  const pizzaData = await getPizzaData()    // async call
  const drinkData = await getDrinkData()    // async call
  const chosenPizza = choosePizza()    // sync call
  const chosenDrink = chooseDrink()    // sync call
  await addPizzaToCart(chosenPizza)    // async call
  await addDrinkToCart(chosenDrink)    // async call
  orderItems()    // async call
})()
```

表面上看代码没有问题，但是这却不是有好的实践，因为它没有实现并发。

### 解释

我们已经使用[IIFE](https://developer.mozilla.org/en-US/docs/Glossary/IIFE)包装我们的代码，订购流程如下：


- 获取pizza列表.

- 获取drink列表.

- 从pizza列表中选择商品.

- 从drink中选择商品.

- 添加pizza到购物车.

- 添加drink到购物车.

- 下单.

### 那么，这样有什么问题吗？

就像之前说的，每句都是依赖上一句，这里并没有并发：pizza和drink应该可以并发运行的，pizza相关的工作和drink相关的工作可以并行进行，但涉及相关工作的各个步骤需要按顺序（逐个）进行。

### 另一个糟糕的示例

此JavaScript代码段将获取购物车中的商品并发出订购请求。

```
async function orderItems() {
  const items = await getCartItems()    // async call
  const noOfItems = items.length
  for(var i = 0; i < noOfItems; i++) {
    await sendRequest(items[i])    // async call
  }
}
```
每次循环我们都必须等待`sendRequest`的完成，但是，我们并不需要等待。我们希望尽快发送所有请求，然后我们可以等待所有请求完成。

## 怎么逃离async/await地狱

### 第一步：查找依赖于其他语句执行的语句

在我们的第一个例子中，我们选择了一个披萨和一杯饮料。我们的结论是，在选择比萨饼之前，我们需要有比萨饼的名单。在将比萨加入购物车之前，我们需要选择比萨饼。所以我们可以说这三个步骤取决于对方。在完成前一件事之前我们不能做一件事。但如果我们更广泛地来看，我们发现选择比萨不依赖于选择饮料，所以我们可以并行选择它们。这是机器可以做得比我们更好的一件事。因此我们发现了一些依赖于其他语句执行的语句，有些则没有。

### 第二步：把相关的操作独立进行封装

我们可以封装`selectPizza()`和`selectDrink()`。

### 第三步：并发运行相关操作函数

然后我们利用事件循环同时运行这些异步非阻塞函数。这样做的两种常见模式是return Promise和Promise.all方法。


## 让我们来解决示例中的问题

```
async function selectPizza() {
  const pizzaData = await getPizzaData()    // async call
  const chosenPizza = choosePizza()    // sync call
  await addPizzaToCart(chosenPizza)    // async call
}

async function selectDrink() {
  const drinkData = await getDrinkData()    // async call
  const chosenDrink = chooseDrink()    // sync call
  await addDrinkToCart(chosenDrink)    // async call
}

(async () => {
  const pizzaPromise = selectPizza()
  const drinkPromise = selectDrink()
  await pizzaPromise
  await drinkPromise
  orderItems()    // async call
})()

// Although I prefer it this way 

(async () => {
  Promise.all([selectPizza(), selectDrink()]).then(orderItems)   // async call
})()
```

针对循环中`sendRequest`的等待问题，我们使用Promise.all来并发请求：

```
async function orderItems() {
  const items = await getCartItems()    // async call
  const noOfItems = items.length
  const promises = []
  for(var i = 0; i < noOfItems; i++) {
    const orderPromise = sendRequest(items[i])    // async call
    promises.push(orderPromise)    // sync call
  }
  await Promise.all(promises)    // async call
}
```

