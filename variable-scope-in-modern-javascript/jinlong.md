# 现代 JavaScript 的变量作用域

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







