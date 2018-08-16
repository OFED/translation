# 现代 JavaScript 的变量作用域

原文链接：[Variable Scope in Modern JavaScript](https://andy-carter.com/blog/variable-scope-in-modern-javascript)

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

