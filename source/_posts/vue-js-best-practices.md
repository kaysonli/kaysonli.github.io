---
title: Vue.js 最佳实践[新手必看]
date: 2019-11-07 18:07:28
tags: Vue.js
---

通过查看 [VueJs 文档](https://vuejs.org/v2/guide/) 和网上的资料，我列了一个最佳实践和风格指南的清单，这些是被普遍认为更加正确的VueJs 用法。

下面列出的这些点，有些是功能和优化相关的，其他的是VueJs命名约定和元素顺序相关。

#### 组件销毁的时候用 `$off`清除事件监听器

当使用`$on`监听事件时，我们应该记得在`destroyed()`中用`$off`移除该事件。这样可以防止内存泄漏。

#### 事件名总是使用 kebab-case 大小写

触发和监听自定义事件时，应该总是使用kebab-case 大小写形式。为什么呢？因为无论怎样事件名总会自动转成小写，根本没有机会监听到驼峰式和首字母大写格式的事件名，因此，跟监听用的格式保持一致更加合理，也就是kebab-case形式。
```
// Emitting
this.$emit('my-event') // instead of myEvent// Listening
v-on:my-event</pre>
```
<!-- more -->
#### 避免在created和watch里调用同一个方法

如果需要在组件初始化和属性变化时触发方法调用，通常的做法是这样的：
```
watch: {
  myProperty() {
    this.doSomething();
  }
},
created() {
  this.doSomething();
},
methods: {
  doSomething() {
     console.log('doing something...');
  }
}
```

尽管看起来没错，但是这里使用`created()`是多余的。我们可以把所有功能都放进`watch`，这样就避免在`created()`中写重复的代码了，同时还能在组件初始化时触发。例如：

```
watch: {
  myProperty: {
    immediate: true, // 初始化时立即执行
    handler() {
      this.doSomething();
    }
  }
},
methods: {
  doSomething() {
     console.log('doing something...');
  }
},

// 这样甚至更好
watch: {
  myProperty: {
    immediate: true, //初始化时立即执行 
    handler() {
      // 只用一次的代码都不需要定义方法了。
      console.log('doing something...'); 
    }
  }
},
```

####  `v-for` 循环总是使用 `:key`

在模板循环里总是加上`:key`是一种常见的最佳实践。不带`:key`的`v-for`会导致很难发现错误，尤其是在动画中。

#### mixins 中的属性名使用$_前缀

mixins 是一种绝佳的方式，它把重复代码放在一起，然后在需要的时候引入。但是，*一个大大的但是*，这会导致一些问题。在这里我们说下**属性冲突**的问题。

但我们在组件中引入一个mixin时，其实是将mixin里的代码和组件代码*合并了*。那么，碰到同名的属性会怎样？答案是，组件的优先级更高。

如果我想要 mixin 的优先级更高怎么办 ？你无法改变优先级，但是可以避免属性合并，甚至选择正确的*命名约定*来避免属性覆盖。

为了区分mixin属性和组件属性，我们可以使用`$_`前缀。为什么是这两个字符？有几个原因：
1.  来自VueJs 风格指南的约定 
2.  `_` 是给Vue 私有属性用的
3.  `$` 是给 Vue 开源生态用的

 [VueJs 风格指南 ](https://vuejs.org/v2/style-guide/#Private-property-names-essential)建议加上mixin的名称，例如：`$_myMixin_updateUser`。

我发现，加上mixin的名称带来的迷惑性多于可读性。不过这也取决于mixin的开发者和使用场景。

通过添加`$_`，例如`$_updateUser`，代码的可读性好多了，也可以轻松区分组件和mixin了。

#### mixin用到的属性应该在mixin中定义

在上一点的基础上，mixin还有另一个问题：有时候会忘了定义相关属性。

如果我们创建了一个mixin，并使用了`this.language`，而它没有在mixin内部定义，那么引入这个mixin的组件必须要包含`language`属性。

想必你也能看出来，这很容易出错。为了避免这些错误，我们应该在mixin**内部**定义用到的东西。不要担心会定义两次，VueJs足够智能，如果检测到已经从Store获取（大部分情况下都是从store获取数据），就不会做重复工作。

#### 单文件组件命名使用 PascalCase 或者 kebab-case 大小写风格

PascalCase 风格能更好地与编辑器集成，在常用的IDE中也有更好的自动完成和导入功能。

如果我们想避免大小写敏感的文件系统的问题，kebab-case 风格比较合适。

#### 基类组件命名使用前缀

展示型、静态的和“纯”组件应该用一个前缀来与其他非“纯”组件区分开来。这样可以提高项目的可读性，以及改善团队和开发者之间的知识传递。

#### 组件名称使用 PascalCase 风格 

在 JavaScript 中，PascalCase 风格是类和原型构造器的传统约定，Vue 组件使用这种风格也比较合理。

如果你只使用通过`Vue.component`定义的全局组件，建议使用 kebab-case 风格。
> Prop 声明时应该总是使用 camelCase，但在模板里是 kebab-case 的。

根据语言各自的约定：JavaScript 使用*(camelCase)* ， HTML 使用 *(kebab-case)*， `prop` 在JS中定义时用camelCase，而在HTML中使用kebab-case是合理的。 

#### 使用风格指南中的组件配置项顺序

听起来可能有点呆板，但是在整个项目中让组件的所有配置项保持一样的顺序，有助于内容查找，也方便创建新的组件。

 VueJs 代码约定见 [风格指南 ](https://vuejs.org/v2/style-guide/#Component-instance-options-order-recommended)。

#### 使用 v-for 的元素不要同时使用  v-if 

这是性能杀手，对于这种错误做法，列表越大，性能损失越大。

我们通过代码来解释下。假设有这么一个场景：

```
<ul>
  <li
    v-for="game in games"
    v-if="game.isActive"
    :key="game.slug"
  >
    {{ game.title }}
  <li>
</ul>
```

这会被解析成类似这样的代码：
```
this.games.map(function (game) {
  if (game.isActive) {
    return game.title
  }
})
```

可以看到，我们必须遍历整个`games`列表，无论符合条件的`games`列表是否有变化。

其他框架，比如 Angular，这种写法根本无法通过编译。（使用*`**ngFor*`*的元素不能有*`**ngIf*`*）

#### Actions 必须有返回

这是长期折腾 `async`/`await` 和 Vuex actions 得出的经验。

举例如下：
```
// Store
[SOME_ACTION] () {
   // 耗时的某个操作
   console.log('Action done');
}
// 使用 action
async doSomething() {
  await dispatch(SOME_ACTION);
  console.log('Do stuff now');
}
这会输出：
// Do stuff now
// Action done
```

这是因为`await`不知道要等待什么，相反，如果我们实际上返回了一个`Promise.resolve()`，`await`就会等待解决，然后继续执行。
```
// Store
[SOME_ACTION] () {
   // 耗时的操作
   console.log('Action done');
   Promise.resolve();
}// 使用action
async doSomething() {
  await dispatch(SOME_ACTION);
  console.log('Do stuff now');
}
这会输出：
// Action done
// Do stuff now
```

####   在 actions 和 getters 内部使用选择器函数

我们创建选择器函数是有原因的，不但可以在整个应用中使用，还可以在Vuex Store 内部使用。

看看代码就明白了：

```
// 先定义选择器函数
export const language = (state) => state.userConfig.language;// 某个Action需要 language 属性
// 不推荐
[GET_GAMES]({ commit, rootState }) {
   const lang = rootState.userConfig.language;
   // Do stuff...
} 
// 推荐
[GET_GAMES]({ commit, rootState }) {
   const lang = language(rootState);
   // Do stuff...
}
```


[原文链接](https://blog.usejournal.com/vue-js-best-practices-c5da8d7af48d)

#### 交流
欢迎关注微信公众号“1024译站”，持续为你奉上技术干货。
![公众号：1024译站](https://upload-images.jianshu.io/upload_images/1618526-840b5a6dfebbe564.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
