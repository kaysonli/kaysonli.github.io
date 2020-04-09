---
title: 面试官：JavaScript 原始数据类型 Symbol 有什么用？
date: 2020-04-05 14:09:59
tags: JavaScript
---
以前提到 JavaScript 原始数据类型时，我们知道有`Number`，`String`，`Null`，`Boolean`，`Undefined`这几种。ES6 引入了新的基本数据类型`Symbol`和`BigInt`。今天我们就来了解下`Symbol`类型。`Symbol`类型是为了解决属性名冲突的问题，顺带还具备模拟私有属性的功能。
<!-- more -->
## 简介

创建`symbol`变量最简单的方法是用`Symbol()`函数。`sysmbol`变量有两点比较特别：
1.  它可以作为对象属性名。只有字符串和 `symbol` 类型才能用作对象属性名。
2.  没有两个`symbol` 的值是相等的。

```
const symbol1 = Symbol();
const symbol2 = Symbol();

symbol1 === symbol2; // false

const obj = {};
obj[symbol1] = 'Hello';
obj[symbol2] = 'World';

obj[symbol1]; // 'Hello'
obj[symbol2]; // 'World'
```

尽管调用`Symbol()` 让它看起来像是对象，实际上`symbol`是 JavaScript 原始数据类型。把`Symbol`当作构造函数来用 `new`会报错。
```
const symbol1 = Symbol();

typeof symbol1; // 'symbol'
symbol1 instanceof Object; // false

// Throws "TypeError: Symbol is not a constructor"
new Symbol();
```

## 描述信息

 `Symbol()`函数只有一个参数，字符串`description`。这个字符串参数的唯一作用是辅助调试，也就是它的`toString()`值。但是请注意，两个具有相同`description`的`symbol`也是**不相等**的。
```
const symbol1 = Symbol('my symbol');
const symbol2 = Symbol('my symbol');

symbol1 === symbol2; // false
console.log(symbol1); // 'Symbol(my symbol)'
```

有一个全局的`symbol`注册中心，用`Symbol.for()`创建的`symbol`会添加到这个注册中心，并用它的 `description`作为索引键。也就是说，如果你用`Symbol.for()`创建带有相同 `description`的两个 `symbol`，它们就是相等的。
```
const symbol1 = Symbol.for('test');
const symbol2 = Symbol.for('test');

symbol1 === symbol2; // true
console.log(symbol1); // 'Symbol(test)'
```

通常来说，除非你有非常好的理由，否则不应该使用全局注册中心，因为这会造成命名冲突。
## 命名冲突

JavaScript 内置了一个 `symbol` ，那就是 ES6 中的[`Symbol.iterator` ](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol/iterator) 。拥有`Symbol.iterator`函数的对象被称为[*可迭代对象*](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols#The_iterable_protocol)，就是说你可以在对象上使用[`for/of` 循环](http://thecodebarbarian.com/for-vs-for-each-vs-for-in-vs-for-of-in-javascript)。
```
const fibonacci = {
  [Symbol.iterator]: function*() {
    let a = 1;
    let b = 1;
    let temp;

    yield b;

    while (true) {
      temp = a;
      a = a + b;
      b = temp;
      yield b;
    }
  }
};

// Prints every Fibonacci number less than 100
for (const x of fibonacci) {
  if (x >= 100) {
    break;
  }
  console.log(x);
}
```

为什么这里要用`Symbol.iterator` 而不是字符串？假设不用`Symbol.iterator` ，可迭代对象需要有一个字符串属性名`'iterator'`，就像下面这个可迭代对象的类：
```
class MyClass {
  constructor(obj) {
    Object.assign(this, obj);
  }

  iterator() {
    const keys = Object.keys(this);
    let i = 0;
    return (function*() {
      if (i >= keys.length) {
        return;
      }
      yield keys[i++];
    })();
  }
}
```

 `MyClass` 的实例是可迭代对象，可以遍历对象上面的属性。但是上面的类有个潜在的缺陷，假设有个恶意用户给 `MyClass` 构造函数传了一个带有`iterator`属性的对象：
```
const obj = new MyClass({ iterator: 'not a function' });
```

这样你在`obj`上使用`for/of`的话，JavaScript 会抛出`TypeError: obj is not iterable`异常。可以看出，传入对象的 `iterator`函数覆盖了类的 `iterator`属性。这有点类似[原型污染](http://thecodebarbarian.com/mongoose-prototype-pollution-vulnerability-disclosure)的安全问题，无脑复制用户数据会对一些特殊属性，比如`__proto__`和`constructor`带来问题。

这里的核心在于，`symbol`让对象的内部数据和用户数据井水不犯河水。由于`sysmbol`无法在 JSON 里表示，因此不用担心给 Express API 传入带有不合适的`Symbol.iterator`属性的数据。另外，对于那种混合了内置函数和用户数据的对象，比如 [Mongoose model](https://mongoosejs.com/docs/models.html)，你可以用`symbol`来确保用户数据不会跟内置属性冲突。
## 私有属性

由于任何两个`symbol`都是不相等的，在 JavaScript 里可以很方便地用来模拟私有属性。`symbol`不会出现在 `Object.keys()`的结果中，因此除非你明确地`export` 一个`symbol`，或者用 [`Object.getOwnPropertySymbols()` ](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/getOwnPropertySymbols)函数获取，否则其他代码无法访问这个属性。
```
function getObj() {
  const symbol = Symbol('test');
  const obj = {};
  obj[symbol] = 'test';
  return obj;
}

const obj = getObj();

Object.keys(obj); // []

// 除非有这个 symbol 的引用，否则无法访问该属性
obj[Symbol('test')]; // undefined

// 用 getOwnPropertySymbols() 依然可以拿到 symbol 的引用
const [symbol] = Object.getOwnPropertySymbols(obj);
obj[symbol]; // 'test'
```

还有一个原因是`symbol`不会出现在`JSON.stringify()`的结果里，确切地说是`JSON.stringify()`会忽略`symbol`属性名和属性值：
```
const symbol = Symbol('test');
const obj = { [symbol]: 'test', test: symbol };

JSON.stringify(obj); // "{}"
```

## 总结

用 `Symbol` 表示对象内部状态，可以很好地隔离用户数据和程序状态。有了它，我们就不再需要某些命名约定了，比如内部属性用`'$'`开头。下次碰到需要定义私有属性的时候，试试`Symbol`类型吧！
