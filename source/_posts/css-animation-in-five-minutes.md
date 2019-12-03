---
title: 5分钟学会 CSS 动画：纯 CSS 实现 loading 效果
date: 2019-12-02 10:55:02
tags: CSS
---


你是不是跟我一样，一直想学会 CSS 动画？今天我们就一起来花五分钟做几个纯 CSS 加载动画。
## 前言

CSS 动画不是什么新鲜玩意了，目前所有主流浏览器基本上都已经支持，可以看[Can I Use 网站](https://caniuse.com/#feat=css-animation) 上的统计数据。文章结尾部分会提到针对老古董浏览器的兼容方案。

网页里经常会用到数据加载中的 loading 效果，当然你也可以用 gif 动图，但更好的做法是用 CSS 动画。
## 语法

废话少说，先上代码：
```
<!-- HTML -->
<div class="simple-spinner"></div>

```

```
/* CSS */
.simple-spinner {
  height: 48px;
  width: 48px;
  border: 5px solid rgba(150, 150, 150, 0.2);
  border-radius: 50%;
  border-top-color: rgb(150, 150, 150);
  animation: rotate 1s 0s infinite ease-in-out alternate;
}
@keyframes rotate {
  0%   { transform: rotate(0);      }
  100% { transform: rotate(360deg); }
}

```
效果是这样的：
<!-- more -->
![](https://upload-images.jianshu.io/upload_images/1618526-a78ad82f05ddb97c.gif?imageMogr2/auto-orient/strip)

看起来还不错吧？这是怎么做到的呢？首先我们用`<div>` 做了一个宽高相等的圆环，设置了边框，其中一边的颜色不一样，`border-radius` 设置成 50%，就成了圆形。为了设置动画效果，我们做了2件事：
1.  给目标元素定义 `animation` 属性
2.  定义 `@keyframes` 规则

### animation 属性一览

`animation` 是一个简写属性，可以同时设置多个动画属性。以上面的例子为例，拆开细看：
| example | 属性 | 描述 |
| --- | --- | --- |
| rotate | [`animation-name`](https://developer.mozilla.org/en-US/docs/Web/CSS/animation-name) | animation 名字 |
| 1s | [`animation-duration`](https://developer.mozilla.org/en-US/docs/Web/CSS/animation-duration) | 动画单次循环持续时长 |
| 0s | [`animation-delay`](https://developer.mozilla.org/en-US/docs/Web/CSS/animation-delay) | 动画开始前的等待时长 |
| infinite | [`animation-iteration-count`](https://developer.mozilla.org/en-US/docs/Web/CSS/animation-iteration-count) | 动画重复次数 |
| ease-in-out | [`animation-timing-function`](https://developer.mozilla.org/en-US/docs/Web/CSS/animation-timing-function) | 动画运行方式 |
| alternate | [`animation-direction`](https://developer.mozilla.org/en-US/docs/Web/CSS/animation-direction) | 动画前进、后退或者来回运动 |

这就是说，我们把动画命名为`rotate`，一个循环是一秒，开始前无需等待，动画持续进行，采用非线性方式来回运动（每秒改变一次方向）。
*注意： 完整列表还有 [`animation-fill-mode`](https://developer.mozilla.org/en-US/docs/Web/CSS/animation-fill-mode) 和 [`animation-play-state`](https://developer.mozilla.org/en-US/docs/Web/CSS/animation-play-state)属性，今天先不讲了。*

###  @keyframes 规则

现在我们来看动画制作的部分。有了 `@keyframes`，可以指定动画在时间轴上的状态。对于每个状态，你可以定义出现的时间点(动画时间的百分比)和应用的样式规则。

我们的例子中，动画从 0% `transform: rotate(0);`（无旋转）到100% `transform: rotate(360deg);`（旋转一圈）。

当然， `@keyframes` 规则可以用在多个元素上，下面这个例子，两个元素用了不同的`animation`规则，但用的是同一个 `@keyframes` ：
```
<!-- HTML -->
<div class="double-spinner"></div>

```

```
/* CSS */
.double-spinner {
  height: 48px;
  width: 48px;
  border: 5px solid rgba(150, 150, 150, 0.2);
  border-radius: 50%;
  border-top-color: rgb(150, 150, 150);
  animation: rotate 1s 0s infinite linear normal;
}
.double-spinner::after {
  content: '';
  height: 40%;
  width: 40%;
  display: block;
  margin: 10px auto;
  border: inherit;
  border-radius: inherit;
  border-top-color: inherit;
  animation: rotate .5s 0s infinite linear reverse;
}
@keyframes rotate {
  0%   { transform: rotate(0);      }
  100% { transform: rotate(360deg); }
}

```
效果：
![](https://upload-images.jianshu.io/upload_images/1618526-cd404342048c54a6.gif?imageMogr2/auto-orient/strip)


对于开始和结束状态（`0%` 和 `100%`），你还可以用关键字 `from` 和 `to`来表示：
```
@keyframes rotate {
  from { transform: rotate(0);      }
  to   { transform: rotate(360deg); }
}

```

再来看一个有更多状态的例子：
```
<!-- HTML -->
<div class="flip-walker"></div>

```

```
/* CSS */
.flip-walker {
  width: 64px;
  height: 64px;
}
.flip-walker::before {
  content: '';
  display: block;
  width: 50%;
  height: 50%;
  background: rgba(150, 150, 150, .5);
  animation: flip 2s 0s infinite ease normal;
}
@keyframes flip {
  0%   { transform: translate(0, 0)       rotateX(0)       rotateY(0);      }
  25%  { transform: translate(100%, 0)    rotateX(0)       rotateY(180deg); }
  50%  { transform: translate(100%, 100%) rotateX(-180deg) rotateY(180deg); }
  75%  { transform: translate(0, 100%)    rotateX(-180deg) rotateY(360deg); }
  100% { transform: translate(0, 0)       rotateX(0)       rotateY(360deg); }
}

```
效果：
![](https://upload-images.jianshu.io/upload_images/1618526-c798e5aee6992d29.gif?imageMogr2/auto-orient/strip)

这里我们用了[CSS transform](https://developer.mozilla.org/en-US/docs/Web/CSS/transform)，在5个状态点上设置不同变换，实现了翻转动画效果。
## 多重动画

可以对同一个元素应用多个动画。如果我们想让方块在翻转的同时还发光，我们可以简单地添加另一个动画，用逗号分隔：
```
/* CSS */
.flip-walker::before {
  /* ... */
  animation:
    flip 2s 0s infinite ease normal,
    glow 2s 0s infinite linear normal;
}
@keyframes flip {
  /* ... */
}
@keyframes glow {
  to { background: white; box-shadow: 0 0 15px white; }
}

```
效果：
![叠加动画](https://upload-images.jianshu.io/upload_images/1618526-555f4b24c8797ae1.gif?imageMogr2/auto-orient/strip)


*注意：这里只设置了 `to` 规则，省略了 `from` 规则。这是因为，针对省略的开始或结束状态，浏览器会使用元素的当前或初始样式。 *

## 自定义 timing 函数

如果你觉得默认的 timing 函数不太好玩，你可以通过贝塞尔曲线自己定义。有很多工具可以定义这种曲线，比如[这个](https://matthewlein.com/tools/ceaser)。

来看最后一个酷炫的例子：
```
<!-- HTML -->
<div class="pulse"></div>

```

```
/* CSS */
.pulse {
  width: 64px;
  height: 64px;
  background: white;
  border-radius: 50%;
  animation: pulse 1s 0s infinite cubic-bezier(0.000, 1.010, 0.5, 1.200) normal;
}
@keyframes pulse {
  from { transform: scale(0);   opacity: 1; }
  to   { transform: scale(1.0); opacity: 0; }
}

```
效果：
![心跳](https://upload-images.jianshu.io/upload_images/1618526-d960e858a2148c01.gif?imageMogr2/auto-orient/strip)

timing 函数替换成了刚刚用工具生成的 `cubic-bezier(0.000, 1.010, 0.5, 1.200)`。
## 兼容方案

如果你想兼容老古董浏览器，可以用 [`@supports` 指令](https://developer.mozilla.org/en-US/docs/Web/CSS/@supports) 查询特性支持情况，然后用 gif 作为备选方案。例如：
```
/* CSS */
.animated {
  background: url(animation-as.gif);
}
@supports (animation-name: test) {
  .animated {
    background: none;
    animation-name: test;
    /* ... */
  }
  @keyframes test {
    /* ... */
  }
}

```

不支持  `animation-name` 属性，甚至都不认识 `@supports` 指令的浏览器，就会显示GIF图片作为元素背景。
## 总结

通过几个简单的例子，我们学会了如何实现 CSS 动画、定义不同的状态、使用多重动画以及自定义 timing 函数。看到没，CSS 动画就是这么简单！
