# 现代 JS 流程控制：从回调函数到 Promises 再到 Async/Await

> 原文链接：[Flow Control in Modern JS: Callbacks to Promises to Async/Await](https://www.sitepoint.com/flow-control-callbacks-promises-async-await/)
>
> 译者：[OFED](https://github.com/OFED/translation/issues/4)

JavaScript 通常被认为是异步的。这意味着什么？对开发有什么影响呢？近年来，它又发生了怎样的变化？

看看以下代码：

```js
result1 = doSomething1();
result2 = doSomething2(result1);
```

大多数编程语言同步执行每行代码。第一行执行完毕返回一个结果。无论第一行代码执行多久，只有执行完成第二行代码才会执行。

## 单线程处理程序

JavaScript 是单线程的。当浏览器选项卡执行脚本时，其他所有操作都会停止。这是必然的，因为对页面 DOM 的更改不能并发执行；一个线程
重定向 URL 的同时，另一个线程正要添加子节点，这么做是危险的。

用户不容易察觉，因为处理程序会以组块的形式快速执行。例如，JavaScript 检测到按钮点击，运行计算，并更新 DOM。一旦完成，浏览器就可以自由处理队列中的下一个项目。

(*附注: 其它语言比如 PHP 也是单线程，但是通过多线程的服务器比如 Apache 管理。同一 PHP 页面同时发起的两个请求，可以启动两个线程运行，它们是彼此隔离的 PHP 实例。*)

## 通过回调实现异步

单线程产生了一个问题。当 JavaScript 执行一个“缓慢”的处理程序，比如浏览器中的 Ajax 请求或者服务器上的数据库操作时，会发生什么？这些操作可能需要几秒钟 - *甚至几分钟*。浏览器在等待响应时会被锁定。在服务器上，Node.js 应用将无法处理其它的用户请求。

解决方案是异步处理。当结果就绪时，一个进程被告知调用另一个函数，而不是等待完成。这称之为*回调*，它作为参数传递给任何异步函数。例如:

```js
doSomethingAsync(callback1);
console.log('finished');

// 当 doSomethingAsync 完成时调用
function callback1(error) {
  if (!error) console.log('doSomethingAsync complete');
}
```

`doSomethingAsync()` 接收回调函数作为参数(只传递该函数的引用，因此开销很小)。`doSomethingAsync()` 执行多长时间并不重要；我们所知道的是，`callback1()` 将在未来某个时刻执行。控制台将显示:

```js
finished
doSomethingAsync complete
```

### 回调地狱

通常，回调只由一个异步函数调用。因此，可以使用简洁、匿名的内联函数:

```js
doSomethingAsync(error => {
  if (!error) console.log('doSomethingAsync complete');
});
```

一系列的两个或更多异步调用可以通过嵌套回调函数来连续完成。例如:

```js
async1((err, res) => {
  if (!err) async2(res, (err, res) => {
    if (!err) async3(res, (err, res) => {
      console.log('async1, async2, async3 complete.');
    });
  });
});
```

不幸的是，这引入了**回调地狱** —— 一个臭名昭著的概念，甚至有[专门的网页介绍](http://callbackhell.com/)！代码很难读，并且在添加错误处理逻辑时变得更糟。

回调地狱在客户端编码中相对少见。如果你调用 Ajax 请求、更新 DOM 并等待动画完成，可能需要嵌套两到三层，但是通常还算可管理。

操作系统或服务器进程的情况就不同了。一个 Node.js API 可以接收文件上传，更新多个数据库表，写入日志，并在发送响应之前进行下一步的 API 调用。

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

## Async/Await

Promise 似乎比较复杂，所以 ***ES2017*** 引进了 `async` 和 `await`。虽然只是语法糖，却使得 Promise 使用更加方便，并且可以避免 `.then()` 链式调用的问题。看下面使用 Promise 的例子：

```js
function connect() {

  return new Promise((resolve, reject) => {

    asyncDBconnect('http://localhost:1234')
      .then(asyncGetSession)
      .then(asyncGetUser)
      .then(asyncLogAccess)
      .then(result => resolve(result))
      .catch(err => reject(err))

  });
}

// 运行 connect 方法 (自执行方法)
(() => {
  connect();
    .then(result => console.log(result))
    .catch(err => console.log(err))
})();
```

使用 `async` / `await` 重写上面的代码：

1. 外部方法用 `async` 声明
1. 基于 Promise 的异步方法用 `await` 声明

```js
async function connect() {

  try {
    const
      connection = await asyncDBconnect('http://localhost:1234'),
      session = await asyncGetSession(connection),
      user = await asyncGetUser(session),
      log = await asyncLogAccess(user);

    return log;
  }
  catch (e) {
    console.log('error', err);
    return null;
  }

}

// 运行 connect 方法 (自执行异步函数)
(async () => { await connect(); })();
```

`await` 函数使每个异步调用看起来像是同步的，同时不耽误 JavaScript 的单个处理线程。此外，`async` 函数总是返回一个 Promise 对象，因此它们可以被其他 `async` 函数调用。

`async` / `await` 可能不会让代码变少，但是有很多优点：

1. 语法更清晰。括号越来越少，出错的可能性也越来越小。
1. 调试更容易。可以在任何 `await` 语句上设置断点。
1. 错误处理更好。`try` / `catch` 块可以与同步代码使用相同的处理方式。
1. 支持度友好。它在所有浏览器(除了 IE 和Opera Mini)和 Node7.6(以及以上)版本中实现。

也就是说，并不都是完美的…

### Promises, 还是Promises

`async` / `await` 仍然依赖 Promise 对象，最终依赖回调。你需要理解 Promise 的工作原理，它也并不等同于`Promise.all()` 和 `Promise.race()`。比较容易忽视的是 `promise.all()`，这个命令比使用一系列无关联的 `await` 命令更有用。

### 同步循环中的异步等待

某些情况下，你想要在同步循环中调用异步函数。例如:

```js
async function process(array) {
  for (let i of array) {
    await doSomething(i);
  }
}
```

不起作用，下面的代码也一样：

```js
async function process(array) {
  array.forEach(async i => {
    await doSomething(i);
  });
}
```

循环本身保持同步，并且总是在内部异步操作之前完成。

ES2018 引入异步迭代器，除了 `next()` 方法返回一个 Promise 对象之外，它与常规迭代器类似。因此，`await` 关键字可以与 `for…of` 循环一起使用，以串行方式运行异步操作。例如:

```js
async function process(array) {
  for await (let i of array) {
    doSomething(i);
  }
}
```

然而，在执行异步迭代器之前，最好将数组项映射到异步函数，并用 `Promise.all()` 运行它们。例如:

```js
const
  todo = ['a', 'b', 'c'],
  alltodo = todo.map(async (v, i) => {
    console.log('iteration', i);
    await processSomething(v);
});

await Promise.all(alltodo);
```

这有利于并行运行任务，但是不能将一次迭代的结果传递给另一次迭代，并且映射大数组可能会比较消耗计算性能。

### 丑陋的 `try` / `catch` 块

如果失败的 `await` 没有 `try` / `catch`，`async` 函数将默认退出。如果你有一长组异步 `await` 命令，您可能需要多个 `try` / `catch` 块。

一个替代方案是高阶函数，用来捕捉错误，不再需要 `try` / `catch` 块:

```js
async function connect() {

  const
    connection = await asyncDBconnect('http://localhost:1234'),
    session = await asyncGetSession(connection),
    user = await asyncGetUser(session),
    log = await asyncLogAccess(user);

  return true;
}

// 用高阶函数来捕获错误
function catchErrors(fn) {
  return function (...args) {
    return fn(...args).catch(err => {
      console.log('ERROR', err);
    });
  }
}

(async () => {
  await catchErrors(connect)();
})();
```

当应用程序采用不同于其他错误的方式处理某些特殊错误时，上面的方式就不实用了。

尽管有一些缺陷，`try` / `catch`是 JavaScript 非常有用的增进补充。更多资源：

+ MDN [async](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/async_function) 和 [await](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/await)
+ [异步函数 - 提高 Promise 的易用性](https://developers.google.com/web/fundamentals/primers/async-functions)
+ [TC39 异步函数规范](https://tc39.github.io/ecmascript-asyncawait/)
+ [用异步函数简化异步编码](https://www.sitepoint.com/simplifying-asynchronous-coding-async-functions/)

## JavaScript 之旅

异步编程是 JavaScript 中无法避免的挑战。回调在大多数应用程序中是必不可少的，但是很容易陷入深度嵌套的函数中。

Promise 取消了回调，但是有许多句法陷阱。转换现有的函数的写法可能是一件苦差事。链式调用 `·then()` 方法仍然很凌乱。

很幸运，`async` / `await` 使代码更简洁。代码看起来是同步的，但是又不独占单个处理线程。它将改变你写 JavaScript 的方式，甚至可以让你感激 Promise 对象。