---
title: 深入理解 BFC
date: 2017-06-06 00:36:04
tags: css
---

我的博客长草了，特地前来除除草。今天想写的内容是 BFC 。
先前师兄师姐给我面试的时候，我听见 BFC 还觉得很陌生，之后研究了一番，发现，其实这是很普遍的一种解决一些常见问题的方式。于是在网络上看了许多文章，找了找 <a href="https://www.w3.org/TR/CSS22/visuren.html#normal-flow">www.w3.org</a> 里面关于这方面的解释，是以总结出以下这篇博文。

<!--more-->

## 什么是 BFC

根据 CSS2.2 可知，文档流分为定位流、浮动流和普通流三种。在普通流中的盒子都属于 formatting context （简称 FC）即格式化上下文。块级盒子参与块级格式化上下文，行内盒子参与行内格式化上下文，表格盒子参与表格格式化上下文，但一个盒子不会同时参与多种格式化上下文。

“块级格式化上下文”，就是 Block Formating Context —— 简称 BFC ，是用于布局块级元素的渲染区域。它是一个独立的渲染区域，规定了内部的盒子如何布局，且容器里的子元素与外部区域互不影响。

### 触发条件

在 <a href="https://www.w3.org/TR/CSS22/visuren.html#normal-flow">www.w3.org</a> 中，写出了哪些元素会产生 BFC 的描述：

> Floats, absolutely positioned elements, block containers (such as inline-blocks, table-cells, and table-captions) that are not block boxes, and block boxes with 'overflow' other than 'visible' (except when that value has been propagated to the viewport) establish new block formatting contexts for their contents.

以上文字也就是说，满足下列条件之一的元素，就会触发 BFC ：

* 绝对布局
* 浮动值不为空
* overflow 不为 visible
* 不是块级元素却拥有块级元素特性的块级包含区域 (eg display 值为 inline-block, table-cell or table-caption)

### 布局特点
 <a href="https://www.w3.org/TR/CSS22/visuren.html#normal-flow">www.w3.org</a> 中，对于 BFC 的布局特点，有以下介绍：

> In a block formatting context, boxes are laid out one after the other, vertically, beginning at the top of a containing block. The vertical distance between two sibling boxes is determined by the 'margin' properties. Vertical margins between adjacent block-level boxes in a block formatting context collapse.

> In a block formatting context, each box's left outer edge touches the left edge of the containing block (for right-to-left formatting, right edges touch). This is true even in the presence of floats (although a box's line boxes may shrink due to the floats), unless the box establishes a new block formatting context (in which case the box itself may become narrower due to the floats).

这两段话的主要意思是：

在一个 BFC 的内部的垂直方向上，内部的盒子是接连布局的。第一个盒子的定位始于包含块的最顶部，在垂直方向上的相邻元素间的间距会只保留 margin 值比较大的那个元素的 margin 值的大小，也就是我们常说的 margin 重叠的现象。

在水平方向上，每一个盒子的左外边距都会紧贴着包含块的左内边距（从右到左布局的相反），在浮动布局中依然成立（尽管盒子的线盒会因为浮动而收缩），直到在 BFC 中的盒子自身建立起了一个新的 BFC （在这种情况下，这个盒子本身可能会因为浮动而变窄）。

我们可以通过这两段话总结并补充出以下观点：

* BFC 内部的垂直方向上，是一个接着一个排列的。
* BFC 内部的相邻元素之间有 margin 重叠的现象。
* BFC 的内部的水平方向上，每个元素的左外边距都紧贴着包含块的左内边距（从右到左布局的相反）。
* BFC 就是页面上的一个独立的容器，容器里面的子元素不会影响到外面的元素。反之也如此。
* BFC 的区域不会与 float box 重叠。
* 计算 BFC 的高度时，浮动的元素也会参与计算。

## BFC 的作用解析

根据先前我们所提及的 BFC 的布局特点，我们可以利用 BFC 的特性，解决一些常见的问题。

### margin 叠加问题

在 BFC 中，垂直方向上的盒子会发生 margin 重叠的现象。例如：

```
<div class="wrapper">
	<div class="box">
		<h1>BFC test</h1>
		<p>just test just test</p>
	</div>
	<div class="box">
		<h1>BFC test</h1>
		<p>this margin</p>
	</div>
</div>
```

以上这段代码，所呈现出来的效果，是这样的：![margin-before](深入理解-BFC/bfc-margin-before.png)

可见，第一个 box 和 第二个 box 之间，是保留了第一个 box 的 margin ，如果要使 margin 不重叠，只需要利用 BFC 中，容器里面的子元素不会影响到容器外面也不会受容器外面影响这一条，把第二个 box 的外面包裹一层叫做 test 的容器，并且使这个容器独立成一个 BFC 即可。

HTML：
```
<div class="wrapper">
	<div class="box">
		<h1>BFC test</h1>
		<p>just test just test</p>
	</div>
	<div class="test">
		<div class="box">
			<h1>BFC test</h1>
			<p>this margin</p>
		</div>
	</div>
</div>
```

CSS：
```
.test {
	overflow: hidden;
}
```

效果如下：![margin-after](深入理解-BFC/bfc-margin-after.png)


### 清除内部浮动

我们还可以利用 BFC 来清除内部浮动。例如下面这个例子，如图所示， .wrapper  里面的 .child 是浮动的盒子，在这个情况下， .wrapper 并不能被 .child 撑开而收缩了。

![float-before](深入理解-BFC/bfc-float1-before.png)

HTML:
```
<div class="wrapper">
	<div class="child"></div>
	<div class="child"></div>
</div>
```

CSS:
```
.wrapper {
	width: 100%;
}

.child {
	width: 150px;
	float: left;
	height: 400px;
	margin: 10px;
	background: #f45460;
}
```

因此，如果要清除内部浮动，撑开这个 .wrapper 的话，我们可以利用 BFC 的这一个布局特点是：计算 BFC 的高度时，浮动的元素也会参与计算。将 .wrapper 触发 BFC 。

![float-after](深入理解-BFC/bfc-float1-after.png)

```
.wrapper {
	overflow: hidden;
}
```

### BFC 与浮动元素构成的分栏布局

以下代码的本意是想要获得一个两栏布局，可是因为左边的 .aside 是浮动的，所以右边的 .main 会挤到右边。

![layout-after](深入理解-BFC/bfc-layout-before.png)

HTML:
```
<div class="wrapper">
	<div class="aside"></div>
	<div class="main">
			ayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayayaya
	</div>
</div>
```

CSS:
```
.wrapper {
	width: 300px;
}

.aside {
	width: 100px;
	height: 150px;
	float: left;
	background: #efefef;
}

.main {
	height: 200px;
	background: #f45460;
	word-wrap: break-word;
}
```
 BFC 有这一特性： BFC 的区域不会与 float box 重叠。因此，只需要触发右边的 .main 的 BFC ，就可以达到分栏布局的目的。

在 .aside 中加一行 `overflow: hidden`;

![layout-after](深入理解-BFC/bfc-layout-after.png)

## 参考资料

 <a href="https://www.w3.org/TR/CSS22/visuren.html#normal-flow">www.w3.org</a> 中对于 Block Famating Context 的解释
