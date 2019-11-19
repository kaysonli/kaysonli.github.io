---
title: 最后一次学习 JavaScript 函数式编程
date: 2019-11-06 18:09:51
tags:
---

作为程序员，你可能希望编写优雅、可维护、可伸缩、可预测的代码。函数式编程(FP)的原则可以极大地帮助实现这些目标。

函数式编程是一种范式或风格，它重视不变性、“一等公民”(first-class)函数、引用透明性和纯函数。如果这些词你都听不懂，别担心！我们将在本文中分解所有这些术语。

函数式编程是从 lambda 演算演化而来的，lambda 演算是一个围绕函数抽象和泛化构建的数学系统。因此，许多函数式编程语言看起来都非常数学化。不过，好消息是：你不需要使用函数式编程语言来为代码引入函数式编程原则。在这篇文章中，我们将使用 JavaScript，它的很多特性使它能够符合函数式编程的要求，同时又不拘泥于这种范式。

## 函数式编程的核心原则

既然我们已经讨论了什么是函数式编程，现在让我们说说FP背后的核心原则。

### 纯函数

我喜欢把函数看作机器——它们接受一个输入或参数，然后输出一些东西，即返回值。纯函数没有与函数输出无关的“副作用”和操作。一些潜在的副作用可能是打印一个值或用`console.log`输出日志，或者在函数外部操作变量。

下面是一个非纯函数的例子：
```
let number = 2;

function squareNumber() {
  number = number * number; // 非纯操作：在函数外部操作变量
  console.log(number); // 非纯操作：打印日志
  return number;
}

squareNumber();

```
下面的函数是纯函数。它接受一个输入并产生一个输出。
```
// 纯函数
function squareNumber(number) {
  return number * number;
}

squareNumber(2);

```

纯函数在函数外部独立于状态运行，所以它们不应该依赖于全局状态，也不应该依赖于自身之外的变量。在第一个示例中，我们使用在函数外部创建的 `number` 变量，并在内部设置它。这违反了纯函数原则。如果你严重依赖于不断变化的全局变量，那么你的代码将是不可预测的，并且很难跟踪。要弄清楚 bug 发生在哪里以及为什么值会发生变化会更加困难。相反，只使用函数的输入、输出和局部变量可以简化调试。

此外，函数应该遵循*引用透明*原则，也就是给定一个输入，它们的输出总是相同的。在上面的函数中，如果我将`2`传递给函数，它将始终返回`4`。对于API调用或生成随机数，情况就不一样了，举两个例子。给定相同的输入，输出可能返回，也可能不返回。
```
// 非*引用透明*
Math.random();
// 0.1406399143589343
Math.random();
// 0.26768924082159495ß

```

### 不变性

函数式编程也优先考虑“不变性”，即不直接修改数据。不变性导致可预测性——你知道数据的值，并且它们不会改变。它使代码简单、可测试，并且能够在分布式和多线程系统上运行。

当我们处理数据结构时，不变性经常起作用。JavaScript 中的许多数组方法直接修改数组。例如，`.pop()`直接从数组末尾删除一个项目，`.splice()`允许你截取数组的一部分。相反，在函数式范型中，我们将复制数组，并在此过程中删除不要的元素。
```
// 直接修改 myArr
const myArr = [1, 2, 3];
myArr.pop();
// [1, 2]

```

```
// 复制数组，不包含最后一个元素，然后保存到一个变量
let myArr = [1, 2, 3];
let myNewArr = myArr.slice(0, 2);
// [1, 2]
console.log(myArr);

```

### 一等函数

在函数式编程中，我们的函数是第一等的，这意味着我们可以像使用其他值一样使用它们。我们可以创建函数数组，将它们作为参数传递给其他函数，并将它们存储在变量中。
```
let myFunctionArr = [() => 1 + 2, () => console.log("hi"), x => 3 * x];
myFunctionArr[2](2); // 6

const myFunction = anotherFunction => anotherFunction(20);
const secondFunction = x => x * 10;
myFunction(secondFunction); // 200

```

### 高阶函数

[高阶函数](https://www.sitepoint.com/higher-order-functions-javascript/)是这样的函数：它们要么接受一个函数作为一个或多个参数，要么返回一个函数。JavaScript 内置了许多第一类高阶函数，比如`map`、 `reduce`和`filter`，我们可以使用它们来操作数组。

[`filter`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/filter) 用于从一个旧数组返回一个新数组，该数组只包含符合我们提供的条件的值。
```
const myArr = [1, 2, 3, 4, 5];

const evens = myArr.filter(x => x % 2 === 0); // [2, 4]

```

[`map`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map) 用于迭代数组中的项，根据提供的逻辑修改每个项。在下面的示例中，通过传递一个将值乘以2的映射函数，将数组中的每个项加倍。
```
const myArr = [1, 2, 3, 4, 5];

const doubled = myArr.map(i => i * 2); // [2, 4, 6, 8, 10]

```

[`reduce`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce) 允许我们基于一个输入的数组输出单个值——它通常用于对一个数组、扁平数组或以某种方式对值进行求和。
```
const myArr = [1, 2, 3, 4, 5];

const sum = myArr.reduce((i, runningSum) => i + runningSum); // 15

```
你也可以自己实现其中的任何一个！例如，你可以像这样写一个过滤器函数:
```
const filter = (arr, condition) => {
  const filteredArr = [];

  for (let i = 0; i < arr.length; i++) {
    if (condition(arr[i])) {
      filteredArr.push(arr[i]);
    }
  }

  return filteredArr;
};

```

第二类高阶函数，即返回其他函数的函数，也是一种相对常见的模式。例如:
```
const createGreeting = greeting => person => `${greeting} ${person}`

const sayHi = createGreeting("Hi")
console.log(sayHi("Ali")) // "Hi Ali"

const sayHello = createGreeting("Hello")
console.log(sayHi("Ali")) // "Hello Ali"

```

[柯里化](https://www.sitepoint.com/currying-in-functional-javascript/) 是一种相关的技术，有兴趣可以了解下。

### 复合函数

复合函数是将多个简单函数组合在一起以创建更复杂的函数。因此，你可以有一个 `averageArray`函数，它结合了`average`函数和`sum` 函数，后者对一个数组求和。单个函数很小，可以用于其他目的，它们组合在一起可以执行更完整的任务。
```
const sum = arr => arr.reduce((i, runningSum) => i + runningSum);
const average = (sum, count) => sum / count;
const averageArr = arr => average(sum(arr), arr.length);

```

## 函数式编程的好处

函数式编程造就模块化代码，你写的一些小函数可以反复使用。了解每个函数的特定功能意味着，准确定位错误和编写测试应该比较简单直接，特别是因为函数输出应该是可预测的。

此外，如果你试图使用多核，你可以跨核分配函数调用，这样可以提高计算效率。

## 如何使用函数式编程？

要吸收这些理念，你不必完全转向函数式编程。你甚至可以将这些理念跟面向对象编程结合使用，而后者经常被认为是前者的对手。

举例来说，React 吸收了很多[函数式原则](https://medium.com/@andrea.chiarelli/the-functional-side-of-react-229bdb26d9a6)，比如不可变状态，但是多年来主要是用 class 语法。几乎可以在任何编程语言中实现函数式编程——你不需要写 Clojure 或 Haskell，除非真的需要。

函数式编程原则可以在你的代码中产生积极的结果，即使你不是一个纯粹主义者。

[原文](https://www.sitepoint.com/what-is-functional-programming/)
