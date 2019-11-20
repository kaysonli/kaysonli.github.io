---
title: 彻底抛弃 jQuery ，不然还留着过年？
date: 2019-11-17 20:04:54
tags: JavaScript,jQuery
---

我以前很喜欢 jQuery，而且说实话，我是先学jQuery，再学 JavaScript 的。所以我写这篇文章有点像是在背叛 jQuery。

我知道，关于为什么不应该用 jQuery 的文章已经汗牛充栋，但我只是想说下自己的亲身体验。

### 不用 jQuery 用什么？

随着 web 的发展，新技术长江后浪推前浪，前浪死在沙滩上。就像有些编程语言曾经辉煌过，现在也消失不见了。我认为 jQuery 也不例外，是时候跟 它说再见了，虽然它曾经给我们带来过美妙的编程体验。

为什么要放弃 jQuery 呢？因为有 Vue 啊！如果你看过我之前的文章，你应该能猜到我是 Vue.js 粉。Vue.js 提供了非常多的工具，我敢说它比jQuery 方便多了。

### DOM 与事件

让我们来看一个点击事件的例子。
<!-- more -->

*请注意，我省略掉了框架的初始化部分*

一个 Vue SFC（别慌，意思就是Single File Component，单文件组件）：
```
<template>
    <button @click="handleClick">点我，用力</button>
</template>

<script>
    export default {
        methods: {
            handleClick() {
                alert('老铁，你点击了按钮');
            }
        }
    }   
</script>

```

用 jQuery 是这样写的：

```
<button id="myButton">点吧</button>

<script>
    $('button#myButton').click({
        alert('这次用 jQuery');
    });
</script>

```

你可能会觉得 Vue.js 的代码更多啊，你说的没错，但也不对！Vue.js 并不是有更多代码，实际上除了 `handleClick() { ... }` 之外的部分只是框架的结构，跟其他行为是共用的。我觉得 Vue.js 看起来更整洁，代码可读性更高。

你心里可能还有一个疑问，你还是用了 Vue.js 啊，五十步笑百步？有本事别用！实际上你完全可以用原生 JavaScript 实现：
```
<button id="myButton">来点我呀</button>

<script>
    document.getElementById('myButton').addEventListener('click', function() {
       alert('来自原生js的问候'); 
    });
</script>

```

所以你看，jQuery 只是背着我们把代码翻译成原生 JavaScript 而已，假装自己很特别。

至于 DOM 元素的选择操作，用原生 JavaScript 可以轻松做到：
```
document.getElementById('myButton'); // jQuery => $('#myButton');
document.getElementsByClassName('a'); // jQuery => $('.a');
document.querySelectorAll('.parent .a'); // jQuery => $('.parent .a');

```

### AJAX 请求

jQuery 被用得最多的方面可能就是 AJAX 了。
我们都知道 jQuery 提供了 `$.ajax()` ，也可以分别用具体的 `$.post()` 和 `$.get()` 。这些 API 可以帮你发送 AJAX 请求，获取结果等等。

你可以用原生 JavaScript 的两个方法，那就是 `XMLHttpRequest` 和 `fetch`。

`XMLHttpRequest` 也算是个老古董了， 跟 `fetch` 相比还不太一样。

`fetch` 更加时新，也比 `XMLHttpRequest` 更常用，而且是基于 promise 的。

`fetch` 默认不发送 cookies，并且如果 HTTP 状态码返回错误码，比如404或500，它不会 reject，因此基本上 `.catch()` 不会运行，而是 `response.ok` 变成 `false`。

在这里就不详细对比它们了。

我们还是来看两段代码。

 这是 jQuery：

```
$.ajax({
  method: "POST",
  url: "http://localhost/api",
  data: { name: "Adnan", country: "Iran" }
}).done(response => console.log(response))
  .fail(() => console.log('error'));

```

*示例代码来自 jQuery 官方文档*

 这是 fetch：

```
fetch(
    'http://localhost/api',
    {
        method: 'POST'
        body: { name: "Adnan", country: "Iran" }
    }
  ).then(response => console.log(response));

```

两段代码做的事情是一样的，但`fetch` 不属于任何库。

请注意， `fetch` 也可以跟 async/await 结合使用，如下所示：
```
async function getData() {
    let response = await fetch('http://localhost/api' {
        method: 'POST'
        body: { name: "Adnan", country: "Iran" }
    });
    return response;
}

```

那我自己用 `fecth` 吗？实际上没有，因为我不太信任它，原因有很多。因此我在用一个叫 axios 的库，也是基于 promise 的，同时非常可靠。

上面的代码用 `axios` 写是这样的：
```
axios({
    method: 'POST',
    url: 'http://localhost/api',
    data: { name: "Adnan", country: "Iran" }
}).then(response => console.log(response))
  .catch(err => console.log(err));

```

`axios` 也可以跟 async/await 结合使用：

```
async function getData() {
    let response = await axios.post('http://localhost/api' {
        name: "Adnan",
        country: "Iran"
    });
    return response;
}

```

### 总结

我现在自已经不再用 jQuery 了，尽管我曾经非常喜欢它，那个时候项目初始化后的第一件事就是安装 jQuery。

我相信我们已经不再需要 jQuery 了，因为其他库和框架实际上能比 jQuery 更方便、更简洁地完成任务。

可能还有大量的插件基于 jQuery，但基本上都有相应的 Vue.js 或者 React.js 组件替代品。

你们怎么看？欢迎在评论区留言！

