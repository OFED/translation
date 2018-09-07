
# D3.js 教程: 使用 JavaScript 创建可交互的柱状图

> 原文链接：[D3.js Tutorial: Building Interactive Bar Charts with JavaScript](https://blog.risingstack.com/d3-js-tutorial-bar-charts-with-javascript/)
>
> 译者：[OFED](https://github.com/OFED/translation/issues/6)

最近，我们有幸参与了一个机器学习项目，该项目涉及 React 和 D3.js 之类的库。在许多任务中，我开发了几个图表用来展示诸如朴素贝叶斯这样的机器学习模型的处理结果，图表以折线图或分组柱状图的形式呈现。

我会在此文中介绍使用 D3.js 的过程，以及通过一个简单的柱状图示例演示库的基本使用。

读完此文后，你将学到如何轻松创建类似的 D3.js 图表:

![bar chart](https://raw.githubusercontent.com/OFED/translation/master/d3-js-tutorial-bar-charts-with-javascript/img/d3-js-tutorial-bar-chart-made-with-javascript-small.gif)

这里有完整的[源代码](https://codepen.io/kingrychan/pen/MqebJZ?editors=0010)

我们在 RisingStack（公司）喜欢 JavaScript 生态，前端，后端都喜欢。就我个人而言，我对前后端都感兴趣。通过后端开发，我可以看透应用程序的底层业务逻辑，同时也有机会在前端创建令人惊叹的效果。这正是 D3.js 的用武之地!

## D3.js 是什么？

D3.js 是一个可以基于数据来操作文档的 JavaScript 库。

>“D3 可以帮助你使用 HTML, CSS, SVG 以及 Canvas 来展示数据。D3 遵循现有的 Web 标准，可以不需要其他任何框架独立运行在现代浏览器中，它结合强大的可视化组件来驱动 DOM 操作。” - [d3js.org](https://d3js.org/)

**首先考虑为什么要用 D3.js 创建图表？为什么不只显示图片呢？**

图表是基于第三方资源的信息，在渲染时需要动态可视化。此外，SVG 是一个非常强大的工具，非常适合这个应用场景。

让我们先看看 SVG 有什么好。

## SVG 的优点

SVG 代表可缩放矢量图形，从技术上讲，这是一种基于 XML 的标记语言。

它通常用于绘制矢量图形，比如线条和形状或修改现有图像。你可以在[这里](https://developer.mozilla.org/en-US/docs/Web/SVG/Element)找到可用元素的列表。

优点:

+ 支持所有主流浏览器；
+ 有 DOM 接口，不需要第三方库；
+ 可伸缩，可保持高分辨率；
+ 和其他图像格式相比，体积更小。

缺点:

+ 只能显示二维图像；
+ 学习曲线长；
+ 对于计算密集型操作，渲染可能需要很长时间。

SVG 尽管有缺点，但它仍是显示图标，logo，插图或者此文提及的图表的优良工具。

## 开始使用 D3.js

我选择以柱状图作为开始，因为它代表了一个低复杂度的视觉元素，同时它还能教会 D3.js 本身的基本应用。没骗你，D3 提供了一套很棒的可视化数据的工具。看看它的 [github page](https://github.com/d3/d3/wiki/Gallery) 页面，欣赏一些非常好的用例！

柱状图可以是水平或垂直的，取决于它的方向。我们从垂直的柱状图开始。

在这个图表中，我将根据 Stack Overlow 2018年[开发者调查结果](https://insights.stackoverflow.com/survey/2018/#technology-most-loved-dreaded-and-wanted-languages)显示前十个最受欢迎的编程语言。

## 画起来！

SVG 的坐标系从左上角开始(0;0)。正 x 轴向右，正 y 轴向下。因此，在计算元素的 y 坐标时，必须考虑 SVG 的高度。

![axis](https://raw.githubusercontent.com/OFED/translation/master/d3-js-tutorial-bar-charts-with-javascript/img/d3-js-bar-chart-tutorial-javascript-coordinate.png)

背景知识差不多了，让我们撸代码吧！

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

以上代码片段中，我用 d3 `select` 选择了 HTML 创建的 `<svg>` 元素。此选择方法接收各种类型的[选择器字符串](https://www.w3.org/TR/selectors-api/)并返回第一个匹配元素。如果想获取所有匹配元素，使用 `selectAll`。

我还定义了一个边距值，它给图表提供了一点间距。间距也可以应用到 `<g>` 元素上，通过 `translate` 移动期望的值。从现在起，我将在这个分组中绘制，确保与页面其它内容保持合理的间距。

```js
const chart = svg.append('g')
    .attr('transform', `translate(${margin}, ${margin})`);
```

往元素添加属性就像调用 `attr` 方法一样简单。方法的第一个参数接收用于所选 DOM 元素的属性。第二个参数是属性值或返回其值的回调函数。以上代码简单将图表的原点移到 SVG 的 (60;60) 位置。

## D3.js 支持的数据源格式

要开始绘图，我需要定义使用的数据源。本教程中，我使用了一个简单的 JavaScript 数组，该数组保存了语言名称及其所占百分比率的对象，但是这里着重提到一点，D3.js 支持多种数据格式。

该库具备从 XMLHttpRequest，.csv 文件，文本文件等数据源加载数据的内置功能。每一种数据源都可能包含 D3.js 可用的数据，最重要的是把它们构建成数组。注意，从 [版本5.0](https://github.com/d3/d3/blob/master/CHANGES.md) 开始，D3 库使用 Promise 取代回调来加载数据，这是一次不向后兼容的更改。

## 缩放，坐标轴

让我们继续讨论图表的坐标轴。为了画 y 轴，我需要设定最小和最大值，分别设置为0和100。

*本教程中，我正在研究使用百分比，但是除了数字之外，还有其他数据类型的实用函数，我将在后面解释。*

我必须将图表的高度在这两个值之间均分。为此，我创建了一个缩放函数。

```js
const yScale = d3.scaleLinear()
    .range([height, 0])
    .domain([0, 100]);
```

线性缩放是最常见的缩放类型。它将连续输入范围转换为连续输出范围。请注意 `range` 和 `domain` 方法。第一个 `range` 方法取的长度应该在 `domain` 的边界值之间。

记住，SVG 坐标系从左上角开始，这就是为什么 `range` 将高度作为第一个参数而不是零。

在左侧创建一个坐标轴跟添加另一个分组一样简单，调用 d3 的 `axisLeft` 方法，并把缩放函数作为参数。

```js
chart.append('g')
    .call(d3.axisLeft(yScale));
```

现在，继续添加 x 轴。

```js
const xScale = d3.scaleBand()
    .range([0, width])
    .domain(sample.map((s) => s.language))
    .padding(0.2)

chart.append('g')
    .attr('transform', `translate(0, ${height})`)
    .call(d3.axisBottom(xScale));
```

![](https://raw.githubusercontent.com/OFED/translation/master/d3-js-tutorial-bar-charts-with-javascript/img/d3-js-tutorial-bar-chart-labels.png)

请注意，我使用 [scaleBand](https://github.com/d3/d3-scale#scaleBand) 方法创建 x 轴，它将 x 轴 分成多段，并且使用余下的间隙计算柱状图的坐标和宽度。

D3.js 还能处理许多其他日期类型。[scaleTime](https://github.com/d3/d3-scale#scaleTime) 与 [scaleLinear](https://github.com/d3/d3-scale#scaleLinear) 非常相似，只是这里的 domain 是一个日期数组。

## 使用 D3.js 绘制柱状图

想想我们需要什么样的输入来画柱条。它们各自代表一个用简单形状，特别是矩形来展示的值。下一段代码中，我把它们添加到已创建的分组元素中了。

```js
chart.selectAll()
    .data(goals)
    .enter()
    .append('rect')
    .attr('x', (s) => xScale(s.language))
    .attr('y', (s) => yScale(s.value))
    .attr('height', (s) => height - yScale(s.value))
    .attr('width', xScale.bandwidth())
```

首先，我 `selectAll` 图表上的所有元素，返回结果为空。然后，`data` 函数根据数组长度通知 DOM 应该更新多少元素。如果数据个数多于 DOM 个数时，则 `enter` 会标识出缺少的元素。`enter` 会返回需要添加的元素。
通常，后面紧跟 `append` 方法会把元素添加到 DOM 中。

基本上，我用 D3.js 给数组每一项都追加了一个矩形。

当前只在彼此顶部添加了没有宽高的矩形。这两个属性必须通过之前的缩放函数计算所得。

我调用 `attr` 方法添加了矩形坐标。第二个参数可以是回调，它返回3个参数：当前绑定的数据，索引和所有数据数组。

```js
.attr(’x’, (actual, index, array) =>
    xScale(actual.value))
```

缩放函数返回给定范围值的坐标。计算坐标就是小菜一碟，诀窍是利用柱子的高度。必须从图表的高度减去计算出的 y 坐标，才能得到正确的列值。

定义矩形的宽度也会用到缩放函数。`scaleBand` 有一个 `bandwidth` 函数，它基于设置的间距返回一个元素的计算宽度。

![](https://raw.githubusercontent.com/OFED/translation/master/d3-js-tutorial-bar-charts-with-javascript/img/d3-js-tutorial-bar-chart-drawn-out-with-javascript.png)

干得不错，但没那么花哨，对吧？

为了防止观众视觉疲劳，让我们添加一些信息改善下视觉效果！

## 制作柱状图的技巧

有一些基本规则值得一提。

+ 避免使用 3D 效果；
+ 直观地排序数据点 - 按字母顺序或按数字排序；
+ 柱条之间保持一定距离；
+ y 轴从 0 开始，而不是从最小值开始；
+ 使用统一的颜色；
+ 添加轴标签、标题、导引线。

## D3.js 网格系统

我想在背景中添加栅格线突出那些值。

垂直和水平的线都可以添加，我的建议是只添加一种。过多的线会分散注意力。以下代码片段演示了如何添加水平和垂直的栅格。

```js
chart.append('g')
    .attr('class', 'grid')
    .attr('transform', `translate(0, ${height})`)
    .call(d3.axisBottom()
        .scale(xScale)
        .tickSize(-height, 0, 0)
        .tickFormat(''))

chart.append('g')
    .attr('class', 'grid')
    .call(d3.axisLeft()
        .scale(yScale)
        .tickSize(-width, 0, 0)
        .tickFormat(''))
```

![](https://raw.githubusercontent.com/OFED/translation/master/d3-js-tutorial-bar-charts-with-javascript/img/d3-js-bar-chart-grid-system-added-with-javascript.png)

此例中，我更喜欢垂直栅格线，因为它可以引导视线，保持整体画面简介明快。

## D3.js 中的标签

我还想添加一些文字指导，从而使图表更加全面。让我们给图表命个名，并为坐标轴添加标签吧。

![](https://raw.githubusercontent.com/OFED/translation/master/d3-js-tutorial-bar-charts-with-javascript/img/d3-js-tutorial-adding-labels-to-bar-chart-javascript.png)

文本是 SVG 元素，同样可以添加到 SVG 或者分组中。它们可以使用 x 和 y 坐标定位，文本对齐是通过 `text-anchor` 属性实现的。
添加标签文字，只需调用文本元素上的 `text` 方法。

```js
svg.append('text')
    .attr('x', -(height / 2) - margin)
    .attr('y', margin / 2.4)
    .attr('transform', 'rotate(-90)')
    .attr('text-anchor', 'middle')
    .text('Love meter (%)')

svg.append('text')
    .attr('x', width / 2 + margin)
    .attr('y', 40)
    .attr('text-anchor', 'middle')
    .text('Most loved programming languages in 2018')
```

## 与 D3.js 交互

我们的图表内容已然丰富，但是仍然可以添加些互动效果。

以下的代码演示了如何给 SVG 元素添加事件监听。

```js
svgElement
    .on('mouseenter', function (actual, i) {
        d3.select(this).attr(‘opacity’, 0.5)
    })
    .on('mouseleave’, function (actual, i) {
        d3.select(this).attr(‘opacity’, 1)
    })
```

*注意，我用了函数表达式而不是箭头函数，因为我通过 this 关键字访问元素。*

当鼠标滑过选中的 SVG 元素时，它的透明度变为原始值的一半，鼠标离开元素时透明度恢复原始值。

你也可以通过 `d3.mouse` 获取鼠标坐标。它返回一个具有 x 和 y 坐标的数组。在光标所在位置显示提示，就可以通过这个实现。

**创建令人瞠目结舌的图表并没那么简单。**

可能需要图形设计师，UX 研究员和其他牛人的智慧。以下例子展示了几个提升图表效果的可能性！

我们的图表显示了非常相似的值，所以为了突出条形值之间的差异，我添加了一个 `mouseenter` 事件。每当用户悬停在特定的列时，该栏的顶部就会画一条水平线。此外，我还计算了与其他柱条的差异，并显示在了相应的柱条上。

![](https://raw.githubusercontent.com/OFED/translation/master/d3-js-tutorial-bar-charts-with-javascript/img/d3-js-tutorial-adding-interactivity-to-bar-chart.png)

很整齐吧？我还在此例中增加了透明度，加大了柱条的宽度。

```js
.on(‘mouseenter’, function (s, i) {
    d3.select(this)
        .transition()
        .duration(300)
        .attr('opacity', 0.6)
        .attr('x', (a) => xScale(a.language) - 5)
        .attr('width', xScale.bandwidth() + 10)

    chart.append('line')
        .attr('x1', 0)
        .attr('y1', y)
        .attr('x2', width)
        .attr('y2', y)
        .attr('stroke', 'red')

    // 部分实现，整体效果见源码
})
```

`transition` 方法表明我想把 DOM 改变绘制成动画。它的时间间隔是用 `duration` 函数设置的，该函数以毫秒作为参数。上面的过渡会淡化带状颜色，并加宽条形的宽度。

要画一条 SVG 线，我需要起点和终点。这可以通过 `x1`，`y1` 和 `x2`，`y2` 坐标来设置。直到我用 `stroke` 属性设置线条的颜色，线条才可见。

这里只展示了 `mouseenter` 事件这部分，切记，必须在 `mouseout` 事件上恢复或删除更改。本文末尾提供了完整的源代码。

## 让我们给图表添加一些样式吧！

回顾下我们目前为止完成了那些功能，以及如何通过样式装扮图表。*可以通过先前用过的 `attr` 方法给 SVG 元素添加 class 属性。*

我们的图表功能丰富，而不是死板的静态图片，鼠标悬停时可以显示各个柱条的差值。标题交代表格的背景，标签帮助识别坐标轴的测量单位。我还在右下角添加了新的标签，注明数据来源。


**剩下的事情就差颜色和字体了！**

深色背景的图表使亮色柱条看起来很酷。我还使用了 `open Sans` 字体，并给不同的标签设置不同的大小和粗细。

<script async src="//jsfiddle.net/matehu/w7h81xz2/38/embed/"></script>

注意到那条虚线了吗？它是通过 `stroke-width` 和 `stroke-dasharray` 属性实现的。使用 `stroke-dasharray`，可以定义虚线的图案和间距，从而改变形状的轮廓。


```css
line#limit {
    stroke: #FED966;
    stroke-width: 3;
    stroke-dasharray: 3 6;
}

.grid path {
    stroke-width: 3;
}

.grid .tick line {
    stroke: #9FAAAE;
    stroke-opacity: 0.2;
}
```

网格线比较讨巧，我给分组中的路径元素使用了 `stroke-width: 0`，为了隐藏表格的框架，我还通过设置线条的透明度降低它们的可见性。

所有其它有关字体大小和颜色的 CSS 可以参照源码。

## 收尾我们的 D3.js 柱状图教程

D3.js 是一个令人惊叹的 DOM 操作库。它的内部埋藏了无数的宝藏等待你去探索（确切的说，不是埋藏，文档也很齐全）。此文仅仅使用了它的工具集的冰山一角，就创建了一个不同凡响的柱状图。

**继续探索吧，定能创造出无比壮观的视觉效果！**

这是本文示例[源代码](https://jsfiddle.net/matehu/w7h81xz2/) 的链接。

你用 D3.js 做过一些炫酷的东西吗？和我们分享一下！你有任何问题，或者想要关于这个主题的另一个教程，欢迎留言！

谢谢阅读，下次再见！

![](https://raw.githubusercontent.com/OFED/translation/master/d3-js-tutorial-bar-charts-with-javascript/img/12345.gif)