---
title: Sass Module 介绍
date: 2019-10-14 20:04:54
tags: CSS
---

Sass刚刚发布了一个你可能从其他语言中认识的主要新功能：**模块系统**。这是最常用的Sass功能之一`@import`向前迈出的一大步。虽然当前的`@import`规则允许引入第三方包，并将Sass分成可管理的多个“partials”，但它有一些限制:

*   `@import` 也是一个CSS特性，两者之间的区别可能会让人感到困惑
*   如果多次`@import`相同的文件，它会降低编译速度，导致覆盖冲突，并生成重复的输出。
*   所有东西都在全局命名空间中，包括第三方包——所以我的`color()`函数可能会覆盖你现有的`color()`函数，反之亦然。
*  当你使用像`color()`这样的函数时，很难确切地知道它是在哪定义的。它来自哪个`@import`?

Sass包的作者(像我一样)试图通过手动为变量和函数添加前缀来解决命名空间问题，但是Sass模块是一个更强大的解决方案。简而言之，`@import`被更明确的`@use`和`@forward`规则所取代。在接下来的几年里，Sass`@import`将被弃用，然后被删除。你仍然可以使用[CSS import](https://developer.mozilla.org/en-US/docs/Web/CSS/@import)，但是它们不会被Sass编译。别担心，有一个[迁移工具](http://sass-lang.com/documentation/cli/migrator)可以帮助你升级!
<!-- more -->
### 使用@use导入文件

```
@use 'buttons';
```

新的`@use` 类似于`@import`。但有一些明显的区别:
*  该文件只导入一次，不管在项目中`@use`它多少次。
*   以下划线(`_`)或连字符(`-`)开头的变量、mixin 和函数(Sass称为"成员变量")被认为是私有的，不会被导入。
*   导入的文件（这里即`buttons.scss`）中的成员变量只在局部可用，而不会传递到未来的导入结果中。
*   类似地，`@extends`将只应用于*上游链*——即只扩展被导入的文件中的选择器，而不是执行导入命令的文件。
*   所有导入的成员变量*默认拥有命名空间*

当我们`@use` 一个文件时，Sass会根据文件名自动生成一个命名空间:

```
@use 'buttons'; // 生成了一个`buttons` 命名空间
@use 'forms'; // 生成了一个 `forms` 命名空间
```

我们现在可以同时访问`buttons.scss` 和`forms.scss`中的成员变量。但是这两个文件之间并不能互相访问变量。由于导入的特性是带命名空间的，所以我们必须使用一个新的点号分割语法来访问它们:

```
// variables: <namespace>.$variable
$btn-color: buttons.$color;
$form-border: forms.$input-border;

// functions: <namespace>.function()
$btn-background: buttons.background();
$form-border: forms.border();

// mixins: @include <namespace>.mixin()
@include buttons.submit();
@include forms.input();
```

我们可以在导入时添加`as <name>`来改变或删除默认的命名空间：
```
@use 'buttons' as *; // 星号会删除所有命名空间
@use 'forms' as 'f';

$btn-color: $color; // 不带命名空间的buttons.$color
$form-border: f.$input-border; // 带有自定义命名空间的forms.$input-border
```
使用`as *`将模块添加到根名称空间，因此不需要前缀，但这些成员仍然在当前文档的本地作用域内。
### 导入内置的Sass模块

内部的Sass特性也已经转移到模块系统中，因此我们可以完全控制全局命名空间。有几个内置的模块：`math`, `color`, `string`, `list`, `map`, `selector`和`meta` ，这些模块必须在使用之前显式地导入到文件中:
```
@use 'sass:math';
$half: math.percentage(1/2);
```

Sass模块也可以导入到全局命名空间中：
```
@use 'sass:math' as *;
$half: percentage(1/2);
```

已经有前缀名称的内部函数，如`map-get`或`str-index`可以不加前缀直接使用：
```
@use 'sass:map';
@use 'sass:string';
$map-get: map.get(('key': 'value'), 'key');
$str-index: string.index('string', 'i');
```
你可以在[Sass模块规范](https://github.com/sass/sass/blob/master/accepted/module-system.md#built-in-modules-1)中找到内置模块、函数和名称更改的完整列表

### 新的和改变的核心功能

作为一个附带好处，这意味着Sass可以安全地添加新的内部mixin和函数，而不会引起命名冲突。这个版本中最激动人心的例子是一个名为`load-css()`的`sass:meta` mixin。跟`@use` 类似 ，但它只返回转换后的CSS输出，我们可以在代码中的任何地方动态使用：
```
@use 'sass:meta';
$theme-name: 'dark';

[data-theme='#{$theme-name}'] {
  @include meta.load-css($theme-name);
}
```

第一个参数是一个模块URL(类似`@use`)，但是它可以被变量动态更改，甚至包括插值表达式，如`theme-#{$name}`。第二个(可选)参数接受配置值项键值对:
```
// 在加载之前设置'theme/dark' 中的 $base-color 变量
@include meta.load-css(
  'theme/dark', 
  $with: ('base-color': rebeccapurple)
);
```

参数 `$with`接受被加载模块中的任何变量的键和值：
*   不以`_`或`-`开头的全局变量(现在用于表示私有)
*   标记为 `!default` 的可配置值

```
// theme/_dark.scss
$base-color: black !default; // 可被配置
$_private: true !default; // 不可被配置，因为是私有的
$config: false; // 不可被配置，因为没有被标记为 !default
```

请注意，`'base-color'` 键将设置`$base-color` 变量。

还有两个新的`sass:meta`函数`module-variables()`和`module-functions()`。每个函数返回被导入模块中的成员名和值的键值对。它们接受一个与模块命名空间匹配的参数:
```
@use 'forms';

$form-vars: module-variables('forms');
// (
//   button-color: blue,
//   input-border: thin,
// )

$form-functions: module-functions('forms');
// (
//   background: get-function('background'),
//   border: get-function('border'),
// )
```

其他几个`sass:meta`函数`global-variable-exists()`, `function-exists()`, `mixin-exists()` 和 `get-function()` 有额外的`$module`参数，允许我们显式地检查每个命名空间。
#### 调整和缩放颜色

 `sass:color` 模块也有一些有趣的注意事项，因为我们试图摆脱一些遗留问题。许多遗留的快捷方式如`lighten()`或者`adjust-hue()`现在被弃用，改为显式的 `color.adjust()` 和`color.scale()`函数：
```
// 原来的 lighten(red, 20%)
$light-red: color.adjust(red, $lightness: 20%);

// 原来的 adjust-hue(red, 180deg)
$complement: color.adjust(red, $hue: 180deg);
```

其中一些旧的函数(如 `adjust-hue`)变得多余和不必要了。其他的比如`lighten`， `darken`，`saturate`等等，需要用更好的内部逻辑重新构建。原来的函数是基于`adjust()`的，它利用了线性数学：在上面的例子中，给当前`red` 变浅`20%`。在大多数情况下，我们实际上想要相对于当前值缩放（`scale()`）颜色深浅的百分比：
```
// 跟white相差20%，而不是在当前深浅度上增加20%
$light-red: color.scale(red, $lightness: 20%);
```

一旦完全弃用并删除，这些快捷函数最终将以`color.scale()`而不是`color.adjust()`为基础重新出现在`sass:color` 中。这是分阶段进行的，以避免出现突然的向后不兼容的变化。与此同时，我建议你手动检查你的代码，看看`color.scale()` 在什么地方更适合你。

### 配置导入的库

第三方或可重用的库通常带有默认的全局配置变量可供覆盖。我们通常在导入之前对变量进行这样的操作：
```
// _buttons.scss
$color: blue !default;

// old.scss
$color: red;
@import 'buttons';
```

由于使用过的模块不能够再访问本地变量，我们需要一种新的方法来设置这些默认值。我们可以通过给`@use`添加配置键值对来实现：
```
@use 'buttons' with (
  $color: red,
  $style: 'flat',
);
```

这类似于`load-css()`中的`$with` 参数。但是我们不使用变量名作为键，而是使用变量本身，以`$`开头。

我喜欢这种显式的配置方式，但是有一条规则让我犯了好几次错误：**一个模块只能配置一次，即第一次使用它时**。导入顺序对Sass来说一直很重要，即便`@import`也是如此。但这些问题总是静默地失败。现在我们得到一个明确的错误，这是好事，有时也令人惊讶。
确保使用`@use`时在“入口”文件(导入所有部分的中心文档)中首先配置库，以便在其他地方`@use`库之前编译这些配置。

(目前来说)不可能在保持可编辑的同时将配置“链接”在一起，但是你可以将已配置的模块与扩展打包，并将其作为新模块传递。

### 用@forward传递文件

我们并不总是需要使用一个文件，并访问它的成员。有时我们只是想把它传给未来的导入操作。假设我们有多个与表单相关的partials，我们希望将它们全部导入为一个命名空间。我们可以用 `@forward`来实现：
```
// forms/_index.scss
@forward 'input';
@forward 'textarea';
@forward 'select';
@forward 'buttons';
```

被转发的文件的成员在当前文档中不可访问，也没有创建命名空间，但是当另一个文件想要`@use` 或`@forward` 整个集合时，这些变量、函数和mixin 就是可访问的。如果转发的部分包含实际的CSS，那么在使用包之前，它也不会生成输出。在这一点上，它将被视为单个模块与单个命名空间:
```
// styles.scss
@use 'forms'; // 导入`forms` 命名空间下的所有被转发的成员
```

**注：**如果你要求Sass导入一个目录，它会寻找一个名为`index`或`_index`的文件)

默认情况下，所有公共成员将使用一个模块进行转发。但我们可以更有选择性地添加`show` 或`hide`语句，来包含或排除指定的成员：
```
// 只转发'input' 中的 border() mixin 和 $border-color 变量
@forward 'input' show border, $border-color;

// 转发'buttons' 里的所有成员， gradient() 函数除外
@forward 'buttons' hide gradient;
```

**注意：**当函数和mixin共享一个名称时，它们会一起显示和隐藏。

为了区分来源，或避免转发模块之间的命名冲突，我们可以在转发时对模块成员使用`as` 前缀：
```
// forms/_index.scss
// @forward "<url>" as <prefix>-*;
// 假设两个模块都有一个background() mixin
@forward 'input' as input-*;
@forward 'buttons' as btn-*;

// style.scss
@use 'forms';
@include forms.input-background();
@include forms.btn-background();
```

而且，如果需要，我们可以对同一模块同时使用 `@use`和`@forward` ：
```
@forward 'forms';
@use 'forms';
```

如果你希望在将库传递给其他文件之前，用配置或任何其他工具包装库，那么这一点特别有用。它甚至可以帮助简化导入路径：
```
// _tools.scss
// 只使用该库一次，并带有配置参数
@use 'accoutrement/sass/tools' with (
  $font-path: '../fonts/',
);
// 随即转发配置后的库
@forward 'accoutrement/sass/tools';

// 在这里添加其他扩展…

// _anywhere-else.scss
// import the wrapped-and-extended library, already configured
@use 'tools';
```

 `@use`和`@forward`必须在(非嵌套的)文档的根部和文件的开头声明。只有`@charset`和简单的变量定义可以出现在导入指令之前。

#### 维护和编写样式

在网站上使用模块非常有趣。新的语法鼓励我已经在使用的代码结构。我所有的全局配置和工具导入都在一个目录中(我称之为 `config`)，其中有一个索引文件，可以转发我需要的所有内容:
```
// config/_index.scss
@forward 'tools';
@forward 'fonts';
@forward 'scale';
@forward 'colors';
```

在我构建网站的其他部分时，我可以在任何需要的地方导入这些工具和配置：
```
// layout/_banner.scss
@use '../config';

.page-title {
  @include config.font-family('header');
}
```
这甚至适用于我现有的Sass库，比如[accout](https://www.oddbird.net/accoutrement/)和[Herman](https://www.oddbird.net/herman/)，它们仍然使用旧的`@import`语法。由于`@import`规则不会在一夜之间被全部替换，所以Sass提供了一个过渡期。现在就可以使用模块功能，但是`@import`在一两年后才会被弃用，并且在一年后才会从语言中删除。在此期间，这两个系统将朝两个方向共同努力：
*   如果我们 `@import`一个包含新的 `@use`/`@forward` 语法的文件，那么只导入公共成员，没有命名空间。
*  如果我们`@use` 或`@forward`一个包含遗留 `@import`语法的文件，我们将以单个命名空间访问所有嵌套的导入。

这意味着您可以立即开始使用新的模块语法，而不必等待你喜欢的库的新版本：而且我可以花一些时间来更新我的所有库!

#### 迁移工具

如果我们使用Jennifer Thakar构建的[Migration Tool](https://sass-lang.com/documentation/cli/migrator)，升级应该不会花很长时间。它可以通过 Node, Chocolatey, 或 Homebrew 安装：
```
npm install -g sass-migrator
choco install sass-migrator
brew install sass/sass/migrator
```

这不是迁移到模块系统的一次性工具。既然Sass已经回到了活跃的开发阶段(参见下面)，那么迁移工具也将得到定期更新，以帮助迁移每个新特性。全局安装它是个好主意，并保留它以备将来使用。

迁移工具可以从命令行运行，并有望跟CodeKit和Scout一样添加到[第三方应用程序集](http://modules.sass-lang.com/install)。将它指向一个Sass文件，比如`style.scss`，并告诉它应该迁移什么内容。在这里，只有一个名为`module`的迁移：
```
# sass-migrator <migration> <entrypoint.scss...>
sass-migrator module style.scss
```

默认情况下，迁移工具将只更新单个文件，但在大多数情况下，我们希望更新主文件*及其所有依赖项*：`import`、`forward`或`use`的任何 partials 。我们可以通过单独指定每个文件，或者添加 `--migrate-deps`标记来做到这一点:
```
sass-migrator --migrate-deps module style.scss
```
对于测试程序，我们可以添加`--dry-run --verbose`(或简写为 `-nv`)，并在不更改任何文件的情况下查看结果。我们可以使用许多其他选项来定制迁移——甚至有一个专门用于帮助库作者删除旧的手动命名空间——但是我不会在这里全部介绍。在[Sass网站](http://sass-lang.com/)上可以找到[迁移工具的完整文档](https://sass-lang.com/documentation/cli/migrator)。

#### 更新已发布的库

我在库方面遇到了一些问题，特别是试图让用户跨文件访问用户配置，以及处理缺少的链式配置。顺序错误可能很难调试，但是结果表明花这些工夫是值得的，我认为我们很快就会看到一些额外的补丁。我仍然需要在复杂的包上试验迁移工具，并可能为库作者撰写后续文章。

现在要知道的重要的事情是，在过渡期，Sass已经替我们解决了大部分问题。不仅import 和 module 可以一起使用，而且我们可以创建“[只有import](http://sass-lang.com/documentation/atrules/import#import-onlyfiles)”的文件，为仍然用 `@import`方式使用库的老用户提供更好的体验。在大多数情况下，这将是主包文件的另一个版本，你可能希望同时使用它们：为模块用户提供`<name>.scss`，为遗留用户提供`<name>.import.scss`。当用户调用`@import <name>`时，它将加载文件的`.import`版本。
```
// load _forms.scss
@use 'forms';

// load _forms.input.scss
@import 'forms';
```

这对于为非模块用户添加前缀特别有用:
```
// _forms.import.scss
// Forward the main module, while adding a prefix
@forward "forms" as forms-*;
```

### 升级Sass

你可能还记得，几年前Sass有一个特性冻结，以使各种实现(LibSass、Node Sass、Dart Sass)都跟上开发节奏，并最终[废弃了原来的Ruby实现](https://css-tricks.com/ruby-sass-to-be-put-to-pasture-on-march-26-2019/)。这一冻结在去年结束了，在GitHub上推出了一些新功能和活跃的[讨论与开发](https://github.com/sass/sass)，但并没有大张旗鼓。如果你错过了这些发布，你可以在[Sass博客](http://sass.logdown.com/)上找到：
*   [CSS Imports 与 CSS 兼容性](http://sass.logdown.com/posts/7807041-feature-watchcss-imports-and-css-compatibility) (Dart Sass v1.11)
*   [内容参数与颜色函数](http://sass.logdown.com/posts/7816585-feature-watchcontent-arguments-and-color-function-syntax) (Dart Sass v1.15)


Dart Sass现在是标准实现，并且通常是第一个实现新特性的。如果你想要最新的，我建议你切换过去。可以使用Node、Chocolatey或Homebrew[安装Dart Sass](http://sass-lang.com/install)。它还可以与现有的[gulp-sass](https://www.npmjs.com/package/gulp-sass)构建步骤一起工作。

跟CSS(从CSS3开始)很像，新的发布不再有一个统一的版本号。所有的Sass实现都遵循相同的规范，但是每个都有一个独特的发布时间表和版本号，这反映在[Jina](https://www.sushiandrobots.com/)设计的[新文档](https://sass-lang.com/documentation)中。

[原文链接](https://css-tricks.com/introducing-sass-modules/)

