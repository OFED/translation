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

### Promises, Promises