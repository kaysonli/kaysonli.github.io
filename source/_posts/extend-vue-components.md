---
title: Vue.js 组件复用和扩展之道
date: 2020-06-22 14:54:48
tags:
---
软件编程有一个重要的原则是 D.R.Y（Don't Repeat Yourself），讲的是尽量复用代码和逻辑，减少重复。组件扩展可以避免重复代码，更易于快速开发和维护。那么，扩展 Vue 组件的最佳方法是什么？

Vue 提供了不少 API 和模式来支持组件复用和扩展，你可以根据自己的目的和偏好来选择。

本文介绍几种比较常见的方法和模式，希望对你有所帮助。

* * *
<!-- more -->

## 扩展组件是否必要

要知道，所有的组件扩展方法都会增加复杂性和额外代码，有时候还会增加性能消耗。

因此，在决定扩展组件之前，最好先看看有没有其他更简单的设计模式能完成目标。

下面几种组件设计模式通常足够替代扩展组件了：
*   `props` 配合模板逻辑
*   slot 插槽
*   JavaScript 工具函数

#### `props` 配合模板逻辑

最简单的方法是通过`props`结合模板条件渲染，来实现组件的多功能。

比如通过 `type` 属性：
*MyVersatileComponent.vue* 

```
<template>
  <div class="wrapper">
    <div v-if="type === 'a'">...</div>
    <div v-else-if="type === 'b'">...</div>
    <!--etc etc-->
  </div>
</template>
<script>
export default {
  props: { type: String },
  ...
}
</script>

```

使用组件的时候传不同的`type`值就能实现不同的结果。

```
// *ParentComponent.vue*
<template>
  <MyVersatileComponent type="a" />
  <MyVersatileComponent type="b" />
</template>

```

如果出现下面两种情况，就说明这种模式不适用了，或者用法不对：
1.  组件组合模式把状态和逻辑分解成原子部分，从而让应用具备可扩展性。如果组件内存在大量条件判断，可读性和可维护性就会变差。
2.  props 和模板逻辑的本意是让组件动态化，但是也存在运行时资源消耗。如果你利用这种机制在运行时解决代码组合问题，那是一种反模式。

#### slot（插槽）

另一种可避免组件扩展的方式是利用 [slots（插槽）](https://vuejs.org/v2/guide/components-slots.html)，就是让父组件在子组件内设置自定义内容。

```
// *MyVersatileComponent.vue*
<template>
  <div class="wrapper">
    <h3>Common markup</div>
    <slot />
  </div>
</template>

```

```
// *ParentComponent.vue*
<template>
  <MyVersatileComponent>
    <h4>Inserting into the slot</h4>
  </MyVersatileComponent>
</template>

```

渲染结果：

```
<div class="wrapper">
  <h3>Common markup</div>
  <h4>Inserting into the slot</h4>
</div>

```

这种模式有一个潜在约束， slot 内的元素从属于父组件的上下文，在拆分逻辑和状态时可能不太自然。`scoped slot`会更灵活，后面会在无渲染组件一节里提到。

#### JavaScript 工具函数

如果只需要在各组件之间复用独立的函数，那么只需要抽取这些 JavaScript 模块就行了，根本不需要用到组件扩展模式。

JavaScript 的模块系统是一种非常灵活和健壮的代码共享方式，所以你应该尽可能地依靠它。
*MyUtilityFunction.js*

```
export default function () {
  ...
}

```

*MyComponent.vue*

```
import MyUtilityFunction from "./MyUtilityFunction";
export default {
  methods: {
    MyUtilityFunction
  }
}

```

## 扩展组件的几种模式

如果你已经考虑过以上几种简单的模式，但这些模式还不够灵活，无法满足需求。那么就可以考虑扩展组件了。

扩展 Vue 组件最流行的方法有以下四种：
*   [Composition 函数](https://vuejsdevelopers.com/2017/06/11/vue-js-extending-components/#composition-functions)
*   [mixin](https://vuejsdevelopers.com/2017/06/11/vue-js-extending-components/#mixins)
*   [高阶组件（HOC）](https://vuejsdevelopers.com/2017/06/11/vue-js-extending-components/#higher-order-components)
*   [无渲染组件](https://vuejsdevelopers.com/2017/06/11/vue-js-extending-components/#renderless-components)

每一种方法都有其优缺点，根据使用场景，或多或少都有适用的部分。
## Composition 函数

组件之间共享状态和逻辑的最新方案是 Composition API。这是 Vue 3 推出的 API，也可以在 Vue 2 里当插件使用。

跟之前在组件定义配置对象里声明`data`，`computed`，`methods`等属性的方式不同，Composition API 通过一个 `setup` 函数声明和返回这些配置。

比如，用 Vue 2 配置属性的方式声明 *Counter* 组件是这样的：
*Counter.vue*

```
<template>
  <button @click="increment">
    Count is: {{ count }}, double is: {{ double }}
  </button>
<template>
<script>
export default {
  data: () => ({
    count: 0
  }),
  methods: {
    increment() {
      this.count++;
    }
  },
  computed: {
    double () {
      return this.count * 2;
    }
  }
}
</script>

```

用 Composition API 重构这个组件，功能完全一样：
*Counter.vue*

```
<template><!--as above--><template>
<script>
import { reactive, computed } from "vue";

export default {
  setup() {
    const state = reactive({
      count: 0,
      double: computed(() => state.count * 2)
    });

    function increment() {
      state.count++
    }

    return {
      count,
      double,
      increment
    }
  }
}
</script>

```

用 Composition API 声明组件的主要好处之一是，逻辑复用和抽取变得非常轻松。

进一步重构，把计数器的功能移到 JavaScript 模块 `useCounter.js`中：
*useCounter.js*

```
import { reactive, computed } from "vue";

export default function {
  const state = reactive({
    count: 0,
    double: computed(() => state.count * 2)
  });

  function increment() {
    state.count++
  }

  return {
    count,
    double,
    increment
  }
}

```

现在，计数器功能可以通过`setup`函数无缝引入到任意 Vue 组件中：
*MyComponent.vue*

```
<template><!--as above--></template>
<script>
import useCounter from "./useCounter";

export default {
  setup() {
    const { count, double, increment } = useCounter();
    return {
      count,
      double,
      increment
    }
  }
}
</script>

```
Composition 函数让功能模块化、可重用，是扩展组件最直接和低成本的方式。
#### Composition API 的缺点

Composition API 的缺点其实不算什么——可能就是看起来有点啰嗦，并且新的用法对一些 Vue 开发者来说有点陌生。

关于 Composition API 优缺点的讨论，推荐阅读：[When To Use The New Vue Composition API (And When Not To)](https://vuejsdevelopers.com/2020/02/17/vue-composition-api-when-to-use)

## mixin

如果你还在用 Vue 2，或者只是喜欢用配置对象的方式定义组件功能，可以用 mixin 模式。mixin 把公共逻辑和状态抽取到单独的对象，跟使用 mixin 的组件内部定义对象合并。

我们继续用之前的*Counter*组件例子，把公共逻辑和状态放到*CounterMixin.js*模块中。
*CounterMixin.js*

```
export default {
  data: () => ({
    count: 0
  }),
  methods: {
    increment() {
      this.count++;
    }
  },
  computed: {
    double () {
      return this.count * 2;
    }
  }
}

```

使用 mixin 也很简单，只要导入对应模块并在`mixins`数组里加上变量就行。组件初始化时会把 mixin 对象与组件内部定义对象合并。
*MyComponent.vue*

```
import CounterMixin from "./CounterMixin";

export default {
  mixins: [CounterMixin],
  methods: {
    decrement() {
      this.count--;
    }
  }
}

```

#### 选项合并

如果组件内的选项跟 mixin 冲突怎么办？

比如，给组件定义一个自带的`increment` 方法，哪个优先级更高呢？
*MyComponent.vue*

```
import CounterMixin from "./CounterMixin";

export default {
  mixins: [CounterMixin],
  methods: {
    // 自带的 `increment`` 方法会覆盖 mixin 的`increment` 吗？
    increment() { ... }
  }
}

```

这个时候就要说到 Vue 的合并策略了。Vue 有一系列的规则，决定了如何处理同名选项。

通常，组件自带的选项会覆盖来自 mixin 的选项。但也有例外，比如同类型的生命周期钩子，不是直接覆盖，而是都放进数组，按顺序执行。

你也可以通过 [自定义合并策略](https://vuejs.org/v2/guide/mixins.html#Custom-Option-Merge-Strategies) 改变默认行为。
#### mixin 的缺点

作为扩展组件的一种模式，mixin 对于简单的场景还算好用，一旦规模扩大，问题就来了。不仅需要注意命名冲突问题（尤其是第三方 mixin），使用了多个 mixin 的组件，很难搞清楚某个功能到底来自于哪里，定位问题也比较困难。
## 高阶组件

高阶组件（HOC）是从 React 借用的概念，Vue 也能使用。

为了理解这个概念，我们先抛开组件，看看两个简单的 JavaScript 函数，`increment` 和 `double`。
```
function increment(x) {
  return x++;
}

function double(x) {
  return x * 2;
}

```

假设我们想给这两个函数都加一个功能：在控制台输出结果。

为此，我们可以用*高阶函数*模式，新建一个 `addLogging`函数，接受函数作为参数，并返回一个带有新增功能的函数。
```
function addLogging(fn) {
  return function(x) {
    const result = fn(x);
    console.log("The result is: ", result);
    return result;
  };
}

const incrementWithLogging = addLogging(increment);
const doubleWithLogging = addLogging(double);

```

组件如何利用这种模式呢？类似地，我们创建一个高阶组件来渲染*Counter*组件，同时添加一个`decrement`方法作为实例属性。

实际代码比较复杂，这里只给出伪代码作为示意：
```
import Counter from "./Counter";

// 伪代码
const CounterWithDecrement => ({
  render(createElement) {
    const options = {
      decrement() {
        this.count--;
      }
    }
    return createElement(Counter, options);
  }
});

```

HOC 模式比 mixin 更简洁，扩展性更好，但是代价是增加了一个包裹组件，实现起来也需要技巧。

## 无渲染组件

如果需要在多个组件上使用相同的逻辑和状态，只是展示方式不同，那么就可以考虑*无渲染组件*模式。

该模式需要用到两类组件：*逻辑*组件用于声明逻辑和状态，*展示*组件用于展示数据。

#### 逻辑组件

还是回到*Counter*的例子，假设我们需要在多个地方重用这个组件，但是展示方式不同。

创建一个*CounterRenderless.js* 用于定义逻辑组件，包含逻辑和状态，但是不包含模板，而是通过 `render`函数声明 `scoped slot`。

`scoped slot`暴露三个属性给父组件使用：状态`count`，方法`increment` 和计算属性 `double`。
*CounterRenderless.js*

```
export default {
  data: () => ({
    count: 0
  }),
  methods: {
    increment() {
      this.count++;
    }
  },
  computed: {
    double () {
      return this.count * 2;
    }
  },
  render() {
    return this.$scopedSlots.default({
      count: this.count,
      double: this.double,
      increment: this.toggleState,
    })
  }
}

```
这里的`scoped slot`是这种模式里逻辑组件的关键所在。

#### 展示组件

接下来是*展示组件*，作为无渲染组件的使用方，提供具体的展示方式。

所有的元素标签都包含在`scoped slot`里。可以看到，这些属性在使用上跟模板直接放在逻辑组件里没什么两样。
*CounterWithButton.vue*

```
<template>
  <counter-renderless slot-scope="{ count, double, increment }">
    <div>Count is: {{ count }}</div> 
    <div>Double is: {{ double }}</div>
    <button @click="increment">Increment</button>
  </counter-renderless>
</template>
<script>
import CounterRenderless from "./CountRenderless";
export default {
  components: {
    CounterRenderless
  }
}
</script>

```

无渲染组件模式非常灵活，也容易理解。但是，它没有前面那几种方法那么通用，可能只有一种应用场景，那就是用于开发组件库。

## 模板扩展

上面的 API 也好，设计模式也罢，都有一种局限性，就是无法扩展组件的**模板**。Vue 在逻辑和状态方面有办法重用，但是对于模板标签就无能为力了。

有一种比较 hack 的方式，就是利用 HTML 预处理器，比如 Pug，来处理模板扩展。

第一步是创建一个基础模板*.pug*文件，包含公共的页面元素。还要包含一个 `block input` ，作为模板扩展的占位符。

*BaseTemplate.pug*

```
div.wrapper
  h3 {{ myCommonProp }} <!--common markup-->
  block input <!--extended markup outlet -->

```

为了能扩展这个模板，需要安装 Vue Loader 的 Pug 插件。然后就可以引入基础模板并利用`block input`语法替换占位部分了：
*MyComponent.vue*

```
<template lang="pug">
  extends BaseTemplate.pug
  block input
    h4 {{ myLocalProp }} <!--gets included in the base template-->
</template>

```

一开始你可能会认为它跟 slot 的概念是一样的，但是有个区别，这里的基础模板不属于任何单独的组件。它在**编译时**跟当前组件合并，而不是像 slot 那样是在**运行时**替换。

参考资料：
*   [Mixins Considered Harmful - Dan Abramov](https://reactjs.org/blog/2016/07/13/mixins-considered-harmful.html)
*   [Renderless Components in Vue.js - Adam Wathan](https://adamwathan.me/renderless-components-in-vuejs/)
