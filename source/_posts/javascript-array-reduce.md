---
title: 自从学会了 Array.reduce() ，再也离不开它
date: 2019-12-21 12:00:54
tags: JavaScript
---


在所有后 ES6 时代的数组方法中，我觉得最难理解的就是`Array.reduce()`。

从表面上看，它似乎是一个简单无趣的方法，并没有太大作用。 但是在不起眼的外表之下，`Array.reduce()`实际上是对开发人员工具包的强大而灵活的补充。

今天，我们就来研究一下通过`Array.reduce()`可以完成的一些有意思的事情。
##  原理

大部分现代的数组方法都返回一个新的数组，而 `Array.reduce()` 更加灵活。它可以返回任意值，它的功能就是将一个数组的内容聚合成单个值。

这个值可以是数字、字符串，甚至可以是对象或新数组。这就是一直难住我的部分，我没想到它这么灵活！
<!-- more -->
## 用法

`Array.reduce()`接受两个参数：一个是对数组每个元素执行的回调方法，一个是初始值。

这个回调也接受两个参数：`accumulator`是当前聚合值，`current`是数组循环时的当前元素。无论你返回什么值，都将作为累加器提供给循环中的下一个元素。初始值将作为第一次循环的累加器。
```

var myNewArray = [].reduce(function (accumulator, current) {
  return accumulator;
}, starting);
```

让我们来看几个实际例子。
## 1\. 数组求和

假设你想把一组数字加在一起。使用`Array.forEach()`大概可以这么做：
```

var total = 0;

[1, 2, 3].forEach(function (num) {
  total += num;
});
```

这是`Array.reduce()`用得最多的例子了。我发现* accumulator *这个单词让人困惑，所以在示例中我改为`sum`，因为这里就是求和的意思。
```

var total = [1, 2, 3].reduce(function (sum, current) {
  return sum + current;
}, 0);
```

这里传入`0`作为初始值。

在回调里，将当前值加入到 `sum`，第一轮循环时它的值是初始值`0`，然后变成`1`（初始值`0`加上当前元素值`1`），然后变成`3`（累加值 `1`加上当前元素值 `2` ），以此类推

## 2\. 组合多个数组方法

假设有一个`wizards` 数组：
```

var wizards = [
  {
    name: 'Harry Potter',
    house: 'Gryfindor'
  },
  {
    name: 'Cedric Diggory',
    house: 'Hufflepuff'
  },
  {
    name: 'Tonks',
    house: 'Hufflepuff'
  },
  {
    name: 'Ronald Weasley',
    house: 'Gryfindor'
  },
  {
    name: 'Hermione Granger',
    house: 'Gryfindor'
  }
];
```

你想创建一个仅包含住在 Hufflepuff 的巫师名字的新数组。一个可行的方法是使用`Array.filter()` 方法获取 `house` 属性为`Hufflepuff`的 `wizards` 。然后用`Array.map()` 方法创建一个只包含过滤后对象的`name` 属性的新数组。
```

var hufflepuff = wizards.filter(function (wizard) {
  return wizard.house === 'Hufflepuff';
}).map(function (wizard) {
  return wizard.name;
});
```

使用`Array.reduce()` 方法，我们可以用一步得到同样的结果，提高了性能。传递一个空数组`[]`作为初始值。每次循环时判断`wizard.house` 是否为`Hufflepuff`。如果是，就加入到`newArr` 中（即`accumulator`），否则啥也不做。

无论判断条件是否成立，最后都返回 `newArr` 作为下一次循环的`accumulator` 。
```

var hufflepuff = wizards.reduce(function (newArr, wizard) {
  if (wizard.house === 'Hufflepuff') {
    newArr.push(wizard.name);
  }
  return newArr;
}, []);
```

## 3\. 从数组生成 HTML 标签

那么，如果想创建一个由住在 Hufflepuff 的巫师组成的无序列表要怎么做呢？这次不是给`Array.reduce()`传一个空数组作为初始值了，而是一个名为 `html`的空字符串`''`。

如果`wizard.house` 等于 `Hufflepuff`，我们就将`wizard.name` 用列表项`li`包裹起来，再拼接到`html` 字符串里。然后返回`html` 作为下一次循环的`accumulator` 。
```

var hufflepuffList = wizards.reduce(function (html, wizard) {
  if (wizard.house === 'Hufflepuff') {
    html += '<li>' + wizard.name + '</li>';
  }
  return html;
}, '');
```

在`Array.reduce()`前后添加无序列表的开始和结束标记，就可以把它插入到 DOM 中了。
```

var hufflepuffList = '<ul>' + wizards.reduce(function (html, wizard) {
  if (wizard.house === 'Hufflepuff') {
    html += '<li>' + wizard.name + '</li>';
  }
  return html;
}, '') + '</ul>';
```


## 4\. 数组元素分组

lodash 有个 `groupBy()`方法，可以将数组元素按照某个标准分组。

假设你有一个数字数组。

如果你想把`numbers` 数组中的元素按照整数部分的值分组，用 lodash 可以这样做：
```

var numbers = [6.1, 4.2, 6.3];

// 返回 {'4': [4.2], '6': [6.1, 6.3]}
_.groupBy(numbers, Math.floor);
```

如果你有一个单词数组，你想根据 `words` 中的单词长度分组，你可以这样做：
```

var words = ['one', 'two', 'three'];

// 返回 {'3': ['one', 'two'], '5': ['three']}
_.groupBy(words, 'length');
```

### 用 `Array.reduce()` 实现 `groupBy()`函数

你可以用`Array.reduce()` 方法实现同样的功能。

我们来创建一个工具函数`groupBy()`，接受数组和分组条件作为参数。在`groupBy()`内部，在数组上执行`Array.reduce()` ，传一个空对象`{}`作为初始值，然后返回结果。
```

var groupBy = function (arr, criteria) {
  return arr.reduce(function (obj, item) {
    // 省略代码
  }, {});
};
```

在 `Array.reduce()` 回调函数内部，我们会判断`criteria`是函数还是 `item`的属性。然后获取当前`item`的值。

如果`obj` 中还不存在这个属性，则创建它，并将一个空数组赋值给它。最后，将`item` 添加到 `key`的数组中，再返回该对象作为下一次循环的`accumulator` 。
```

var groupBy = function (arr, criteria) {
  return arr.reduce(function (obj, item) {

    // 判断criteria是函数还是属性名
    var key = typeof criteria === 'function' ? criteria(item) : item[criteria];

    // 如果属性不存在，则创建一个
    if (!obj.hasOwnProperty(key)) {
      obj[key] = [];
    }

    // 将元素加入数组
    obj[key].push(item);

    // 返回这个对象
    return obj;

  }, {});
};
```

## 5\. 合并数据到单个数组

还记得前面的`wizards`数组吗？
```

var wizards = [
  {
    name: 'Harry Potter',
    house: 'Gryfindor'
  },
  {
    name: 'Cedric Diggory',
    house: 'Hufflepuff'
  },
  {
    name: 'Tonks',
    house: 'Hufflepuff'
  },
  {
    name: 'Ronald Weasley',
    house: 'Gryfindor'
  },
  {
    name: 'Hermione Granger',
    house: 'Gryfindor'
  }
];
```

如果还有另一份数据，每个巫师获得的的积分对象：
```

var points = {
  HarryPotter: 500,
  CedricDiggory: 750,
  RonaldWeasley: 100,
  HermioneGranger: 1270
};
```

假设你想把两份数据合并到一个数组，也就是把 `points` 数值添加到每个巫师对象上。你会怎么做？

 `Array.reduce()` 方法特别适合！
```

var wizardsWithPoints = wizards.reduce(function (arr, wizard) {

  // 移除巫师名字中的空格，用来获取对应的 points
  var key = wizard.name.replace(' ', '');

  // 如果wizard有points，则加上它，否则设置为0
  if (points[key]) {
    wizard.points = points[key];
  } else {
    wizard.points = 0;
  }

  // 把wizard对象加入到新数组里
  arr.push(wizard);

  // 返回这个数组
  return arr;

}, []);
```
其实这里用`Array.map`也很方便实现。
## 6\. 合并数据到单个对象

如果你想合并两个来源的数据到一个对象中，也就是巫师的名字作为属性名，house 和 points 作为属性值，要怎么做呢？同样， `Array.reduce()` 很合适。
```

var wizardsAsAnObject = wizards.reduce(function (obj, wizard) {

  // 移除巫师名字中的空格，用来获取对应的 points
  var key = wizard.name.replace(' ', '');

  // 如果wizard有points，则加上它，否则设置为0
  if (points[key]) {
    wizard.points = points[key];
  } else {
    wizard.points = 0;
  }

  // 删除 name 属性
  delete wizard.name;

  // 把 wizard 数据添加到新对象中
  obj[key] = wizard;

  // 返回该对象
  return obj;

}, {});
```

## 总结： `Array.reduce()` 真香

 `Array.reduce()` 方法从我曾经认为不堪大用的东西，变成我最喜欢的 JavaScript 方法。那么，你应该使用它吗？什么时候可以用？

`Array.reduce()` 方法有着良好的浏览器支持。所有的现代浏览器都支持，包括 IE9 及以上。移动端浏览器也在很早之前就支持了。如果你还需要支持更老的浏览器，你可以添加一个 polyfill 来支持到 [IE6](https://vanillajstoolkit.com/polyfills/arrayreduce/)。

 `Array.reduce()`最大的槽点可能就是对于从来没接触过的人来说有点费解。组合使用`Array.filter()` 和`Array.map()`执行起来更慢，并且包含多余的步骤，但是更容易阅读，从方法名可以明显看出它要做的事情。

尽管如此，有时候`Array.reduce()` 也可以让复杂的事情看起来更简单。 `groupBy()` 工具函数就是个很好的例子。

最后，它应该成为你的工具箱里的另一个工具，一个使用得当就威力无穷的工具。
[](https://24ways.org/2019/five-interesting-ways-to-use-array-reduce/)
