---
title: (翻译)flex-basis和width不同
tags:
  - CSS
  - 翻译
abbrlink: df26
---

> 原文： [The Difference Between Width and Flex Basis](https://gedd.ski/post/the-difference-between-width-and-flex-basis/) 有删节

### Flex Item项目公式

```
centent -> width -> flex-basis（被max-width | min-width） 限制
```

如果没有`flex-basis`属性，那么`flex-basis`将会和 `width`保持一致

如下代码

```css
container{
    display: flex;
    width: 1000px;
}
```

<!--more-->

![](https://chaochaogege.net/images/20181103100849.png)

### 设置一个宽度

让我们放置几个确定宽度的`item`

```css
item{
    width: 200px;
    height: 200px;
}
```
![](https://chaochaogege.net/images/20181103101404.png)

我们的 `flex container` 有足够的空间，所以我们的`item`可以很好的适配

![](https://chaochaogege.net/images/20181103101710.png)

在这个例子中我们没有明确`flex-basis` ， 所以它默认是 `auto` ，这会和`width`保持一致

到目前为止，设置一个`width`等同于设置`flex-basis`

### 设置 `flex-basis`

让我们看一下给已经设置`width`的容器设置 `flex-basis` 会发生什么。

```css
item{
    width: 30px;
    flex-basis: 250px;
}
```
![](https://chaochaogege.net/images/20181103102107.png)

正如你所看到的，当添加了`flex-basis` ， 我们盒子的 `width` 被忽略了， 所以我们根本没必要去设置 `width`

```css
item{
    flex-basis: 250px;
}
```

最终我们每个 `item` 的 `flex-basis`  是 250px， 因为我们有足够的空间去盛放他们， 他们将会很好的适配 `container`

![](https://chaochaogege.net/images/20181103102342.png)

记住 `flex-items` 的公式

```
content -> width -> flex-basis(由 max|min-width限制)
```

最重要的是公式最后的 `flex-basis` ，所以我们应该只使用 `flex-basis` 而不是 `width` 或者 `height`。 例外情况是因为 `Safari` 仍然有一个古老的 flexbox bug， 它不会正确的将 `flex-shrink` 应用到 `items` 上， 这时候应该使用 `height` 而不是 `flex-basis`

### flex-basis 被 max-width限制

`flex-basis` 的值被`item` 的 `min-width` 和`max-width` 限制。 看看当我们能给 `item` 设置 `max-width` 的时候会发生什么？

```css
item{
    flex-basis: 250px;
    max-width: 100px;
}
```

![](https://chaochaogege.net/images/20181103102903.png)

尽管我们的 `flex-basis` 被设置成 250px，但是却被 `max-width` 的100px限制。 所以在这种情况下我们的 `flex-basis` 是100px。 我们的`item`会像下面这样适配我们的 `flex container`

![](https://chaochaogege.net/images/20181103103029.png)

现在让我们在 `item` 上使用 `min-width` 来看看会发生什么

```css
item{
    flex-basis: 100x;
    min-width: 250px;
}
```

![](https://chaochaogege.net/images/20181103103142.png)

即使我们给 `flex-basis` 规定了 100px， 但他却不能小于 `min-width` 规定的250px。 所以我们最终的 `flex-basis` 是250px

![](https://chaochaogege.net/images/20181103103725.png)


## 那么准确来说 flex-basis 到底是什么

现在我们理解了 `width` 仅仅是 `flex-basis` 缺失时的一个替代， `min-width` 和`max-width` 仅仅是 最终 `flex-basis` 的低限制和高限制

你应该已经注意到我们所有的插图都配有一张我们在放置到 `flex-container` **之前** 的图片。

我们这样做是因为准确说来 `flex-basis` 是**flex-items 在放到 flex-container 之前的大小**。 它是理想的或者假象的大小。 但是 `flex-basis` 并**不保证**大小。只要浏览器将 `flex-items` 放置到 `flex-container` ， 那么一切都将取决于 flexbox 的其他属性。 在我们上面的例子中你可以看到 `flex-items` 很完美的适配了 `flex-container` ， 这是因为所有最终的 `items `的`flex-basis` 之和 小于等于 `flex-container` 的宽度（1000px)， 这样很好，但是在 全部的 `items` 的 `flex-basis` 相加之后， `flex-container` 经常会没有空间 

## 当没有足够空间的时候

假如我们想要放置到 `container` 中更多 `flex-basis：200px` 的 `items`

![](https://chaochaogege.net/images/20181103104744.png)

在放置到 `container` 之前他们每个占据 200px 空间， 总共占用 1600px。 但是我们的 `container` 只有仅仅 1000px， 当根据 `flex-basis` 的值使得 `items` 没有足够的空间放置的时候， `flex-items` 默认会缩小去适配。

![](https://chaochaogege.net/images/20181103104947.png)

所有的 `items` 起初是 200px 宽度， 但是因为我们没有足够的空间，他们会缩小到每个 items 占据 125px， 这个收缩比例是由 `flex-shrink` 规定（默认是 `flex-shrink：1` ， 即以相同比例缩小）

当然你也可以覆写 `flex-shrink` 默认值 ，来决定缩小的比例，或者使用 `flex-shrink:0` 来明确不缩小

## 当有更多的空间可以获得的时候

在计算完 `flex-basis` 之后，我们经常会有更多的空间剩余

```css
item{
    flex-basis: 100px;
}
```

![](https://chaochaogege.net/images/20181103110033.png)

我们可以让 `flex-items` 在放置到 `container` 之后去填充满剩余的空间

这就是 `flex-grow` 属性做的工作。 默认值是 0 ，即使有剩余的空间也不使用

让我们在每一个 `item` 上都设置 `flex-grow： 1` 来等比例填充满剩余的空间

```css
item{
    flex-basis: 100px;
    flex-grow: 1;
}
```

![](https://chaochaogege.net/images/20181103110338.png)

> 如果 `flex-grow` 会使得`item`宽度为300px，但 `max-width：250px` ， 那么最终的宽度为250px，同理 `min-width`

填充和缩小都是 `flexbox` 酷炫的地方， 也正是因为这样他才非常适合 响应式UI设计

## width vs flex-basis

你现在知道了 `width` 和 `flex-basis` 的区别以及 `min-width` 和 `max-width` 如何影响最终的 `flex-basis`， 当你使用 `flex-direction： column | column-reverse` 时，上面的也同时适用于 `height`
