---
title: Vue.js 无限滚动列表性能优化方案
date: 2019-12-03 09:46:17
tags: Vue.js
---

### 问题
大家都知道，Web 页面修改 DOM 是开销较大的操作，相比其他操作要慢很多。这是为什么呢？因为每次 DOM 修改，浏览器往往需要重新计算元素布局，再重新渲染。也就是所谓的重排（reflow）和重绘（repaint）。尤其是在页面包含大量元素和复杂布局的情况下，性能会受到影响。那对用户有什么实际的影响呢？

一个常见的场景是大数据量的列表渲染。通常表现为可无限滚动的无序列表或者表格，当数据很多时，页面会出现明显的滚动卡顿，严重影响了用户体验。怎么解决呢？

### 解决方案

既然问题的根源是 DOM 元素太多，那就想办法限制元素数量。
<!-- more -->

- 限制列表对用户可见的元素数量。我们把可见区域称为 ViewPort
- 当列表滚动时，列表种的其他元素怎么由不可见变为可见？
- 监听列表容器元素的滚动事件，当列表里的元素进入可视区域，则添加到DOM中
- 问题是如果一直这么滚下去，列表会越来越大。所以需要在列表元素离开 ViewPort 的时候从DOM中移除
- 问题又来了，由于 ViewPort 刚好是一屏的大小，滚动的时候元素还没来得及渲染，会出现一段时间的空白。解决办法就是上下增加一部分数据渲染。
![无限滚动](/uploads/1618526-e3d68e99ac070746.webp?imageMogr2/auto-orient/strip)

无限滚动的性能优化方案基本思路就是这样。

在实际项目中，我们可能不需要自己从头实现一个无限滚动列表组件，Vue.js 就有一个现成的轮子：[vue-virtual-scroller](https://akryum.github.io/vue-virtual-scroller/)。

在项目中安装这个插件：
```
$ npm install -D vue-virtual-scroller
```
项目入口文件 `main.js` 引入这个插件：
```
import "vue-virtual-scroller/dist/vue-virtual-scroller.css";
import Vue from "vue";
import VueVirtualScroller from "vue-virtual-scroller";

Vue.use(VueVirtualScroller);
```
### 案例一：VirtualList

我们来看一个简单的例子，用`vue-virtual-scroller`渲染一个包含大量数据的列表。
先用 [JSON-Generator](https://www.json-generator.com/) 生成 5000 条数据的 JSON 对象，并保存到 `data.json` 文件。可以用下面的规则：

```
[
  '{{repeat(5000)}}',
  {
    _id: '{{objectId()}}',
    age: '{{integer(20, 40)}}',
    name: '{{firstName()}} {{surname()}}',
    company: '{{company().toUpperCase()}}'
  }
]

```

新建一个 *VirtualList.vue* 文件，引入`data.json`，并将它赋值给组件的`items`属性。然后套一个 `<virtual-scroller>`组件：

VirtualList.vue：

```
<template>
  <virtual-scroller :items="items" item-height="40" content-tag="ul">
    <template slot-scope="props">
      <li :key="props.itemKey">{{props.item.name}}</li>
    </template>
  </virtual-scroller>
</template>

<script>
import items from "./data.json";

export default {
  data: () => ({ items })
};
</script>

```

`virtual-scroller` 组件必须设置 `item-height` 。另外，由于我们要创建一个列表，可以设置`content-tag="ul"`，表示内容渲染成 `<ul>`标签。

 `vue-virtual-scroller` 支持使用 `scoped slots`，增加了内容渲染的灵活性。通过使用`slot-scope="props"`，我们可以访问 `vue-virtual-scroller` 暴露的数据。

`props` 有一个`itemKey`属性，出于性能考虑，我们应该在内容部分的根元素上绑定 `:key="props.itemKey"`。然后我们就可以通过 `props.item` 拿到 JSON 里的原始数据了。

如果你要给列表设置样式，可以给 `virtual-scroller` 设置 `class`属性：
```
<template>
  <virtual-scroller class="virtual-list" ...></virtual-scroller>
</template>

<style>
.virtual-list ul {
  list-style: none;
}
</style>

```

或者也可以用`scoped` 样式，用 `/deep/`选择器：
```
<style scoped>
.virtual-list /deep/ ul {
  list-style: none;
}
</style>

```

### 案例二： VirtualTable

类似 `VirtualList`，我们再看一个表格组件`VirtualTable`：
VirtualTable.vue:

```
<template>
  <virtual-scroller :items="items" item-height="40" content-tag="table">
    <template slot-scope="props">
      <tr :key="props.itemKey">
        <td>{{props.item.age}}</td>
        <td>{{props.item.name}}</td>
        <td>{{props.item.company}}</td>
      </tr>
    </template>
  </virtual-scroller>
</template>

<script>
import items from "./data.json";

export default {
  data: () => ({ items })
};
</script>

```

这里有个小问题，我们需要增加一个 `<thead>`标签，用于显示列名： *Age*, *Name* 和 *Company*

幸好 virtual-scroller 支持 slot，可以定制各部分内容：
```
<main>
  <slot name="before-container"></slot>
  <container>
    <slot name="before-content"></slot>
    <content>
      <!-- Your items here -->
    </content>
    <slot name="after-content"></slot>
  </container>
  <slot name="after-container"></slot>
</main>

```

这些 slot 都可以放置自定义内容。`container` 会被 `container-tag` 属性值替换，默认是`div`，`content` 被 `content-tag` 值替换。

这里用 `before-content` slot 加一个`thead` 就行了：
```
<template>
  <virtual-scroller
    :items="items"
    item-height="40"
    container-tag="table"
    content-tag="tbody"
    >
      <thead slot="before-content">
        <tr>
          <td>Age</td>
          <td>Name</td>
          <td>Company</td>
        </tr>
      </thead>
      <template slot-scope="props">
        <tr :key="props.itemKey">
          <td>{{props.item.age}}</td>
          <td>{{props.item.name}}</td>
          <td>{{props.item.company}}</td>
        </tr>
      </template>
  </virtual-scroller>
</template>

```

请注意，我们把`content-tag="table"` 改成了`content-tag="tbody"`，因为我们设置了`container-tag="table"`，这是为了构造` table` 标签的常规结构。

如果要加一个 `tfoot`，应该知道怎么做了吧。

### 总结
我们了解了无限滚动列表的性能优化原理，以及利用`vue-virtual-scroller` Vue 插件创建了 *VirtualList* 和  *VirtualTable* 组件。如果用它们来展示前面生成的 5000 条数据，应该可以比较流畅地渲染和滚动了。更多用法可以参考 [vue-virtual-scroller 文档](https://github.com/Akryum/vue-virtual-scroller) 。

文章涉及的源码 [戳这里](https://codesandbox.io/s/jnxyxr21lv)。更多技术干货，欢迎关注我的微信公众号：1024译站。
