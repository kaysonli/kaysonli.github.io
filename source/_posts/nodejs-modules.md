---
title: Node.js 模块系统入门
date: 2020-01-07 17:57:04
tags: node.js
---


>在编程领域中，模块是自包含的功能单元，可以跨项目共享和重用。它们使开发人员的生活更加轻松，因为我们可以使用它来增加应用程序的功能，而不必亲自编写这些功能。它还让我们可以组织和解耦代码，从而使应用程序更容易理解、调试和维护。

在本文中，我们来探究如何使用 Node.js 中的模块，主要介绍如何导出和导入。
## 不同的模块格式

由于JavaScript 最初没有模块的概念，随着时间的推移出现了各种相互竞争的格式。以下是主流的几种格式：
*   [Asynchronous Module Definition (AMD)](https://en.wikipedia.org/wiki/Asynchronous_module_definition)格式， 用于浏览器端，使用 `define` 函数定义模块。
*   [CommonJS (CJS)](https://en.wikipedia.org/wiki/CommonJS) 格式，用于 Node.js，使用 `require` 和 `module.exports` 定义依赖和模块。
*    [ES Module (ESM)](https://www.sitepoint.com/understanding-es6-modules/) 格式。从 ES6 (ES2015)开始，JavaScript 支持原生模块格式。它使用`export` 关键字导出模块的公开 API，并用 `import` 关键字导入。
*    [System.register](https://github.com/systemjs/systemjs/blob/master/docs/system-register.md) 格式被设计用于在ES5 中 支持 ES6 模块。
*    [Universal Module Definition (UMD)](https://riptutorial.com/javascript/example/16339/universal-module-definition--umd-) 格式，在浏览器和 Node.js 中都可以使用。当模块需要被多种模块加载程序导入时，这个很有用。

请注意，本文只讨论 **CommonJS 格式**，它是Node.js 中的标准格式。如果你想深入了解其他格式，我推荐你阅读[这篇文章](https://www.jvandemo.com/a-10-minute-primer-to-javascript-modules-module-formats-module-loaders-and-module-bundlers/)，作者是 SitePoint 的 Jurgen Van de Moere。
<!-- more -->
## 加载模块
Node.js 有一系列的[内置模块](https://www.w3schools.com/nodejs/ref_modules.asp)，我们在代码中无需安装即可使用。为此，我们需要使用`require`关键字加载这个模块，并把它赋值给一个变量。这样就可以调用模块暴露的任何方法了。

例如，要列出目录的内容，你可以使用[文件系统模块](https://nodejs.org/api/fs.html) 的 `readdir` 方法：
```
const fs = require('fs');
const folderPath = '/home/jim/Desktop/';

fs.readdir(folderPath, (err, files) => {
  files.forEach(file => {
    console.log(file);
  });
});

```

注意，在 CommonJS 里，模块是按照出现的顺序同步加载和处理的。
## 创建和导出模块

现在我们来看看如何创建模块并导出，以便在程序的其他地方使用。先创建一个 `user.js` 文件，并添加如下内容。

```
const getName = () => {
  return 'Jim';
};

exports.getName = getName;

```

然后在同一个目录中创建一个 `index.js` 文件，添加以下内容：
```
const user = require('./user');
console.log(`User: ${user.getName()}`);

```

用命令`node index.js` 运行程序，你应该会在控制台看到如下输出：
```
User: Jim

```

这里发生了什么？如果你查看一下`user.js`文件，你会注意到我们定义了一个`getName`函数，然后使用`exports`关键字使其可以在其他地方导入。然后是 `index.js`文件，我们导入了这个方法并执行它。还要注意的是在`require` 语句中，模块名字加了前缀 `./`，因为它是本地文件。另外就是不需要加文件扩展名。
### 导出多个方法和值

我们可以用同样的方式导出多个方法和值：
```
const getName = () => {
  return 'Jim';
};

const getLocation = () => {
  return 'Munich';
};

const dateOfBirth = '12.01.1982';

exports.getName = getName;
exports.getLocation = getLocation;
exports.dob = dateOfBirth;

```

 `index.js` 文件：

```
const user = require('./user');
console.log(
  `${user.getName()} lives in ${user.getLocation()} and was born on ${user.dob}.`
);

```

上面的代码结果如下：
```
Jim lives in Munich and was born on 12.01.1982.

```

注意，我们为导出的`dateOfBirth`变量指定的名称可以是任何我们想要的名称(本例中为`dob`)。它不必与原来的变量名相同。
### 多种语法形式

还需要提到的是，你可以在中途导出方法和值，并不一定要在文件末尾。
例如：

```
exports.getName = () => {
  return 'Jim';
};

exports.getLocation = () => {
  return 'Munich';
};

exports.dob = '12.01.1982';

```

另外要感谢[解构赋值](https://www.sitepoint.com/es6-destructuring-assignment/)，我们可以根据需要选择性导入：
```
const { getName, dob } = require('./user');
console.log(
  `${getName()} was born on ${dob}.`
);

```

如你所料，这会输出日志：
```
Jim was born on 12.01.1982.

```

## 导出默认值

在上面的例子中，我们分别导出了函数和值。这对于整个应用程序都需要的辅助函数来说是很方便的，但是当你的模块只导出单个对象时，通常使用`module.exports`：
```
class User {
  constructor(name, age, email) {
    this.name = name;
    this.age = age;
    this.email = email;
  }

  getUserStats() {
    return `
      Name: ${this.name}
      Age: ${this.age}
      Email: ${this.email}
    `;
  }
}

module.exports = User;

```

 `index.js`文件：

```
const User = require('./user');
const jim = new User('Jim', 37, 'jim@example.com');

console.log(jim.getUserStats());

```

上面的代码输出日志：
```
Name: Jim
Age: 37
Email: jim@example.com

```

##  `module.exports` 和 `exports` 的区别是什么

你在网上可能会遇到以下语法：
```
module.exports = {
  getName: () => {
    return 'Jim';
  },

  getLocation: () => {
    return 'Munich';
  },

  dob: '12.01.1982',
};

```

这里我们把要导出的函数和值赋给 `module` 的 `exports` 属性——当然这也是可以的：
```
const { getName, dob } = require('./user');
console.log(
  `${getName()} was born on ${dob}.`
);

```

输出日志如下：
```
Jim was born on 12.01.1982.

```

那么， `module.exports`和 `exports`之间的区别到底是什么？后者只是个别名吗？

嗯，有一点，但不完全是。

为了说明我的意思，让我们更改`index.js` 中的代码，输出`module`的值：
```
console.log(module);

```

输出结果为：

```
Module {
  id: '.',
  exports: {},
  parent: null,
  filename: '/home/jim/Desktop/index.js',
  loaded: false,
  children: [],
  paths:
   [ '/home/jim/Desktop/node_modules',
     '/home/jim/node_modules',
     '/home/node_modules',
     '/node_modules' ] }

```

可以看到，`module` 有一个`exports` 属性。让我们再加点东西：
```
// index.js
exports.foo = 'foo';
console.log(module);

```

这会输出：

```
Module {
  id: '.',
  exports: { foo: 'foo' },
  ...

```

给`exports` 添加属性，也会添加到 `module.exports`。这是因为`exports`是`module.exports`的一个引用。
### 我应该用哪个？

既然 `module.exports` 和 `exports` 都指向同一个对象，你使用哪一个通常无关紧要。例如：
```
exports.foo = 'foo';
module.exports.bar = 'bar';

```

这段代码将会使模块的导出对象变成`{ foo: 'foo', bar: 'bar' }`。

不过，这里有个需要注意的地方。你赋值给`module.exports`的内容将成为模块导出的值。

举例如下：
```
exports.foo = 'foo';
module.exports = () => { console.log('bar'); };

```

这样只会导出一个匿名函数。`foo`变量将被忽略。

如果你想了解更多，我推荐你阅读 [这篇文章](https://www.hacksparrow.com/nodejs/exports-vs-module-exports.html)。
## 总结

模块已经成为 JavaScript 生态系统不可分割的一部分，它让我们能用更小的部分组成大型程序。我希望本文为你在 Node.js 中使用模块做了良好介绍，以及帮助你了解模块语法。
> 作者：James Hibbard
来源：[SitePoint](https://www.sitepoint.com/understanding-module-exports-exports-node-js/)
翻译：1024译站

