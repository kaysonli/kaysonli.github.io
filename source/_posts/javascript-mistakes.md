---
title: JavaScript 初学者容易犯的几个错误，你中招没？
date: 2020-01-16 18:09:30
tags: JavaScript
---

JavaScript 是对初学者比较友好的一门编程语言，基本上花个半小时看下语法就能写出能运行的代码。JavaScript 是动态脚本语言，对数据类型没有太多的限制，写起来非常灵活。但正因为如此，初学者如果不深入了解语言本身，往往会犯一些错误，从而导致一些很难发现的 bug。

抛开 JavaScript 语言设计层面的问题不说，毕竟它是 Brendan Eich 当年用短短十天时间设计出来的，有点缺陷也是在所难免。作为开发者，我们该怎样避免一些常见的低级错误呢？本文就列举几个常见错误，看看你有没有似曾相识。


### 混淆 undefined 和 null

JavaScript 中的`undefined` 和 `null` 都可用来表示*没有值*，但是二者之间有所区别。`undefined`字面意思是“未定义”，但它的含义其实已经超出了变量未定义的范畴：尝试读取对象不存在的属性、没有`return`语句的函数的返回值、声明后没有赋值的变量甚至显式赋值为`undefined`的变量等，它们的结果都是`undefined`。用`typeof`测试它的类型，是字符串 `'undefined'`。而 `null` 就比较纯粹了，变量只有设置为`null` 才有这个值。另外，`null` 是对象类型，即`typeof(null)`的值是字符串`'object'`。
<!-- more -->

需要注意的是，用`if`判断这两个值都是`false`，而且`null==undefined`是成立的，这一点初学者通常容易搞混。因此，尽量统一把“没有值”都设置为`undefined`，这样就省去了判断区分的麻烦。

返回 `undefined`的函数：
```
const f = () => {}

```

设置变量的值为 `undefined`：
```
x = undefined;

```

判断属性是否为 `undefined`：
```
typeof obj.prop === 'undefined'
obj.prop === undefined

```

判断变量是否为 `undefined`：
```
typeof x === 'undefined'

```

变量声明后没有赋值，自动就有了 `undefined`值。
 
如果一定要判断`null`，用全等判断：
```
obj.prop === null
x === null
```

使用 `typeof` 是无法判断 `null`的，因为它是对象类型。


### 混淆数字相加和字符串拼接

在 JavaScript 中，加号 `+`操作符既可用于数字相加，也可以用于字符串拼接。由于 JavaScript 是动态语言，操作符会自动将变量转成相同数据类型再运算。比如：
```
let x = 1 + 1; // 2

```
结果是 `2`，是我们期望的数字相加操作，因为两个值都是数字。

但是，如果是下面这种表达式：
```
let x = 1 + '1'; // “11”

```
结果是`'11'`，因为第一个数字会转换成字符串。这里的加号`+`运算符被用作字符串拼接，而不是数字相加。这里能直接看到表达式的值还算清楚，如果是由多个变量组成的表达式就很难判断类型了。

为了解决这个问题，我们可以把字符串都转成数字类型，再进行运算。例如：

```
let x = 1;  
let y = '2';  
let z = Number(x) + Number(y);

```
这样，运行结果就是`3`了。 `Number`函数接收任意类型的值，如果能转成数字就返回数字，否则返回`NaN`。还可以用 `new Number(...).valueOf()`函数：

```
let x = 1;  
let y = '2';  
let z = new Number(x).valueOf() + new Number(y).valueOf();

```

由于 `new Number(...)` 是实例化一个构造函数，返回的是一个对象，并不是数字类型。如果要得到原始的数字类型，需要用该对象的`valueOf`方法。其实还有个更简洁的方法：
```
let x = 1;  
let y = '2';  
let z = +x + +y;

```

变量前面的 `+` 作用是将它转换成数字，或者`NaN` ，跟`Number`函数的作用相同。


### return 语句换行问题

JavaScript 语法规定换行代表语句结束。例如：
```
const add = (a, b) => {  
  return  
  a + b;  
}
add(1,2); // undefined
```
本以为会返回 `3`，实际上是`undefined`。这是因为在`a + b`之前，函数已经执行了`return`。要解决这个问题，有两个做法：要么把表达式跟`return`放在一行，要么把表达式套一层括号。


```
const add = (a, b) => {  
  return a + b;  
}
// 或者
const add = (a, b) => {  
  return (  
    a + b  
  );  
}
```
加括号为什么可以换行呢？因为括号里的是表达式，不是语句。表达式可以拆成多行，如果很长的话。用箭头函数会更直观：

```
const add = (a, b) => a + b

```
箭头函数里的单行表达式自带`return`效果，当然也可以在表达式外面套一层括号：


```
const add = (a, b) => (a + b)

```
这个括号在返回对象字面量的箭头函数里有点用处，因为不加圆括号`()`的话，`{}`只是函数体的开始和结束标记，要返回对象字面量，还要显式`return {...}`。

如果某行代码中的语句不完整，JavaScript 解析器会将下一行的语句合并一起解析。比如：
```
const power = (a) => {  
  const  
    power = 10;  
  return a ** 10;  
}
// 等同于：
const power = (a) => {  
  const  power = 10;  
  return a ** 10;  
}
```
但是对于完整的语句，比如`return`，就不会合并多行。
### 用 return 跳出 forEach 循环

JavaScript 数组有个 `forEach` 方法，用于对数组元素进行循环操作。初学者很容易联想到 `for`循环的`break`或`continue`关键字，用来中止循环。但是对不起，`forEach`没有这两个关键字。那用`return`行不行？可以用，但它的作用就是提前返回函数，跟`continue`的效果类似，用于结束本次循环。要跳出整个循环，`return`做不到。
```
const nums = [1, 2, 3, 4, 5, 6];
let firstEven;
nums.forEach(n => {
  if (n % 2 ===0 ) {
    firstEven = n;
    return n;
  }
});
console.log(firstEven); // 6
```
代码本意是想找出第一个偶数，找到就退出循环。但实际并没有退出循环，因此最终结果是最后一个偶数。
有解决办法吗？这种情况可以用`for`循环，或者用数组`filter`、`find`之类的方法。

### 总结
虽然 JavaScript 很容易上手，但稍不注意还是比较容易犯错。本文简单介绍了几种容易犯的错，希望对你有所帮助。
