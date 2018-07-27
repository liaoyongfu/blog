---
title: jest测试工具学习（入门篇）
date: 2018-6-19 11:33:50
tag: jest
---

## 使用匹配器

### 普通匹配器

- `toBe`：测试值的方法是看是否精确匹配，相反的匹配器是`.not.toBe`;

- `toEqual`: 递归检查对象或数组的每个字段

### Truthiness

- toBeNull

- toBeUndefined

- toBeDefine

- toBeTruthy: 匹配任何 if 语句为真

- toBeFalsy: 匹配任何 if 语句为假

### 数字

大多数的比较数字有等价的匹配器。

```
test('two plus two', () => {
  const value = 2 + 2;
  expect(value).toBeGreaterThan(3);
  expect(value).toBeGreaterOrEqual(3.5);
  expect(value).toBeLessThan(5);
  expect(value).toBeLessOrEqual(4.5);
});
```

对于比较浮点数相等，使用 toBeCloseTo 而不是 toEqual，因为你不希望测试取决于一个小小的舍入误差。

```
test('两个浮点数字相加', () => {
  const value = 0.1 + 0.2;
  //expect(value).toBe(0.3);      这句会报错，因为浮点数有舍入误差
  expect(value).toBeCloseTo(0.3); // 这句可以运行
});
```

### 字符串

您可以检查对具有`toMatch`正则表达式的字符串︰

```
test('there is no I in team', () => {
  expect('team').not.toMatch(/I/);
});

test('but there is a "stop" in Christoph', () => {
  expect('Christoph').toMatch(/stop/);
});
```

### 数组

你可以检查数组是否包含特定子项使用 toContain︰

```
const shoppingList = [
  'diapers',
  'kleenex',
  'trash bags',
  'paper towels',
  'beer',
];

test('购物清单（shopping list）里面有啤酒（beer）', () => {
  expect(shoppingList).toContain('beer');
});
```

### 例外

如果你想要测试的特定函数抛出一个错误，在它调用时，使用 toThrow。

```
function compileAndroidCode() {
  throw new ConfigError('you are using the wrong JDK');
}

test('compiling android goes as expected', () => {
  expect(compileAndroidCode).toThrow();
  expect(compileAndroidCode).toThrow(ConfigError);

  // You can also use the exact error message or a regexp
  expect(compileAndroidCode).toThrow('you are using the wrong JDK');
  expect(compileAndroidCode).toThrow(/JDK/);
});
```

## 测试异步代码

### 回调

使用单个参数调用 done，而不是将测试放在一个空参数的函数。 Jest会等done回调函数执行结束后，结束测试。

```
test('the data is peanut butter', done => {
  function callback(data) {
    expect(data).toBe('peanut butter');
    done();
  }

  fetchData(callback);
});
```

如果 done()永远不会调用，这个测试将失败，这也是你所希望发生的。

### Promises

```
test('the data is peanut butter', () => {
  expect.assertions(1);
  return fetchData().then(data => {
    expect(data).toBe('peanut butter');
  });
});
```

一定要返回 Promise - 如果你省略 return 语句，您的测试将在 fetchData 完成之前完成。如果你想要 Promise 被拒绝，使用 .catch 方法。 请确保添加 expect.assertions 来验证一定数量的断言被调用。 否则一个fulfilled态的 Promise 不会让测试失败︰

```
test('the fetch fails with an error', () => {
  expect.assertions(1);
  return fetchData().catch(e => expect(e).toMatch('error'));
});
```

### .resolves / .rejects

您还可以使用 .resolves 匹配器在您期望的声明，Jest 会等待这一 Promise 来解决。如果 Promise 被拒绝，则测试将自动失败。

```
test('the data is peanut butter', () => {
  expect.assertions(1);
  return expect(fetchData()).resolves.toBe('peanut butter');
});
```

如果你想要 Promise 被拒绝，使用 .catch 方法。 它参照工程 .resolves 匹配器。 如果 Promise 被拒绝，则测试将自动失败。

```
test('the fetch fails with an error', () => {
  expect.assertions(1);
  return expect(fetchData()).rejects.toMatch('error');
});
```

### Async/Await

```
test('the data is peanut butter', async () => {
  expect.assertions(1);
  await expect(fetchData()).resolves.toBe('peanut butter');
});

test('the fetch fails with an error', async () => {
  expect.assertions(1);
  await expect(fetchData()).rejects.toMatch('error');
});
```

<!-- more -->

## 钩子函数

### 为多次测试重复设置

```
beforeEach(() => {
  return initializeCityDatabase();
});

afterEach(() => {
  return clearCityDatabase();
});

test('city database has Vienna', () => {
  expect(isCity('Vienna')).toBeTruthy();
});

test('city database has San Juan', () => {
  expect(isCity('San Juan')).toBeTruthy();
});
```

### 一次性设置

在某些情况下，你只需要在文件的开头做一次设置。 当这种设置是异步行为时，可能非常恼人，你不太可能一行就解决它。 Jest 提供 beforeAll 和 afterAll 处理这种情况。

```
beforeAll(() => {
  return initializeCityDatabase();
});

afterAll(() => {
  return clearCityDatabase();
});

test('city database has Vienna', () => {
  expect(isCity('Vienna')).toBeTruthy();
});

test('city database has San Juan', () => {
  expect(isCity('San Juan')).toBeTruthy();
});
```

### 作用域

默认情况下，before 和 after 的块可以应用到文件中的每个测试。 此外可以通过 describe 块来将测试分组。 当 before 和 after 的块在 describe 块内部时，则其只适用于该 describe 块内的测试。

比如说，我们不仅有一个城市的数据库，还有一个食品数据库。我们可以为不同的测试做不同的设置

```
// Applies to all tests in this file
beforeEach(() => {
  return initializeCityDatabase();
});

test('city database has Vienna', () => {
  expect(isCity('Vienna')).toBeTruthy();
});

test('city database has San Juan', () => {
  expect(isCity('San Juan')).toBeTruthy();
});

describe('matching cities to foods', () => {
  // Applies only to tests in this describe block
  beforeEach(() => {
    return initializeFoodDatabase();
  });

  test('Vienna <3 sausage', () => {
    expect(isValidCityFoodPair('Vienna', 'Wiener Schnitzel')).toBe(true);
  });

  test('San Juan <3 plantains', () => {
    expect(isValidCityFoodPair('San Juan', 'Mofongo')).toBe(true);
  });
});
```

```
beforeAll(() => console.log('1 - beforeAll'));
afterAll(() => console.log('1 - afterAll'));
beforeEach(() => console.log('1 - beforeEach'));
afterEach(() => console.log('1 - afterEach'));
test('', () => console.log('1 - test'));
describe('Scoped / Nested block', () => {
  beforeAll(() => console.log('2 - beforeAll'));
  afterAll(() => console.log('2 - afterAll'));
  beforeEach(() => console.log('2 - beforeEach'));
  afterEach(() => console.log('2 - afterEach'));
  test('', () => console.log('2 - test'));
});

// 1 - beforeAll
// 1 - beforeEach
// 1 - test
// 1 - afterEach
// 2 - beforeAll
// 1 - beforeEach
// 2 - beforeEach
// 2 - test
// 2 - afterEach
// 1 - afterEach
// 2 - afterAll
// 1 - afterAll
```

#### desribe和test块的执行顺序

Jest在执行任何实际测试之前执行所有描述处理程序的测试文件。这是在`before*`和`after*`的处理程序中进行设置和拆卸的另一个原因，而不是在描述块中,一旦描述块完成，默认情况下，Jest将按照它们在收集阶段遇到的顺序依次运行所有测试，等待每个测试完成并在继续之前进行整理。

```
describe('outer', () => {
  console.log('describe outer-a');

  describe('describe inner 1', () => {
    console.log('describe inner 1');
    test('test 1', () => {
      console.log('test for describe inner 1');
      expect(true).toEqual(true);
    });
  });

  console.log('describe outer-b');

  test('test 1', () => {
    console.log('test for describe outer');
    expect(true).toEqual(true);
  });

  describe('describe inner 2', () => {
    console.log('describe inner 2');
    test('test for describe inner 2', () => {
      console.log('test for describe inner 2');
      expect(false).toEqual(false);
    });
  });

  console.log('describe outer-c');
});

// describe outer-a
// describe inner 1
// describe outer-b
// describe inner 2
// describe outer-c
// test for describe inner 1
// test for describe outer
// test for describe inner 2
```

### 通用建议

如果测试失败，第一件要检查的事就是，当仅运行这条测试时，它是否仍然失败。 在 Jest 中很容易地只运行一个测试 — — 只需暂时将 test 命令更改为 test.only:

```
test.only('this will be the only test that runs', () => {
  expect(true).toBe(false);
});

test('this test will not run', () => {
  expect('A').toBe('A');
});
```

如果你有一个测试，当它作为一个更大的用例中的一部分时，经常运行失败，但是当你单独运行它时，并不会失败，所以最好考虑其他测试对这个测试的影响。 通常可以通过修改 beforeEach 来清除一些共享的状态来修复这种问题。 如果不确定某些共享状态是否被修改，还可以尝试在 beforeEach 中 log 数据来 debug。

## Mock函数

```
const mockCallback = jest.fn();
forEach([0, 1], mockCallback);

// The mock function is called twice
expect(mockCallback.mock.calls.length).toBe(2);

// The first argument of the first call to the function was 0
expect(mockCallback.mock.calls[0][0]).toBe(0);

// The first argument of the second call to the function was 1
expect(mockCallback.mock.calls[1][0]).toBe(1);

// The return value of the first call to the function was 42
expect(mockCallback.mock.results[0].value).toBe(42);
```

### .mock 属性

```
const myMock = jest.fn();

const a = new myMock();
const b = {};
const bound = myMock.bind(b);
bound();

console.log(myMock.mock.instances);
// > [ <a>, <b> ]
```

```
// The function was called exactly once
expect(someMockFunction.mock.calls.length).toBe(1);

// The first arg of the first call to the function was 'first arg'
expect(someMockFunction.mock.calls[0][0]).toBe('first arg');

// The second arg of the first call to the function was 'second arg'
expect(someMockFunction.mock.calls[0][1]).toBe('second arg');

// The return value of the first call to the function was 'return value'
expect(someMockFunction.mock.results[0].value).toBe('return value');

// This function was instantiated exactly twice
expect(someMockFunction.mock.instances.length).toBe(2);

// The object returned by the first instantiation of this function
// had a `name` property whose value was set to 'test'
expect(someMockFunction.mock.instances[0].name).toEqual('test');
```

### Mock 的返回值

```
const myMock = jest.fn();
console.log(myMock());
// > undefined

myMock
  .mockReturnValueOnce(10)
  .mockReturnValueOnce('x')
  .mockReturnValue(true);

console.log(myMock(), myMock(), myMock(), myMock());
// > 10, 'x', true, true
```

```
const filterTestFn = jest.fn();

// Make the mock return `true` for the first call,
// and `false` for the second call
filterTestFn.mockReturnValueOnce(true).mockReturnValueOnce(false);

const result = [11, 12].filter(filterTestFn);

console.log(result);
// > [11]
console.log(filterTestFn.mock.calls);
// > [ [11], [12] ]
```

### Mocking Modules

```
// users.test.js
import axios from 'axios';
import Users from './users';

jest.mock('axios');

test('should fetch users', () => {
  const resp = {data: [{name: 'Bob'}]};
  axios.get.mockResolvedValue(resp);

  // or you could use the following depending on your use case:
  // axios.get.mockImplementation(() => Promise.resolve(resp))

  return Users.all().then(users => expect(users).toEqual(resp.data));
});
```

### Mock 实现

```
const myMockFn = jest.fn(cb => cb(null, true));

myMockFn((err, val) => console.log(val));
// > true

myMockFn((err, val) => console.log(val));
// > true
```

```
// foo.js
module.exports = function() {
  // some implementation;
};

// test.js
jest.mock('../foo'); // this happens automatically with automocking
const foo = require('../foo');

// foo is a mock function
foo.mockImplementation(() => 42);
foo();
// > 42
```

```
const myMockFn = jest
  .fn()
  .mockImplementationOnce(cb => cb(null, true))
  .mockImplementationOnce(cb => cb(null, false));

myMockFn((err, val) => console.log(val));
// > true

myMockFn((err, val) => console.log(val));
// > false
```

```
onst myMockFn = jest
  .fn(() => 'default')
  .mockImplementationOnce(() => 'first call')
  .mockImplementationOnce(() => 'second call');

console.log(myMockFn(), myMockFn(), myMockFn(), myMockFn());
// > 'first call', 'second call', 'default', 'default'
```

```
const myObj = {
  myMethod: jest.fn().mockReturnThis(),
};

// is the same as

const otherObj = {
  myMethod: jest.fn(function() {
    return this;
  }),
};
```

### Mock 名称

```
const myMockFn = jest
  .fn()
  .mockReturnValue('default')
  .mockImplementation(scalar => 42 + scalar)
  .mockName('add42');
```

### 自定义匹配器

```
// 这个 mock 函数至少被调用一次
expect(mockFunc).toBeCalled();

// 这个 mock 函数至少被调用一次，而且传入了特定参数
expect(mockFunc).toBeCalledWith(arg1, arg2);

// 这个 mock 函数的最后一次调用传入了特定参数
expect(mockFunc).lastCalledWith(arg1, arg2);

// 所有的 mock 的调用和名称都被写入了快照
expect(mockFunc).toMatchSnapshot();
```

```
// 这个 mock 函数至少被调用一次
expect(mockFunc.mock.calls.length).toBeGreaterThan(0);

// 这个 mock 函数至少被调用一次，而且传入了特定参数
expect(mockFunc.mock.calls).toContain([arg1, arg2]);

// 这个 mock 函数的最后一次调用传入了特定参数
expect(mockFunc.mock.calls[mockFunc.mock.calls.length - 1]).toEqual([
  arg1,
  arg2,
]);

//  这个 mock 函数的最后一次调用的第一个参数是`42`
// （注意这个断言的规范是没有语法糖的）
expect(mockFunc.mock.calls[mockFunc.mock.calls.length - 1][0]).toBe(42);

// 快照会检查 mock 函数被调用了同样的次数，
// 同样的顺序，和同样的参数 它还会在名称上断言。
expect(mockFunc.mock.calls).toEqual([[arg1, arg2]]);
expect(mockFunc.mock.getMockName()).toBe('a mock name');
```

## Jest平台工具

您可以挑选Jest的特定功能，并将它们作为独立软件包使用。以下是可用软件包的列表：

### jest-changed-files

用于识别git / hg存储库中已修改文件的工具。出口两个功能：

getChangedFilesForRoots 返回一个承诺，解析为更改后的文件和回购对象。
findRepos 返回一个承诺，解析为包含在指定路径中的一组存储库。

```
const {getChangedFilesForRoots} = require('jest-changed-files');

// 打印出当前目录最后修改过的一组文件
getChangedFilesForRoots(['./'], {
  lastCommit: true,
}).then(result => console.log(result.changedFiles));
```

### jest-diff

用于可视化数据更改的工具

```
const diff = require('jest-diff');

const a = {a: {b: {c: 5}}};
const b = {a: {b: {c: 6}}};

const result = diff(a, b);

// print diff
console.log(result);
```

### jest-docblock

```
const {parseWithComments} = require('jest-docblock');

const code = `
/**
 * This is a sample
 *
 * @flow
 */

 console.log('Hello World!');
`;

const parsed = parseWithComments(code);

// prints an object with two attributes: comments and pragmas.
console.log(parsed);
```

### jest-get-type

```
const getType = require('jest-get-type');

const array = [1, 2, 3];
const nullValue = null;
const undefinedValue = undefined;

// prints 'array'
console.log(getType(array));
// prints 'null'
console.log(getType(nullValue));
// prints 'undefined'
console.log(getType(undefinedValue));
```

### jest-validate

用于验证用户提交的配置的工具

```
const {validate} = require('jest-validate');

const configByUser = {
  transform: '<rootDir>/node_modules/my-custom-transform',
};

const result = validate(configByUser, {
  comment: '  Documentation: http://custom-docs.com',
  exampleConfig: {transform: '<rootDir>/node_modules/babel-jest'},
});

console.log(result);
```

### jest-worker

用于任务并行化的模块

```
// heavy-task.js

module.exports = {
  myHeavyTask: args => {
    // long running CPU intensive task.
  },
};
```

```
// main.js

async function main() {
  const worker = new Worker(require.resolve('./heavy-task.js'));

  // run 2 tasks in parallel with different arguments
  const results = await Promise.all([
    worker.myHeavyTask({foo: 'bar'}),
    worker.myHeavyTask({bar: 'foo'}),
  ]);

  console.log(results);
}

main();
```

### pretty-format

导出一个将任何JavaScript值转换为可读的字符串的函数。支持所有内置的JavaScript类型，并允许通过用户定义的插件扩展特定于应用程序的类型。

```
const prettyFormat = require('pretty-format');

const val = {object: {}};
val.circularReference = val;
val[Symbol('foo')] = 'foo';
val.map = new Map([['prop', 'value']]);
val.array = [-0, Infinity, NaN];

console.log(prettyFormat(val));
```