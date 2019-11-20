---
title: 花式玩转 JavaScript 数组，看完后我直接删了 lodash
date: 2019-11-18 20:04:54
tags: JavaScript
---

数组是 JavaScript 里面最常见的概念，它为我们处理数据提供了各种可能性。考虑到数组是 JavaScript 中最基本的主题之一，可能在你学习编程之初就接触到了，这篇文章就给你介绍一些你可能不知道的技巧，在日常开发中非常有用！

### 1. 数组去重

这是面试当中经常出现的问题，如何把数组中重复的元素去掉？有一个非常简单快捷的方法，就是利用  `Set()` 对象。这里给你展示两种方式，一个用 `.from()` ，另一个用展开操作符(`…`)。
```
var fruits = [“banana”, “apple”, “orange”, “watermelon”, “apple”, “orange”, “grape”, “apple”];


// 方法一
var uniqueFruits = Array.from(new Set(fruits));
console.log(uniqueFruits); // returns [“banana”, “apple”, “orange”, “watermelon”, “grape”]
// 方法二
var uniqueFruits2 = […new Set(fruits)];
console.log(uniqueFruits2); // returns [“banana”, “apple”, “orange”, “watermelon”, “grape”]
```
很容易吧？
<!-- more -->
### 2. 替换数组中的某些元素

编码过程中有时候需要替换数组中某些值，这里有一个你可能不知道的简便方法，就是利用 `.splice(start, 删除的元素个数, 添加的元素) `，传入三个参数，指定修改的起始位置，以及要删除的元素个数和新的元素。
```
var fruits = [“banana”, “apple”, “orange”, “watermelon”, “apple”, “orange”, “grape”, “apple”];
fruits.splice(0, 2, “potato”, “tomato”);
console.log(fruits); // returns [“potato”, “tomato”, “orange”, “watermelon”, “apple”, “orange”, “grape”, “apple”]
```
### 3. 无需 .map() 也可映射数组

可能每个人都知道数组的 `.map()` 方法，但是还有一种方法可以达到类似的效果，代码也很简洁，那就是 `.from() `方法。
```
var friends = [
    { name: ‘John’, age: 22 },
    { name: ‘Peter’, age: 23 },
    { name: ‘Mark’, age: 24 },
    { name: ‘Maria’, age: 22 },
    { name: ‘Monica’, age: 21 },
    { name: ‘Martha’, age: 19 },
]


var friendsNames = Array.from(friends, ({name}) => name);
console.log(friendsNames); // returns [“John”, “Peter”, “Mark”, “Maria”, “Monica”, “Martha”]
```
### 4. 清空数组

你有一个包含多个元素的数组，出于某种目的，你想清空它，但不想一个一个的删除，怎么办？一行代码搞定，只要把数组的长度设置为`0`，就行了！
```
var fruits = [“banana”, “apple”, “orange”, “watermelon”, “apple”, “orange”, “grape”, “apple”];

fruits.length = 0;
console.log(fruits); // returns []
```
### 5. 将数组转成对象

假设有一个数组，出于某种目的，我们想把这些数据转成对象。最快的方法就是利用著名的展开操作符 (`…`)。
```
var fruits = [“banana”, “apple”, “orange”, “watermelon”];
var fruitsObj = { …fruits };
console.log(fruitsObj); // returns {0: “banana”, 1: “apple”, 2: “orange”, 3: “watermelon”, 4: “apple”, 5: “orange”, 6: “grape”, 7: “apple”}
```
### 6. 用数据填充数组

在某些情况下，当我们创建一个数组时，我们想用一些数据填充它，或者我们需要一个具有相同值的数组，在这种情况下，`.fill()`  方法提供了一种简单而干净的解决方案。
```
var newArray = new Array(10).fill(“1”);
console.log(newArray); // returns [“1”, “1”, “1”, “1”, “1”, “1”, “1”, “1”, “1”, “1”, “1”]
```
### 7. 合并数组

你知道不使用 `.concat() `方法怎样合并多个数组吗？有一个简单的方法，只要一行代码就可以合并任意数量的数组。你可能知道了，展开操作符 (`…`) 在数组操作上非常有用，这里也是。
```
var fruits = [“apple”, “banana”, “orange”];
var meat = [“poultry”, “beef”, “fish”];
var vegetables = [“potato”, “tomato”, “cucumber”];
var food = […fruits, …meat, …vegetables];
console.log(food); // [“apple”, “banana”, “orange”, “poultry”, “beef”, “fish”, “potato”, “tomato”, “cucumber”]
```
### 8. 查找两个数组的交集

这也是在 JavaScript 面试中最常见的问题之一，因为它展示了你对数组方法的熟悉程度和你的逻辑能力。为了找到两个数组的交集，我们将使用前面提到的方法之一，确保我们检查的数组中的值不重复，我们会用到 `.filter` 和`.include`方法。结果将得到两个数组中都包含的值的数组。请看代码:
```
var numOne = [0, 2, 4, 6, 8, 8];
var numTwo = [1, 2, 3, 4, 5, 6];
var duplicatedValues = […new Set(numOne)].filter(item => numTwo.includes(item));
console.log(duplicatedValues); // returns [2, 4, 6]
```
### 9. 从数组中删除假值

我们先定义什么是假值。在 JavaScript 中，假值是 `false`, `0`， `""`，` null`, `NaN`, `undefined`。然后我们就想办法从数组中删除这类值。为此，我们将使用  `.filter()`方法。
```
var mixedArr = [0, “blue”, “”, NaN, 9, true, undefined, “white”, false];
var trueArr = mixedArr.filter(Boolean);
console.log(trueArr); // returns [“blue”, 9, true, “white”]
```
### 10. 从数组中随机取值

有时我们需要从数组中随机选择一个值。要以一种简单、快速、简短的方式创建它，并保持代码整洁，我们可以根据数组长度获得一个随机索引值。来看看代码:
```
var colors = [“blue”, “white”, “green”, “navy”, “pink”, “purple”, “orange”, “yellow”, “black”, “brown”];
var randomColor = colors[(Math.floor(Math.random() * (colors.length)))]
```
### 11. 数组倒序

当我们需要倒转数组时，没有必要通过复杂的循环和函数来创建，有一个简单的数组方法可以为我们做所有的事情，只需一行代码，我们就可以让数组颠倒顺序。来看代码:
```
var colors = [“blue”, “white”, “green”, “navy”, “pink”, “purple”, “orange”, “yellow”, “black”, “brown”];
var reversedColors = colors.reverse();
console.log(reversedColors); // returns [“brown”, “black”, “yellow”, “orange”, “purple”, “pink”, “navy”, “green”, “white”, “blue”]
```
### 12. .lastIndexOf() 方法

在 JavaScript 中，有一个有趣的方法，它允许查找给定元素的最后一次出现的索引。例如，如果我们的数组有重复的值，我们可以找到它最后一次出现的位置。让我们看看代码示例:
```
var nums = [1, 5, 2, 6, 3, 5, 2, 3, 6, 5, 2, 7];
var lastIndex = nums.lastIndexOf(5);
console.log(lastIndex); // returns 9
```
### 13. 数组求和

在 JavaScript 面试中经常出现的又一个问题。也没什么难的，可以在一行代码中使用`.reduce`方法来解决。来看代码:
```
var nums = [1, 5, 2, 6];
var sum = nums.reduce((x, y) => x + y);
console.log(sum); // returns 14
```
### 总结

本文介绍了关于数组的13个技巧，它们是保持代码简洁的得力助手。另外要告诉你的是，JavaScript 还有很多值得研究的技巧，不仅是关于数组的，还包括不同的数据类型。希望本文提供的解决方案能帮你有效提升开发效率。之前 JavaScript 数组的原生方法还没有这么多，有时候我们不得不依赖 lodash 这样的库完成比较复杂的数组操作。随着 ES6 的普及，大部分操作都可以用原生方法了，效率会高很多。你甚至可以删掉 lodash 了。
