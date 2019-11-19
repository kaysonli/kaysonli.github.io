---
title: 一文搞懂JavaScript 新数据类型BigInt
date: 2019-10-12 20:04:54
tags: JavaScript
---


`BigInt`数据类型是为了让JavaScript程序能表示超出`Number` 类型支持的数值范围。在对大整数进行数学运算时，以任意精度表示整数的能力尤为重要。有了`BigInt`，整数溢出将不再是一个问题。

此外，你可以安全地使用高精度时间戳、大整数 ID 等，而不必使用任何变通方法。`BigInt`目前处于 stage 3 提案阶段。一旦加入到规范中，它将成为JavaScript中的第二种数字数据类型，这将使支持的数据类型总数达到8个:
*   Boolean
*   Null
*   Undefined
*   Number
*   BigInt
*   String
*   Symbol
*   Object

在本文中，我们将仔细研究`BigInt`，并了解它如何帮助克服JavaScript中`Number`类型的限制。
### 问题

对于来自其他语言的程序员来说，JavaScript中缺乏显式整数类型常常令人困惑。许多编程语言支持多种数字类型，如float、double、integer和bignum，但JavaScript不是这样。在JavaScript中，所有数字都以[双精度64位浮点格式](http://en.wikipedia.org/wiki/Double_precision_floating-point_format)表示，这是由[IEEE 754-2008](https://en.wikipedia.org/wiki/IEEE_754-2008_revision)标准定义的。

在此标准下，无法精确表示的非常大的整数将自动四舍五入。准确地说，JavaScript中的`Number` 类型只能安全地表示 -9007199254740991 (-(2<sup>53</sup>-1))和 9007199254740991 (2<sup>53</sup>-1)之间的整数。任何超出此范围的整数值都可能丢失精度。

这个很容易验证，执行以下代码:

```
console.log(9999999999999999);    // → 10000000000000000

```

该整数大于JavaScript可以用 `Number`原始类型表示的最大数字。因此,它被舍入了。意外的舍入可能会损害程序的可靠性和安全性。这是另一个例子:
```
// 注意最后一位
9007199254740992 === 9007199254740993;    // → true

```

JavaScript提供了`Number.MAX_SAFE_INTEGER` 常量，允许你在JavaScript中快速获得最大安全整数。类似地，你可以通过使用`Number.MIN_SAFE_INTEGER`常量获得最小安全整数:

```
const minInt = Number.MIN_SAFE_INTEGER;

console.log(minInt);         // → -9007199254740991

console.log(minInt - 5);     // → -9007199254740996

// 注意它是如何输出与上面相同的值的
console.log(minInt - 4);     // → -9007199254740996

```


### 解决办法

为了解决这些限制，一些JavaScript开发人员使用`String`类型表示大整数。例如，[Twitter API](https://developer.twitter.com/en/docs/basics/twit-IDs)在使用JSON响应时向对象添加了一个id的字符串版本。此外，还开发了一些库，如[bignumber.js](https://github.com/MikeMcl/bignumber.js/)，以便更容易地处理大整数。

有了 `BigInt`，应用程序不再需要一个变通方法或库来安全地表示`Number.MAX_SAFE_INTEGER` 和`Number.Min_SAFE_INTEGER`之外的整数。现在可以在标准JavaScript中执行对大整数的算术操作，而不会有丢失精度的风险。在第三方库上使用原生数据类型的好处是更好的运行时性能。

要创建`BigInt`，只需将`n`附加到整数的末尾。对比一下:
```
console.log(9007199254740995n);    // → 9007199254740995n
console.log(9007199254740995);     // → 9007199254740996

```

或者，你可以调用`BigInt()`构造函数：
```
BigInt("9007199254740995");    // → 9007199254740995n

```

`BigInt` 字面量也可以写成二进制、八进制或十六进制形式：

```

// 二进制
console.log(0b100000000000000000000000000000000000000000000000000011n);
// → 9007199254740995n

// 十六进制
console.log(0x20000000000003n);
// → 9007199254740995n

// 八进制
console.log(0o400000000000000003n);
// → 9007199254740995n

//注意，不支持旧式八进制语法
console.log(0400000000000000003n);
// → SyntaxError

```

记住，不能使用严格的相等运算符来比较`BigInt`和普通数字，因为它们不是同一类型的:

```
console.log(10n === 10);    // → false

console.log(typeof 10n);    // → bigint
console.log(typeof 10);     // → number

```

相反，你可以使用相等运算符，它在处理操作数之前执行隐式类型转换:
```
console.log(10n == 10);    // → true

```

所有算术运算符都可以在`BigInt`上使用，除了一元加号(`+`)运算符:
```
10n + 20n;    // → 30n
10n - 20n;    // → -10n
+10n;         // → TypeError: Cannot convert a BigInt value to a number
-10n;         // → -10n
10n * 20n;    // → 200n
20n / 10n;    // → 2n
23n % 10n;    // → 3n
10n ** 3n;    // → 1000n

let x = 10n;
++x;          // → 11n
--x;          // → 10n

```

不支持一元加号(`+`)运算符的原因是，有些程序可能依赖于这样的结果：`+`总是产生`Number`类型的值，或者抛出异常。改变`+` 的行为也会破坏asm.js代码。

当然，当与`BigInt`操作数一起使用时，算术运算符应该返回一个`BigInt`值。因此，除法(`/`)运算符的结果会自动四舍五入到最接近的整数。例如:
```
25 / 10;      // → 2.5
25n / 10n;    // → 2n

```


### 隐式类型转换

因为隐式类型转换可能丢失信息，所以不允许`BigInt`和`Number`之间的混合操作。当混合使用大整数和浮点数时，结果值可能无法用`BigInt`或`Number`准确表示。看看下面的例子:
```
(9007199254740992n + 1n) + 0.5

```

这个表达式的结果在`BigInt`和`Number`的范围之外。带有小数部分的`Number`不能准确地转换为`BigInt`。大于2<sup>53</sup>的`BigInt`不能准确转换为`Number`。

由于这个限制，不能使用`Number`和 `BigInt`操作数的组合来执行算术运算。你也不能将 `BigInt`传递给Web API和期望`Number`类型参数的内置JavaScript函数。试图这样做会导致`TypeError`:
```
10 + 10n;    // → TypeError
Math.max(2n, 4n, 6n);    // → TypeError

```

注意，关系运算符不遵循此规则，如下例所示:
```
10n > 5;    // → true

```

如果希望使用`BigInt`和 `Number`执行算术计算，首先需要确定应该在哪个域中执行操作。为此，只需通过调用`Number()`或`BigInt()`来转换操作数:
```
BigInt(10) + 10n;    // → 20n
// or
10 + Number(10n);    // → 20

```

当遇到`Boolean`上下文时，`BigInt`被视为类似于`Number`。换句话说，只要不是`0n`， `BigInt`就被认为是一个布尔真值:
```
if (5n) {
    // 这个代码块将被执行
}

if (0n) {
    // 但这个不会
}

```

对`BigInt`和 `Number`进行排序时，不会发生隐式类型转换:
```
const arr = [3n, 4, 2, 1n, 0, -1n];

arr.sort();    // → [-1n, 0, 1n, 2, 3n, 4]

```

按位运算符如`|`, `&`, `<<`, `>>`和 `^` 操作 `BigInt`与`Number`类似。负数被解释为无穷长二进制补码。不允许混合操作数。以下是一些例子:
```
90 | 115;      // → 123
90n | 115n;    // → 123n
90n | 115;     // → TypeError

```


### BigInt 构造函数

与其他基本类型一样，可以使用构造函数创建`BigInt`。如果可能，传递给`BigInt`的参数会自动转换为`BigInt`:
```
BigInt("10");    // → 10n
BigInt(10);      // → 10n
BigInt(true);    // → 1n

```

无法转换的数据类型和值会抛出异常:
```
BigInt(10.2);     // → RangeError
BigInt(null);     // → TypeError
BigInt("abc");    // → SyntaxError

```

你可以直接对通过`BigInt` 构造函数创建的变量执行算术运算：
```
BigInt(10) * 10n;    // → 100n

```

当用作严格相等运算符的操作数时，使用构造函数创建的`BigInt`操作数与常规操作数类似:
```
BigInt(true) === 1n;    // → true

```

### 库函数

JavaScript提供了两个库函数来把`BigInt`值表示为有符号或无符号整数:
*   `BigInt.asUintN(width, BigInt)`: 包装一个介于 0 和 2<sup>width</sup>-1之间的 `BigInt`
*   `BigInt.asIntN(width, BigInt)`: 包装一个介于 -2<sup>width-1</sup> 和2<sup>width-1</sup>-1之间的`BigInt` 

这些函数在执行64位算术操作时特别有用。这样你就可以保持在预定的范围内。
### 浏览器支持及转换

在撰写本文时，Chrome +67和Opera +54完全支持`BigInt` 数据类型。不幸的是，Edge和Safari还没有实现它。火狐默认不支持`BigInt` ，但可以通过在`about:config`中将`javascript.options.bigint`设置为`true`来启用。支持的浏览器的最新列表可以在[Can I use…](https://caniuse.com/#search=bigint)上找到。

不幸的是，转换`BigInt` 是一个[极其复杂的过程](https://github.com/googlechromelabs/jsbi#why)，这会导致严重的运行时性能损失。也不可能直接填充`BigInt` ，因为该提议改变了几个现有操作符的行为。目前，更好的选择是使用[JSBI](https://github.com/GoogleChromeLabs/jsbi)库，它是`BigInt` 建议的纯JavaScript 实现。

这个库提供了一个与内置`BigInt` 行为完全相同的API。下面是使用JSBI的方法:
```
import JSBI from './jsbi.mjs';

const b1 = JSBI.BigInt(Number.MAX_SAFE_INTEGER);
const b2 = JSBI.BigInt('10');

const result = JSBI.add(b1, b2);

console.log(String(result));    // → '9007199254741001'

```

使用JSBI的一个优点是，一旦浏览器支持得到改进，你就不需要重写代码了。相反，您可以使用[babel plugin](https://github.com/googlechromelabs/babel-plugin-transform-jsbito-BigInt)将你的JSBI代码自动编译成本地的`BigInt` 代码。此外，JSBI的性能与内置的`BigInt` 实现相当。你可以期待更广泛的浏览器支持`BigInt` 。

### 结论

`BigInt`是一种新的数据类型，用于当整数值大于`Number` 数据类型支持的范围时。这种数据类型允许我们安全地对大整数执行算术操作，表示高精度时间戳，使用大整数 ID 等等，而不需要使用库。

重要的是要记住，不能使用`Number`和`BigInt`操作数的组合来执行算术运算。你需要通过显式转换操作数来确定操作应该在哪个域中执行。此外，出于兼容性的原因，不允许在`BigInt`上使用一元加号(`+`)操作符。

你觉得怎么样?你觉得`BigInt`有用吗？欢迎评论！
