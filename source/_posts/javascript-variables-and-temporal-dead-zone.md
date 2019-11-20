---
title: 不可不知的JavaScript 变量 Temporal Dead Zone
date: 2019-10-09 20:04:54
tags: JavaScript
---

先问个简单的问题，下面哪段代码会报错？

1. 先创建类实例，然后定义这个类：

```
new Car('red'); // Does it work?

class Car {
  constructor(color) {
    this.color = color;
  }
}
```
2. 先调用函数，再定义这个函数：

```
greet('World'); // Does it work?

function greet(who) {
  return `Hello, ${who}!`;
}
```
正确答案是：第一段代码会抛出`ReferenceError`异常，第二段代码可以正常运行。
<!-- more -->

如果你答错了，或者只是猜对了但不知道背后的原理，那么你应该了解下什么是*Temporal Dead Zone* (TDZ)

TDZ 负责管理`let`, `const`, 和`class` 语句的可用性。这就需要知道JavaScript变量是如何工作的。

##  什么是 *Temporal Dead Zone*

先看一个简单的 `const`变量声明。如果先声明变量，然后访问它，那么一切正常：

```
const white = '#FFFFFF';

white; // => '#FFFFFF'
```
现在试着在声明之前访问`white` 变量：

```
white; // 抛出`ReferenceError`异常
const white = '#FFFFFF';

white;
```
这段代码里，在`const white = '#FFFFFF'`语句出现之前，变量 `white` 处于Temporal Dead Zone。

在TDZ内访问`white`，JavaScript会抛出异常`ReferenceError: Cannot access 'white' before initialization`.

![image.png](/uploads/1618526-97f5b9c0fd68e02f.webp)


> *Temporal Dead Zone* 语义禁止在未声明之前访问变量。它强调了这样的规则：*不要在未声明前使用任何东西*

##  TDZ影响到的语句

我们来看看哪些语句会受 TDZ 的影响。

###  *const* 变量

前面已经看到了，`const` 变量在声明和初始化前位于 TDZ：

```
// 跪了！
pi; // throws `ReferenceError`
const pi = 3.14;
```
必须要在声明后使用`const` 变量：

```
const pi = 3.14;

// 稳了！
pi; // => 3.14
```

### 2.2 *let* 变量

`let` 变量声明同样受 TDZ 影响：
```
// Does not work!
count; // throws `ReferenceError`
let count;

count = 10;
```
同样，要在声明之后才能使用：

```
let count;

// 稳了！
count; // => undefined
count = 10;

// Works!
count; // => 10
```

### *class* 语句
前面说了，不能在定义`class`之前使用它：

```
// 跪了!
const myNissan = new Car('red'); // throws `ReferenceError`
class Car {
  constructor(color) {
    this.color = color;
  }
}
```
需要先定义再使用：

```
class Car {
  constructor(color) {
    this.color = color;
  }
}

// 稳了！
const myNissan = new Car('red');myNissan.color; // => 'red'
```

###  *constructor()* 里的*super()* 

如果一个类继承自另一个类，构造函数里的 `this` 在调用`super()` 之前位于TDZ：
```
class MuscleCar extends Car {
  constructor(color, power) {
    this.power = power;   
    super(color); 
 }
}

// 跪了！
const myCar = new MuscleCar('blue', '300HP'); // `ReferenceError`
```

在 `constructor()`内部， `this` 在 `super()` 调用之前不可用。要改成这样：
```
class MuscleCar extends Car {
  constructor(color, power) {
    super(color);    
    this.power = power;  
  }
}

// 稳了！
const myCar = new MuscleCar('blue', '300HP');
myCar.power; // => '300HP'
```

###  函数默认参数

默认参数处于一个中间作用域，不同于全局作用域和函数作用域。默认参数同样会受到TDZ的限制：

```
const a = 2;
function square(a = a) {
  return a * a;
}
// 跪了！
square(); // throws `ReferenceError`
```
参数`a`在表达式`a = a`的右边被使用，同时未声明。这会产生引用错误。
要确保在声明和初始化后再使用默认参数，比如这样：

```
const init = 2;
function square(a = init) {
  return a * a;
}
// 稳了！
square(); // => 4
```

##  *var*, *function*, *import* 语句
跟前面提到的语句相反，`var` 和`function` 并不受 TDZ 影响，因为会被作用域提升。

如果你在声明之前访问变量`var` ，只是得到 `undefined`值：

```
// 能运行，但不建议这么做
value; // => undefined
var value;
```
而函数不管在哪定义，都可以直接使用：

```
// 稳了！
greet('World'); // => 'Hello, World!'
function greet(who) {
  return `Hello, ${who}!`;
}

// 稳了！
greet('Earth'); // => 'Hello, Earth!'
```
有意思的是， `import` 的模块也能被提升：

```
// 也很稳！
myFunction();
import { myFunction } from './myModule';
```
尽管如此，还是建议在 JS 文件最顶部加载模块依赖。

##  TDZ 的*typeof* 行为表现

`typeof` 操作符可以用来判断当前作用域的某个变量是否已定义。
例如，变量`notDefined`没有定义。对它使用`typeof` 操作符并不会抛错：

```
typeof notDefined; // => 'undefined'
```
但是对Temporal Dead Zone里的变量使用`typeof` 操作符，会有不同的表现：

```
typeof variable; // throws `ReferenceError`
let variable;
```
这是因为，通过静态分析（用眼睛看）可以看出 `variable` 并不是未定义。

##  当前作用域的TDZ 行为
Temporal Dead Zone 只会影响到声明语句所在的作用域内的变量。

![image.png](/uploads/1618526-62f6b54573047a97.webp)

我们来看个例子：

```
function doSomething(someVal) {
  // 函数作用域
  typeof variable; // => undefined 
  if (someVal) {
    // 内部块级作用域
    typeof variable; // 抛出`ReferenceError`   
    let variable;
  }
}
doSomething(true);
```

这里有两个作用域：

1.  函数作用域
2.  `let` 变量所在的块级作用域
函数作用域内，`typeof variable` 直接返回`undefined`。这里的`let ` 变量语句不归TDZ管。
在块级作用域内， `typeof variable` 使用了未声明的变量，于是抛出异常 `ReferenceError: Cannot access 'variable' before initialization`。TDZ只在该作用域内起作用。.

##  结论

TDZ是一个重要的概念，它影响着`const`、`let` 和 `class` 语句的可用性。它不允许在声明之前使用变量。
相反， `var`变量沿用了旧有行为，可以在声明之前使用变量。但是应该避免这么做。

我觉得TDZ 是个好东西，它从语言规范层面指导了最佳编码实践。

