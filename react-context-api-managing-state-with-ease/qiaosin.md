# React Context API: 轻松管理状态

#### 使用最新的 React Context API 管理状态非常容易。现在就跟随我一起学习下它和 Redux 的区别以及它是如何使用的吧。

## 前言：

The React Context API 在 React 生态系统中并不是个新鲜事物。不过，在 React `16.3.0` 版本中做了一些[改进](https://auth0.com/blog/whats-new-in-react-16-3/)。这些改进是如此巨大，以至于大大减少了我们对 Redux 和其他高级状态管理库的需求。在本文中，你将通过一个实用教程了解到新的 React Context API 是如何取代 Redux 完成小型应用的状态管理的。

## Redux 快速回顾

在直奔主题之前，我们先来快速回顾下 Redux，以便我们更好的比较两者的区别。[redux 是一个便于状态管理的JavaScript库](https://redux.js.org/)。Redux 本身和 React 并没有关系。来自世界各地的众多开发者选择在流行的前端框架（比如 *React* 和 *Angular* ）中使用 Redux。

说明一点，在本文中，状态管理指的是处理单页面应用（SPA）中产生的基于特定事件而触发的状态变化。比如，一个按钮的点击事件或者一条来自服务器的异步信息等，都可以触发应用状态的变化。

在 Redux 中，你尤其需要注意下面几点：

1. 整个 app 的状态存储在单个对象中（该对象被称作数据源）。
2. 如果要改变状态，你需要通过 dispatch 方法触发`actions`，actions 描述了需要发生的事情。
3. 在 Redux 中，你不能更改对象的属性或更改现有数组，必须始终返回新对象或新数组。

如果你对 Redux 并不熟悉并且你想要了解更多，请移步 [Redux 的实用教程](https://auth0.com/blog/redux-practical-tutorial/)学习。
