
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

## D3.js支持的数据源格式

要开始绘图，我需要定义我工作的数据源。在本教程中，我使用了一个简单的 JavaScript 数组，该数组保存了带有语言名称及其所占百分比率的对象，但是这里要提到很重要的一点是 D3.js 支持多种数据格式。

该库具有从 XMLHttpRequest，.CSV 文件，文本文件等加载内容的内置功能。这些源中的每一个都可能包含 D3.js 可以使用的数据，唯一重要的事是通过它们构建一个数组。请注意，从 [版本5.0](https://github.com/d3/d3/blob/master/CHANGES.md) 开始，D3 库使用 Promise 而不是回调来加载数据，这是一种非向后兼容的更改。

## 缩放，坐标轴

让我们继续讨论图表的坐标轴。为了画 y 轴，我需要设定最低和最高的极限值，在这种情况下是 0 和 100。

*在本教程中，我正在研究使用百分比，但是除了数字之外，还有其他数据类型的实用函数，我将在后面解释。*

我必须将图表的高度在这两个值之间分成相等的部分。为此，我创建了一个叫做缩放函数的东西。

```js
const yScale = d3.scaleLinear()
    .range([height, 0])
    .domain([0, 100]);
```

线性缩放是最常见的缩放类型。它将连续输入域转换为连续输出范围。请注意 `range` 和 `domain` 方法。第一个 `range` 方法取的长度值应该在 `domain` 值的界限之间划分的长度。

记住，SVG坐标系从左上角开始，这就是为什么 `range` 将高度作为第一个参数而不是零。

在左边创建一个轴很简单，就是添加另一个组并调用 D3 的 axisLeft 方法，该方法将缩放函数作为参数。

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

请注意，我对 x 轴使用 [scaleBand](https://github.com/d3/d3-scale#scaleBand) 方法，用于将 range 分成多个bands，并使用额外的填充来计算 bars 的坐标和宽度。

D3.js 还能够处理许多其他日期类型。[caleTime](https://github.com/d3/d3-scale#scaleTime) 与 [scaleLinear](https://github.com/d3/d3-scale#scaleLinear) 非常相似，只是这里的 domain 是一系列日期。

## 使用 D3.js 绘制图条

想想我们需要什么样的输入来画图条。它们各自代表一个用简单形状，特别是矩形来说明的值。在下一个代码片段中，我将它们追加到创建的 group 元素中。

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

首先，我 `selectAll` 图表上的所有元素，这些元素返回空的结果集。然后，`data` 函数根据数组长度告诉 DOM 应该更新多少元素。如果数据输入长于所选内容，则 `.enter` 标识出缺少的元素。这将返回一个新的选择项，代表需要被添加的元素。通常，后面是一个向 DOM 中添加元素的 `append` 方法。

基本上，我使用 D3.js 为数组的每个成员追加一个矩形。

现在，这只会在彼此的顶部添加没有高度或宽度的矩形。必须计算这两个属性，这也是缩放函数再次派上用场的地方。

看，我用 `attr` 方法添加了矩形的坐标。第二个参数可以是回调，它采用3个参数:输入数据的实际成员、索引和整个输入。

```js
.attr(’x’, (actual, index, array) =>
    xScale(actual.value))
```

scaling 函数返回给定域值的坐标。计算坐标是小菜一碟，诀窍是用 bar 的高度。必须从图表的高度减去计算出的 y 坐标，才能得到正确的列表示值。

我也用缩放函数定义矩形的宽度。`scaleBand` 有一个 `bandwidth` 函数，它基于设置的填充返回一个元素的计算宽度。

![](https://raw.githubusercontent.com/OFED/translation/master/d3-js-tutorial-bar-charts-with-javascript/img/d3-js-tutorial-bar-chart-drawn-out-with-javascript.png)

干得不错，但没那么花哨，对吧？

为了防止观众眼睛流血，让我们添加一些信息并改善视觉效果！

## 制作条形图的技巧

条形图有一些基本规则值得一提。

+ 避免使用 3D 效果；
+ 直观地排序数据点 - 按字母顺序或按数字排序；
+ 保持 bands 之间的距离；
+ y 轴从 0 开始，但不使用最小值；
+ 使用统一的颜色；
+ 添加轴标签、标题、源行。

## D3.js 网格系统

我想通过在背景中添加网格线来突出显示这些值。

继续，用垂直线和水平线做实验，但是我的建议是只显示其中一条。过多的线条会分散注意力。这段代码介绍了如何添加这两种解决方案。

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

在这种情况下，我更喜欢垂直网格线，因为它们可以引导眼睛，保持整体画面简单明了。

## D3.js中的标签

我还想通过添加一些文本指导来使图表更加全面。让我们给图表命名，并为坐标轴添加标签。

![](https://raw.githubusercontent.com/OFED/translation/master/d3-js-tutorial-bar-charts-with-javascript/img/d3-js-tutorial-adding-labels-to-bar-chart-javascript.png)

文本是可以附加到 SVG 或 groups 上的 SVG 元素。它们可以用 x 和 y 坐标定位，而文本对齐是用 `text-anchor` 属性完成的。要添加标签，只需调用文本元素上的 `text` 方法。

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

## D3.js 交互效果

我们得到了相当丰富的图表，但是仍然有可能让它具有互动性。

在下一个代码块中，我将向你展示如何将事件监听器添加到 SVG 元素中。

```js
svgElement
    .on('mouseenter', function (actual, i) {
        d3.select(this).attr(‘opacity’, 0.5)
    })
    .on('mouseleave’, function (actual, i) {
        d3.select(this).attr(‘opacity’, 1)
    })
```

*请注意，我使用函数表达式而不是箭头函数，因为我通过 this 关键字访问元素。*

我将鼠标悬停时所选 SVG 元素的不透明度设置为原始值的一半，并在光标离开该区域时重置它。

你也可以用 `d3.mouse` 获得鼠标坐标。它返回一个具有 x 和 y 坐标的数组。这样，在光标顶端显示工具提示就没问题了。

**创建令人瞠目结舌的图表不是一种简单的艺术形式。**

人们可能需要借鉴图形设计师、UX研究人员和其他人的智慧。在下面的例子中，我将展示一些提升图表的可能性！

我在图表上显示了非常相似的值，所以为了突出条形值之间的差异，我为 `mouseenter` 事件设置了一个事件监听器。每当用户悬停在特定的一列上时，该栏的顶部就会画一条水平线。此外，我还计算了与其他 brands 的差异，并将其显示在条形上。

![](https://raw.githubusercontent.com/OFED/translation/master/d3-js-tutorial-bar-charts-with-javascript/img/d3-js-tutorial-adding-interactivity-to-bar-chart.png)

很整洁吧？我还在这个例子中增加了不透明度，并增加了 bar 的宽度。

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

    // this is only part of the implementation, check the source code
})
```

`transition` 方法表明我想把 DOM 的改变绘制成动画。它的间隔是用 duration 函数设置的，该函数需要毫秒作为参数。上面的过渡会淡化带状颜色，并加宽条形的宽度。

要画一条 SVG 线，我需要起点和终点。这可以通过 `x1`，`y1` 和 `x2`，`y2` 坐标来设置。直到我用 `stroke` 属性设置线条的颜色，线条才可见。

我在这里只展示了 `mouseenter` 事件的一部分，所以请记住，必须在 `mouseout` 事件上恢复或删除更改。本文末尾提供了完整的源代码。

## 让我们给图表添加一些样式吧！

让我们看看到目前为止我们取得了什么成就，以及如何用某种风格改变这张图表。*你可以使用我们以前使用过的 attr 函数向 SVG 元素添加类属性。*

这个图表有一套很好的功能。它也揭示了鼠标悬停时所代表的值之间的差异，而不是单调的静态图片。标题将图表放入上下文中，标签有助于用测量单位来识别是哪个轴。我还在右下角添加了一个新标签来标记输入源。

**唯一剩下的就是升级颜色和字体！**

深色背景的图表使亮色条看起来很酷。我还将 `open Sans` 字体系列应用于所有文本并且不同标签设定不同的字体大小和字体粗细。

你注意到这条线被冲了吗？这可以通过设置 `stroke-width` 和 `stroke-dasharray` 属性来实现。用 `stroke-dasharray` 你可以定义改变形状轮廓的虚线和间隙的样式。

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

网格线变得棘手。我必须将 `stroke-width: 0` 应用于组中的路径元素以隐藏图表的框架，我还通过设置线条的不透明度来降低它们的可见性。

所有其他 CSS 规则涵盖了源代码中可以找到的字体大小和颜色。

## 结束我们的 D3.js 条形图教程

D3.js 是一个令人惊叹的 DOM 操作库。它的深度隐藏了无数等待发现的珍宝, 实际上并未真正隐藏，已有很好的记录。这篇文章只涵盖了工具集的片段，这些片段有助于创建一个不那么平庸的条形图。

**继续，探索它，使用它，创造壮观的视觉效果！**

顺便说一下，这是 [源代码](https://codepen.io/kingrychan/pen/MqebJZ?editors=0010) 的链接。

你用 D3.js 创造了一些酷的东西吗？和我们分享！如果你有任何问题，或者想要关于这个主题的另一个教程，请留言！

谢谢你的阅读，下次再见！

![]()