---
title: 这些特效即将告别JavaScript，迎来CSS时代！
date: 2019-11-02 20:04:54
tags: JavaScript,CSS
---

随着CSS和JavaScript的发展，这两种语言之间的界限逐渐变得模糊。

从自定义属性（即变量）到滤镜、动画或数学操作，CSS采用了很多我们以前在JavaScript（或流行的CSS预处理器）中使用的东西，并为我们提供了原生支持。

两种语言都有不同的用途。现代浏览器的每次发布，都带来了新的特性支持，CSS正在成为一种非常强大的语言，能够处理我们以前依赖JavaScript的功能。

在这篇文章中，我们将学习一些你可能没有听说过的CSS特性，它们能处理**平滑滚动**，**暗黑模式**，**表单验证**，以及更多以前需要多行JS代码技巧才能完成的功能。
## 平滑滚动

曾经有一段时间，我们不得不依靠JavaScript(甚至jQuery)来做到这一点，用的是`window.scrollY`。多亏了[scroll-behavior](https://developer.mozilla.org/en-US/docs/Web/CSS/scroll-behavior)属性，那些日子已经一去不复返了。我们现在可以用一行CSS代码就能实现网页的平滑滚动：
```
html {
  scroll-behavior: smooth;
}

```

浏览器支持率[约75%](https://caniuse.com/#feat=css-scroll-behavior)， Edge很快也将推出76版本。看看它是如何工作的：
[CodePen 示例代码](https://codepen.io/imjuangarcia/pen/RwbQLPe)
<!-- more -->
## 暗黑模式

自从[macOS 发布了暗黑模式功能](https://developer.apple.com/design/human-interface-guidelines/macos/visual-design/dark-mode/)后，暗黑模式最近变得非常流行。Safari 实现了`prefers-color-scheme`媒体特性，它允许我们监测用户是否默认启用了暗黑模式。

你可能认为暗黑模式是一个复杂的特性，它包括CSS变量、每种方案的不同颜色和一些JavaScript代码来处理必要的单击事件以完成对网站的更改。这么说也没错，它是目前的[标准实现方法](https://dev.to/ananyaneogi/create-a-dark-light-mode-switch-with-css-variables-34l8)。

Wei Gao[在她的博客上向我们展示了](https://dev.wgao19.cc/sun-moon-blending-mode/)用 `mix-blend-mode: difference`(CSS支持的一种混合模式)实现类似效果(更多的是一种“反色模式”)。如果你曾经尝试过[Photoshop混合模式](https://helpx.adobe.com/photoshop/using/blending-modes.html)，这跟它是一样的，只不过是在浏览器上。

它的一些优点包括不必指定反转的颜色(混合模式可以为你做到这一点)，并且你可以隔离不想更改的元素。它的一些限制是，你没有一个完整的颜色范围，性能可能是一个问题。

目前，[浏览器原生支持率77%，带前缀的话再多出 13%](https://caniuse.com/#search=mix-blend-mode)(全球91%)，Edge在76版本开始支持。

要更深入地了解这种混合模式的工作原理，请务必查看 [Wei 的文章](https://dev.wgao19.cc/sun-moon-blending-mode/) 。如果想体验下有趣的实验，请看这个[CodePen 示例代码](https://codepen.io/imjuangarcia/embed/xxKLMEN)
![夜间模式](https://upload-images.jianshu.io/upload_images/1618526-a669630d07c82407.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![白天模式](https://upload-images.jianshu.io/upload_images/1618526-74a2a50db5f781dc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 截断文本

这是我个人最喜欢的特性之一，因为这是在开发CMS网站页面会遇到的一个常见问题。可变的文字长度可能会使你设计良好的卡片在不同的尺寸或分辨率下看起来不一致。

在过去，我总是在不加思索地寻找基于JavaScript 的解决方案，因为[实现这个效果的大多数CSS技术都相当“hacky”](https://css-s.com/clampin/)。但是，最近我发现了一些CSS属性，它们结合在一起可能能够提供一个原生的、易于实现的解决方案。这就是`text-overflow`和`line-clamp`。

## 文本溢出

这是一个被广泛采用，[并且浏览器完全支持的](https://caniuse.com/#search=text-overflow)原生css解决方案，用于控制文本在溢出其包含元素时的行为。你可以将其值设置为`ellipsis`，这样就会得到Unicode字符 `…` 。到目前为止还不错，但它的主要限制是无论文本的长度如何，总是会有一行被截断的文本。因此，这可能对标题非常适合，但对长段落的多行文本不太适用。

这个时候 `line-clamp` 就派上用场了。

## Line-clamp

`line-clamp`属性也不是什么新东西。大约十年前，[Dave DeSandro](https://twitter.com/desandro)向我们展示了[这项技术](https://dropshado.ws/post/1015351370/webkit-line-clamp)(因此需要使用老的flexbox实现，也就是带前缀的`display: -webkit-box` 和`-webkit-box-orient: vertical`)。

那么，有什么新动向吗？Firefox[在68版本中实现了它](https://bugzilla.mozilla.org/show_bug.cgi?id=WebKit-line-clamp)，但是需要带`-webkit`前缀。现在Edge是基于Chromium的，我们也可以使用 `-webkit`前缀，这样[浏览器支持率高达92%](https://caniuse.com/#search=line-clamp)。

这意味着我们现在可以使用三行CSS代码来截断多行文本，[像这样](https://codepen.io/imjuangarcia/pen/dybdYbB)。
![文本截断](https://upload-images.jianshu.io/upload_images/1618526-d1220947f2fc0033.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

唯一的问题是，截断多行文本的规范是[CSS Overflow Module Level 3](https://drafts.csswg.org/css-overflow-3/#propdef--webkit-line-clamp)的一部分，而它目前还是编辑草案，这可能意味着规范可能会有一些变化。另一个需要考虑的事情是，你无法控制要显示的字符数，这可能会导致一些[不太方便(但很有趣)的截断场景。](https://twitter.com/search?f=tweets&vertical=default&q=karenmcgrane%20truncation%20is%20not&src=typd)除了这个问题，愉快地玩耍CSS 截断文本吧！

## 滚动停顿

CSS 滚动停顿是Chrome用户已经使用了一段时间的另一个方便的功能，现在 FireFox 最新的68版本也支持了。另外，Edge 76 版本也将支持， [整体支持程度已经覆盖主流浏览器](https://caniuse.com/#feat=css-snappoints)。

你是否注意到一些花哨的网站会创建全屏区域，并在你滚动时锁定特定位置的视口？这里有一个这样的[例子](https://www.artandscience.jp/)。

在此之前，要实现这样的功能还是挺麻烦的，需要大量的数学计算和 JavaScript 代码。但是现在，CSS 就能原生实现这样的[交互效果了](https://codepen.io/imjuangarcia/pen/zYORdMK)。

它的工作原理类似于Flexbox或CSS网格，你需要一个容器元素，给它设置`scroll-snap-type` ，加上多个设置了`scroll-snap-align`的子元素，就像这样：

HTML代码：
```
<main class=”parent”>
  <section class=”child”></section>
  <section class=”child”></section>
  <section class=”child”></section>
</main>

```
CSS 代码：
```
.parent {
  scroll-snap-type: x mandatory;
}

.child {
  scroll-snap-align: start;
}

```
 `scroll-snap-type` 有两个值：`mandatory` 强制停顿在元素的顶部或底部，`proximity` 会替你计算好位置，当内容足够接近目标位置时停顿。

在父元素上还可以设置的一个属性是 `scroll-padding`，如果在布局中有固定定位的元素时非常好用。比如固定的头部，或者 cookie 策略通知[😅](https://s.w.org/images/core/emoji/12.0.0-1/72x72/1f605.png))，不然的话有些内容会被遮挡。

对于子元素，目前只有`scroll-snap-align`，用于告诉容器在哪些位置停顿（顶部，中间或底部）。

既然知道了这项神技通过几行CSS代码，无需数学计算就可以实现这个效果，你可能经不住诱惑给整个网站都加上滚动停顿效果。但是请记住[这条唯一的网页设计法则](https://www.robinrendle.com/notes/scrolljacking)，也就是Robin Rendle 说的：**不要瞎几把滚动**。这项技术可能对滑动幻灯片、图库预览或页面特定部分有用，但是用的时候要克制点，因为它会影响性能和整体的用户体验。

## Sticky 导航

之前的一些特性需要大量的JavaScript计算，而且实现性能方面的代价也相当高昂，现在好了，我们有了 sticky 定位方式。之前我们需要用 `offsetTop`和 `window.scrollY`，现在全都交给神奇的`position: sticky` ！sticky 定位的元素默认表现为相对定位元素，直到视口达到某个位置时它就会变成 fixed 定位的元素。[带上 `-webkit` 前缀，浏览器支持率高达 92% ](https://caniuse.com/#feat=css-sticky)

因此，只要这样写就能轻松实现：
```
header {
  position: sticky;
  top: 0;
}

```

要正确地将 header 设置为 sticky 定位，HTML 结构非常重要。因此，假如你的HTML结构像这样：
```
<main>
  <header>
    <h1>This is my sticky header!</h1>
  </header>
  <section>This is my content</section>
</main>

```
header 只会附着在父元素（这里是`<main>`标签）所在区域。“sticky 父元素”决定了“sticky 子元素”的作用范围。Elad Shechter 的[这篇文章](https://medium.com/@elad/css-position-sticky-how-it-really-works-54cd01dc2d46) 很好地解释了这个问题。另外，这里还有一个利用这个特性的[有趣实验](https://codepen.io/imjuangarcia/pen/QWLQjqQ)。

## 最后的福利： @supports

正如前面说到的，尽管这些 CSS 特性 已经被广泛采用和支持，你可能还是想在使用之前检查[浏览器是否支持](https://caniuse.com/)。如果是这样，你可以用`@supports` 特性查询，这个功能已经被[浏览器广泛采用](https://caniuse.com/#feat=css-featurequeries)。它可以测试浏览器在应用样式之前是否支持特定的`property: value` 。语法如下：
```
@supports (initial-letter: 4) {
  p::first-letter {
    initial-letter: 4;
  }
}

```
有了这个方便的特性查询，你可以在支持那些特性的浏览器上应用特定样式。它的语法看起来很熟悉，因为媒体查询也是这样写的。这是很棒的一种方式实现所谓的*渐进增强*，在支持的浏览器上使用这些最新的特性，而避免了不支持的浏览器的行为不一致。

好了，就此结束！希望这些最新的CSS特性能帮你用更小的JavaScript代码实现有趣又吸引人的界面。另外，可以再看看这些[Codepens](https://codepen.io/collection/DrGkMr) 示例代码。

[原文](https://dev.to/bnevilleoneill/5-things-you-can-do-with-css-instead-of-javascript-975)

