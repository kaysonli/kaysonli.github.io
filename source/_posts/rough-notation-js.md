---
title: 只需几行 JavaScript 代码，网页瞬间有气质了！
date: 2020-06-02 14:44:36
tags: JavaScript
---

最近在网上闲逛，发现一个特别好玩的 JavaScript 库，叫 RoughNotation。干嘛用的呢？就是在网页上给文字加标注，比如下划线、方框、高亮文字背景等，不过是手写风格的！截图给大家感受下：
![手绘风格的标注](https://s1.ax1x.com/2020/06/29/NfuV8U.png)

怎么样？是不是感觉网页瞬间就生动了？是不是有种手捧纸质书，用彩笔在纸上做笔记的感觉？满满的文艺范！
<!-- more -->
它支持多种标注形式，除了前面提到的，还有圆圈、叉、删除线等，还能够自定义样式（颜色和粗细等），并且支持动画效果。

使用也很简单，可以通过`npm`安装：
```
npm install --save rough-notation
```
或者直接通过 ES 模块引入（浏览器支持才行）：
```
<script type="module" src="https://unpkg.com/rough-notation?module"></script>
```
还可以直接引入 IIFE 格式的脚本，这样就会得到一个全局变量`RoughNotation `：
```
<script src="https://unpkg.com/rough-notation/lib/rough-notation.iife.js"></script>
```
用法也很简单，找到需要标注的 DOM 元素，调用 API 就行：
```
import { annotate } from 'rough-notation';

const e = document.querySelector('#myElement');
const annotation = annotate(e, { type: 'underline' });
annotation.show();
```
更多用法可参考官方文档和 Github 仓库。

作为技术人，其实我们更关心的是它怎么实现的。RoughNotation 基于 RoughJS 这个库开发，后者就是一个用手绘风格绘图的库。打开 Chrome 控制台查看下元素，发现这些标注是通过 SVG 绘制出来的。SVG 使用了绝对定位，根据目标 DOM 元素位置计算坐标。
![image.png](https://s1.ax1x.com/2020/06/29/NfuG8O.png)

好了，就简单介绍到这。有兴趣的可以去安装使用下，分分钟让你的页面变得个性十足，快分享给你的小伙伴们吧！
