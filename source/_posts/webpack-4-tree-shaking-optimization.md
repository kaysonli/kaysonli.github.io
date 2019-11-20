---
title: Webpack 4 Tree Shaking 终极优化指南
date: 2019-11-15 18:07:28
tags: Webpack
---

### 简介

几个月前，我的任务是将我们组的 React 构建配置升级到 Webpack 4。我们的主要目标之一是利用 tree-shaking 的优势，即 Webpack 去掉了实际上并没有使用的代码来减少包的大小。现在，tree-shaking 的好处将根据你的代码库而有所不同。由于我们的几个架构决策，我们从公司内部的其他库中提取了大量代码，而我们只使用了其中的一小部分。

我写这篇文章是因为恰当地优化 Webpack 并不简单。一开始我以为这是一种简单的魔法，但后来我花了一个月的时间在网上搜索我遇到的一系列问题的答案。我希望通过这篇文章，其他人会更容易地处理类似问题。

### 先说好处

在讨论技术细节之前，让我先总结一下好处。不同的应用程序将看到不同程度的好处。主要的决定因素是应用程序中死代码的数量。如果你没有多少死代码，那么你就看不到 tree-shaking 的多少好处。我们项目里有**很多**死代码。

在我们部门，最大的问题是共享库的数量。从简单的自定义组件库，到企业标准组件库，再到莫名其妙地塞到一个库中的大量代码。很多都是技术债务，但一个大问题是我们所有的应用程序都在导入所有这些库，而实际上每个应用程序都只需要其中的一小部分

总的来说，一旦实现了 tree-shaking，我们的应用程序就会根据应用程序的不同，缩减率从25%到75%。平均缩减率为52%，主要是由这些庞大的共享库驱动的，它们是小型应用程序中的主要代码。

同样，具体情况会有所不同，但是如果你觉得你打的包中可能有很多不需要的代码，这就是如何消除它们的方法。
<!-- more -->

### 没有示例代码仓库

对不住了各位老铁，我做的项目是公司的财产，所以我不能分享代码到 GitHub 仓库了。但是，我将在本文中提供简化的代码示例来说明我的观点。

因此，废话少说，让我们来看看如何编写可实现 tree-shaking 的最佳 webpack 4 配置。

### 什么是死代码

很简单：就是 Webpack 没看到你使用的代码。Webpack 跟踪整个应用程序的 import/export 语句，因此，如果它看到导入的东西最终没有被使用，它会认为那是“死代码”，并会对其进行 tree-shaking 。

死代码并不总是那么明确的。下面是一些死代码和“活”代码的例子，希望能让你更明白。请记住，在某些情况下，Webpack 会将某些东西视为死代码，尽管它实际上并不是。请参阅《副作用》一节，了解如何处理。
```
// 导入并赋值给 JavaScript 对象，然后在下面的代码中被用到
// 这会被看作“活”代码，不会做 tree-shaking
import Stuff from './stuff';
doSomething(Stuff);
// 导入并赋值给 JavaScript 对象，但在接下来的代码里没有用到
// 这就会被当做“死”代码，会被 tree-shaking
import Stuff from './stuff';
doSomething();
// 导入但没有赋值给 JavaScript 对象，也没有在代码里用到
// 这会被当做“死”代码，会被 tree-shaking
import './stuff';
doSomething();
// 导入整个库，但是没有赋值给 JavaScript 对象，也没有在代码里用到
// 非常奇怪，这竟然被当做“活”代码，因为 Webpack 对库的导入和本地代码导入的处理方式不同。
import 'my-lib';
doSomething();
```

### 用支持 tree-shaking 的方式写 import

在编写支持 tree-shaking 的代码时，导入方式非常重要。你应该避免将整个库导入到单个 JavaScript 对象中。当你这样做时，你是在告诉 Webpack 你需要整个库， Webpack 就不会摇它。

以流行的库 Lodash 为例。一次导入整个库是一个很大的错误，但是导入单个的模块要好得多。当然，Lodash 还需要其他的步骤来做 tree-shaking，但这是个很好的起点。
```
// 全部导入 (不支持 tree-shaking)
import _ from 'lodash';
// 具名导入(支持 tree-shaking)
import { debounce } from 'lodash';
// 直接导入具体的模块 (支持 tree-shaking)
import debounce from 'lodash/lib/debounce';
```

### 基本的 Webpack 配置

使用 Webpack 进行 tree-shaking 的第一步是编写 Webpack 配置文件。你可以对你的 webpack 做很多自定义配置，但是如果你想要对代码进行 tree-shaking，就需要以下几项。

首先，你必须处于生产模式。Webpack 只有在压缩代码的时候会 tree-shaking，而这只会发生在生产模式中。

其次，必须将优化选项 “usedExports” 设置为true。这意味着 Webpack 将识别出它认为没有被使用的代码，并在最初的打包步骤中给它做标记。

最后，你需要使用一个支持删除死代码的压缩器。这种压缩器将识别出 Webpack 是如何标记它认为没有被使用的代码，并将其剥离。TerserPlugin 支持这个功能，推荐使用。

下面是 Webpack 开启  tree-shaking 的基本配置：
```
// Base Webpack Config for Tree Shaking
const config = {
 mode: 'production',
 optimization: {
  usedExports: true,
  minimizer: [
   new TerserPlugin({...})
  ]
 }
};
```

### 有什么副作用

仅仅因为 Webpack 看不到一段正在使用的代码，并不意味着它可以安全地进行 tree-shaking。有些模块导入，只要被引入，就会对应用程序产生重要的影响。一个很好的例子就是全局样式表，或者设置全局配置的JavaScript 文件。

Webpack 认为这样的文件有“副作用”。具有副作用的文件不应该做 tree-shaking，因为这将破坏整个应用程序。Webpack 的设计者清楚地认识到不知道哪些文件有副作用的情况下打包代码的风险，因此默认地将所有代码视为有副作用。这可以保护你免于删除必要的文件，但这意味着 Webpack 的默认行为实际上是不进行 tree-shaking。

幸运的是，我们可以配置我们的项目，告诉 Webpack 它是没有副作用的，可以进行 tree-shaking。

### 如何告诉 Webpack 你的代码是无副作用的

`package.json` 有一个特殊的属性 `sideEffects`，就是为此而存在的。它有三个可能的值：

`true` 是默认值，如果不指定其他值的话。这意味着所有的文件都有副作用，也就是没有一个文件可以 tree-shaking。

`false` 告诉 Webpack 没有文件有副作用，所有文件都可以 tree-shaking。

第三个值 `[…]` 是文件路径数组。它告诉 webpack，除了数组中包含的文件外，你的任何文件都没有副作用。因此，除了指定的文件之外，其他文件都可以安全地进行 tree-shaking。

每个项目都必须将 `sideEffects` 属性设置为 `false` 或文件路径数组。在我公司的工作中，我们的基本应用程序和我提到的所有共享库都需要正确配置 `sideEffects` 标志。

下面是 `sideEffects` 标志的一些代码示例。尽管有 JavaScript 注释，但这是 JSON 代码：
```
// 所有文件都有副作用，全都不可 tree-shaking
{
 "sideEffects": true
}
// 没有文件有副作用，全都可以 tree-shaking
{
 "sideEffects": false
}
// 只有这些文件有副作用，所有其他文件都可以 tree-shaking，但会保留这些文件
{
 "sideEffects": [
  "./src/file1.js",
  "./src/file2.js"
 ]
}
```

### 全局 CSS 与副作用

首先，让我们在这个上下文中定义全局 CSS。全局 CSS 是直接导入到 JavaScript 文件中的样式表(可以是CSS、SCSS等)。它没有被转换成 CSS 模块或任何类似的东西。基本上，import 语句是这样的：
```
// 导入全局 CSS
import './MyStylesheet.css';
```

因此，如果你做了上面提到的副作用更改，那么在运行 webpack 构建时，你将立即注意到一个棘手的问题。以上述方式导入的任何样式表现在都将从输出中删除。这是因为这样的导入被 webpack 视为死代码，并被删除。

幸运的是，有一个简单的解决方案可以解决这个问题。Webpack 使用它的模块规则系统来控制各种类型文件的加载。每种文件类型的每个规则都有自己的 `sideEffects` 标志。这会覆盖之前为匹配规则的文件设置的所有 `sideEffects` 标志。

所以，为了保留全局 CSS 文件，我们只需要设置这个特殊的 `sideEffects ` 标志为 `true`，就像这样:
```
// 全局 CSS 副作用规则相关的 Webpack 配置
const config = {
 module: {
  rules: [
   {
    test: /regex/,
    use: [loaders],
    sideEffects: true
   }
  ]
 } 
};
```

Webpack 的所有模块规则上都有这个属性。处理全局样式表的规则必须用上它，包括但不限于 CSS/SCSS/LESS/等等。

### 什么是模块，模块为什么重要

现在我们开始进入秘境。表面上看，编译出正确的模块类型似乎是一个简单的步骤，但是正如下面几节将要解释的，这是一个会导致许多复杂问题的领域。这是我花了很长时间才弄明白的部分。

首先，我们需要了解一下模块。多年来，JavaScript 已经发展出了在文件之间以“模块”的形式有效导入/导出代码的能力。有许多不同的 JavaScript 模块标准已经存在了多年，但是为了本文的目的，我们将重点关注两个标准。一个是 “commonjs”，另一个是 “es2015”。下面是它们的代码形式：
```
// Commonjs
const stuff = require('./stuff');
module.exports = stuff;

// es2015 
import stuff from './stuff';
export default stuff;
```

默认情况下，Babel 假定我们使用 es2015 模块编写代码，并转换 JavaScript 代码以使用 commonjs 模块。这样做是为了与服务器端 JavaScript 库的广泛兼容性，这些 JavaScript 库通常构建在 NodeJS 之上(NodeJS 只支持 commonjs 模块)。但是，Webpack **不支持**使用 commonjs 模块来完成 tree-shaking。

现在，有一些插件(如 common-shake-plugin)声称可以让 Webpack 有能力对 commonjs 模块进行 tree-shaking，但根据我的经验，这些插件要么不起作用，要么在 es2015 模块上运行时，对 tree-shaking 的影响微乎其微。我不推荐这些插件。

因此，为了进行 tree-shaking，我们需要将代码编译到 es2015 模块。

### 为 es2015 模块配置 Babel

据我所知，Babel 不支持将其他模块系统编译成 es2015 模块。但是，如果你是前端开发人员，那么你可能已经在使用 es2015 模块编写代码了，因为这是全面推荐的方法。

因此，为了让我们编译的代码使用 es2015 模块，我们需要做的就是告诉 babel 不要管它们。为了实现这一点，我们只需将以下内容添加到我们的 babel.config.js 中(在本文中，你会看到我更喜欢JavaScript 配置而不是 JSON 配置)：
```
// es2015 模块的基本 Babel 配置
const config = {
 presets: [
  [
   '[@babel/preset-env](http://twitter.com/babel/preset-env)',
   {
    modules: false
   }
  ]
 ]
};
```

把 `modules` 设置为 `false`，就是告诉 babel 不要编译模块代码。这会让 Babel 保留我们现有的 es2015 import/export 语句。

**划重点：**所有可需要 tree-shaking 的代码必须以这种方式编译。因此，如果你有要导入的库，则必须将这些库编译为 es2015 模块以便进行 tree-shaking 。如果它们被编译为 commonjs，那么它们就不能做 tree-shaking ，并且将会被打包进你的应用程序中。许多库支持部分导入，lodash 就是一个很好的例子，它本身是 commonjs 模块，但是它有一个 lodash-es 版本，用的是 es2015模块。

此外，如果你在应用程序中使用内部库，也必须使用 es2015 模块编译。为了减少应用程序包的大小，必须将所有这些内部库修改为以这种方式编译。
### 然而 Jest 又跪了

其他测试框架情况类似，我们用的是 Jest。

不管怎么样，如果你走到了这一步，你会发现 Jest 测试开始失败了。你会像我当时一样，看到日志里出现各种奇怪的错误，慌的一批。别慌，我会带你一步一步解决。

出现这个结果的原因很简单：NodeJS。Jest 是基于 NodeJS 开发的，而 NodeJS 不支持 es2015 模块。为此有一些方法可以配置 Node，但是在 jest 上行不通。因此，我们卡在这里了：Webpack 需要 es2015 进行 tree shaking，但是 Jest 无法在这些模块上执行测试。

就是为什么我说进入了模块系统的“秘境”。这是整个过程中耗费我最多时间来搞清楚的部分。建议你仔细阅读这一节和后面几节，因为我会给出解决方案。

解决方案有两个主要部分。第一部分针对项目本身的代码，也就是跑测试的代码。这部分比较容易。第二部分针对库代码，也就是来自其他项目，被编译成 es2015 模块并引入到当前项目的代码。这部分比较复杂。

# 解决项目本地 Jest 代码

针对我们的问题，babel 有一个很有用的特性：环境选项。通过配置可以运行在不同环境。在这里，开发和生产环境我们需要 es2015 模块，而测试环境需要 commonjs 模块。还好，Babel 配置起来非常容易：
```
// 分环境配置Babel 
const config = {
 env: {
  development: {
   presets: [
    [
     '[@babel/preset-env](http://twitter.com/babel/preset-env)',
     {
      modules: false
     }
    ]
   ]
  },
  production: {
   presets: [
    [
     '[@babel/preset-env](http://twitter.com/babel/preset-env)',
     {
      modules: false
     }
    ]
   ]
  },
  test: {
   presets: [
    [
     '[@babel/preset-env](http://twitter.com/babel/preset-env)',
     {
      modules: 'commonjs'
     }
    ]
   ],
   plugins: [
    'transform-es2015-modules-commonjs' // Not sure this is required, but I had added it anyway
   ]
  }
 }
};
```

设置好之后，所有的项目本地代码能够正常编译，Jest 测试能运行了。但是，使用 es2015 模块的第三方库代码依然不能运行。

### 解决 Jest 中的库代码

库代码运行出错的原因非常明显，看一眼node_modules 目录就明白了。这里的库代码用的是 es2015 模块语法，为了进行 tree-shaking。这些库已经采用这种方式编译过了，因此当 Jest 在单元测试中试图读取这些代码时，就炸了。注意到没有，我们已经让 Babel 在测试环境中启用 commonjs 模块了呀，为什么对这些库不起作用呢？这是因为，Jest (尤其是 babel-jest) 在跑测试之前编译代码的时候，默认忽略任何来自node_modules 的代码。

这实际上是件好事。如果 Jest 需要重新编译所有库的话，将会大大增加测试处理时间。然而，虽然我们不想让它重新编译所有代码，但我们希望它重新编译使用 es2015 模块的库，这样才能在单元测试里使用。

幸好，Jest 在它的配置中为我们提供了解决方案。我想说，这部分确实让我想了很久，并且我感觉没必要搞得这么复杂，但这是我能想到的唯一解决方案。

###  配置 Jest 重新编译库代码

```
// 重新编译库代码的 Jest 配置 
const path = require('path');
const librariesToRecompile = [
 'Library1',
 'Library2'
].join('|');
const config = {
 transformIgnorePatterns: [
  `[\\\/]node_modules[\\\/](?!(${librariesToRecompile})).*$`
 ],
 transform: {
  '^.+\.jsx?$': path.resolve(__dirname, 'transformer.js')
 }
};
```

以上配置是 Jest 重新编译你的库所需要的。有两个主要部分，我会一一解释。

`transformIgnorePatterns` 是 Jest 配置的一个功能，它是一个正则字符串数组。任何匹配这些正则表达式的代码，都不会被 babel-jest 重新编译。默认是一个字符串“node_modules”。这就是为什么Jest 不会重新编译任何库代码。

当我们提供了自定义配置，就是告诉 Jest 重新编译的时候如何忽略代码。也就是为什么你刚才看到的变态的正则表达式有一个负向先行断言在里面，目的是为了匹配除了库以外的所有代码。换句话说，我们告诉 Jest 忽略 node_modules 中除了指定库之外的所有代码。

这又一次证明了 JavaScript 配置比 JSON 配置要好，因为我可以轻松地通过字符串操作，往正则表达式里插入库名字的数组拼接。

第二个是 transform 配置，他指向一个自定义的 babel-jest 转换器。我不敢100%确定这个是必须的，但我还是加上了。设置它用于在重新编译所有代码时加载我们的 Babel 配置。
```
// Babel-Jest 转换器
const babelJest = require('babel-jest');
const path = require('path');
const cwd = process.cwd();
const babelConfig = require(path.resolve(cwd, 'babel.config'));
module.exports = babelJest.createTransformer(babelConfig);
```

这些都配置好后，你在测试代码应该又能跑了。记住了，任何使用库的 es2015 模块都需要这样配置，不然测试代码跑不动。

### Npm/Yarn Link 就是魔鬼

接下来轮到另一个痛点了：链接库。使用 npm/yarn 链接的过程就是创建一个指向本地项目目录的符号链接。结果表明，Babel 在重新编译通过这种方式链接的库时，会抛出很多错误。我之所以花了这么长时间才弄清楚 Jest 这档子事儿，原因之一就是我一直通过这种方式链接我的库，出现了一堆错误。

解决办法就是：**不要**使用 npm/yarn link。用类似 “yalc” 这样的工具，它可以连接本地项目，同时能模拟正常的 npm 安装过程。它不但没有 Babel 重编译的问题，还能更好地处理传递性依赖。

### 针对特定库的优化。

如果完成了以上所有步骤，你的应用基本上实现了比较健壮的 tree shaking。不过，为了进一步减少文件包大小，你还可以做一些事情。我会列举一些特定库的优化方法，但这绝对不是全部。它尤其能为我们提供灵感，做出一些更酷的事情。

MomentJS 是出了名的大体积库。幸好它可以剔除多语言包来减少体积。在下面的代码示例中，我排除了 momentjs 所有的多语言包，只保留了基本部分，体积明显小了很多。
```
// 用 IgnorePlugin 移除多语言包
const { IgnorePlugin } from 'webpack';
const config = {
 plugins: [
  new IgnorePlugin(/^\.\/locale$/, /moment/)
 ]
};
```

Moment-Timezone 是 MomentJS 的老表，也是个大块头。它的体积基本上是一个带有时区信息的超大 JSON 文件导致的。我发现只要保留本世纪的年份数据，就可以将体积缩小90%。这种情况需要用到一个特殊的 Webpack 插件。
```
// MomentTimezone Webpack Plugin
const MomentTimezoneDataPlugin = require('moment-timezone-data-webpack-plugin');
const config = {
 plugins: [
  new MomentTimezoneDataPlugin({
   startYear: 2018,
   endYear: 2100
  })
 ]
};
```

Lodash 是另一个导致文件包膨胀的大块头。幸好有一个替代包 Lodash-es，它被编译成 es2015 模块，并带有 sideEffects 标志。用它替换 Lodash 可以进一步缩减包的大小。

另外，Lodash-es，react-bootstrap 以及其他库可以在 Babel transform imports 插件的帮助下实现瘦身。该插件从库的 index.js 文件读取 import 语句，并使其指向库中特定文件。这样就使 webpack 在解析模块树时更容易对库做 tree shaking。下面的例子演示了它是如何工作的。
```
// Babel Transform Imports
// Babel config
const config = {
 plugins: [
  [
   'transform-imports',
   {
    'lodash-es': {
     transform: 'lodash/${member}',
     preventFullImport: true
    },
    'react-bootstrap': {
     transform: 'react-bootstrap/es/${member}', // The es folder contains es2015 module versions of the files
     preventFullImport: true
    }
   }
  ]
 ]
};
// 这些库不再支持全量导入，否则会报错
import _ from 'lodash-es';
// 具名导入依然支持
import { debounce } from 'loash-es';
// 不过这些具名导入会被babel编译成这样子
// import debounce from 'lodash-es/debounce';

```

### 总结

全文到此结束。这样的优化可以极大地缩减打包后的大小。随着前端架构开始有了新的方向（比如微前端），保持包大小最优化变得比以往更加重要。希望本文能给那些正在给应用程序做tree shaking的同学带来一些帮助。

[原文](https://medium.com/@craigmiller160/how-to-fully-optimize-webpack-4-tree-shaking-405e1c76038)
