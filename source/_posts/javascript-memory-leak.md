---
title: JavaScript 内存泄漏防范之道
date: 2020-06-07 15:11:05
tags: JavaScript
---

一般情况下，忽视内存管理不会对传统的网页产生显著的后果。这是因为，用户刷新页面后，内存数据都被清理了。

但是随着SPA（单页应用）的普及，我们不得不更加关注页面的内存管理。用户在 SPA 上往往很少刷新页面，随着页面停留时间的增长，内存可能越占越多，轻则影响页面性能，严重的可能导致标签页崩溃。

在这篇文章中，我们将探讨导致 JavaScript 中内存泄露的常见原因，以及如何改善内存管理。
## 什么是内存泄漏以及如何发现

浏览器将对象保留在堆内存中，通过引用链可从根对象到达这些对象。垃圾回收器（GC）是 JavaScript 引擎中的一个后台进程，它可以识别无法到达的对象，将其删除，并回收相应的内存。
<!-- more -->
![引用链 - GC - 对象关系图](https://s1.ax1x.com/2020/06/23/NND039.md.png)

当内存中本应在垃圾回收循环中被清理的对象，通过另一个对象意外的引用从而维持可访问状态，就会发生内存泄漏。将多余的对象保持在内存中，会导致应用程序内部的内存使用量过大，进而影响性能。

![内存泄露](https://s1.ax1x.com/2020/06/23/NNDsnx.md.png)

如何判断代码是否存在内存泄漏呢？内存泄漏通常比较隐蔽，难以发现和定位。造成内存泄漏的 JavaScript 代码看上去挺正常，浏览器在运行的时候也不会抛出错误。如果发现页面性能越来越差，通常是内存泄漏的征兆，可以通过浏览器内置的工具判断是否存在内存泄漏，并分析出原因。

最快的方法是查看浏览器的任务管理器（注意，不是操作系统的任务管理器）。它提供了浏览器运行中的所有 tab 页和进程的资源使用情况，比如内存占用、CPU 占用和进程 ID 等。Chrome 的任务管理器可通过 Shift+Esc 快捷键打开，Firefox 可在地址栏输入`about:performance`打开。

如果页面都没有任何交互，内存占用却越来越多，很可能存在泄漏。

![Chrome 任务管理器](https://s1.ax1x.com/2020/06/23/NND2ND.md.png)

浏览器 DevTools 则提供了更丰富的内存管理功能。可以在 Chrome 的性能面板录制页面运行情况，查看可视化的性能分析数据。
![Chrome 性能面板](https://s1.ax1x.com/2020/06/23/NND4gA.md.png)

除此之外，Chrome 和 Firefox 的 DevTools 还有专门的内存工具用于分析内存使用情况。通过比较连续的内存快照，可以看出内存分配情况。
## JavaScript 代码内存泄露的常见原因

通过前面的分析，内存泄露的根本原因就是代码在无意之中引用了本该被 GC 回收的对象。那么，哪些情况容易造成内存泄露呢？
### 1. 意外的全局变量

全局变量一直处于可访问状态，不会被 GC 回收。在非严格模式下，有时会不小心让局部变量变成全局变量。
*   给未声明的变量赋值
*   使用指向全局对象的 `this`

```
function createGlobalVariables() {
    leaking1 = '变成全局变量了'; // 给未声明的变量赋值
    this.leaking2 = '这也是全局变量'; // 'this' 指向全局对象
};
createGlobalVariables();
window.leaking1; // '变成全局变量了'
window.leaking2; // '这也是全局变量'
```

**如何避免：** 严格模式 ("use strict") 会避免意外的全局变量，以上代码在严格模式下会报错。

### 2\. 闭包

函数作用域变量在函数执行完后会被清理，前提是在函数外部没有引用它。闭包会让变量一直处于被引用状态，即使它的执行上下文和作用域已经不存在了。
```
function outer() {
    const potentiallyHugeArray = [];
    return function inner() {
        potentiallyHugeArray.push('Hello'); 
        console.log('Hello');
    };
};
const sayHello = outer(); // 包含了对 inner 的引用

function repeat(fn, num) {
    for (let i = 0; i < num; i++){
        fn();
    }
}
repeat(sayHello, 10); // 每次调用 sayHello 都会添加 'Hello' 到potentiallyHugeArray 

// 如果是10万次呢？阔怕： repeat(sayHello, 100000)
```

上面例子中的数组 `potentiallyHugeArray` 没有从任何函数中直接返回，因此无法直接访问，但是它却不停地膨胀，取决于我们调用了多少次 `function inner()`。

**如何避免：** 闭包是 JavaScript 语言的特性之一，如果无法避开，那就请注意两点：

*   清楚闭包是何时创建的，以及哪些对象会被保留在内存中；
*   清楚闭包的生命周期和用途（尤其是当做回调函数的时候）

### 3\. 定时器

在 `setTimeout` 或 `setInterval` 的回调函数中引用某些对象，是防止被 GC 回收的常见做法。如果在代码里设置循环定时器（`setTimeout`也能像`setInterval`一样定时重复执行，只要设置成递归调用），只要定时器还在运行，回调函数中的对象就会一直保持在内存中。

下面的例子中，`data` 对象会在清除定时器后被 GC 回收。但我们没有获取 `setInterval`的返回值，也就没办法用代码清除这个定时器，因此尽管完全没有用到，`data.hugeString` 也会一直保留在内存中，直到进程结束。
```
function setCallback() {
    const data = {
        counter: 0,
        hugeString: new Array(100000).join('x')
    };
    return function cb() {
        data.counter++; // data 对象现在已经属于回调函数的作用域了
        console.log(data.counter);
    }
}
setInterval(setCallback(), 1000); // 没法停止定时器了
```

**如何避免：** 对于生命周期不确定的回调函数，我们应该：

*   注意被定时器回调函数引用的对象
*   使用定时器返回的句柄，在必要时清除它

也可以通过分离变量的方式，减少大对象被引用：
```
function setCallback() {
    // 分开定义变量
    let counter = 0;
    const hugeString = new Array(100000).join('x'); // setCallback执行完即可被回收
    return function cb() {
        counter++; // 只剩 counter 位于回调函数作用域
        console.log(counter);
    }
}

const timerId = setInterval(setCallback(), 1000); // 保存定时器 ID

// 执行某些操作 ...

clearInterval(timerId); // 停止定时器
```

### 4\. 事件监听器
活动的事件监听器会阻止作用域内的变量被 GC 回收。事件监听器一直处于活动状态，直到用 `removeEventListener()` 显式移除，或者关联的 DOM 元素被移除。

对于有些事件来说，监听器需要一直保留，直到页面被销毁。比如按钮点击事件，我们可能需要重复使用。但是，有时候我们希望某个事件只执行特定次数。
```
const hugeString = new Array(100000).join('x');
document.addEventListener('keyup', function() { // 匿名监听器无法移除
    doSomething(hugeString); // hugeString 会一直处于回调函数的作用域内
});
```
上面例子中的事件监听器用了匿名函数，这样就没法用`removeEventListener()`移除了。同时，`document`元素也无法删除，因此事件回调函数内的变量会一直保留，哪怕我们只想触发一次事件。

**如何避免：** 事件监听器不再需要时，要记得解除绑定。使用具名函数方式获取引用，通过`removeEventListener()`解除绑定。

```
function listener() {
    doSomething(hugeString);
}
document.addEventListener('keyup', listener); 
document.removeEventListener('keyup', listener); 
```

如果事件监听器只需要执行一次， `addEventListener()`可以接受第三个参数，是一个配置对象。指定`{once: true}`，监听器函数会在事件触发一次执行后自动移除（匿名函数也可以）。
```
document.addEventListener('keyup', function listener(){
    doSomething(hugeString);
}, {once: true}); // 执行一次后自动移除事件监听器
```

### 5\. 缓存

如果持续不断地往缓存里增加数据，没有定时清除无用的对象，也没有限制缓存大小，那么缓存就会像滚雪球一样越来越大。
```
let user_1 = { name: "Kayson", id: 12345 };
let user_2 = { name: "Jerry", id: 54321 };
const mapCache = new Map();

function cache(obj){
    if (!mapCache.has(obj)){
        const value = `${obj.name} has an id of ${obj.id}`;
        mapCache.set(obj, value);

        return [value, 'computed'];
    }

    return [mapCache.get(obj), 'cached'];
}

cache(user_1); // ['Kayson has an id of 12345', 'computed']
cache(user_1); // ['Kayson has an id of 12345', 'cached']
cache(user_2); // ['Jerry has an id of 54321', 'computed']

console.log(mapCache); // ((…) => "Kayson has an id of 12345", (…) => "Jerry has an id of 54321")
user_1 = null; 

//Garbage Collector
console.log(mapCache); // ((…) => "Kayson has an id of 12345", (…) => "Jerry has an id of 54321") // 依然在缓存里
```

上面的例子中，缓存依然保留了`user_1` 的数据。因此我们需要把不再使用的数据从缓存中删除。
**可能的解决方案：** 为了解决这个问题，可以使用 `WeakMap`。 `WeakMap` 是一种数据结构，它只用对象作为键，并保持对象键的弱引用，如果这个对象被置空了，相关的键值对会被 GC 自动回收。

```
let user_1 = { name: "Kayson", id: 12345 };
let user_2 = { name: "Jerry", id: 54321 };
const weakMapCache = new WeakMap();

function cache(obj){
    // 代码跟前一个例子相同，只不过用的是 weakMapCache

    return [weakMapCache.get(obj), 'cached'];
}

cache(user_1); // ['Kayson has an id of 12345', 'computed']
cache(user_2); // ['Jerry has an id of 54321', 'computed']
console.log(weakMapCache); // ((…) => "Kayson has an id of 12345", (…) => "Jerry has an id of 54321"}
user_1 = null; 

// Garbage Collector

console.log(weakMapCache); // ((…) => "Jerry has an id of 54321") - 第一条记录已被 GC 删除
```

### 6\. 分离的 DOM 元素

如果 DOM 节点被 JavaScript 代码直接引用，即使从 DOM 树分离，也不会被 GC 回收。

下面的例子中，`removeChild()` 达不到预期效果，堆快照会显示`HTMLDivElement `处于分离状态，因为有个变量指向了这个`div`。
```
function createElement() {
    const div = document.createElement('div');
    div.id = 'detached';
    return div;
}

// 即使调用了deleteElement() ，依然保存着 DOM 元素的引用
const detachedDiv = createElement();
document.body.appendChild(detachedDiv);
function deleteElement() {
document.body.removeChild(document.getElementById('detached'));
}

deleteElement(); // Heap snapshot will show detached div#detached
```

**如何避免：** 一种方法是把DOM 引用限制为局部作用域。

```
function createElement() {...} // 
// DOM 引用位于函数作用域内

function appendElement() {
    const detachedDiv = createElement();
    document.body.appendChild(detachedDiv);
}

appendElement();

function deleteElement() {
     document.body.removeChild(document.getElementById('detached'));
}

deleteElement(); // no detached div#detached elements in the Heap Snapshot
```

## 总结

对于重要的前端应用，定位和解决 JavaScript 内存问题是一项颇具挑战性的任务。因此，理解典型的内存泄露原因，从而在源头上避免，是做好内存管理的必要工作。