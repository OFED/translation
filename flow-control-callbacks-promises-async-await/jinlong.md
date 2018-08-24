# 现代 JS 流程控制：从回调函数到 Promises 再到 Async/Await

> 原文链接：https://www.sitepoint.com/flow-control-callbacks-promises-async-await/
>
> 译者：[OFED]()


## Promises

[ES2015(ES6) 引入了 Promises](https://www.sitepoint.com/overview-javascript-promises/)。回调函数依然有用，但是 Promises 提供了更清晰的链式异步命令语法，因此可以串联运行（[下个章节](https://www.sitepoint.com/flow-control-callbacks-promises-async-await/#asynchronouschaining)会讲）。

打算基于 Promise 封装，异步回调函数必须返回一个 Promise 对象。Promise 对象会执行以下两个函数（作为参数传递的）其中之一：

- `resolve`：执行成功回调
- `reject`：执行失败回调

以下例子，database API 提供了一个 `connect()` 方法，接收一个回调函数。外部的 `asyncDBconnect()` 函数立即返回了一个新的 Promise，一旦连接创建成功或失败，`resolve()` 或 `reject()` 便会执行：

```js
const db = require('database');

// 连接数据库
function asyncDBconnect(param) {

  return new Promise((resolve, reject) => {

    db.connect(param, (err, connection) => {
      if (err) reject(err);
      else resolve(connection);
    });

  });

}
```

Node.js 8.0 以上提供了 [util.promisify() 功能](https://nodejs.org/api/util.html#util_util_promisify_original)，可以把基于回调的函数转换成基于 Promise 的。有两个使用条件：

1. 传入一个唯一的异步函数
1. 传入的函数希望是错误优先的（比如：(err, value) => ...），error 参数在前，value 随后

举例：

```js
// Node.js: 把 fs.readFile promise 化
const
  util = require('util'),
  fs = require('fs'),
  readFileAsync = util.promisify(fs.readFile);

readFileAsync('file.txt');
```

各种库都会提供自己的 promisify 方法，寥寥几行也可以自己撸一个：

```js
// promisify 只接收一个函数参数
// 传入的函数接收 (err, data) 参数
function promisify(fn) {
  return function() {
      return new Promise(
        (resolve, reject) => fn(
          ...Array.from(arguments),
        (err, data) => err ? reject(err) : resolve(data)
      )
    );
  }
}

// 举例
function wait(time, callback) {
  setTimeout(() => { callback(null, 'done'); }, time);
}

const asyncWait = promisify(wait);

ayscWait(1000);
```

### 异步链式调用

任何返回 Promise 的函数都可以通过 `.then()` 链式调用。前一个 `resolve` 的结果会传递给后一个：

```js
asyncDBconnect('http://localhost:1234')
  .then(asyncGetSession)      // 传递 asyncDBconnect 的结果
  .then(asyncGetUser)         // 传递 asyncGetSession 的结果
  .then(asyncLogAccess)       // 传递 asyncGetUser 的结果
  .then(result => {           // 同步函数
    console.log('complete');  //   (传递 asyncLogAccess 的结果)
    return result;            //   (结果传给下一个 .then())
  })
  .catch(err => {             // 任何一个 reject 触发
    console.log('error', err);
  });
```

同步函数也可以执行 `.then()`，返回的值传递给下一个 `.then()`（如果有）。

当任何一个前面的 `reject` 触发时，`.catch()` 函数会被调用。触发 `reject` 的函数后面的 `.then()` 也不再执行。贯穿整个链条可以存在多个 `.catch()` 方法，从而捕获不同的错误。

ES2018 引入了 `.finally()` 方法，它不管返回结果如何，都会执行最终逻辑 - 例如，清理操作，关闭数据库连接等等。当前仅有  Chrome 和 Firefox 支持，但是 TC39 技术委员会已经发布了 [.finally() 补丁](https://github.com/tc39/proposal-promise-finally/blob/fd934c0b42d59bf8d9446e737ba14d50a9067216/polyfill.js)。

```js
function doSomething() {
  doSomething1()
  .then(doSomething2)
  .then(doSomething3)
  .catch(err => {
    console.log(err);
  })
  .finally(() => {
    // 清理操作放这儿！
  });
}
```

## 使用 Promise.all() 处理多个异步操作

Promise `.then()` 方法用于相继执行的异步函数。如果不关心顺序 - 比如，初始化不相关的组件 - 所有异步函数同时启动，直到最慢的函数执行 `resolve`，整个流程结束。

`Promise.all()` 适用于这种场景，它接收一个函数数组并且返回另一个 Promise。举例：

```js
Promise.all([ async1, async2, async3 ])
  .then(values => {           // 返回值的数组
    console.log(values);      // (与函数数组顺序一致)
    return values;
  })
  .catch(err => {             // 任一 reject 被触发
    console.log('error', err);
  });
```

任意一个异步函数 `reject`，`Promise.all()` 会立即结束。

### 使用 Promise.race() 处理多个异步操作

`Promise.race()` 与 `Promise.all()` 极其相似，不同之处在于，当首个 Promise resolve 或者 reject 时，它将会 resolve 或者 reject。仅有最快的异步函数会被执行：

```js
Promise.race([ async1, async2, async3 ])
  .then(value => {            // 单一值
    console.log(value);
    return value;
  })
  .catch(err => {             // 任一 reject 被触发
    console.log('error', err);
  });
```

### 前途光明吗？

Promise 减少了回调地狱，但是引入了其他的问题。

教程常常不提，整个 Promise 链条是异步的，一系列的 Promise 函数都得返回自己的 Promise 或者在最终的 `.then()`，`.catch()` 或者 `.finally()` 方法里面执行回调。

我也承认：Promise 困扰了我很久。语法看起来比回调要复杂，好多地方会出错，调试也成问题。可是，学习基础还是很重要滴。

延伸阅读：

- [MDN Promise documentation](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)
- [JavaScript Promises: an Introduction](https://developers.google.com/web/fundamentals/primers/promises)
- [JavaScript Promises … In Wicked Detail](http://www.mattgreer.org/articles/promises-in-wicked-detail/)
- [Promises for asynchronous programming](http://exploringjs.com/es6/ch_promises.html)


