---
title: 重新认识border-radius
date: 2016-07-18
tags: [css3, css]
---

其实在之前也接触过border-radius，不过只是简单的用“border-radius: 50%;”  来画一个圆；或者是用"border-radius: 50px;" 来画出想要的圆角矩形。然而，这个礼拜听了果姐姐讲的关于border-radius的内容，并且翻阅了几篇博文，终于让我解锁了border-radius的新姿势。

<!--more-->

##  border-radius的原理与规则

首先，border-radius的中文含义是边框半径。它是通过设置边框四周的角的边框半径从而达到画圆角矩形，圆，椭圆的效果。

和margin，padding有些相似，border-radius也是有四个按照顺时针顺序值重复的值，然而和margin，padding不同的是，border-radius的四个属性分别是“topleft, topright, bottomright, bottomleft”。border-radius可以同时设置1-4个值，规则如下：

``` javascript
border-radius:50px; //表示4个圆角都使用这个值

border-radius:50px 25px; //表示左上角和右下角使用第一个值，右上角和左下角使用第二个值

border-radius:25px 10px 50px; //左上角使用第一个值，右上角和左下角使用第二个值，右下角使用第三个值

border-radius:25px 10px 50px 0;//左上角、右上角、右下角、左下角(顺时针顺序)
```

而且border-radius还有水平半径和垂直半径之分。顾名思义，在水平方向上的半径叫做水平半径，在竖直方向上的半径叫做垂直半径。

水平半径和垂直半径之间用/来区分。如果只设置一组值，如同前面我们所说的，表示水平半径与垂直半径相等，如果设置两个值，并且用/来区分，第一组值表示水平半径，第二组值表示垂直半径它们都遵循值重复的原则。

``` python
border-radius:50px/25px;  //表示水平半径为50px，垂直半径为25px

border-radius: 100px 25px 80px 5px / 45px 25px 30px 15px;      //表示水平半径为左上100px，右上25px，右下80px，左下5px，垂直半径为左上45px，右上25px，右下30px，左下15px

```

除了同时设置四个圆角以外，还可以单独对每个角进行设置。对应四个角，他们的属性分别是：

　　border-top-left-radius
　　border-top-right-radius
　　border-bottom-right-radius
　　border-bottom-left-radius

同样的，它们也遵循上述值重复与水平半径和垂直半径的原则。以下不再赘述。

## border-radius好玩的应用

知道了border-radius的原理和规则，我们就能愉快的用它来搞(wei)大(suo)新(yu)闻(wei)了。

1. 圆角矩形

这玩意应该是比较常接触的border-radius的一种应用了。他的用法很简单，可以用以下代码实现。
``` python
border-radius: 50px;
```

2. 实心圆

border-radius可以用百分比来定义圆角的半径，因此，当其单个值被设置为50%时，可以绘出1/4个圆弧。因此，当div的宽高相同，border-radius设置为50%时，即可以画出一个圆。画实心圆可以用以下代码实现

``` python
div{
width：200px；
height：200px；
background-color：#FFF；
border-radius: 50%;
}
```

3. 空心圆

画空心圆，可以用div内部颜色与背景色相同的方法，设置边框颜色，其中需要注意的是，border（边框）的宽度需要小于高度，宽度的一半。代码如下

``` python
div{
width：200px；
height：200px；
background-color：#FFF；
border：5px #efefef solid；
border-radius: 50%;
}
```

4. 操场图案

操场是两边半圆中间夹着一个矩形。前面提到，当border-radius的单个圆角值设置为50%时，可以画出一个1/4的圆弧。然而我们知道50%是相对于div而言的，也就是说，如果用border-radius画出一个操场图案的话，可以把圆角半径设置为较短的一边的50%，使得两边画出半圆。代码如下：

``` python
div{
width：600px；
height：200px；
background-color：#FFF；
border-radius: 100px;
}
```

5. 椭圆

椭圆是一段光滑的曲线，因此，如果要画椭圆的话，需要把水平半径设置为宽度的一半，垂直半径设置为高度的一半。代码如下：

``` python
div{
width：200px；
height：100px；
background-color：#FFF；
border-radius: 100px/50px;//此处用50%也行。
}
```

## 注意兼容

border-radius只有在以下版本的浏览器：Firefox4.0+、Google Chrome 10.0+、Opera 10.5+、IE9+支持border-radius标准语法格式，对于老版的浏览器，border-radius需要根据不同的浏览器内核添加不同的前缀。比说Mozilla内核需要加上“-moz”，而Webkit内核需要加上“-webkit”等，那么为了能兼容各大内核的老版浏览器，我们需要作出兼容。
