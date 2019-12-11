---
title: 用原生 JavaScript 实现 DOM 树上下元素查找
date: 2019-12-10 17:59:25
tags: JavaScript
---

在最近的项目里，我需要在 DOM 树向上查找拥有某个class, ID, 或者 data 属性的第一个元素。

我知道用 jQuery 实现这个功能简直小菜一碟，但今天我想分享一下用原生 JavaScript 方法实现 jQuery 里的 `.closest()`，`.parent/s()`和 `.find()` 这些API。

## 向上查找 DOM 元素

jQuery 提供了几个向上查找元素的方法。下面是一些可能的应用场景。
### 向上查找第一个匹配的元素

jQuery  里的`.closest()` 会向上查找 DOM 树，直到找到匹配指定选择器的第一个元素。来看原生 JS 方法怎么实现：
<!-- more -->
```
/**
 * 向上查找最近的匹配元素
 * @private
 * @param  {Element} elem     起始元素
 * @param  {String}  selector 选择器
 * @return {Boolean|Element}  如果找不到匹配结果则返回null
 */
var getClosest = function ( elem, selector ) {

	// Element.matches() polyfill
	if (!Element.prototype.matches) {
		Element.prototype.matches =
			Element.prototype.matchesSelector ||
			Element.prototype.mozMatchesSelector ||
			Element.prototype.msMatchesSelector ||
			Element.prototype.oMatchesSelector ||
			Element.prototype.webkitMatchesSelector ||
			function(s) {
				var matches = (this.document || this.ownerDocument).querySelectorAll(s),
					i = matches.length;
				while (--i >= 0 && matches.item(i) !== this) {}
				return i > -1;
			};
	}

	// 获取最近的匹配
	for ( ; elem && elem !== document; elem = elem.parentNode ) {
		if ( elem.matches( selector ) ) return elem;
	}

	return null;

};

```

用法：

```
var elem = document.querySelector('#example');
var closestElem = getClosest(elem, '.sample-class');

```

如果你想从父元素开始查找，只要这样写：
```
var elem = document.querySelector('#example');
var closestElem = getClosest(elem.parentNode, '#sample-id');

```

### 向上查找所有匹配元素

jQuery 里的`.parents()`向上查找并返回所有父元素。如果指定了选择器，则只返回匹配的元素。等价的原生 JavaScript 代码如下：
```
/**
 * 获取元素的所有父元素
 * @param  {Node}   elem     该元素
 * @param  {String} selector 要匹配的选择器（可选）
 * @return {Array}           父元素集合
 */
var getParents = function ( elem, selector ) {

	// Element.matches() polyfill
	if (!Element.prototype.matches) {
		Element.prototype.matches =
			Element.prototype.matchesSelector ||
			Element.prototype.mozMatchesSelector ||
			Element.prototype.msMatchesSelector ||
			Element.prototype.oMatchesSelector ||
			Element.prototype.webkitMatchesSelector ||
			function(s) {
				var matches = (this.document || this.ownerDocument).querySelectorAll(s),
					i = matches.length;
				while (--i >= 0 && matches.item(i) !== this) {}
				return i > -1;
			};
	}

	// 父元素数组
	var parents = [];

	// 获取匹配的所有父元素
	for ( ; elem && elem !== document; elem = elem.parentNode ) {

		// 添加匹配的父元素到数组
		if ( selector ) {
			if ( elem.matches( selector ) ) {
				parents.push( elem );
			}
		} else {
			parents.push( elem );
		}

	}

	return parents;

};

```

用法：

```
var elem = document.querySelector('#some-element');
var parents = getParents(elem, '.some-class');
var allParents = getParents(elem.parentNode);

```

同样，如果想从父元素开始查找：
```
var elem = document.querySelector('#example');
var parents = getClosest(elem.parentNode, '[data-sample]');

```

### 向上查找所有匹配元素，直到找到某个匹配父的元素

jQuery 的 `.parentsUntil()` 向上查找DOM树，返回所有父元素，直到某个匹配的父元素被找到。如果指定选择器，则只返回匹配的结果。等价的原生 JavaScript 代码如下：
```
/**
 * Get all of an element's parent elements up the DOM tree until a matching parent is found
 * @param  {Node}   elem     The element
 * @param  {String} parent   The selector for the parent to stop at
 * @param  {String} selector The selector to filter against [optionals]
 * @return {Array}           The parent elements
 */
var getParentsUntil = function ( elem, parent, selector ) {

	// Element.matches() polyfill
	if (!Element.prototype.matches) {
		Element.prototype.matches =
			Element.prototype.matchesSelector ||
			Element.prototype.mozMatchesSelector ||
			Element.prototype.msMatchesSelector ||
			Element.prototype.oMatchesSelector ||
			Element.prototype.webkitMatchesSelector ||
			function(s) {
				var matches = (this.document || this.ownerDocument).querySelectorAll(s),
					i = matches.length;
				while (--i >= 0 && matches.item(i) !== this) {}
				return i > -1;
			};
	}

	// Setup parents array
	var parents = [];

	// Get matching parent elements
	for ( ; elem && elem !== document; elem = elem.parentNode ) {

		if ( parent ) {
			if ( elem.matches( parent ) ) break;
		}

		if ( selector ) {
			if ( elem.matches( selector ) ) {
				parents.push( elem );
			}
			break;
		}

		parents.push( elem );

	}

	return parents;

};

```

用法：

```
var elem = document.querySelector('#some-element');
var parentsUntil = getParentsUntil(elem, '.some-class');
var parentsUntilByFilter = getParentsUntil(elem, '.some-class', '[data-something]');
var allParentsUntil = getParentsUntil(elem);
var allParentsExcludingElem = getParentsUntil(elem.parentNode);

```

## 向下查找 DOM 树

jQuery 的`.find()` 方法提供了简单的方式用选择器向下查找元素。原生 JS 里向下查找其实比向上查找更简单，因为有现成的标准 API 可用，就是`querySelector` 和 `querySelectorAll`

### 向下查找第一个匹配的元素

```
var elem = document.querySelector('#example');
var firstElem = elem.querySelector('.sample-class');

```

###  向下查找所有匹配的元素

```
var elem = document.querySelector('#example');
var allElems = elem.querySelectorAll('[data-sample]');

```

### 总结

可能有人会说，jQuery 不香吗？为什么要自己费劲实现这些简单的方法？没错，jQuery 以前确实很方便，它抹平了浏览器之间的差异，提供了非常好用的 API，让 DOM 操作变得很轻松。但如今浏览器越来越标准化，标准 API 的功能也越来越强大，再加上有 Vue 和 React 这样的框架，实际项目需要手动操作 DOM 的场景越来越少，没必要为了那几个 API 引入一个库。再说，原生 API 操作 DOM 也是合格前端的必备技能，面试经常会问到。你怎么看？

