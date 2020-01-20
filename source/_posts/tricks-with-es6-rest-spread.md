---
title: ES6 展开操作符的几个妙用，老板都说好！
date: 2020-01-11 18:00:10
tags: JavaScript
---

[](https://blog.bitsrc.io/6-tricks-with-resting-and-spreading-javascript-objects-68d585bdc83)

ES6 新增了`...`操作符，通常用于在函数中提取剩余参数和展开数组。但其实它的用途不止于此，本文就介绍几个技巧，用它来操作 JavaScript 对象。
# 1\. 添加属性

复制对象的同时，为其添加新的属性。

例子中复制了`user`对象到`userWithPass`，并添加了 `password` 属性。
```
const user = { id: 110, name: 'Kayson Li'}
const userWithPass = { ...user, password: 'Password!' }

user //=> { id: 110, name: 'Kayson Li'}
userWithPass //=> {  id: 110, name: 'Kayson Li', password: 'Password!' }
```
<!-- more -->
# 2\. 合并多个对象

利用`...`可以合并多个对象到一个新的对象。
`part1` 和 `part2` 合并到 `user1`：

```
const part1 = {  id: 110, name: 'Kayson Li' }
const part2 = { id: 110, password: 'Password!' }

const user1 = { ...part1, ...part2 }
//=> { id: 110, name: 'Kayson Li', password: 'Password!' }
```


# 3\. 移除对象属性

可以结合使用解构和剩余操作符删除对象的属性。例子中 `password`被解构出来，其他属性保留在 `rest`对象里并被返回。
```
const noPassword = ({ password, ...rest }) => rest
const user = {
  id: 110,
  name: 'Kayson Li',
  password: 'Password!'
}

noPassword(user) //=> { id: 110, name: 'Kayson Li' }
```

# 4. 动态移除属性

来看一个例子。`removeProperty` 函数接受一个参数 `prop` ，利用[动态属性名](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Object_initializer#New_notations_in_ECMAScript_2015)这个特性，`prop` 可以动态地从拷贝对象中移除。
```
const user1 = {
  id: 110,
  name: 'Kayson Li',
  password: 'Password!'
}
const removeProperty = prop => ({ [prop]: _, ...rest }) => rest
//                     ----       ------
//                          \   /
//                      动态解构

const removePassword = removeProperty('password')
const removeId = removeProperty('id')

removePassword(user1) //=> { id: 110, name: 'Kayson Li' }
removeId(user1) //=> { name: 'Kayson Li', password: 'Password!' }
```

# 5\. 调整属性顺序

有时候对象的属性并不是我们想要的顺序。利用一些技巧，可以将属性移到最前面或最后面。

为了把`id`移到最前面，可以把 `id: undefined` 放在展开 `object`的前面：
```
const user3 = {
  password: 'Password!',
  name: 'Bruce',
  id: 300
}

const organize = object => ({ id: undefined, ...object })
//                            -------------
//                          /
//  移动 id 到第一个属性位置

organize(user3)
//=> { id: 300, password: 'Password!', name: 'Bruce' }
```

要把 `password` 移动到最后位置，先从 `object`中解构出 `password` ，然后把 `password` 放在展开`object`的后面：
```
const user3 = {
  password: 'Password!',
  name: 'Bruce',
  id: 300
}

const organize = ({ password, ...object }) =>
  ({ ...object, password })
//              --------
//             /
// 把 password 移动到最后

organize(user3)
//=> { name: 'Bruce', id: 300, password: 'Password!' }
```

# 6\. 设置属性默认值

当对象不存在某个属性时，我们有时候需要给对象添加这个属性，并设置一个默认值。

例如，`user2`没有 `quotes`属性， `setDefaults` 函数的作用是确保所有对象都有 `quotes` ，并有个默认值`[]`。

执行 `setDefaults(user2)`，返回的对象包含`quotes: []`。

执行  `setDefaults(user4)`，由于`user4`已经有 `quotes`了，那它就保持不变。
```
const user2 = {
  id: 200,
  name: 'Jack Ma'
}

const user4 = {
  id: 400,
  name: '鲁迅',
  quotes: ["我没说过这句话……"]
}

const setDefaults = ({ quotes = [], ...object}) =>
  ({ ...object, quotes })

setDefaults(user2)
//=> { id: 200, name: 'Jack Ma', quotes: [] }

setDefaults(user4)
//=> {
//=>   id: 400,
//=>   name: '鲁迅',
//=>   quotes: ["我没说过这句话……"]
//=> }
```

如果想让这个属性在最前面，可以这样写：
```
const setDefaults = ({ ...object}) => ({ quotes: [], ...object })
```

# 7: 属性重命名

结合前面的几个技巧，我们可以写一个给属性重命名的函数。

假设某些对象都有大写的 `ID`属性，我们想让它们都变成小写，该怎么做呢？先从`object`中解构出`ID`的值，然后再把这个值合并到新对象里，改成小写的`id`：
```
const renamed = ({ ID, ...object }) => ({ id: ID, ...object })

const user = {
  ID: 500,
  name: "张全蛋"
}

renamed(user) //=> { id: 500, name: '张全蛋' }
```

# 8\. 还有更秀的操作

可以根据条件决定是否添加某个属性，这在构造请求参数的时候非常有用。比如，我们只在 `password`有值的情况才添加该属性：
```
const user = { id: 110, name: 'Kayson Li' }
const password = 'Password!'
const userWithPassword = {
  ...user,
  id: 100,
  ...(password && { password })
}

userWithPassword //=> { id: 110, name: 'Kayson Li', password: 'Password!' }
```

# 总结

本文介绍了关于剩余参数和扩展操作符`...`的几个鲜为人知的技巧，希望对你有所帮助。如果你还知道更多的技巧，欢迎在评论区分享！

