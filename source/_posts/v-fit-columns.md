---
title: （几乎）完美实现 el-table 列宽自适应
date: 2020-08-28 17:07:15
tags: Vue.js
---

### 背景
Element UI 是 PC 端比较流行的 Vue.js UI 框架，它的组件库基本能满足大部分常见的业务需求。但有时候会有一些定制性比较高的需求，组件本身可能没办法满足。最近在项目里就碰到了。

很多页面都需要用到表格组件`el-table`。如果没有给`el-table-column`指定宽度，默认情况下会平均分配给剩余的列。在列数比较多的情况，如果`el-table`宽度限定在容器内，单元格里的内容就会换行。强制不换行，内容要么在单元格内滚动，要么就会溢出或被截断。

产品想要的效果是：内容保持单行显示，列间距保持一致，表格超出容器允许水平滚动。`el-table-column`是支持设置固定宽度的，在内容宽度可预知的情况下，也能满足这个需求。问题就在于如何让列宽动态适应内容的宽度。在官方文档也没找到这样的选项，应该是组件本身不支持。

### 技术方案
于是想到了动态计算内容宽度的方案。网上也有人提过这个思路，做法是根据内容字符数来计算宽度。这种方法有几个局限：
1. 内容必须是文本
2. 不同字符宽度不一，结算结果不够准确
3. 需要在渲染前操作数据，不利于解耦

我采用了另一种思路，还是动态计算内容宽度，但是根据实际渲染后的 DOM 元素宽度，这样就能解决上面三个问题。
<!-- more -->
具体怎么做呢？通过查看渲染后的 DOM 元素发现，`el-table` 的表头和内容分别用了一个原生`table`，通过`colgroup`设置每列的宽度。就从这里入手，`col`的`name`属性值和对应的 `td`的`class`值是一致的，这样就可以遍历对应列的所有单元格，找出宽度最大的单元格，用它的内容宽度加上一个边距作为该列的宽度。
![](http://ww1.sinaimg.cn/large/6bb8ee92gy1gicf5sm51yj20sq0lowh1.jpg)

### 具体实现
怎么计算内容宽度呢？这是个比较关键的步骤。渲染后的每个单元格有个`.cell`类，用`white-space: nowrap; overflow: auto;`设置为不允许换行，内容超出后可滚动，同时设置`display: inline-block;`以便计算实际内容宽度。这样，最终的宽度可通过`.cell`元素的`scrollWidth`属性得到。

```
function adjustColumnWidth(table) {
  const colgroup = table.querySelector("colgroup");
  const colDefs = [...colgroup.querySelectorAll("col")];
  colDefs.forEach((col) => {
    const clsName = col.getAttribute("name");
    const cells = [
      ...table.querySelectorAll(`td.${clsName}`),
      ...table.querySelectorAll(`th.${clsName}`),
    ];
    // 忽略加了"leave-alone"类的列
    if (cells[0]?.classList?.contains?.("leave-alone")) {
      return;
    }
    const widthList = cells.map((el) => {
      return el.querySelector(".cell")?.scrollWidth || 0;
    });
    const max = Math.max(...widthList);
    const padding = 32;
    table.querySelectorAll(`col[name=${clsName}]`).forEach((el) => {
      el.setAttribute("width", max + padding);
    });
  });
}
```

中间的探索过程比较繁琐，但最终的代码实现却非常简洁。在什么时候触发列宽计算呢？自然是组件渲染完成后。为了方便重用，我采用了 Vue 自定义指令的方式。

```
Vue.directive("fit-columns", {
  update() {},
  bind() {},
  inserted(el) {
    setTimeout(() => {
      adjustColumnWidth(el);
    }, 300);
  },
  componentUpdated(el) {
    el.classList.add("r-table");
    setTimeout(() => {
      adjustColumnWidth(el);
    }, 300);
  },
  unbind() {},
});
```
更进一步，我封装了一个 Vue 插件叫`v-fit-columns`，已经发布到 npm 仓库，直接安装即可使用。
安装：
```
npm install v-fit-columns --save
```
引入：
```
import Vue from 'vue';
import Plugin from 'v-fit-columns';
Vue.use(Plugin);
```
使用：
```
<el-table v-fit-columns>
  <el-table-column label="No." type="index" class-name="leave-alone"></el-table-column>
  <el-table-column label="Name" prop="name"></el-table-column>
  <el-table-column label="Age" prop="age"></el-table-column>
</el-table>

```
源码仓库在这：https://github.com/kaysonli/v-fit-columns ，欢迎各位不吝赐教和 Star！

### 总结
这个方案多少有点 Hack 的意味，只顾实现需求，可能在其他方面还有点瑕疵，比如渲染完后会稍微闪一下（因为要重新调整宽度，会出现 reflow）。不过从最终实现的效果来看，还算令人满意，至少产品经理提着他的两米大刀走了……他可是为了这个效果，蹲守在我电脑前半个下午，非要我实现不可！在手摸手的胁迫下，总算完事交差了……
