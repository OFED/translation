# 使用 Vue 编写一个长按指令

> 原文链接：[Building a long press directive in Vue](https://blog.logrocket.com/building-a-long-press-directive-in-vue-3408d60fb511)
>
> 译者：[OFED](https://github.com/OFED)

![Alt text](https://raw.githubusercontent.com/OFED/translation/master/building-a-long-press-directive-in-vue/img/1_0neT5mha2f00A3CXxA8r6w.png)

有没有想过只需按住一个按钮几秒钟就能在你的 Vue 应用中触发一个功能？

有没有想过创建一个按钮，按下一次就可以清除单次输入（或者持续按住可以清除所有输入）？

想过？太好了，英雄所见略同。

本文就是讲解如何在按下（或者按住）一个按钮时，既执行一个函数，又清除输入。

首先，我会讲解如何使用纯 JS 实现。而后也会创建一个 Vue 指令。

请系好安全带。好戏在后头呢。

## 原理

要实现长按，用户需要按下并按住按钮几秒钟。

想通过代码模拟这一效果，我们需要在鼠标“点击”按下按钮时，启动一个计时器监听用户按下的时长，如果时间超过我们期望的时长，就执行相应的函数。

非常简单！然而，我们需要知道用户何时按住按钮。

## 如何实现

当用户点击按钮时，在点击事件之前会触发另外两个事件: **mousedown** 和 **mouseup**。

当用户按下按钮时触发 `mousedown` 事件，用户松开按钮时调用 `mouseup` 事件。

我们需要做的是:

1. `mousedown` 事件触发时，启动计时器。
1. 一旦 `mouseup` 事件在预期的 2 秒前被触发，就清除计时器，不要执行相应的函数。就当作一个普通的点击事件。

只要计时器在我们预设的时间内没有被清除，即 `mouseup` 事件没有被触发——那么可以断定用户没有释放按钮。因此，可以判定为一次长按，可以执行关联的函数。

## 实践

让我们深入代码，完成这一功能。

首先，我们必须定义三件事，即:

1. 一个 **变量** 用于存储计时器。
1. 一个 **启动** 功能函数，用于启动计时器。
1. 一个 **取消** 功能函数，用于取消计时器。

### 变量

这个变量主要用来保存 `setTimeout` 的值，以便当鼠标 `mouseup` 事件触发时我们可以取消它。

```js
let pressTimer = null;
```

我们把变量值设置为 `null` 是为了在执行取消操作前，检查这个变量的值判断当前是否有一个正在运行的计时器。

### 启动函数

这个函数包括一个 [setTimeout](https://www.w3schools.com/jsref/met_win_settimeout.asp)，它是 JavaScript 中的一个基本方法，允许在特定时间之后执行一个函数。

注意，`click` 事件执行的过程中，会触发另外两个事件。但是我们需要启动计时器的是 `mousedown` 事件。如果只是点击事件，不需要启动计时器。

```js
// 创建计时器 ( 1s之后执行函数 )
let start = (e) => {
    // 如果是点击事件，不启动计时器
    if (e.type === 'click' && e.button !== 0) {
        return;
    }
    // 在启动一个定时器之前确保没有正在运行的计时器
    if (pressTimer === null) {
        pressTimer = setTimeout(() => {
            // 执行任务 !!!
        }, 1000)
    }
}
```

### 取消函数

这个函数见名知意，用来取消启动函数创建的 `setTimeout`。

要取消 `setTimeout` ，可以使用 JavaScript 中的 [clearTimeout](https://www.w3schools.com/jsref/met_win_cleartimeout.asp) 方法，它主要用来清除 [setTimeout()](https://www.w3schools.com/jsref/met_win_settimeout.asp) 方法设置的计时器。

在使用 `clearTimeout` 之前，需要检查 ***pressTimer*** 变量是否为 `null`。如果没有为 `null`，意味着有一个正在运行的计时器。因此，我们需要先清除它，并且将 ***pressTimer*** 变量设置为 ***null***。

```js
let cancel = (e) => {
    // 检查 pressTimer 的值是否为 null
    if (pressTimer !== null) {
        clearTimeout(pressTimer)
        pressTimer = null
    }
}
```

一旦 ***mouseup*** 事件触发，这个函数就会被调用。

### 设置触发器

剩下的就是将事件监听器添加到想要长按效果的按钮上。

```js
addEventListener("mousedown", start);
addEventListener("click", cancel);
```

以上代码合到一起是这样:

```js
// 定义变量
let pressTimer = null;

// 创建计时器（ 1秒后执行函数 ）
let start = (e) => {

    if (e.type === 'click' && e.button !== 0) {
        return;
    }

    if (pressTimer === null) {
        pressTimer = setTimeout(() => {

            // 执行任务 !!!

        }, 1000)
    }
}

// 停止计时器
let cancel = (e) => {

    // 检查是否有正在运行的计时器
    if ( pressTimer !== null ) {
        clearTimeout(pressTimer);
        pressTimer = null;
    }
}

// 选择 id 为 longPressButton 的元素
let el = document.getElementById('longPressButton');

// 添加事件监听器
el.addEventListener("mousedown", start);

// 长按事件取消，取消计时器
el.addEventListener("click", cancel);
el.addEventListener("mouseout", cancel);
```

### 用 Vue 指令包装

创建 Vue 指令时，可以创建全局或局部指令，本文中，我们采用全局指令。

首先，我们必须声明自定义指令的名称。

```js
Vue.directive('longpress', {

})
```

这就注册了一个名为 ***v-longpress*** 的全局自定义指令。

接下来，我们添加带参数的 bind [钩子函数](https://vuejs.org/v2/guide/custom-directive.html#Hook-Functions)，它允许我们引用指令绑定的元素，获取传递给指令的值，并标识指令使用的组件。

```js
Vue.directive('longpress', {
    bind: function(el, binding, vNode) {

    }
})
```

接下来，我们在 bind 函数中添加长按功能的代码。

```js
Vue.directive('longpress', {
    bind: function(el, binding, vNode) {

        // 定义变量
        let pressTimer = null;

        // 定义函数处理程序
        // 创建计时器（ 1秒后执行函数 ）
        let start = (e) => {

            if (e.type === 'click' && e.button !== 0) {
                return;
            }

            if (pressTimer === null) {
                pressTimer = setTimeout(() => {

                    // 执行任务 !!!

                }, 1000)
            }
        }

        // 取消计时器
        let cancel = (e) => {

            // 检查是否有正在运行的计时器
            if ( pressTimer !== null ) {
                clearTimeout(pressTimer);
                pressTimer = null;
            }
        }

        // 添加事件监听器
        el.addEventListener("mousedown", start);

        // 取消计时器
        el.addEventListener("click", cancel);
        el.addEventListener("mouseout", cancel);
    }
})
```

接下来，我们需要添加一个函数来运行传递给 ***longpress*** 指令的方法。

```js
Vue.directive('longpress', {
    bind: function(el, binding, vNode) {

        // 定义变量
        let pressTimer = null;

        // 定义函数处理程序
        // 创建计时器（ 1秒后执行函数 ）
        let start = (e) => {

            if (e.type === 'click' && e.button !== 0) {
                return;
            }

            if (pressTimer === null) {
                pressTimer = setTimeout(() => {
                    // 执行函数
                    handler();
                }, 1000)
            }
        }

        // 停止计时器
        let cancel = (e) => {

            // 检查是否有正在运行的计时器
            if ( pressTimer !== null ) {
                clearTimeout(pressTimer);
                pressTimer = null;
            }
        }

        // 运行函数
        const handler = (e) => {
            // 执行传递给指令的方法
            binding.value(e)
        }

        // 添加事件监听器
        el.addEventListener("mousedown", start);

        // 取消计时器
        el.addEventListener("click", cancel);
        el.addEventListener("mouseout", cancel);
    }
})
```

现在，可以在 Vue 应用中使用这个指令了，除非使用者给指令传入的值不是一个函数。因此，我们需要通过警告反馈给使用者。

为了反馈给使用者，我们在 bind 函数中添加了以下内容:

```js
// 确保提供的表达式是函数
if (typeof binding.value !== 'function') {
    // 获取组件名称
    const compName = vNode.context.name;
    // 将警告传递给控制台
    let warn = `[longpress:] provided expression '${binding.expression}' is not a function, but has to be `;
    if (compName) { warn += `Found in component '${compName}' ` }

    console.warn(warn);
}
```

最后，如果这个指令也适用于触屏设备，那会是极好的。因此，我们添加了 [touchstart](https://developer.mozilla.org/en-US/docs/Web/Events/touchstart)、[touchend](https://developer.mozilla.org/en-US/docs/Web/Events/touchend) 和 [touchcancel](https://developer.mozilla.org/en-US/docs/Web/Events/touchcancel) 事件监听器。

最终代码如下:

```js
Vue.directive('longpress', {
    bind: function(el, binding, vNode) {

        // 确保提供的表达式是函数
        if (typeof binding.value !== 'function') {
            // 获取组件名称
            const compName = vNode.context.name;
            // 将警告传递给控制台
            let warn = `[longpress:] provided expression '${binding.expression}' is not a function, but has to be `;
            if (compName) { warn += `Found in component '${compName}' `}

            console.warn(warn);
        }

        // 定义变量
        let pressTimer = null;

        // 定义函数处理程序
        // 创建计时器（ 1秒后执行函数 ）
        let start = (e) => {

            if (e.type === 'click' && e.button !== 0) {
                return;
            }

            if (pressTimer === null) {
                pressTimer = setTimeout(() => {
                    // 执行函数
                    handler();
                }, 1000)
            }
        }

        // 取消计时器
        let cancel = (e) => {

            // 检查计时器是否有值
            if ( pressTimer !== null ) {
                clearTimeout(pressTimer);
                pressTimer = null;
            }
        }

        // 运行函数
        const handler = (e) => {
            // 执行传递给指令的方法
            binding.value(e)
        }

        // 添加事件监听器
        el.addEventListener("mousedown", start);
        el.addEventListener("touchstart", start);

        // 取消计时器
        el.addEventListener("click", cancel);
        el.addEventListener("mouseout", cancel);
        el.addEventListener("touchend", cancel);
        el.addEventListener("touchcancel", cancel);
    }
})
```

现在可以在 Vue 组件里使用了:

```html
<template>
    <div>
        <button v-longpress="incrementPlusTen" @click="incrementPlusOne">{{value}}</button>
    </div>
</template>

<script>
export default {
    data() {
        return {
            value: 10
        }
    },
    methods: {
        // 增加1
        incrementPlusOne() {
            this.value++
        },
        // 增加10
        incrementPlusTen() {
            this.value += 10
        }
    }
}
</script>
```

<center> . . . </center>

如果你想知道更多关于 **自定义指令**、可用的 **钩子函数**、可以传递到这个钩子函数中的 **参数**、**函数简写** 的信息, 参照 [@vuejs](https://vuejs.org/v2/guide/custom-directive.html) 官方文档，作者做了很好的解释。

收工，干杯！

<center> . . . </center>
