---
title: javascript 函数式编程（一）：Applicative 函子
date: 2017-07-15 17:22:04
tags: [javascript, 函数式编程]
---

函数式编程逐渐成为一种编程流行趋势，不论是前端还是后端。函数是 JavaScript 的一等公民，JavaScript 当然支持函数式编程。

然而，我为什么选择学习函数式编程呢？仅仅因为它是一个热门的编程范式吗？

当然不是。我选择函数式编程的原因有：
+ 函数式编程是以函数作为行为单元，在一些情况下能避免 JavaScript 因为本身的缺陷而产生的一些问题，更为安全。
+ 函数式编程高度抽象，有利于函数单元的模块化，使代码更为简洁，更易于修改和维护。
+ 我渐渐意识到，目前我对于 JavaScript 的学习缺乏对于数据流的处理相关知识。而当前流行的 MV* 架构需要一定的对于数据的处理的能力。

<!--more-->

言归正传。本篇博文主要讲的，是函数式编程中的 Applicative 函子。
在《 JavaScript 函数式编程》中，有如下解释：”Applicative 函子定义为函数 A 作为参数提供给函数 B “。而 Applicative 函子的三个典型的例子是 `map`, `reduce`. 和 `filter`。我将在接下来的篇幅中借助数组原生的方法对 Applicative 函子进行略微的介绍。

## 用数组自带方法解决数组问题

`map`, `reduce`. 和 `filter`，都是 `Array.prototype` 的方法。在最近写的代码中，对于数组的处理，我渐渐的少使用 `for` ，`while` 等循环，而倾向于使用原生的方法。这样使得我的代码更加简洁。

### Array.prototype.map()
介绍数组的原生迭代方法， `map` 当然是要说明的了。

`map()` 方法使给定的数组中的每个元素都调用一个提供的函数，然后将返回的结果按顺序拼接成一个新的数组。

```javascript
let array = arr.map(function callback(currentValue, index, array) { 
    // 此处必须有 return 值，否则将构成空数组
}[, thisArg])
```

`map` 方法接受如下几个参数：
+ 给数组中每个元素执行的函数  `callback` 
> + `currentValue`：当前数组正在处理的值
> + `index`：当前数组正在处理的值的索引
> + `array`：调用 `map` 方法的数组

+ `thisArg`：在执行 `callback` 时使用的 `this` 值

之所以说 `map` 是 Applicative 函子的典型例子，正是以为其定义了一个回调函数作为参数，这个回调函数返回的值按照原本数组的顺序依次构成一个新的数组。这种编程方式使得代码更加简洁优雅。

说了这么多，上一个 MDN 上的例子来解释 `map` 的用法

```javascript
const numbers = [1, 2, 3, 4, 5];

let arr = numbers.map((currentValue, index, array) => {
    console.log(`currentValue = `, currentValue);
    console.log(`index = `, index);
    console.log(`array= `, array);
    return currentValue * 2;
}, numbers);

console.log(`arr `, arr);

// 1, 0, [1, 2, 3, 4, 5]
// 2, 1, [1, 2, 3, 4, 5]
// 3, 2, [1, 2, 3, 4, 5]
// 4, 3, [1, 2, 3, 4, 5]
// 5, 4, [1, 2, 3, 4, 5]

// [2, 4, 6, 8, 10]

```

可见，`map` 可以让数组中的每一个元素进行一个函数，然后返回的值会构成一个新的数组。这里需要说明的是，`map` 的回调中，必须有 `return` 值，否则将返回一个空数组，那使用 `map` 也就没有意义了。

### Array.prototype.reduce()

`reduce` 方法是对数组中的每个元素逐一调用，以上一次保存的值（初次调用可设置一个初始值）和当前数组正在处理的值为参数，执行一个指定的函数，最终返回前几次调用的累加结果。

```javascript
array.reduce(function(accumulator, currentValue, index, array), initialValue)
```

`reduce` 的参数和 `map` 的类似：
+ 给数组中每个元素执行的函数  `callback` 
> + `accumulator`：数组调用过程中前几次累积的值（第一次调用为设定的初始值 `initialValue`）
> + `currentValue`：当前数组正在处理的值
> + `index`：当前数组正在处理的值的索引
> + `array`：调用 `reduce` 方法的数组

+ `initialValue`：给定的初始值

```javascript
var total = [0, 1, 2, 3];
total.reduce(function(sum, cur) {
  return sum + cur;
}, 0);
// total is 6

var mul = [1, 2, 3];
mul.reduce(function(sum, cur) {
  return sum * cur;
}, 1);
// mul is 6

var flattened = [[0, 1], [2, 3], [4, 5]].reduce(function(a, b) {
  return a.concat(b);
}, []);
// [0, 1, 2, 3, 4, 5]
```

很多地方把 `reduce` 方法翻译成一个从左到右处理的累加器，需要注意的是，此处的“累加”并不是简单地是加法的累加。根据上述例子，可以知道，此处的“累加”其实是对于上一次调用和这一次调用的结果的累加，可以是加法——累加，乘法——累乘，除法——累除，甚至是对于数组的操作。

同样的， `reduce` 也需要有返回值。

## 自己实践的 Applicative 函子

上述介绍了何为 `map()` ，何为 `reduce()` ，何为 Applicative 函子。接下来要说的，是我从 <a herf="https://nodeschool.io/zh-cn/#workshoppers">NodeSchool</a> 的编程训练中遇到的一个问题和这个问题带给我的启发。

> 根据 `reduce` 方法自己实现 `map` 方法。

以下是我的解法：

```javascript
function arrayMap(arr, fn, thisArg) {
	return arr.reduce(function(acc, curVal, index, arr){
		acc.push(fn.call(thisArg, curVal, index, arr));
		return acc;
	}, []);
}

// 其中，fn 为 map 调用的 callback

// 调用时可为：
// arrayMap(arr, function(){
//	 /** map 的回调函数 **/
// }, thisArg);
```

根据上述所说可知，`reduce` 是对于前几次操作的累加。也就是说，我们可以通过这个性质，将初始值定为一个数组，在通过 `reduce` 操作，使数组中的每一个元素都进行 `fn` 函数（`map` 的回调函数），然后将 `fn` 返回的结果 `push` 到 `arr` 数组中。
因为 `map` 方法还可以设置 `thisArg` 参数，改变在执行 `callback` 时使用的 `this` 值，而改变想要 `this` 的指向，那在这里就应该使用 `call()` 了。
由于 `reduce` 方法会累加前几次进行的结果的值，所以在 `fn` 执行完毕后，返回 `arr` 数组，就达到实现原生 `map` 方法的效果——让指定的 `arr` 执行指定的 `callback` 操作。

在这个练习之前，我很狭隘的把 `reduce` 认识为只是单纯的对数字的累加，后来查阅资料，发现原来还可以把初始值设定为一个空数组。因此才有了上述反复说明的， `reduce` 不仅仅是对加法的操作。

## 总结

经过这几天初探，我竟对函数式萌发出“相见恨晚”的感觉。高度抽象和简洁优雅是我对函数式编程的初印象。更重要的是，这让我更加深入认识了，以前没有主要到的一些方法的细节。
本篇博文只是用个别例子介绍了 Applicative 函子以及函数式编程给我的最初感受。当然，数组中的原生方法还有很多，这里就不再一一赘述。毕竟本篇博文只是我对函数式编程学习心得记录的一个开端。

--------------------------------------------------------------

参考资料：
+ 《 JavaScript 函数式编程》
+ 源于Array 的原生的方法在<a herf="https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array"> MND </a>上的描述
