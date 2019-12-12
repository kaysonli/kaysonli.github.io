---
title: 又学到一招：巧用CSS 伪类控制表单，太秀了！
date: 2019-12-12 09:50:21
tags: CSS
---

假设你正在做一个简单的搜索表单，用户在输入框中输入关键词，然后点击按钮进行搜索。

大概长这样：
```
<input placeholder="搜...">
<button>搜他丫的</button>
```

这里有个要求，搜索按钮应该只在搜索框有内容时才可点击。毕竟搜索空字符串不太合理。
## JavaScript 方式

用 JavaScript 实现也比较简单：
<!-- more -->
```
inputElement.oninput = function(e) {
    if(inputElement.value != "") {
        activateSearchButton();
    } else {
        deactivateSearchButton();
    }
};
```

## 该 CSS 出场了

作为有追求的前端，怎么能满足于此呢？我们的信条是：能用 CSS 实现的，坚决不用 JavaScript。这里就可以通过 CSS 伪类`:not()` 和 `:placeholder-shown`实现。
*   **`:placeholder-shown`** — 当输入框的placeholder 可见时被激活。输入框为空时，placeholder 可见，输入内容时不可见。
*   **`:not()`** — 选择器取反，也就是选择不符合选择器的元素

因此`:not(:placeholder-shown)` 意思就是当输入框的 placeholder 不可见时应用样式规则，也就是在输入了内容的情况下。

这里还可以结合 CSS  `+` 选择器操作符完成搜索按钮的控制：
```
button {
    display: none;
}

input:not(:placeholder-shown) + button {
    display: block;
}
```

代码的效果是，只有当输入框有内容时按钮才显示。跟 JavaScript 版本效果一样，但是完全不用。
![GIF Video of :not(:placeholder-shown) Demo](/uploads/1618526-65f21e694d015a6b.webp?imageMogr2/auto-orient/strip)

## 再秀一波

我们还可以结合使用`:focus`伪类，实现仅在用户输入内容时才应用样式，请看大屏幕：
```
<input type="text" placeholder="type something">
<p id="otherElement">You are typing</p>
```


```
#otherElement {
    display: none;
}
input:focus:not(:placeholder-shown) + #otherElement {
    display: block;
}
```

![:not(:placeholder-shown) Demo](/uploads/1618526-246b7271f45a8234.webp?imageMogr2/auto-orient/strip)

## 总结

这个小技巧在构建搜索表单、登录表单等类似场景下比较实用，希望对你有所帮助。

请注意，本文开头引用的 JavaScript 代码并不完整，不能直接运行。另外， `:not(:placeholder-shown)`伪类可能不适用于某些浏览器，建议在应用之前先检查浏览器兼容性。



