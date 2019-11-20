---
title: 顶层 await 释疑
date: 2019-10-15 20:04:54
tags: JavaScript
---

[顶层`await`](https://github.com/tc39/proposal-top-level-await)允许开发人员在异步函数之外使用`await`关键字。它就像一个大的异步函数，会让导入它们的其他模块在开始解析主体代码之前等待。

## 原有行为

当 `async`/`await` 首次被提出时，试图在 `async`函数之外使用`await` 会导致`SyntaxError`。许多开发人员利用立即调用的异步函数表达式来使用该特性。
```
await Promise.resolve(console.log('🎉'));
// → SyntaxError: await 只在 async 函数中有效

(async function() {
  await Promise.resolve(console.log('🎉'));
  // → 🎉
}());
```
<!-- more -->
## 新的行为

使用顶层`await`，上面的代码将按照在[模块](https://v8.dev/features/modules)中期望的方式工作:
```
await Promise.resolve(console.log('🎉'));// → 🎉
```

**注意：**顶层的`await`*只能*在模块的顶层作用域使用。不支持在传统脚本或非异步函数中运行。

## 用例

这些用例来自于 [规范提议代码仓库](https://github.com/tc39/proposal-top-level-await#use-cases)。

### 动态依赖路径

```
const strings = await import(`/i18n/${navigator.language}`);
```

这允许模块使用运行时的值来确定依赖项。这对于开发环境和生产环境区分、国际化、运行环境区分等非常有用。

### 资源初始化

```
const connection = await dbConnector();
```
这允许模块描述资源，并在无法使用模块的情况下产生错误。

### 依赖项备选方案

下面的示例尝试从CDN A加载JavaScript库，如果失败则返回到CDN B：
```
let jQuery;
try {
  jQuery = await import('https://cdn-a.example.com/jQuery');
} catch {
  jQuery = await import('https://cdn-b.example.com/jQuery');
}
```

## 模块执行顺序

顶层`await` 对JavaScript最大的变化之一是模块的执行顺序。JavaScript引擎采用[后序遍历](https://en.wikibooks.org/wiki/A-level_Computing/AQA/Paper_1/Fundamentals_of_algorithms/Tree_traversal#后序)方式执行模块：从模块树最左边的子树开始，模块被解析执行，数据绑定被导出，接着是兄弟模块被解析执行，然后执行父模块。此算法递归地运行，直到它执行完模块树的根。

在顶层 `await`之前，这个顺序总是同步和确定的：多次运行代码，模块树的执行顺序一定是相同的。一旦有了顶层`await`，只有在不使用顶层`await`的情况下才能保证这个执行顺序。

下面是在模块中使用顶层 `await`时的情况:
1.  当前模块的执行将被延迟，直到解决所等待的 Promise。
2.  父模块的执行被延迟，直到调用了`await`的子模块及其所有兄弟模块导出绑定之后。
3.  兄弟模块和父模块的兄弟模块能够以相同的同步顺序继续执行——假设依赖树中没有循环依赖或其他处于`await`中的Promise。
4.  调用 `await`的模块在 `await`的Promise解决后继续执行。
5.  只要没有其他的 `await`中的Promise，父模块和后续树将继续以同步顺序执行。

## 开发工具不是已经支持了吗？

的确如此！[Chrome DevTools REPL](https://developers.google.com/web/updates/2017/08/devtools-release-notes#await)，[Node.js](https://github.com/nodejs/node/issues/13209), 以及Safari Web Inspector 已经支持顶层`await`有一段时间了。但是，这个功能是非标准化的，并且只限于在 REPL中！它与顶层`await` 提案不同，后者是语言规范的一部分，只适用于模块。要想以完全符合spec语义的方式测试依赖于顶层`await` 的生产代码，请确保在实际的应用程序中测试，而不仅仅是在DevTools或Node.js REPL中测试!

##顶层 `await` 是不是鸡肋？

也许你已经看过这个[臭名昭著的gist](https://gist.github.com/richharris/0b6f317657f5167663b493c722647221) ，发布者是[Rich Harris](https://twitter.com/Rich_Harris)，它最初列出了对顶层`await`的一些担忧，并敦促JavaScript语言不要实现该特性。一些具体关心的问题是:
*   顶层 `await` 会阻塞代码执行
*   顶层 `await` 会阻塞资源获取
*   CommonJS模块没有明确的互操作方案。

stage 3 版本的提案直接处理了这些问题：

*   由于兄弟模块能够执行，所以不存在阻塞。
*   顶层`await` 发生在模块树的执行阶段。此时，所有资源都已经被获取和链接。没有阻塞资源的风险。
*   顶层`await` 只限于在模块中使用。明确表示不支持普通脚本或CommonJS模块。

与任何新的语言特性一样，总是存在意外行为的风险。例如，对于顶层`await`，循环模块依赖关系可能会导致死锁。

没有顶层 `await`，JavaScript开发人员经常使用异步立即调用的函数表达式来访问 `await`。不幸的是，这种模式导致了代码树执行的不确定性和影响应用程序的静态可分析性。由于这些原因，缺乏顶层`await`被视为比引入该功能风险更大。
## 顶层`await`支持情况


[详见 https://v8.dev/features/support](https://v8.dev/features/support)

