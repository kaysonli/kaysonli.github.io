---
title: 关于 JavaScript 字符串的一个小知识
date: 2020-08-14 17:15:29
tags: JavaScript
---

![](http://ww1.sinaimg.cn/large/6bb8ee92gy1gicfbv580lj20xc0e9hdt.jpg)
说起字符串，我们再熟悉不过了。接触编程的第一个经典任务就是输出字符串：`Hello, world`。但是你知道 JavaScript 字符串在计算机里是怎么表示的吗？

最简单直观但不太准确的的理解就是，字符串就是由英文字母、数字和标点符号等这些字符组成的序列。比如下面这个字符串就是由5个字母和一个感叹号组成的：
```
const message = 'Hello!';
```
同时也可以看出该字符串的字符数是6：
```
const message = 'Hello!';
message.length; // => 6
```
如果字符串是由这些可见字符（也就是 127 个 ASCII 字符） 组成的，这样理解没有问题。但是，一旦碰到不常见的符号，比如一些表情字符😀, 😁, 😈，可能会得到意外的结果：
```
const smile = '😀';
smile.length; // => 2
```
是不是很奇怪？明明只有一个字符，长度怎么会是 2 呢？这是因为，JavaScript 字符串实际上是由编码单元构成的，而不是可见字符序列。
<!-- more -->
[ECMA 262](https://tc39.es/ecma262/#sec-ecmascript-language-types-string-type) 规范里是这么描述 JavaScript 字符串的：
> `String` 类型是由零或多个 16 位无符号整数值组成的有序序列的集合。字符串类型通常用于表示运行中的 ECMAScript 程序中的文本数据，在这种情况下，字符串中的每个元素都被视为 UTF-16 编码单元值。

简单说，JavaScript 字符串就是 UTF-16 编码单元序列，一串数字而已。

一个编码单元就是位于 `0x0000` 和 `0xFFFF` 之间的一个数字，编码单元与字符之间有个对应关系。例如，编码单元 `0x0048` 对应了实际的字符 `H`：
```
const letter = '\u0048';
letter === 'H' // => true
```
如果把一整个字符串`'Hello!'`用编码单元表示就是这样：
```
const message = '\u0048\u0065\u006C\u006C\u006F\u0021';
message === 'Hello!'; // => true
message.length;       // => 6
```
可以看到，这个字符串有6个编码单元，每个编码单元对应一个字符。基本多文种平面 BMP(Basic Multilingual Plane)中的任意一个字符，都可以用一个 UTF-16 编码单元表示。但是，在这个范围以外的字符，就需要 2 个 UTF-16 编码单元来表示了。比如前面提到的笑脸符号，编码是`\uD83D\uDE00`：
```
const smile = '\uD83D\uDE00';
smile === '😀'; // => true
smile.length;  // => 2
```
这两个编码单元是成对存在的，用于表示超出 `0xFFFF` 的字符。不能拆开，否则就变成无法识别的乱码了。另外，这里的`.length`是2，说明这个属性其实是字符串编码单元的个数，而不是字符数。在需要判断字符数量的时候就要注意了，根据`.length`得到的结果是不准确的。那要怎么解决呢？可以用这种办法：

```
const message = 'Hello!';
const smile = '😀';

[...message].length; // => 6
[...smile].length;   // => 1
```
