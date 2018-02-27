---
title: 浅析JavaScript之核心——作用域与作用域链
date: 2016-07-27 01:12:30
tags: javascript
---

如果说上个礼拜的学习，什么使我最为印象深刻，最为醍醐灌顶，那恐怕是作用域与作用域链莫属了。
作用域与作用域链是JavaScript的核心，同时，它更是JavaScript的基础。学习一门语言，不仅仅是学习它的惯常用法，还需要学习在日常使用中背后的一些语法规则。学习计算机语言也是如此。
于是乎，接下来我们就来讨论一下JavaScript的核心之作用域与作用域链。

<!--more-->

## 作用域是什么

理解作用域与作用域链，要首先理解环境的概念。执行环境，也称为环境，定义了变量或函数有权访问的其他数据，决定了它们各自的行为。而一个变量或者函数的作用域，是程序中定义这个变量或是函数的区域。当代码在一个环境中执行，会创建变量对象的一个作用域链。

## 作用机制

按照《JavaScript高级程序设计》中所讲解的，作用域与作用域链的作用机制其实是：每个函数都有自己的执行函数。当执行流进入一个函数时，函数的环境就会被推入一个环境栈中。在函数执行之后，栈将其环境弹出，把控制权返回给之前的执行函数。

其实，按照我的理解，作用域与作用域链的作用机制，可以用三句话来概括。

①  作用域的前端始终是当前执行的函数，而全局环境的变量对象始终是作用域链中的最后一个对象。
②  子域可以访问父域，而父域不能访问子域。
③  JavaScript解析器会搜寻最近的定义的值。如果搜寻不到，将沿着作用域链逐级向上寻找。

下面举个例子来说明一下：

``` javascript
var color = "blue";

function changeColor(){
	var anotherColor = "red";

	function swapColors(){
		var tempColor = anotherColor;
		anotherColor = color;
		color = tempColor;

		//这里可以访问color anotherColor 和tempColor;		
	}

	//这里可以访问color 和 anotherColor  但不能访问 tempColor swapColors();
}   

//这里只能访问color；
changeColor();

``` 

在这个例子中，我们可以看到总共有三个执行环境：全局环境，changeColor()的局部环境和 swapColors() 的局部环境。这块代码的作用域链是：swapColors() 在作用域链的前端，有着tempColor 变量，接下来是changeColor() 有着anotherColor变量，最后是全局环境下的color变量。因此，按照子域可以访问父域，而父域不能访问子域的原则，在swapColors()局部环境中，可以访问color anotherColor 和tempColor; 在changeColor()的局部环境中，可以访问color 和 anotherColor  但不能访问 tempColor swapColors();而在全局环境下，只能访问color。

然而，我们再来看看下面这个例子：

```javascript

var func = function() {
	var num = 10;
	var sub_func = function() {
		 var num = 20;
		 alert(num);
	};
	sub_func();
	//alert(num);
};
func();

```

在这个例子中，作用域链为sub_func，下有num变量，过来是func，下也有一个num变量，上述代码打印出来的数字是20，倘若将sub_func 中的alert(num)去掉，并取消func中alert(num)的注释，则打印出来的数字是10。

再来重新做一个实验，如果将num = 20；去掉，则打印出来的数字是10，如果再把num=10；去掉，则打印出来的数字是undefined。这个实验验证了子域可以访问父域，而父域不能访问子域的原则，并且表明，浏览器会在当前作用域中，寻找是否有这个变量，如果没有，则去父域中寻找，如果沿着作用域与作用域链依然找不到该变量，则该变量为undefined。

## 常见错误之变量提升

变量提升，指的是把后面定义的变量或函数提升到前面的定义中。

首先，来看看变量的提升，上例子：

``` javascript
x=1;
alert(x);
var y = function() {
	alert(x); 
	var x = 2;
	alert(x); 
}
y();
```
这段代码其实会输出1，undefined，2.为什么呢？原因在于，JavaScript解析器在执行这个y函数的时候，会把它里面的声明变量提前到他的作用域的最前面进行声明，上述代码等同于下面这段代码：

``` javascript
x=1;
alert(x);
var y = function() {
	var x;
	alert(x); 
	x = 2;
	alert(x); 
}
y();
```
在第一次打印时，x的值为1，第二次打印时，x变量被提升到该函数的最前端，此时的x尚未定义，所以第二次打印出undefined，第三次打印时，x已经被定义为2，所以打印出2。

然后，再来看看函数的提升

一般常用的定义函数的方式有两种，一种是使用函数声明语法定义的，如下

```javascript
function sum (num1, num2){
	return sum1 + sum2;
}
```

一种是用函数表达式定义的，如下

```javascript
var sum = function(num1, num2){
	return sum1 + sum2;
}
```

这两种定义函数的区别就在于，解析器在向执行环境中加载数据时，会率先读取函数声明，并使其在执行任何代码之前可用（可以访问）；而对于函数表达式，则必须等到解析器执行到他所在的代码行，才会真正被执行。看下面的例子：

```javascript
alert(sum(10,10));
function sum (num1, num2){
	return sum1 + sum2;
}
```

这段代码可以执行。原因在于，此处的函数是函数声明语法定义的。在代码执行之前，解析器就已经通过函数声明提升的过程，读取并将函数声明添加到执行环境中。对代码求值之前，引擎会在第一遍声明函数并将它们放在源代码树的顶部，所以，即使函数声明在调用它的函数后面，JavaScript引擎也能将声明提升到顶部，从而不影响函数的调用。

然而，我们把代码改写成等价的函数表达式，代码如下：

```javascript
alert(sum(10,10));
var sum = function(num1, num2){
	return sum1 + sum2;
}
```

上述代码将会在执行期间发生错误。因为在于，在执行到函数表达式代码行之前，JavaScript引擎不会执行这行代码，更不会将函数表达式提升到顶部。返回这个例子，在执行到调用语句时，变量sum中不会保存有对函数的引用，因此将会导致"unexpected identified"，不会执行到下一行。 

最后，我们再来看看一个坑：

```javascript
foo(); 

var color = red;

if(color) {
    function foo() {
        console.log("red");
    }
} else {
    function foo() {
        console.log("blue");
    }
}}
```

这段代码最终打印出来的值是blue，而不是red。按照我们之前所说的，函数声明会被提升到顶部，然而同名函数不会重复定义两次，而是后面定义的函数会覆盖前面定义的内容。因此，foo()函数在一开始就定义为：console.log(blue)，于是这个if函数就变得不起作用了。

## 何用之有？

在一开始说过，作用域与作用域链是JavaScript的核心，也是基础。因此，运用作用域与作用域链，可以更好的理解关于闭包，以及处理好一些坑。
再者，由于变量提升是先定义变量在给变量赋值，那么这样无疑是消耗了一些时间，对于网页的性能也是有所降低。
所以，这基础的东西并不是没有用处的~
