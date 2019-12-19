---
title: 刨根问底：深入研究 JavaScript 全局变量
date: 2019-12-15 12:57:48
tags: JavaScript
---
![](/uploads/1618526-2a722cbf2d490498.webp?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

本文的内容比较硬核，我们一起来看下 JavaScript 全局变量的底层机制到底是怎样的。文章会涉及脚本作用域、全局对象等概念。
### 作用域

变量的**词法作用域**（简称**作用域**）是程序中可以访问它的区域。JavaScript 的作用域是静态的（在运行时不会改变），并可以嵌套——例如：
```
function func() { // (A)
  const aVariable = 1;
  if (true) { // (B)
    const anotherVariable = 2;
  }
}
```

`if` 语句（B 注释行）引入的作用域嵌套在`func()`（A注释行）的作用域内。

包围作用域 S 的最内层的作用域称为 S 的**外部作用域**。在本例中， `func`是`if`的外部作用域。
<!-- more -->
### 词法环境

在 JavaScript 语言规范中，作用域是通过**词法环境**实现的。它包括两个组成部分：
*   一个将变量名映射到变量值的 *环境记录*。这是作用域内变量的实际存储空间。记录中的键值对称为 *bindings*。
*   对 *外部环境* 的引用——即外部作用域环境

嵌套作用域的树形结构其实就是用互相链接的环境的树形结构表示的。
### 全局对象

全局对象是一个特殊的对象，它的属性就是全局变量。（后面马上讲到它是怎样跟环境树形结构对应上的）可以通过以下全局变量访问它：
*   全平台可用的 `globalThis`。 之所以叫这个名字，是因为它跟全局作用域里的`this`值相同。
*   其他几个针对特定平台的变量：
    *   `window`是引用全局对象的经典方式。它可以在普通的浏览器代码中使用，但不能在*Web Workers* 和 Node.js 中使用。
    *  `self` 可用于浏览器（包括Web Workers），但不能在 Node.js 中使用。
    *   `global` 只在 Node.js 中可用。

### 浏览器中的`globalThis` 并不直接指向全局对象

在浏览器中，`globalThis` 并不是直接指向全局对象的，而是间接指向的。例如，假设 web 页面里有个 iframe：
*   每当 iframe 的`src` 发生变化时，它都会获得一个新的全局对象。
*   但是 `globalThis` 的值总是保持不变。 可以用下面的例子来验证：

`parent.html`文件：

```

<iframe src="iframe.html?first"></iframe>
<script>
  const iframe = document.querySelector('iframe');
  const icw = iframe.contentWindow; // iframe 中的`globalThis`  

  iframe.onload = () => {
    // 访问iframe全局对象的属性
    const firstGlobalThis = icw.globalThis;
    const firstArray = icw.Array;
    console.log(icw.iframeName); // 'first'

    iframe.onload = () => {
      const secondGlobalThis = icw.globalThis;
      const secondArray = icw.Array;

      // 不同的全局对象
      console.log(icw.iframeName); // 'second'
      console.log(secondArray === firstArray); // false

      // 但是 globalThis 仍然是一样的
      console.log(firstGlobalThis === secondGlobalThis); // true
    };
    iframe.src = 'iframe.html?second';
  };
</script>
```

`iframe.html` 文件：

```

<script>
  globalThis.iframeName = location.search.slice(1);
</script>
```

浏览器是怎么保证 `globalThis` 不变的呢？其实，在内部是通过这两个对象来区分的：
*   [`Window`](https://html.spec.whatwg.org/multipage/window-object.html#the-window-object) 全局对象，随着地址改变而改变。
*   [`WindowProxy`](https://html.spec.whatwg.org/multipage/window-object.html#the-windowproxy-exotic-object) 代理对象，负责转发对当前`Window`的访问，这个对象不会改变。

`globalThis` 在浏览器中指向`WindowProxy`，在其他环境里中直接指向全局对象。
### 全局环境

全局作用域是“最外层”的作用域——它没有外层作用域。它的环境就是 *全局环境*。 每个环境都链接到它的外层环境引用，形成一个链条，从而链接到全局环境。全局环境的外部环境引用值为`null`。

全局环境记录使用两个环境记录来管理其中的变量：
*   *对象环境记录*：跟普通环境记录的接口是一样的，只不过把 bindings  放在 JavaScript 对象里，也就是全局对象。

*   普通（声明式）环境记录：具有自己的 bindings 存储。

这两个记录是怎么用的，后面会讲到。
#### 脚本作用域和模块作用域

在 JavaScript 中，只有在脚本的顶层属于全局作用域。相反，每个模块都有自己的作用域，它是脚本作用域的子作用域。

如果我们忽略变量 bindings 添加到全局环境的复杂规则，全局作用域和模块作用域其实就像嵌套的代码块：
```
{ // 全局作用域 (*所有* script 的作用域)

  // (全局变量)

  { // module 1 作用域
    ···
  }
  { // module 2 作用域
    ···
  }
  // (其他模块作用域)
}
```

#### 创建变量：声明式记录和对象记录

为了创建一个真正的全局变量，必须处于全局作用域——也就是在脚本的最顶层。
*  顶层 `const`，`let`和 `class` 在声明式环境记录中创建 bindings。
*   顶层 `var` 和 函数声明在对象环境记录里创建 bindings。

```
<script>
  const one = 1;
  var two = 2;
</script>
<script>
  // 所有 script 共享同一个顶层作用域
  console.log(one); // 1
  console.log(two); // 2

  // 并非所有声明都会创建全局对象的属性
  console.log(globalThis.one); // undefined
  console.log(globalThis.two); // 2
</script>
```

#### 读取和设置变量

当我们获取或设置一个变量并且两个环境记录都具有该变量的 binding 时，则声明式记录优先级更高：
```
<script>
  let myGlobalVariable = 1; // 声明式环境记录
  globalThis.myGlobalVariable = 2; // 对象环境记录

  console.log(myGlobalVariable); // 1 (声明式记录优先)
  console.log(globalThis.myGlobalVariable); // 2
</script>
```

#### 全局 ECMAScript 变量和全局宿主变量

除了通过`var`和函数声明创建的变量之外，全局对象还包含以下属性：
*   所有内置 ECMAScript 全局变量
*   所有内置宿主平台全局变量（浏览器、Node.js 等）

使用`const`或`let`保证全局变量声明不影响ECMAScript 和宿主平台的内置全局变量（或免受其影响）。
例如，浏览器有全局变量 [ `.location`](https://developer.mozilla.org/en-US/docs/Web/API/Window/location):

```
// 会改变当前文档的地址：
var location = 'https://example.com';

// 不会改变 window.location
let location = 'https://example.com';
```

如果已经存在一个变量（例如本例中的`location`），则带有初始化程序的`var`声明的行为就类似于赋值。这就是我们在此示例中遇到麻烦的原因。

请注意，只有全局作用域才有这个问题。在模块中，不属于全局作用域（除非使用`eval()`或类似的东西）。


### 总结

为什么 JavaScript 既有普通全局变量又有全局对象？

全局对象通常被认为是一个错误设计。因此，较新的特性实现，如 `const`，`let`，和`class` 则会创建普通全局变量(在 script 作用域内)。

值得庆幸的是，大多数用现代 JavaScript 编写的代码都位于[ECMAScript 模块和 CommonJS 模块](https://exploringjs.com/impatient-js/ch_modules.html)中。每个模块都有自己的作用域，这就是为什么控制全局变量的规则很少影响基于模块的代码。

[](https://exploringjs.com/deep-js/ch_global-scope.html#the-global-object)
