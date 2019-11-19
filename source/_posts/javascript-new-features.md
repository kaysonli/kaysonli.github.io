---
title: 七种武器：JavaScript 新特性闪亮登场
date: 2019-10-18 18:12:40
tags: JavaScript
---


JavaScript(或ECMA Script)是一门不断发展的语言，有许多关于如何前进的建议和想法。TC39(技术委员会39)是负责定义JS标准和特性的委员会，今年他们非常活跃。以下是目前处于“Stage 3阶段”的一些提案摘要，这是“完成”之前的最后一个阶段。这意味着这些特性将很快在浏览器和其他引擎中实现。事实上，其中一些现在就有了。
<!-- more -->
## 1. 私有字段`#`

*Chrome 和 NodeJS 12 已支持*

是的，你没看错。JS终于在类中支持私有字段了。不再有 `this._doPrivateStuff()`、定义闭包来存储私有值或者使用`WeakMap`来间接实现私有属性。

语法是这样的：
```
// 私有字段必须以 '#' 开头
// and they can't be accessed outside the class block

class Counter {
  #x = 0;

  #increment() {
    this.#x++;
  }

  onClick() {
    this.#increment();
  }

}

const c = new Counter();
c.onClick(); // 正常
c.#increment(); // 出错

```

提案： [https://github.com/tc39/proposal-class-fields](https://github.com/tc39/proposal-class-fields)

## 2\. 可选链式调用 `?.`

以往需要访问嵌套在对象内部好几层的属性时，会得到臭名昭著的错误`Cannot read property 'stop' of undefined`。然后你就要改变代码来处理属性链中每一个可能的`undefined`对象，比如:
```
const stop = please && please.make && please.make.it && please.make.it.stop;

// 或者使用 'object-path' 这样的库
const stop = objectPath.get(please, "make.it.stop");

```
有了可选链式调用 ，你只要这样写就可以做同样的事情:
```
const stop = please?.make?.it?.stop;

```

提案： [https://github.com/tc39/proposal-optional-chaining](https://github.com/tc39/proposal-optional-chaining)

## 3\. 空合并操作符 `??`

变量的可选值可能没有，如果没有则使用默认值。这种情况很常见：
```
const duration = input.duration || 500;

```

使用`||`的问题是，它会覆盖所有的假值，如(`0`, `''`, `false`)，这些值可能是在某些情况下有效的输入。

输入空合并操作符，它只覆盖`undefined`或`null`。
```
const duration = input.duration ?? 500;

```

提案： [https://github.com/tc39/proposal-nullish-coalescing](https://github.com/tc39/proposal-nullish-coalescing)

## 4\. BigInt `1n`

*Chrome 和 NodeJS 12 已支持*

JS在数学方面一直很糟糕的一个原因是，我们无法可靠地存储大于`2 ^ 53`的数字，这使得处理相当大的数字非常困难。幸运的是，`BigInt`是解决这个特定问题的提案。

```
// 可以通过附加'n'到一个数字字面量来定义BitInt
const theBiggestInt = 9007199254740991n;

// 使用构造器
const alsoHuge = BigInt(9007199254740991);

// 或则字符串形式
const hugeButString = BigInt('9007199254740991');

```

你也可以在`BigInt`上使用与普通数字相同的运算符，例如 `+`, `-`, `/`, `*`, `%`等等。不过有一个问题，在大多数操作中，不能将 `BigInt`与`Number`混合使用。比较`Number`和 `BigInt`是可以的，但是不能把它们相加。
```
1n < 2 
// true

1n + 2
// 🤷‍♀️ Uncaught TypeError: Cannot mix BigInt and other types, use explicit conversions

```

提案： [https://github.com/tc39/proposal-bigint](https://github.com/tc39/proposal-bigint)
参考阅读：[一文搞懂JavaScript 新数据类型BigInt](https://www.jianshu.com/p/46b4341bb96c)

## 5. `static` 字段

*Chrome & NodeJS 12 已支持*

这个很简单。它允许类拥有静态字段，类似于大多数OOP语言。静态字段可以用来代替枚举，也可以用于私有字段。
```
class Colors {
  // public static 字段
  static red = '#ff0000';
  static green = '#00ff00';

  // private static 字段
  static #secretColor = '#f0f0f0';

}

font.color = Colors.red;

font.color = Colors.#secretColor; // Error

```

提案： [https://github.com/tc39/proposal-static-class-features](https://github.com/tc39/proposal-static-class-features)

## 6\. 顶层 `await`

*Chrome 已支持*

允许您在代码的顶层作用域使用await。这对于在浏览器控制台中调试异步内容（如 `fetch`）非常有用，而不需要在异步函数中包装它。
[![using await in browser console](https://upload-images.jianshu.io/upload_images/1618526-b9d2299a6db84aef.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)](https://res.cloudinary.com/practicaldev/image/fetch/s--wCIIk7Oa--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://thepracticaldev.s3.amazonaws.com/i/y5ur91fgud4pu7hh5ypv.png) 

另一个杀手级用例是，它可以在用异步方式初始化的ES模块的顶层使用(比如建立数据库连接)。当这样一个异步模块被导入时，模块系统将等待它解析，然后再执行依赖它的模块。这将使处理异步初始化比目前通过返回一个初始化promise并等待它解决来得更容易。模块将不知道它的依赖是否异步。


```
// db.mjs
export const connection = await createConnection();

```

```
// server.mjs
import { connection } from './db.mjs';

server.start();

```

在本例中，`server.mjs` 中不执行任何操作，直到在`db.mjs`中完成连接。
提案：[https://github.com/tc39/proposal-top-level-await](https://github.com/tc39/proposal-top-level-await)

参考阅读：[顶层 await 释疑](https://www.jianshu.com/p/33b80cf4514b)


## 7. `WeakRef`

* Chrome 和 NodeJS 12 已支持*

对象的弱引用是不足以对象处于活动状态的引用。当我们使用(`const`, `let`, `var`)创建一个变量时，垃圾收集器(GC)将永远不会从内存中删除该变量，只要它的引用仍然是可访问的。这些都是强引用。然而，弱引用引用的对象如果没有强引用，则GC可以在任何时候删除它。`WeakRef`实例有一个方法`deref`，它返回引用的原始对象，如果原始对象被回收，则返回`undefined` 。

这对于缓存廉价对象可能很有用，因为你不想将所有对象都永远存储在内存中。
```

const cache = new Map();

const setValue =  (key, obj) => {
  cache.set(key, new WeakRef(obj));
};

const getValue = (key) => {
  const ref = cache.get(key);
  if (ref) {
    return ref.deref();
  }
};

const fibonacciCached = (number) => {
  const cached = getValue(number);
  if (cached) return cached;
  const sum = calculateFibonacci(number);
  setValue(number, sum);
  return sum;
};

```
对于缓存远程数据来说，这可能不是一个好主意，因为远程数据可能会不可预测地从内存中删除。在这种情况下，最好使用LRU之类的缓存。
提案：[https://github.com/tc39/proposal-weakrefs](https://github.com/tc39/proposal-weakrefs)

* * *
以上。希望你和我一样对使用这些很酷的新功能感到兴奋。要了解更多关于这些提案和其他我没有提到的细节，请关注[github上的TC39提案](https://github.com/tc39提案)

原文：[7 Exciting New JavaScript Features You Need to Know](https://dev.to/gafi/7-new-exciting-javascript-features-you-need-to-know-1fkh)
