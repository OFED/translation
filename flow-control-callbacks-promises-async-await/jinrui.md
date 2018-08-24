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