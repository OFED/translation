
# D3.js 教程: 用 JavaScript 构建交互式条形图

> 原文链接：[D3.js Tutorial: Building Interactive Bar Charts with JavaScript](https://blog.risingstack.com/d3-js-tutorial-bar-charts-with-javascript/)
>
> 译者：[OFED](https://github.com/OFED/translation/issues/6)

最近，我们有幸参与了一个机器学习项目，该项目涉及 React 和 D3.js 这样的库。在许多任务中，我开发了很多图表来展示诸如朴素贝叶斯这样的机器学习模型的结果，图表以折线图或分组条形图的形式呈现。

在这篇文章中，我想用 D3.js 展示我迄今为止的进展，并通过一个简单的条形示例展示这个库的基本用法。

阅读本文后，你将学会如何轻松创建这样的 D3.js 图表:

![bar chart](https://raw.githubusercontent.com/OFED/translation/master/d3-js-tutorial-bar-charts-with-javascript/img/d3-js-tutorial-bar-chart-made-with-javascript-small.gif)

这里有完整的[源代码](https://codepen.io/kingrychan/pen/MqebJZ?editors=0010)

无论前端还是后端，JavaScript 生态系统以一种上升的趋势越来越得到大家的喜欢。就我个人而言，我对前后端都很感兴趣。在后端，我可以看透应用程序的底层业务逻辑，同时我也有机会在前端创建令人惊叹的东西。这就是 D3.js 发挥作用的地方!

## D3.js 是什么？

D3.js 是一个数据驱动的 JavaScript 库，用于操作 DOM 元素。

> " D3 帮助您使用 HTML、SVG 和 CSS 将数据变为现实。D3 对 web 标准的强调为您提供了现代浏览器的全部功能，无需使用专有框架，将强大的可视化组件和数据驱动的 DOM 操作方法相结合。” - [d3js.org](https://d3js.org/)

### 为什么首先要用 D3.js 创建图表？为什么不只是显示图像呢？

嗯，图表是基于来自第三方资源的信息，在渲染时需要动态可视化。此外，SVG 是一个非常强大的工具，非常适合这个应用案例。

让我们先绕道看看使用 SVG 有什么好处。

## SVG 的好处

SVG 代表可缩放矢量图形，从技术上讲，这是一种基于 XML 的标记语言。

它通常用于绘制矢量图形、指定线条和形状或修改现有图像。你可以在[这里](https://developer.mozilla.org/en-US/docs/Web/SVG/Element)找到可用元素的列表。

优点:

+ 所有主流浏览器都支持 SVG;
+ 有 DOM 接口，不需要第三方库;
+ 大小可变，可以保持高分辨率;
+ 和其他图像格式相比，体积更小;

缺点:

+ 只能显示二维图像;
+ 学习曲线长，成本大;
+ 对于计算密集型操作，渲染可能需要很长时间;

尽管 SVG 有其缺点，但它是显示图标、徽标、插图以及本文这种情况的图表的伟大工具。

## 开始使用 D3.js

我选择了条形图作为开始，因为它代表了一个低复杂度的视觉元素，同时它还能教会 D3.js 本身的基本应用。没有骗你，D3 提供了一套很好的可视化数据的工具。查看它的 [github page](https://github.com/d3/d3/wiki/Gallery) 页面，了解一些非常好的用例！

条形图可以是水平的，也可以是垂直的。我将使用垂直的，也就是柱状图。

在这个图表中，我将根据 Stack overlow 2018年[开发者调查结果](https://insights.stackoverflow.com/survey/2018/#technology-most-loved-dreaded-and-wanted-languages)显示前十名最受欢迎的编程语言。

## 开始绘制

SVG的坐标系从左上角开始( 0，0 )。正x轴向右，而正y轴向下。因此，在计算元素的y坐标时，必须考虑SVG的高度。

![axis](https://raw.githubusercontent.com/OFED/translation/master/d3-js-tutorial-bar-charts-with-javascript/img/d3-js-bar-chart-tutorial-javascript-coordinate.png)

背景知识就够了，让我们写些代码吧！

我想创建一个宽1000像素、高600像素的图表。

```html
<body>
    <svg />
</body>
<script>
    const margin = 60;
    const width = 1000 - 2 * margin;
    const height = 600 - 2 * margin;

    const svg = d3.select('svg');
</script>
```

在上面的代码片段中，我用 D3 `select` 选择了 HTML 文件中创建的 `svg` 元素。此选择方法接受所有类型的[选择器字符串](https://www.w3.org/TR/selectors-api/)并返回第一个匹配元素。如果你想得到所有匹配元素，使用 `selectAll`。

我还定义了一个边距值，它给图表提供了一点额外的填充。填充可以应用在由所需值转换的 `g` 元素上。从现在起，我将利用这个分组元素与页面的任何其他内容保持合理的间距。

```js
const chart = svg.append('g')
    .attr('transform', `translate(${margin}, ${margin})`);
```

向元素添加属性就像调用 `attr` 方法一样简单。方法的第一个参数接收想要应用于所选DOM元素的属性。第二个参数是值或返回其值的回调函数。上面的代码简单地将图表的原点移到 SVG 的( 60，60 )位置。

## 