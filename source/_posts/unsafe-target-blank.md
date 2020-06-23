---
title: 网页外链用了 target="_blank"，结果悲剧了
date: 2020-03-15 11:23:26
tags: 安全
---

今天给大家分享一个 Web 知识点。如果你有过一段时间的 Web 开发经验，可能已经知道了。不过对于刚接触的新手来说，还是有必要了解一下的。

我们知道，网页里的`a`标签默认在当前窗口跳转链接地址，如果需要在新窗口打开，需要给 `a` 标签添加一个`target="_blank"`属性。
```
<a href="http://kaysonli.com/" target="_blank">1024译站</a>
```
<!-- more -->
顺便提下一个有意思的现象，很早之前我就发现，国外网站倾向于在当前页跳转，而国内网站喜欢打开新窗口。不信你们可以去验证下。我不知道这是交互设计上的文化差异，还是技术上的开发习惯。

当然，这两种方式各有优缺点。当前页跳转显得操作比较有连贯性，不会贸然打断用户的注意力，也会减少浏览器的窗口（tab 页）数量。但是对于需要反复回到初始页面的场景来说，就很麻烦了。比如搜索结果页面，通常需要查看对比几个目标地址，保留在多个窗口还是比较方便。

今天要说的不只是用户体验上的差别，而是涉及安全和性能。

### 安全隐患
如果只是加上`target="_blank"`，打开新窗口后，新页面能通过`window.opener`获取到来源页面的`window`对象，即使跨域也一样。虽然跨域的页面对于这个对象的属性访问有所限制，但还是有漏网之鱼。

![window.opener](https://s1.ax1x.com/2020/03/15/815rS1.md.png)

这是某网页打开新窗口的页面控制台输出结果。可以看到`window.opener`的一些属性，某些属性的访问被拦截，是因为跨域安全策略的限制。

即便如此，还是给一些操作留下可乘之机。比如修改`window.opener.location`的值，指向另外一个地址。你想想看，刚刚还是在某个网站浏览，随后打开了新窗口，结果这个新窗口神不知鬼不觉地把原来的网页地址改了。这个可以用来做什么？钓鱼啊！等你回到那个钓鱼页面，已经伪装成登录页，你可能就稀里糊涂把账号密码输进去了。
![815UwF.jpg](https://s1.ax1x.com/2020/03/15/815UwF.jpg)
还有一种玩法，如果你处于登录状态，有些操作可能只是发送一个`GET`请求就完事了。通过修改地址，就执行了非你本意的操作，其实就是 CSRF 攻击。

### 性能问题
除了安全隐患外，还有可能造成性能问题。通过`target="_blank"`打开的新窗口，跟原来的页面窗口共用一个进程。如果这个新页面执行了一大堆性能不好的 JavaScript 代码，占用了大量系统资源，那你原来的页面也会受到池鱼之殃。

### 解决方案
尽量不使用`target="_blank"`，如果一定要用，需要加上`rel="noopener"`或者`rel="noreferrer"`。这样新窗口的`window.openner`就是`null`了，而且会让新窗口运行在独立的进程里，不会拖累原来页面的进程。当然，有些浏览器对性能做了优化，即使不加这个属性，新窗口也会在独立进程打开。不过为了安全考虑，还是加上吧。

我特意用自己的博客网站 [1024译站](http://kaysonli.com/) 试了一下，点击里面的外链打开新页面，`window.openner`都是`null`。查看页面元素发现，`a`标签都加上了` rel="noreferrer"`。博客是用 Hexo 生成的，看来这种设置已经成了基本常识了。

另外，对于通过`window.open`的方式打开的新页面，可以这样做：
```
var yourWindow = window.open();
yourWindow.opener = null;
yourWindow.location = "http://someurl.here";
yourWindow.target = "_blank";
```