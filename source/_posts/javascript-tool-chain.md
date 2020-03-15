---
title: 前端劝退预警：JavaScript 工具链不完全指南
date: 2020-03-11 11:14:50
tags: 开发工具
---

![宇宙中最重的物质](https://s1.ax1x.com/2020/03/15/814UPA.md.jpg)

经过这么多年的发展，JavaScript 早已经不是当年那个不太起眼的脚本语言。如今的 JavaScript 可以说是风光无限，在 Web 前端、移动端、服务端甚至物联网设备上都大展身手，到处都有它的身影。

在 JavaScript 语言日渐强大的同时，与其配套的开发工具也蓬勃发展。现在的 Web 前端项目，早已不是写几个 HTML 页面，加点 CSS 和 JS 就完事了。随便一个实用的项目，可能都需要用到一些框架和第三方库，以及相应的脚手架、依赖包管理、预编译、构建打包、压缩合并等等工具。纯手工完成这些任务，已经几乎不太可能了。

科学技术是第一生产力，而工具就是其中的一个体现。工欲善其事必先利其器，既然工具解放了人力，我们就应该拥抱它们。本文总结了围绕 JavaScript 的一系列工具，看看一个常见的项目到底需要用到哪些工具。
<!-- more -->
### 静态类型检查
JavaScript 本身是一门动态脚本语言，是弱类型的。也就是说，没有编译阶段的数据类型检查，只能在运行时确定类型。好处是比较灵活，简单易学，代码量也比较少。缺点也是明显的，就是稍不注意容易出 bug，特别是大型项目，如果没有开发规范，任由开发人员自由发挥，维护起来简直就是灾难。为了弥补这个缺陷，一些自带类型系统、可转译成 JavaScript 的语言出现了。

**[Flow](https://flow.org/en/)**：Facebook 出品，用 OCaml 写的 JavaScript 静态类型检查系统。支持类型系统、类型标注，可定义 library，提供代码 lint 等。Vue.js 2.x 就是用 Flow 写的。不过，Vue 3.0 改用另一种静态类型语言了，那就是 TypeScript。

**[TypeScript](https://www.typescriptlang.org/)**：TypeScript 是微软开发的语言，是 JavaScript 的超集，能够编译成 JavaScript。现在 TypeScript 越来越流行，获得了大量项目的认可。

### 代码风格检查（Linter）
由于JavaScript 动态语言的特性，在写法上过于灵活，往往导致多人协作的项目代码风格各异，给维护和扩展带来不少麻烦。另外，部分语言特性容易导致 bug，最佳实践里通常不推荐使用。因此，代码 Lint 工具就派上用场了。它不但可以检查代码书写格式，还可以检查出引用未定义的变量等低级错误。

曾经出现过很多种 Lint 工具，比如 JSLint、JSHint、StandardJS、JSCS 等等，现在有大一统的趋势，基本都用 ESLint 了。

刚开始用 Lint 工具的时候可能会不适应，因为限制太多，动不动就警告和报错。但从长远来看，JavaScript 项目引入 Linter 还是有必要的。

### 包管理器
JavaScript 之所以能够遍地开花，很大程度是因为技术生态非常繁荣，各种第三方库应有尽有。如此庞大的第三方库集合，势必需要一个管理平台，负责第三方包的托管、版本管理、下载安装和依赖管理等。

**npm：**JavaScript 包管理器的集大成者，目前最主流的工具。 基于 Node.js，包含网站和 CLI，上面的包总数超过 100万，基本涵盖了 JavaScript 项目所需的方方面面。

**yarn**：2016 年 Facebook 推出的包管理器，跟 npm registry 兼容，主打 CLI 的快速、安全和确定性。

**bower**：曾经比较流行，用于管理前端的 JavaScript 包，因为当初 npm 只支持 node 环境的包管理。随着 npm 和 yarn 同时支持 node 和浏览器端的包管理，bower 也逐渐淡出历史了。

**pnpm**：通过硬链接（hard links）的方式在硬盘上对某个版本的模块只保存一份，可以在多个项目之间共用，从而节省了大量硬盘空间，顺便也加快了模块安装速度。

### 模块加载器
JavaScript 早期没有语言层面的模块化支持，导致大型项目的依赖管理非常不方便。变量名冲突、全局变量污染、模块加载顺序等问题比较突出。曾经出现过各种模块化方案，比如 CommonJS Modules (CJS)，Asynchronous Module Definition （AMD）和 Universal Module Definition (UMD)。从 2015 年开始，ES6 从语言层面支持了模块化，即 ECMAScript Modules (ESM)。

除了上述常见的 JavaScript 模块格式，还有像 [System.register](https://github.com/systemjs/systemjs/blob/master/docs/system-register.md) 或全局模块，以及一些非 JavaScript 模块，比如 JSON modules，CSS modules，Web Assembly 等。

模块加载器就是加载和处理上面各种类型模块代码的工具，有同步和异步、静态和动态之分。通常，一个模块放在一个单独的文件里，通过一些指令来导入。当一个项目有几十上百个模块的时候，如何保证它们的加载执行先后顺序就是个问题。不用担心，模块加载器会帮你处理好依赖关系。

**[RequireJS](https://github.com/requirejs/requirejs)：**实现了 AMD 模块加载，主要用于浏览器端，也可以在 Node 里使用。

**[SystemJS](https://github.com/systemjs/systemjs)**：一个动态模块加载器，可以加载所有类型的 JavaScript 模块，甚至包括非 JavaScript 模块，可用于浏览器和 Node。

**[StealJS](https://github.com/stealjs/steal/)**：可以加载 ESM，ADM 和 CJS 格式的模块。

**ES Module Loader**： [浏览器](https://caniuse.com/#search=modules)实现的模块加载器，Node.js 里加上  `--experimental-modules` 标志也能使用。

以上模块加载器是在**运行时**加载和执行代码的。不要跟构建工具里的各种 loader 混淆，那些是在**构建时**进行预处理的工具，只是做一些格式转换。

### 打包工具
一个实际的生产项目，通常不是把源码直接发布到服务器上。而是通过一些工具对各种模块和静态资源进行整合处理，最终生成的代码可能跟源码完全不一样了。这就是打包工具的作用。比较常见的打包工具有：

**Webpack**：可以说是最流行的打包器，几乎是大部分项目的标配。它本身只能识别 JavaScript 和 JSON 文件，但是它的架构设计提供了无限的可能，通过各种 loader 可以处理任意类型的资源，通过插件可以优化打包结果和进行资源管理、注入环境变量等。它的构建目标也可以根据平台定制，比如浏览器、Node.js、web worker, Electron 等。

**Rollup**：也是一个优秀的打包器，支持输出 library 和 application。它最大的卖点就是默认支持 ES 模块，很早就实现了 tree-shaking 功能。同时也具备插件和 hook 功能，可定制化也比较好。

**Parcel**：工具界的后起之秀，号称“零配置*的打包器，如果你曾经被 Webpack 繁琐的配置困扰过，那它可能会吸引你。

**Browserify**：应用范围稍窄的打包器，专门用于转换 Node.js 包以便能在浏览器运行。它跟 Node.js 使用相同的模块系统，有些模块只用于 Node 平台，通过它的转换就可以在浏览器端使用了。它只能处理纯 JavaScript 模块，通常跟 Gulp 配合使用。

**Metro**：React Native 专用的打包器，通过一个入口文件和各种配置，最后打成包含所有代码和依赖的单个 JavaScript 文件。

### 任务管理工具（Task Runner）
Task Runner 的作用是自动化执行项目所需的各种重复性动作，比如：
- CSS 预处理 (Less, Sass) 
- CSS 自动添加新特性属性前缀（Autoprefixer）
- 优化图片
- 合并、压缩 JavaScript 文件
- 监听文件变化，自动执行任务

等等。
比较主流的任务管理器是 Grunt 和 Gulp，Webpack 也算一个。

**Grunt**：命令行工具，通过精细化的配置和丰富的插件可以完成很多复杂的任务。

**Gulp***：跟 Grunt 不同，Gulp 采用流式管道组合多个任务，任务之间的临时结果是放在内存中的，执行效率会高很多。

**Webpack**：又是你，Webpack。任务管理只是 Webpack 强大功能的一部分，通过不同的插件在构建的不同阶段执行个性化的任务。

如果你觉得上面几个任务管理器有点杀鸡用牛刀，你也可以根据实际情况选用 bash 脚本，npm 脚本或者 Makefile 等实现一些简单的任务管理。

### 转译器
JavaScript 转译器的作用是将非 JavaScript 语言（TypeScript，CoffeeScript，LiveScript 等）或不同版本的 JavaScript（ES6，ES7，ES8 等）翻译成符合目标平台要求（兼容性、变量混淆、严格模式等）的等价代码。

大部分转译器在处理源码和代码优化的过程中都使用抽象语法树（AST）作为中间格式。AST 将源码逐步拆分解析成带有元数据的树形结构：
```
Code --(parse)-->AST--(transform)-->AST--(generate)-->Code
```
[Babel](https://babeljs.io/) 是目前主流的 JavaScript 转译器，它的工具链体系主要用于将 ES6 以上版本的 JavaScript 转译成向后兼容浏览器的代码。Babel 可以转换语法、提供目标平台缺失的特性支持、转换源码等。

有了 Babel，我们可以尽早用上新的语言特性，而不用担心目标浏览器是否支持。

### 构建工具
构建其实是个综合概念，它包含了模块打包、源码转译、任务管理等多个步骤。其他语言和平台有各种 Build 工具，比如 Make，Gradle，Ant，Maven，Rake 或 MSBuild 等。其中 Make 是最常见的通用构建工具，通常用于 C 语言，但其实也可以用于构建 JavaScript 项目。

对于 JavaScript 工具链来说，构建的目标可能是 npm 包、网站、Node 服务器、RN 应用、Electron 应用等等。有些任务可用专门的工具完成，但构建是个复杂的过程，通常需要综合使用多个工具。

比如 Build 可以用 [Buck](https://buck.build/)，[Bazel](https://bazel.build/)，[Lerna](https://lerna.js.org/) 等工具同时管理多个模块，实现增量 Build（只重新构建有改动的模块，提高效率）。任务管理使用 Grunt 和 Gulp，模块打包用 Webpack、Rollup、Parcel、Browserify 等。

### 调试工具

调试器在开发过程中必不可少，它可以在 Node.js 和浏览器中跟踪查看运行中的代码。通常会提供断点、单步执行、监视变量、查看内存和 CPU 使用情况等功能。

**[Chrome DevTools](https://developers.google.com/web/tools/chrome-devtools)**：Chrome 浏览器自带的开发者工具，是目前浏览器开发工具里最好用的（没有之一）。它的功能之强大，用法之多可以写本书了。

**[node-inspector](https://github.com/node-inspector/node-inspector)**：早期用于调试 Node.js 代码的工具，现在基本不用了，因为 Node.js 已经内置了基于 DevTools 的调试器。

**[VS Code](https://vscode.readthedocs.io/en/latest/editor/debugging/)**：又一调试神器，内置 Node.js 调试器，还能调试 TypeScript 和其他能转译成 JavaScript 的语言。

### Node 进程管理器

Node 进程管理器用于管理运行中的 Node 应用程序。提供高可用、自动重启、文件监控、性能和资源监控和集群等功能。

**[Forever](https://github.com/foreverjs/forever)**：顾名思义，它的作用是保持 Node.js 程序永远运行。它是一个简单的命令行工具，对小型 Node.js 应用比较方便。

**[PM2](https://github.com/Unitech/pm2)**：用于生产环境的 Node.js 进程管理工具，内置负载均衡、自动重启和日志、监控和集群管理等功能。

**[StrongLoop Process Manager](http://strong-pm.io/) (Strong-PM)**：跟 PM2 差不多，也可用于生产环境的 Node.js 进程管理，有多主机部署功能。 

另外，SystemD 是 Linux 系统里的默认进程管理器，可用它把 Node.js 应用作为服务运行。

### 项目脚手架
随着项目复杂度的提升，创建新项目的步骤也变得越来越繁琐，可能需要重复大量的配置工作。这个时候就需要项目脚手架了。很多框架都提供了脚手架工具，比如 Vue.js 有 Vue CLI，Angular 有 Angular CLI，React 有 create-react-app 等。也有通用的脚手架，Yeoman 就是最流行的一个。
- 快速创建新项目
- 创建模块或 package
- 启动新服务
- 统一代码风格、最佳实践等
- 项目推广，让用户快速上手示例 app
- 等等等等等等，自己探索

## [](https://dev.to/hoangbkit/javascript-tooling-online-book-5f39#popular-tools)

### 总结
本文列举了 JavaScript 常规项目可能要用到的工具链，零零散散几十个。每一个工具都解决了特定领域的问题，值得继续挖掘，本文由于篇幅所限只能蜻蜓点水式地一笔带过，有兴趣的可以自己深入研究。
