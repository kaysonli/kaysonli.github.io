---
title: 下次面试再问JavaScript怎么实现深拷贝，我就不客气了！
date: 2019-11-03 20:04:54
tags: JavaScript
---

#### 背景
大家都知道，JavaScript 中的基础数据类型，比如 `number`, `boolean`, `string`, `null`, `undefined` 这些类型的变量在赋值的时候会分配独立的内存空间。而复合类型，比如`Object`，这种类型的变量是引用型的，也就是保存内存的引用地址，可能多个变量指向的是同一个内存地址。这样在修改变量的某个属性时，其他变量的属性也跟着变了。

```
// 值类型
const a = 5;
let b = a; 
b = 6;
console.log(b) // 6
console.log(a) // 5

// 引用类型
const person1 = {
  name: 'tom',
};
let person2 = person1;
person2.name = 'jerry';

console.log(person1.name); // jerry
```
因此，这种情况在有的时候会造成数据互相影响，导致意外的结果。这就需要用到对象的深拷贝了。深拷贝的方法有多种，面试的时候也经常会被问到。今天就来总结下，都有哪些常用的深拷贝实现方式。
<!-- more -->

#### 使用嵌套的展开操作符


```
const original = {name: 'Jane', work: {employer: 'Acme'}};
const copy = {name: original.name, work: {...original.work}};

// 拷贝成功
assert.deepEqual(original, copy);
// 确实是深拷贝
assert.ok(original.work !== copy.work);

```

#### 通过 JSON 字符串

是一种取巧的方式，但是非常快捷。为了深拷贝一个对象`original`，先把它转成JSON字符串，然后再解析这个JSON字符串：
```
function jsonDeepCopy(original) {
  return JSON.parse(JSON.stringify(original));
}
const original = {name: 'Jane', work: {employer: 'Acme'}};
const copy = jsonDeepCopy(original);
assert.deepEqual(original, copy);

```

这种方法的显著缺点是，只能复制JSON格式支持的属性名和值。

不支持的属性名和值会直接忽略：

```
assert.deepEqual(
  jsonDeepCopy({
    [Symbol('a')]: 'abc',
    b: function () {},
    c: undefined,
  }),
  {} // empty object
);

```

其他情况会抛出异常：
```
assert.throws(
  () => jsonDeepCopy({a: 123n}),
  /^TypeError: Do not know how to serialize a BigInt$/);

```

#### 实现通用深拷贝

下面是一个通用的深拷贝函数 ：

```
function deepCopy(original) {
  if (Array.isArray(original)) {
    const copy = [];
    for (const [index, value] of original.entries()) {
      copy[index] = deepCopy(value);
    }
    return copy;
  } else if (typeof original === 'object' && original !== null) {
    const copy = {};
    for (const [key, value] of Object.entries(original)) {
      copy[key] = deepCopy(value);
    }
    return copy;
  } else {
    // 基础类型无需拷贝
    return original;
  }
}

```

这个函数处理了三种情况：

*   如果`original`是一个数组，我们就创建一个新数组，并将`original`里的元素深复制进去。
*   如果`original`这一个对象，我们使用类似的方法。
*   如果`original`是原始类型的值，我们什么也不用做。

我们尝试调用一下 `deepCopy()`:

```
const original = {a: 1, b: {c: 2, d: {e: 3}}};
const copy = deepCopy(original);

// 副本和原始值深度相等吗？
assert.deepEqual(copy, original);

// 是否真的复制了所有层级
// (内容相等，对象不同）
assert.ok(copy     !== original);
assert.ok(copy.b   !== original.b);
assert.ok(copy.b.d !== original.b.d);

```

注意，`deepCopy()`只解决了展开操作符的一个问题。其他问题仍然存在：原型没有拷贝，特殊对象只有部分被拷贝，不可枚举属性被忽略，大部分属性特被忽略。

实现通用的完整拷贝几乎是不可能的：并非所有的数据都是树状的，有时候你不需要复制所有的属性等等。
##### 更简洁版的`deepCopy()`  

如果使用 `.map()` 和 `Object.fromEntries()`，前面实现的`deepCopy()` 可以更加简洁：

```
function deepCopy(original) {
  if (Array.isArray(original)) {
    return original.map(elem => deepCopy(elem));
  } else if (typeof original === 'object' && original !== null) {
    return Object.fromEntries(
      Object.entries(original)
        .map(([k, v]) => [k, deepCopy(v)]));
  } else {
    // 原始类型值无需拷贝
    return original;
  }
}

```

#### 在类中实现深拷贝（进阶）

实现类的实例拷贝，通常会用到两种技术：
*   `.clone()` 方法
*   复制构造器

##### `.clone()` 方法

这种技术为需要实例深拷贝的类引入一个`.clone()`方法。它返回`this`的一个深拷贝。下面这个例子展示了三个可以复制的类。
```
class Point {
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }
  clone() {
    return new Point(this.x, this.y);
  }
}
class Color {
  constructor(name) {
    this.name = name;
  }
  clone() {
    return new Color(this.name);
  }
}
class ColorPoint extends Point {
  constructor(x, y, color) {
    super(x, y);
    this.color = color;
  }
  clone() {
    return new ColorPoint(
      this.x, this.y, this.color.clone()); // (A)
  }
}

```

带注释行 A 展示了这种技术的一个重要方面：复合型的实例属性值必须也要递归地复制。

##### 静态的工厂方法

*复制构造器*是一种利用当前类的另一个实例来初始化当前实例的构造器。复制构造器在静态语言中非常流行，比如和C++和Java。你可以通过*静态重载*来提供多个版本的构造器。*静态*的意思是它发生在编译时。

在 JavaScript 中，你可以这样做（虽然不太优雅） ：

```
class Point {
  constructor(...args) {
    if (args[0] instanceof Point) {
      // 复制构造器
      const [other] = args;
      this.x = other.x;
      this.y = other.y;
    } else {
      const [x, y] = args;
      this.x = x;
      this.y = y;
    }
  }
}

```

这个类的使用方式如下：
```
const original = new Point(-1, 4);
const copy = new Point(original);
assert.deepEqual(copy, original);

```
相反，JavaScript 中*静态工厂方法*更合适。（*静态*意味着它是类方法）

下面这个例子中，三个类 `Point`，`Color` 和 `ColorPoint` 各有一个静态工厂方法 `.from()`：

```
class Point {
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }
  static from(other) {
    return new Point(other.x, other.y);
  }
}
class Color {
  constructor(name) {
    this.name = name;
  }
  static from(other) {
    return new Color(other.name);
  }
}
class ColorPoint extends Point {
  constructor(x, y, color) {
    super(x, y);
    this.color = color;
  }
  static from(other) {
    return new ColorPoint(
      other.x, other.y, Color.from(other.color)); // (A)
  }
}

```

带注释行A中，我们再次用到了递归拷贝。

 `ColorPoint.from()` 的用法如下：

```
const original = new ColorPoint(-1, 4, new Color('red'));
const copy = ColorPoint.from(original);
assert.deepEqual(copy, original);
```

下次面试再碰到这个问题，不用跟面试官客气！

