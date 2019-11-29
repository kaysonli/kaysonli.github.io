---
title: 脑洞大开：CSS 也能玩转随机数？
date: 2019-11-09 20:04:54
tags: CSS
---

CSS 让你可以在网页上创建动态布局和界面，但是作为一门语言，它是静态的：一个值一旦设定就不能改变。要玩转随机数，好像不太可能？运行时生成随机数是JavaScript 领域的事情，而不是 CSS。果真如此吗？最近在 CSS-Tricks 上看到一位大牛的神操作，通过借助一点用户交互，巧妙地在CSS 里面实现了一定程度上的随机性，让人脑洞大开！

### 编程范式：声明式和命令式

关于编程范式，有声明式和命令式的区分。声明式编程是以数据结构的形式来表达程序执行的逻辑。它的主要思想是告诉计算机应该做什么，但不指定具体要怎么做。而命令式编程的主要思想是关注计算机执行的步骤，即一步一步告诉计算机先做什么再做什么。CSS 就是声明式的，各种选择器和规则告诉了浏览器该怎么渲染，但不包括变量和流程控制。JavaScript 则是命令式的，它可以灵活实现各种逻辑。

这么说来，生成随机数已经超出 CSS 的能力范围了。 你可能会说，我们有 SASS 和 LESS 这样的预处理器啊，不是支持各种函数、变量和循环之类的操作么？我们可以用它来生成随机数呀。其实，预处理器最终是要编译成 CSS 的，一旦完成编译，相关的值就确定了，随机性也就不存在了。

有几种利用css变量实现动态随机性的方法， Robin Rendle 在 CSS-Tricks 上的[一篇文章](https://css-tricks.com/random-numbers-css/) 对此有过解释。但是这些方案并不是100% CSS，因为需要 JavaScript 用新的随机值更新 CSS 变量。
<!-- more -->

### 如何实现 CSS 的随机性

掷骰子（还有抛硬币）是公认的随机行为，事先无法预测会出现什么结果。下面我们来看看这位大神是怎么在 CSS 里模拟掷骰子的效果的。

大体思路是这样的：通过叠加多层 label ，并使用CSS动画来“旋转”和交换最顶层。像这样：
![image](/uploads/1618526-e2b223a7c2cd5bcd.gif?imageMogr2/auto-orient/strip)

模拟这种随机化的代码不是很复杂，可以通过动画和不同的延迟时间来实现：
HTML:
```
<label for="d1">Click here to roll the dice</label>
<label for="d2">Click here to roll the dice</label>
<label for="d3">Click here to roll the dice</label>
<label for="d4">Click here to roll the dice</label>
<label for="d5">Click here to roll the dice</label>
<label for="d6">Click here to roll the dice</label>

<div>
  <input type="radio" id="d1" name="dice">
  <input type="radio" id="d2" name="dice">
  <input type="radio" id="d3" name="dice">
  <input type="radio" id="d4" name="dice">
  <input type="radio" id="d5" name="dice">
  <input type="radio" id="d6" name="dice">
  <p>You got a: <span id="random-value"></span></p>
</div>
```
CSS:
```
/* z-index 最大值相当于骰子上的数字 */ 
@keyframes changeOrder {
  from { z-index: 6; } 
  to { z-index: 1; } 
} 

/* 所有的 label 通过绝对定位互相重叠 */ 
label { 
  animation: changeOrder 3s infinite linear;
  background: #ddd;
  cursor: pointer;
  display: block;
  left: 1rem;
  padding: 1rem;
  position: absolute;
  top: 1rem; 
  user-select: none;
} 

/* 延迟时间用负数，这样动画的所有部分都动起来了 */ 
label:nth-of-type(1) { animation-delay: -0.0s; } 
label:nth-of-type(2) { animation-delay: -0.5s; } 
label:nth-of-type(3) { animation-delay: -1.0s; } 
label:nth-of-type(4) { animation-delay: -1.5s; } 
label:nth-of-type(5) { animation-delay: -2.0s; } 
label:nth-of-type(6) { animation-delay: -2.5s; }

#d1:checked ~ p span::before { content: "1"; }
#d2:checked ~ p span::before { content: "2"; }
#d3:checked ~ p span::before { content: "3"; }
#d4:checked ~ p span::before { content: "4"; }
#d5:checked ~ p span::before { content: "5"; }
#d6:checked ~ p span::before { content: "6"; }
```

动画已经被放慢，这样交互更容易(但还是太快了，这会导致一个问题，下面会解释到)。伪随机性也更明显。

这样能得到随机数，但有时点击却没有任何反应。尝试增加动画的时间，这似乎有点用，但仍然会得到一些意想不到的值。

这是什么原因呢？

简单来说，问题就是**浏览器只在鼠标按下时的活动元素与松开时的活动元素相同时才会触发click/press事件**。

由于旋转动画，鼠标按下的顶部 label 不是鼠标松开时的顶部 label，除非点击得足够快或足够慢，让动画绕一圈。这就是为什么增加动画时间可以缓解这个问题。

解决方案是应用一个“static”定位来打破堆叠上下文，并使用一个伪元素，如`::before`或 `::after` 和一个更高的`z-index` 来占据它的位置。这样，当鼠标松开时，当前活动的 label 总是位于顶部。
```
/* 活动状态的 label 是 static 定位，并位于窗口之外的位置 */ 
label:active {
  margin-left: 200%;
  position: static;
}

/* 拥有更高 z-index 的 label 伪元素占据了所有空间 */
label:active::before {
  content: "";
  position: absolute;
  top: 0;
  right: 0;
  left: 0;
  bottom: 0;
  z-index: 10;
}
```

### 不足之处
当然，这种取巧的方式存在一些不足：

*   **需要用户输入：**必须点击一个标签才能触发“随机数生成”。
*   **它的伸缩性不是很好：**它在小范围数值集合上工作得很好，但是对于大的范围来说就很麻烦了。
*   **它不是真正的随机，而是伪随机：**计算机可以很容易地检测出在每个时刻会产生哪些值

但另一方面，它是100% CSS(不需要预处理程序或其他外部帮助程序)，对于人类用户来说，它看起来是100%随机的。

利用这种办法，这位大神还做了一个剪刀锤子布的游戏：
![石头剪刀布](/uploads/1618526-b0fcbd5764264214.webp?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

[CodePen 源码：https://codepen.io/alvaromontoro/pen/BaaBYyz](https://codepen.io/alvaromontoro/pen/BaaBYyz)
