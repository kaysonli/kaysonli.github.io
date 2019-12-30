---
title: JavaScript 构造树形结构的一种高效算法
date: 2019-12-30 09:27:29
tags:
---

# 引言

我们经常会碰到树形数据结构，比如组织层级、省市县或者动植物分类等等数据。下面是一个树形结构的例子：
![树形结构](/uploads/1618526-afcc0d3ca19a31ce.webp?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在实际应用中，比较常见的做法是将这些信息存储为下面的结构，特别是当存在1对多的父/子节点关系时：
```
const data = [
  { id: 56, parentId: 62 },
  { id: 81, parentId: 80 },
  { id: 74, parentId: null },
  { id: 76, parentId: 80 },
  { id: 63, parentId: 62 },
  { id: 80, parentId: 86 },
  { id: 87, parentId: 86 },
  { id: 62, parentId: 74 },
  { id: 86, parentId: 74 },
];
```

那么，如何将这种对象数组格式转换为层级树的格式呢？其实，利用 JavaScript 对象引用的特性，实现起来会非常简单。它可以不用递归，在O(n)时间内完成。
<!-- more -->
## 术语

为了表述方便，我们先来定义几个术语。我们把数组中的每个元素（也就树形图里的每个圆圈）称为**“节点”**。节点可以是多个节点的**“父节点”**，也可以是某个节点的**“子节点”**。上图中，`节点 86`是`节点 80` 和`节点 87`的“父节点”，`节点 86` 是`节点 74`的子节点。树的最顶部节点称为**“根节点”**。
# 思路

为了构造树形结构，我们需要：
1.  遍历`data`数组
2.  找到当前元素的父元素
3.  在父元素对象上添加一个对该子元素的引用
4. 元素如果没有父元素，那我们就认为它是树的根节点

我们可以看到到，引用被保存在对象树下，这就是为什么我们可以在O(n)时间内完成这个任务！
# 建立 ID-数组索引映射关系

虽然不是必需的，但是这个映射关系可以帮我们快速找到元素的位置，方便找到到父元素的引用。
```
const idMapping = data.reduce((acc, el, i) => {
  acc[el.id] = i;
  return acc;
}, {});
```

映射结果如下，后面你会看到它的用处有多大：
```
{
  56: 0,
  62: 7,
  63: 4,
  74: 2,
  76: 3,
  80: 5,
  81: 1,
  86: 8,
  87: 6,
};
```

# 构造树形结构

现在我们开始构造这个树形结构。遍历这个对象数组，找到每个元素的父元素对象，然后添加对这个元素的引用。现在你应该看到了，这个 `idMapping`用来定位元素的位置多么方便（常数时间）。
```
let root;
data.forEach(el => {
  // 判断根节点
  if (el.parentId === null) {
    root = el;
    return;
  }
  // 用映射表找到父元素
  const parentEl = data[idMapping[el.parentId]];
  // 把当前元素添加到父元素的`children`数组中
  parentEl.children = [...(parentEl.children || []), el];
});
```

完事！用`console.log` 打印 `root` 看下：
```
console.log(root);
```

```
{
  id: 74,
  parentId: null,
  children: [
    {
      id: 62,
      parentId: 74,
      children: [{ id: 56, parentId: 62 }, { id: 63, parentId: 62 }],
    },
    {
      id: 86,
      parentId: 74,
      children: [
        {
          id: 80,
          parentId: 86,
          children: [{ id: 81, parentId: 80 }, { id: 76, parentId: 80 }],
        },
        { id: 87, parentId: 86 },
      ],
    },
  ],
};
```

# 原理

为什么可以这么做呢？这是因为，`data` 数组里的每个元素都是内存里的一个对象引用， `forEach`循环里的`el`变量其实是指向内存里的一个对象，`parentEl`也引用了一个对象。

如果内存中的一个对象引用了一个 children 数组，这些子元素同样可以引用自己的子元素数组，这些关联关系都是通过引用完成的。
# 总结

对象引用是 JavaScript 中最基本的概念之一，需要更多的学习和理解。真正理解这个概念后，既可以避免棘手的 bug，又可以为看似复杂的问题提供相对简单的解决方案。
[](https://typeofnan.dev/an-easy-way-to-build-a-tree-with-object-references/)
