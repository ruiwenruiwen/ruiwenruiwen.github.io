---
title: JavaScript 函数式编程（二）：柯里化函数与偏函数
date: 2017-07-18 21:14:19
tags: [javascript, 函数式编程]
---

这是我在 JavaScript 函数式编程学习的第二篇笔记，写的内容是柯里化函数和偏函数。柯里化函数和偏函数都是高阶函数。因此，在进入正文之前，我想先描述一下，什么是高阶函数。

高阶函数至少满足以下一条特性：
+ 以一个函数作为参数
+ 返回值为一个函数

如下例子所示。这是一个简单的高阶函数。

<!--more-->

```javascript
function add(x, y, fn) {
	return fn(x) + fn(y);
}
```

显然，这个函数接受 `x`，`y` 和函数 `fn` 作为参数，返回 `fn(x) + fn(y)` 的值。假如 `fun` 定义为

```javascript
function fun(a){
	return a * a;
}
```

那么 `add(1, 2, fun) = 5`。

## 柯里化函数

柯里化函数是把接受多个参数的函数变换成接受一个单一参数(最初函数的第一个参数)的函数，并且返回接受余下的参数且返回结果的新函数的技术。把上述高阶函数变换成柯里化函数的表现形式：

```javascript
function add(fn){
	return (x) => {
		return (y) => {
			return fn(x) + fn(y);
		}
	}
}

let squareAdd = add(fun); 
// 把 add 函数里面的函数 fn 定义为 fun 函数  即 return fun(x) + fun(y)

let squareAdd1 = squareAdd(1); 
// 把 add 函数里面的参数 x 定义为 1  即 return fun(2) + fun(y)

console.log(squareAdd1(2));     // 5   =>  fun(1) + fun(2) 

console.log(squareAdd1(3));     // 10  =>  fun(1) + fun(3) 


let squareAdd2 = squareAdd(2); 
// 把 add 函数里面的参数 x 定义为 2  即 return fun(2) + fun(y)
 
console.log(squareAdd2(2));     // 8   =>  fun(2) + fun(2)

console.log(add(fun)(1)(2));    // 5   =>  fun(1) + fun(2)
```

从上述例子可以知道，在柯里化函数中，可以一次性赋值所有的参数，也可以一步一步的将参数赋值。其中，对于部分赋值后的参数，如果给定一个变量去保存，则可以重复使用。这是柯里化函数的一个特点：提高专用性。然而柯里化函数也有一定的缺点。创建大量嵌套作用域和闭包会带来开销，而且柯里化函数逐渐返回消耗参数的函数， 消耗比较大。因此，出于各种层面上考虑，偏函数比柯里化更加实用。

## 偏函数

前面说了，柯里化函数会消耗参数。而偏函数指的是固化函数的一个或一些参数，从而产生一个新的函数，它每次处理的不一定只有一个参数。

上述例子，用偏函数来实现：

```javascript
function partial(fn, a){
	return function(){
		return fn(a) + fn.apply(null, arguments);
	}
}

var addFun = partial(fun, 1);

console.log(addFun(2));         // 5   =>  fun(1) + fun(2)
```

其中，`addFun` 定义 `partial` 函数的参数，固化 `fn` 和 `a` 这两个参数，在引用的时候，再将 `partial` 的最后一个参数赋进去。可见，在偏函数应用的例子里面，最内层的处理函数是确定的。也就是说，我们已经预知了最终处理的方式，只需等待在运用时，把参数传入再完成。

实现偏函数的方法有 `apply()` `call()` 和 `bind()`。以下简单的描述一下这几种方法。

### apply() 和 call()

`apply()` 和 `cal()` 方法调用一个函数，其具有一个指定的 `this` 值，以及接受一些参数。`apply()` 和 `call()` 不同的点在于，`apply()` 接受的是作为一个数组（或是类数组对象）提供的参数；而 `call()` 接受的是若干个参数。

他们的用法是：
```javascript
fun.call(this, arg1, arg2);
fun.apply(this, [arg1, arg2])；
```

举个栗子，找出数组中最大的值。

```javascript
var  numbers = [7, 233 , 666 , 0 ]; 
var maxWithApply = Math.max.apply(Math, numbers),          // 666
    maxWithCall = Math.max.call(Math, 7, 233 , 666 , 0); // 666
```

`number` 是一个数组对象，他本身没有 `max` 方法，但是 `Math`对象 有，我们就可以借助 call 或者 apply 使用其方法。然后遵循 `apply()` 接受的是作为一个数组（或是类数组对象）提供的参数；而 `call()` 接受的是若干个参数的原则，可以按照如上例子引用。

### bind()

`bind()` 方法创建一个新的函数, 当被调用时，将其 `this` 关键字设置为提供的值，在调用新函数时，在任何提供之前提供一个给定的参数序列。

借用 MDN 中对 `bind()` 的一个解释：

```javascript
function list() {
	return Array.prototype.slice.call(arguments);
}

var list1 = list(1, 2, 3); // [1, 2, 3]

// Create a function with a preset leading argument
var leadingThirtysevenList = list.bind(undefined, 37);

var list2 = leadingThirtysevenList(); // [37]
var list3 = leadingThirtysevenList(1, 2, 3); // [37, 1, 2, 3]
```

`bind` 可以使一个函数拥有预设的初始参数。这些参数（如果有的话）作为 `bind()` 的第二个参数跟在 `this`（或其他对象）后面，之后它们会被插入到目标函数的参数列表的开始位置，传递给绑定函数的参数会跟在它们的后面。因此，尽管是没有定义 `list2` ， `list3` 的值依旧是 `[37, 1, 2, 3]`。因为 `bind()` 绑定的值会定义在目标函数的参数列表最开始，而 `call()` 和 `apply()` 所绑定的值会在其之后。

而且，由于 `bind` 的内部实现机制，在 Javascript 中，多次 `bind()` 是无效的。这是因为， `bind()` 的实现，相当于使用函数在内部包绑定了一个 `call / apply` ，第二次 `bind()` 相当于再绑定第一次 `bind()` ,故第二次以后的 `bind` 是无法生效的。

### apply(), call() 和 bind() 的比较

从上述描述，可以知道，apply(), call() 和 bind() 有以下特点：
1. 三者都是改变原本函数的 this 值，使得让原本没有此方法的对象运用此方法。
2. 三者的第一个参数都是执行的函数指向的 this 值，第二个参数是函数执行时的参数。
3. `bind` 返回的是一个函数，并且等待下一次调用，而 `apply` 和 `call` 是立即调用，返回调用的结果。

## 总结

柯里化函数和偏函数的异同，在于

+ 柯里化函数：是一种解构技巧，用于把多元函数分解为多个可链式调用的层叠式的一元函数，这种解构可以允许你在其中定义一个或多个参数，但是柯里化函数本身不提供任何参数——它提供的是调用链里的最终处理函数。
+ 偏函数应用：是一种转换技巧，通过预先传入一个或多个参数来把多元函数转变为更少一些元的函数甚或是一元函数。最终的处理方式和结果是限定的，每一层的函数调用所传入的参数都将逐渐参与最终处理的过程。
