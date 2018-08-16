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

## 局部作用域

现在我们回到*局部作用域*

```js
var a = 'Daffy Duck'; // a 是全局变量
function delta(b) {
  // b 是传入 delta 的局部变量
  console.log(b);
}
function epsilon() {
  // c 被定义成局部作用域变量
  var c = 'Bugs Bunny';
  console.log(c);
}
delta(a); // 输出 'Daffy Duck'
epsilon(); // 输出 'Bugs Bunny'
console.log(b); // 抛出错误：b 在全局作用域未定义
```

在函数内部定义的变量，作用域限制在函数内。以上例子中，b 和 c 对于各自的函数而言是局部的。可是出现以下的写法，输出结果会是什么呢？

```js
var d = 'Tom';
function zeta() {
  if (d === undefined) {
    var d = 'Jerry';
  }
  console.log(d);
}
zeta();
```

答案是 'Jerry'，这可能是常考的面试题之一。zeta 函数内部定义了一个新的局部变量 d，当用 `var` 定义变量的时候，JavaScript 会在当前作用域的顶部初始化它，不管它在代码的哪一部分。

```js
var d = 'Tom';
function zeta() {
  var d;
  if (d === undefined) {
    d = 'Jerry';
  }
  console.log(d);
}
zeta();
```

这被称之为*提升*，它是 JavaScript 的特性之一，而且需要注意的是，没在作用域的顶部初始化变量，容易引起一些 bug。还好 `let` 和 `const` 的出现解救了我们。那么让我们看看如何使用 `let` 创建块级作用域。

## 块级作用域





## 全局 + 局部作用域

当我们在局部作用域重新定义已经存在的全局变量时会发生什么呢。

```js
var a = 'Johnny Bravo'; // 全局作用域
function iota() {
  var a = 'Momma'; // 局部作用域
  console.log(a); // 输出 'Momma'
  console.log(window.a); // 输出 'Johnny Bravo'
}
iota();
console.log(a); // 输出 'Johnny Bravo'
```
当我们在局部作用域重定义全局变量的时候，JavaScript 初始化了一个新的局部变量。例子中，已有一个全局变量 a，函数 iota 内部又创建了一个新的局部变量 a。新的局部变量并没有修改全局变量，如果我们想在函数内部访问全局变量的值，需要使用全局的 `window` 对象。

对我而言，以下代码更易读，使用全局命名空间代替了全局变量，使用块级作用域重写了我们的函数:-

```js
var globals = {};
globals.a = 'Johnny Bravo'; // 全局作用域
function iota() {
    let a = 'Momma'; // 局部作用域
    console.log(a); // 输出 'Momma'
    console.log(globals.a); // 输出 'Johnny Bravo'
}
iota();
console.log(globals.a); // 输出 'Johnny Bravo'
```

## 局部 + 块级作用域

希望以下的代码如你所愿。

```js
function kappa() {
    var a = 'Him'; // 局部作用域
    if (true) {
        let a = 'Mojo Jojo'; // 块级作用域
        console.log(a); // 输出 'Mojo Jojo'
    }
    console.log(a); // 输出 'Him'
}
kappa();
```

以上代码并不是特别易读，但是块级作用域变量只能在定义的块级内访问。在块级作用域外面修改块级变量毫无效果，用 `let` 重定义变量 a，同样没效果，如下例：

```js
function kappa() {
    let a = 'Him';
    if (true) {
        let a = 'Mojo Jojo';
        console.log(a); // 输出 'Mojo Jojo'
    }
    console.log(a); // 输出 'Him'
}
kappa();
```

## var，let 还是 const？

我希望此篇作用域的总结能让大家更好的理解 JavaScript 如何处理变量。贯穿全文的示例中我使用 var，let 和 const 定义变量。伴随着 ES6 的降临，我们大可以使用 let 和 const 取代 var。

那么 var 多余了吗？没有真正对错的答案，但是个人而言，我依旧习惯用 var 定义顶层的全局变量。可是，我会保守地使用全局变量，而是用全局命名空间代替。此外，不再改变的变量我用 const，剩余的其他情况我用 let。

最终你会如何定义变量呢，还是希望你能更好的理解代码中的作用域范围。