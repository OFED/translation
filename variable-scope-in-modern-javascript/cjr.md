# 现代 JavaScript 的变量作用域

当与其他 JavaScript 开发人员交谈时，令我经常感到惊讶的是，有很多人不知道**变量作用域**是如何在 JavaScript 里起作用的。这里我们说的作用域指的是代码里变量的可见性；或者换句话说，哪部分代码可以访问和修改变量。我发现大家在代码中经常用 `var` 声明变量，而并不知道 JavaScript 将如何处理这些变量。

过去几年中，JavaScript 经历了一些巨大的变化；这些变化包括新的变量声明关键字以及新的作用域处理方式。ES6(ES2015) 新增了 `let` 和 `const` 命令，至今已经有三年时间，浏览器支持很好，对于其他新增特性，可以使用 [Babel](https://babeljs.io/) 将 ES6 转换成广泛支持的 JavaScript。现在是时候回顾下如何声明变量，以及增加对作用域的了解了。

在这篇博文中，我将通过大量 JavaScript 示例来展示*全局*、*局部*和*块级*作用域是如何工作的。我们还将为那些仍不熟悉这部分内容的人演示如何使用 `let` 和 `const` 来声明变量。

## 全局作用域

让我们从*全局作用域*说起。全局定义的变量在代码中任何地方都可以访问和修改(几乎都可以，但是我们稍后会提到例外的情况)。

声明在任何函数之外的顶层作用域的变量就是全局变量。

```js
var a = Fred Flinstone; // 全局变量
function alpha() {
    console.log(a);
}
alpha(); // 输出 'Fred Flinstone'
```

在这个例子中，`a` 是一个全局变量；因此，在任何函数中都能被轻松获取。因此，我们可以从方法 `alpha` 输出 `a` 的值。当我们调用 `alpha` 方法时，控制台输出 `Fred Flinstone`。

在 web 浏览器声明全局变量时，它会作为全局 `window` 对象的属性。看看这个例子:

```js
var b = 'Wilma Flintstone';
window.b = 'Betty Rubble';
console.log(b); // 输出 'Betty Rubble'
```

`b` 可以作为 `window` 对象的属性（`window.b`）被访问/修改。当然，没有必要通过 `window` 对象修改 `b` 的值，这只是为了证明这一点。我们更有可能将上述情况写成下面的形式：

```js
var b = 'Wilma Flintstone';
b = 'Betty Rubble';
console.log(b); // 输出 'Betty Rubble'
```

使用全局变量要小心。它们会导致代码的可读性变差，同时变得很难测试。我已经看到许多开发人员在查找变量值何时被重置时遇到了意想不到的问题。将变量作为参数传递给函数要比依赖全局变量好得多。全局变量应该尽量少用。

如果你确实需要使用全局变量，最好定义命名空间，使它们成为全局对象的属性。例如，创建一个名为 `globals` 或 `app` 的全局对象。

```js
var app = {}; // 全局对象
app.foo = 'Homer';
app.bar = 'Marge';
function beta() {
    console.log(app.bar);
}
beta(); // 输出 'Marge'
```

如果你正在使用 NodeJS，则顶层作用域与全局作用域不同。如果在 NodeJS 模块中使用 `var foobar` ，则它是该模块的局部变量。要在 NodeJS 中定义全局变量，我们需要使用[全局命名空间对象](https://nodejs.org/api/globals.html#globals_global)，`global` 。

```js
global.foobar = 'Hello World!'; // 在 NodeJS 里是一个全局变量
```

需要注意的是，如果没有使用关键字 `var`、`let` 或 `const` 之一来声明变量，那么变量属于全局作用域。

```js
function gamma() {
    c = 'Top Cat';
}
gamma();
console.log(c); // 输出 'Top Cat'
console.log(window.c); // 输出 'Top Cat'
```

我们推荐始终使用一种变量关键字定义变量。这样，代码中的每一个变量作用域是可控的。正如以上例子，希望你能意识到不用关键字的潜在危险。
