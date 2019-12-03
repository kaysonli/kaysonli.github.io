---
title: 喜大普奔：微软 Edge 79 将采用 Chromium 引擎，前端开发者的福音！
date: 2019-11-27 09:46:17
tags: 行业
---

>原文：[https://www.infoq.com/news/2019/11/microsoft-edge-chromium/](https://www.infoq.com/news/2019/11/microsoft-edge-chromium/)
翻译：1024译站

随着Edge 79的发布，微软将从其专有的[EdgeHTML](https://en.wikipedia.org/wiki/EdgeHTML)引擎过渡到 [Chromium](https://www.chromium.org/Home)， 这个支撑Chrome 的流行开源引擎。

此举旨在增强微软 Edge 与主导市场的基于Chromium 的浏览器之间的兼容性。因此，开发人员将不再需要执行其他测试或添加特定的代码来完全支持 Edge 浏览器，从而可以提高浏览器的使用率。
<!-- more -->
![Microsoft Edge browser comparison](/uploads/1618526-33aaf4b06cd2de0f.webp?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

上面的屏幕截图是 microsoft.com 网站通过当前版本的 Edge 版本（左）和即将推出的带有 Chromium 引擎的 Edge 79版本（使用 beta 版本分支）的渲染效果对比。

用户界面的差异相对较小，主要集中在标签区域。它稍微苗条一些，并包含几个用于管理标签的其他按钮，而内容本身似乎是相同的。另一方面，像[webassign.net](http://webassign.net/)这样的网站以前不支持 Microsoft Edge，并且报告了各种显示问题，这些问题导致用户无法完全使用该网站，现在应该可以正常工作了。

决定采用开源 Chromium 引擎可能代表了 Microsoft 内部的一次更大转变，Microsoft 迄今为止已使用了诸如 EdgeHTML 引擎之类的专有解决方案。关于这一点，在2018年12月首次宣布这一举动时，有人提到转向开源哲学是微软决定采取此举的主要原因之一。

值得一提的是，尽管许多人将改编开源 Chromium 引擎视为 Microsoft 朝着正确方向迈出的一步，但它确实带来了隐性成本，因为它将减少浏览器市场的多样性。

Mozilla 首席执行官 Chris Beard 在一封公开信中对此表示关注，他说：“如果像 Chromium 这样的产品具有足够的市场份额，则 Web 开发人员和企业可以不用担心他们的服务和网站是否可以运行在非 Chromium 平台上。”

FireFox 在2002年首次发布时就是这种情况。当时，Microsoft Internet Explorer 主导了市场，并在浏览器中集成了许多非标准功能。结果，试图切换到 Firefox 的用户经常发现他们最喜欢的许多网站不能正常运行了。同时，开发人员不愿满足新的浏览器要求，因为它大大增加了他们的开发成本。

Microsoft Edge 79 RC 版已于11月在Microsoft Ignite 2019活动期间宣布，可以通过[Microsoft Edge内部人员频道](https://www.microsoftedgeinsider.com/zh-cn/download)下载。 希望开始使用 Edge 79的用户可以在三个发布选项之间进行选择——Beta，Developer 和 Canary——代表不同的发布周期（六周，每周和每天），其中 Beta 版本最为稳定，而Canary 版本包含最新的更改。

Edge 正式版本计划于1月15日发布。
