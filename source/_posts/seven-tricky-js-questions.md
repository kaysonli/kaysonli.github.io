---
title: 7道简单的 JavaScript 面试题，三个月没招到一个人
date: 2019-10-23 16:34:48
tags: JavaScript,面试题
---


如果你符合JavaScript高级开发人员的资格，在编码面试中很有可能会被问到一些刁钻的问题。

我知道这不公平。一些不知名的人把你放在角落上下打量，似乎想看你是什么做成的。这是一次不愉快的经历。
![image.png](/uploads/1618526-2b825c828e81ddec.webp)


你能做什么?

遵循这个建议：“熟能生巧”。通过投入足够的时间，更好地定期深入了解JavaScript，将改善你的编码，并顺便提高你的面试技巧。

在这篇文章中，你会发现7个乍一看很简单，但实际上很棘手的JavaScript面试题。

虽然一开始这些问题看起来是随机的，但是它们试图与JavaScript的重要概念挂钩。所以你最好在下次面试前练习一下!
<!-- more -->
## 1\. 意外的全局变量

#### 问题

在以下代码中，`typeof a`和`typeof b`的值分别是什么：
```
function foo() {
  let a = b = 0;
  a++;
  return a;
}

foo();
typeof a; // => ???typeof b; // => ???
```

#### 答案

让我们仔细看看第2行：`let a = b = 0`。这个语句确实声明了一个局部变量`a`。但是，它确实声明了一个*全局*变量`b`。

在`foo()`作用域或全局作用域中都没有声明变量 `b` ”。因此JavaScript将表达式 `b = 0` 解释为 `window.b = 0`。
![image.png](/uploads/1618526-fd971166d5f7a3e3.webp)

`b`是一个偶然创建的全局变量。

在浏览器中，上述代码片段相当于:
```
function foo() {
  let a;  window.b = 0;  a = window.b;  a++;
  return a;
}

foo();
typeof a;        // => 'undefined'
typeof window.b; // => 'number'
```

`typeof a`是 `'undefined'`。变量`a`仅在 `foo()`范围内声明，在外部范围内不可用。

`typeof b`等于`'number'`。`b`是一个值为 `0`的全局变量。
## 2\. 数组 length 属性

#### 问题

`clothes[0]` 的值是什么：

```
const clothes = ['jacket', 't-shirt'];
clothes.length = 0;

clothes[0]; // => ???
```

#### 答案

数组对象的 `length` 属性有一个 [特殊的行为](http://www.ecma-international.org/ecma-262/6.0/#sec-properties-of-array-instances-length):

> 减少`length`属性的值有一个副作用，就是会删除索引位于新旧长度值之间的元素。

因为 `length`的这种行为，当JavaScript执行`clothes.length = 0` 时，数组 `clothes` 中的所有项都被删除了。

`clothes[0]` 是`undefined`，因为 `clothes` 数组被清空了。

## 3\. 鹰眼测试

#### 问题

`numbers` 数组的内容是什么：

```
const length = 4;
const numbers = [];
for (var i = 0; i < length; i++);{
  numbers.push(i + 1);
}

numbers; // => ???
```

#### 答案

让我们仔细看看出现在左花括号`{`前面的分号`;` ：

![image.png](/uploads/1618526-51c6253c69cd9d04.webp)

很容易忽略这个分号，而它创建了一个*空语句*。空语句是不做任何事情的语句。

`for()` 在空语句（什么也不做）上循环了 4 次，忽略了实际上往数组里添加元素的代码块`{ numbers.push(i + 1); }`。

上述代码等同于：
```
const length = 4;
const numbers = [];
var i;
for (i = 0; i < length; i++) {
  // does nothing
}
{ 
  // a simple block
  numbers.push(i + 1);
}

numbers; // => [5]
```

`for()`递增变量`i`直到`4`。然后JavaScript 进入代码块 `{ numbers.push(i + 1); }`，将`4 + 1` 添加 到`numbers`数组中。

这样 `numbers` 就是 `[5]`.

#### 这个问题背后我的故事

> *很久以前，当我面试我的第一份工作时，我被问到这个问题。*
*面试时，我被要求在1小时内回答20个编码问题。空语句问题位列其中。*
*当我在匆忙中解决这个问题时，我没有看到逗号`;` 就在花括号 `{` 的前面。所以我答错成 [1,2,3,4]*
*我对这些不公平的伎俩有点失望。我问面试官这样做的原因是什么?面试官回答：*
*“因为我们需要注重细节的人。”*
*幸运的是，我最终没有在那家公司上班。*

你怎么看这个问题？

## 4\. 自动插入分号

#### 问题

`arrayFromValue()` 返回什么值？

```
function arrayFromValue(item) {
  return
    [items];
}

arrayFromValue(10); // => ???
```

#### 答案

很容易忽略`return`关键字和`[items]`表达式之间的换行。

换行使JavaScript自动在`return`和`[items]`表达式之间插入一个分号。

这里有一段等价的代码，它在`return`后插入分号：
```
function arrayFromValue(item) {
  return;  [items];
}

arrayFromValue(10); // => undefined
```

函数中的 `return;` 导致它返回 `undefined`。

因此 `arrayFromValue(10)` 的值是 `undefined`。

查看 [这篇文章](https://dmitripavlutin.com/7-tips-to-handle-undefined-in-javascript/#24-function-return-value) 阅读更多关于自动插入分号的内容。

## 5\. 经典问题：坑爹的闭包

#### 问题

以下脚本将会在控制台输出什么：
```
let i;
for (i = 0; i < 3; i++) {
  const log = () => {
    console.log(i);  }
  setTimeout(log, 100);
}
```

#### 答案

如果你之前没有听说过这个棘手的问题，你的答案很可能是`0`, `1` 和 `2`，这是不正确的。当我第一次尝试解答它时，我的答案也是这样！

执行这个代码段包含两个步骤。
**步骤 1**

1.  `for()`迭代3次。在每次迭代过程中，都会创建一个新的函数`log()`，它捕获变量 `i`。然后`setTimout()`执行`log()`。
2. 当`for()`循环完成时，`i`变量的值为`3`。

`log()`是一个捕获变量 `i` 的闭包，它在`for()`循环的外部作用域定义。重要的是要理解闭包从词法上捕获了*变量*`i` 。
**步骤 2**

第2步在 100 毫秒后发生：

`setTimeout()`调用了队列中的3个`log()` 回调。`log()` 读取变量 `i`的*当前值*，即`3`，并记录到控制台`3`。

这就是为什么控制台输出`3`, `3` 和`3`。

*你知道怎样让代码输出 `0`, `1`, 和 `2`吗？请在评论里写出你的方案。*

## 6\. 浮点数问题

#### 问题

等号判断的结果是什么？
```
0.1 + 0.2 === 0.3 // => ???
```

#### 答案

首先，我们看看`0.1 + 0.2` 的值：
```
0.1 + 0.2; // => 0.30000000000000004
```

 `0.1` 和 `0.2` 的和 *不完全等于*  `0.3`，而是略大于 `0.3`。

由于浮点数在二进制中的编码机制，像浮点数的加法这样的操作会受到舍入误差的影响。

简单地说，直接比较浮点数是不精确的。

因此 `0.1 + 0.2 === 0.3` 是 `false`.

查看 [0.30000000000000004.com](https://0.30000000000000004.com/) 获取更多信息。

## 7\. 变量提升

#### 问题

如果在声明之前访问`myVar`和`myConst`会发生什么?
```
myVar;   // => ???myConst; // => ???
var myVar = 'value';
const myConst = 3.14;
```

#### 答案

变量提升和暂时性死区是影响JavaScript变量生命周期的两个重要概念。
![image.png](/uploads/1618526-52cfe23f40515c52.webp)

在声明之前访问 `myVar` 结果为`undefined`。一个被提升的`var`变量，在它的初始化之前，有一个 `undefined`的值。

但是，在声明之前访问`myConst`会抛出 `ReferenceError`。在声明行`const myConst = 3.14`之前，`const` 变量处于暂时死区。

查看指南 [JavaScript 变量提升详解](https://dmitripavlutin.com/javascript-hoisting-in-details/) ，彻底掌握变量提升的概念。

## 8\. 思考

你可以认为有些问题对面试毫无用处。我也有同样的感觉，尤其是关于[鹰眼测试](https://dmitripavlutin.com/simple-but-tricky-javascript-interview-questions/#3-eagle-eye-test)。尽管如此，还是有人会问这些问题。

不管怎样，这些问题中的大多数都可以测试你是否熟悉JavaScript。如果你在阅读这篇文章的时候很难答出其中一些问题，这就是提示你接下来要学习什么了!

*在面试中问一些刁钻的问题公平吗？说说你的看法。*

[原文](https://dmitripavlutin.com/simple-but-tricky-javascript-interview-questions/)
