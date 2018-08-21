> 原文链接：[Flow Control in Modern JS: Callbacks to Promises to Async/Await](https://www.sitepoint.com/flow-control-callbacks-promises-async-await/)
>
> 译者：[OFED](https://github.com/OFED)

# 现代 JS 中的流控制: Callbacks、Promises、Async/Await

JavaScript 通常被认为是异步的。它意味着什么？它如何影响发展？近年来，它发生了怎样的变化？

看下面的代码：

```js
result1 = doSomething1();
result2 = doSomething2(result1);
```

大多数编程语言同步执行每行代码。前一行代码执行完返回一个结果，之后才运行下一行代码。