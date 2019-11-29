---
title: Resize Observer API 初探
date: 2019-10-27 20:04:54
tags: JavaScript
---

> [Resize Observer](https://wicg.github.io/ResizeObserver/)是一个新的JavaScript API，它非常类似于其他的观察者API，比如[Intersection Observer API](https://ator.io/js/observer/)。它允许在元素的大小发生变化时得到通知。

一个元素的大小变化最常见的原因是当视口被调整大小或设备的方向（纵向和横向）发生改变。到目前为止，我们只能依靠监听全局 `window.resize`事件并检查某些元素是否改变了大小。由于大量地触发事件，很容易导致性能问题。换句话说，就是使用 `window.resize` 通常是很浪费的，因为它会在每次视口大小变化时通知我们，而不仅仅是在元素的大小实际发生变化的时候。

还有另一个**Resize Observer API**的用例，窗口的resize 事件无法帮助我们：当动态地从DOM中添加或删除元素时，会影响父元素的大小。这种情况在现代单页应用程序中越来越常见。
## 基本用法
<!-- more -->
使用ResizeObserver很简单，只需实例化一个新的*ResizeObserver*对象，然后传入一个回调函数来接收观察到的元素：
```
const myObserver = new ResizeObserver(entries => {
  // 遍历 entries，做某些操作
});
```

然后，我们可以在我们的实例上调用*observe*，并传入一个元素来观察:
```
const someEl = document.querySelector('.some-element');
const someOtherEl = document.querySelector('.some-other-element');

myObserver.observe(someEl);
myObserver.observe(someOtherEl);
```

对于每个entry，我们将得到一个具有*`contentRect`*和*`target`*属性的对象。**`target`**是DOM元素本身，**`contentRect`**是一个具有以下属性的对象：*`width`*， *`height`*， *`x`*， *`y`*， *`top`*， *`right`*， *`bottom`*和*`left`*。

与元素的`getBoundingClientRect`不同，`contentRect`的宽度和高度值不包括`padding`。`contentRect.top`是元素的顶部内边距，`contentRect.left`是元素的左侧内边距。
* * *

例如，当元素的大小发生变化时，我们想要记录观察到的元素的宽度和高度，我们可以这样做:
```
const myObserver = new ResizeObserver(entries => {
  entries.forEach(entry => {
    console.log('width', entry.contentRect.width);
    console.log('height', entry.contentRect.height);
  });
});

const someEl = document.querySelector('.some-element');
myObserver.observe(someEl);
```

## 简单的例子

下面是一个简单的例子，演示如何使用Resize Observer API。通过调整浏览器窗口的大小来尝试一下，你会发现只有当元素的大小受到影响时，渐变角度和文本内容才会改变:
![image.png](/uploads/1618526-03bce9f2e2f48a17.webp?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* * *

让我们来看看这个简单的演示。首先，我们从一些简单的标记开始:
```
<div class="box">
  <h3 class="info"></h3>
</div>
<div class="box small">
  <h3 class="info"></h3>
</div>

```

和一些样式：
```
.box {
  text-align: center;
  height: 20vh;
  border-radius: 8px;
  box-shadow: 0 0 4px var(--subtle);

  display: flex;
  justify-content: center;
  align-items: center;
}

.box h3 {
  color: #fff;
  margin: 0;
  font-size: 5vmin;
  text-shadow: 0 0 10px rgba(0,0,0,0.4);
}

.box.small {
  max-width: 550px;
  margin: 1rem auto;
}
```

请注意，我们不需要将渐变背景应用于*`.box`*元素。当页面第一次加载时，resize observer 将被调用一次，然后应用我们的渐变。

现在，就是见证奇迹的时刻，当我们添加以下JavaScript代码:
```
const boxes = document.querySelectorAll('.box');

const myObserver = new ResizeObserver(entries => {
  for (let entry of entries) {
    const infoEl = entry.target.querySelector('.info');
    const width = Math.floor(entry.contentRect.width);
    const height = Math.floor(entry.contentRect.height);

    const angle = Math.floor(width / 360 * 100);
    const gradient = `linear-gradient(${ angle }deg, rgba(0,143,104,1) 50%, rgba(250,224,66,1) 50%)`;

    entry.target.style.background = gradient;

    infoEl.innerText = `I'm ${ width }px and ${ height }px tall`;
  }
});

boxes.forEach(box => {
  myObserver.observe(box);
});
```

这里我们使用一个[`for…of`](https://ator.io/js/for-for-in-loops/)循环迭代观察者回调中的元素，但是在 *`entries`*上调用 `forEach` 也是一样的。

请注意，我们还必须遍历可以观察的元素，并对每个元素调用`observe`。
##  浏览器支持情况

目前的浏览器支持情况不太乐观，只有Chrome 64+支持Resize Observer 。所幸，我们可以同时使用 [这个 polyfill](https://github.com/que-etc/resize-observer-polyfill)，它基于[MutationObserver API](https://developer.mozilla.org/en-US/docs/Web/API/MutationObserver)。

[原文](https://alligator.io/js/resize-observer/)

