# 现代JavaScript的变量作用域

当与其他 JavaScript 开发人员交谈时，令我经常感到惊讶的是，有很多人不知道**变量作用域**是如何在 JavaScript 里起作用的。这里我们说的作用域指的是代码里变量的可见性;或者换句话说，哪部分代码可以访问和修改变量。我发现大家在代码中经常用 `var` 声明变量，而并不知道JavaScript将如何处理这些变量。

过去几年中，JavaScript 经历了一些巨大的变化;这些变化包括新的变量声明关键字以及新的作用域处理方式。ES6(ES2015) 新增了 `let` 和 `const` 命令，至今已经有三年时间，浏览器支持很好，对于其他新增特性，可以使用 **Babel** 将 ES6 转换成能被广泛采用的 JavaScript 。现在我们来再次审视如何声明变量，并进一步对它们的作用域增加了解。

在这篇博客文章中，我将通过大量JavaScript示例来展示全局作用域、局部作用域和块作用域是如何起作用的。我们还将为那些仍不熟悉这部分内容的人演示如何使用 `let` 和 `const` 来声明变量。

## 全局作用域

让我们从**全局作用域**说起。代码中的任何地方都可以访问和修改全局声明的变量(几乎可以，但是我们稍后会提到例外的情况)。

声明在任何函数之外的顶级作用域的变量就是全局变量。

```js
var a = Fred Flinstone; // 全局变量
function alpha() {
    console.log(a);
}
alpha(); // 输出 'Fred Flinstone'
```

在这个例子中，`a` 是一个全局变量；因此，在我们代码中的任何函数中都能被轻松获得。在这里，我们可以从方法 alpha 输出 a 的值。当我们调用 alpha 方法时，控制台输出 Fred Flinstone。

在web浏览器中声明全局变量时，JavaScript 默认有一个全局对象 `window` ，全局作用域的变量实际上被绑定到 `window` 的一个属性。看看这个例子:

```js
var b = 'Wilma Flintstone';
window.b = 'Betty Rubble';
console.log(b); // 输出 'Betty Rubble'
```

`b` 可以作为 `window` 对象的属性被访问/修改 `window.b`。当然，没有必要通过 `window` 对象修改 `b` 的值，这只是为了证明这一点。我们更有可能将上述内容写成下面的形式：

```js
var b = 'Wilma Flintstone';
b = 'Betty Rubble';
console.log(b); // 输出 'Betty Rubble'
```

使用全局变量时要小心。它们会导致代码的可读性变差，发生错误时也很难测试。我已经看到许多开发人员在查找变量值何时被重置时遇到了意想不到的问题。将变量作为参数传递给函数要比依赖全局变量好得多。全局变量应该尽量少用。

如果你真的需要使用全局变量，最好给你的变量命名，使它们成为全局对象的属性。例如，创建一个名为 global 或 app 的全局对象。

```js
var app = {}; // 全局对象
app.foo = 'Homer';
app.bar = 'Marge';
function beta() {
    console.log(app.bar);
}
beta(); // 输出 'Marge'
```

如果您正在使用NodeJS，则顶级作用域与全局作用域不同。如果在NodeJS模块中使用 `var foobar` ，则它是该模块的局部变量。要在NodeJS中定义全局变量，我们需要使用**全局命名空间对象** `global` 。

```js
global.foobar = 'Hello World!'; // 在NodeJS里是一个全局变量
```

我们要意识到重要的一点，如果没有使用关键字 `var`、`let` 或 `const` 之一来声明变量，那么变量将被赋予全局作用域。

```js
function gamma() {
    c = 'Top Cat';
}
gamma();
console.log(c); // 输出 'Top Cat'
console.log(window.c); // 输出 'Top Cat'
```

最好先用变量关键字来声明变量。这样，我们就可以控制代码中的每个变量作用域。就像上面的例子一样，希望你能意识到如果不使用关键词的潜在危险。
