---
title: 这15个Vue指令，让你的项目开发爽到爆
date: 2019-10-11 18:07:28
tags: Vue.js
---


受 AngularJS 的启发，Vue 内置了一些非常有用的指令（比如`v-html` 和 `v-once`等），每个指令都有自身的用途。完整的指令列表可以在[这里查看](https://vuejs.org/v2/api/#Directives).

这还没完，更棒的是可以开发自定义指令。Vue.js 社区因此得以通过发布自定义指令npm 包，解决了无数的代码问题。

以下就是我最喜欢的 Vue.js 自定义指令列表。不用说，这些指令为我的项目开发节省了大量时间！😇

## 1\. V-Hotkey

**仓库地址**: [https://github.com/Dafrok/v-hotkey](https://github.com/Dafrok/v-hotkey)
**Demo**: [戳这里 https://dafrok.github.io/v-hotkey](https://dafrok.github.io/v-hotkey/#/)
**安装**: `npm install --save v-hotkey`
这个指令可以给组件绑定一个或多个快捷键。你想要通过按下 Escape 键后隐藏某个组件，按住 Control 和回车键再显示它吗？小菜一碟：

```
<template>
  <div
    v-show="show"
    v-hotkey="{
      'esc': onClose,
      'ctrl+enter': onShow
    }"
  >
	  Press `esc` to close me!
  </div>
</template>

<script>
export default {
    data() {
        return {
            show: true
        }
    },

    methods: {
        onClose() {
            this.show = false
        },

        onShow() {
            this.show = true
        },
    }
}
</script>

```

## 2\. V-Click-Outside

**仓库地址**: [https://github.com/ndelvalle/v-click-outside](https://github.com/ndelvalle/v-click-outside)
**Demo**: [https://codesandbox.io/s/zx7mx8y1ol?module=%2Fsrc%2Fcomponents%2FHelloWorld.vue](https://codesandbox.io/s/zx7mx8y1ol?module=%2Fsrc%2Fcomponents%2FHelloWorld.vue)
**安装**: `npm install --save v-click-outside`

你想要点击外部区域关掉某个组件吗？用这个指令可以轻松实现。这是我每个项目必用的指令之一，尤其在弹框和下拉菜单组件里非常好用。

```
<template>
  <div
    v-show="show"
    v-click-outside="onClickOutside"
  >
    Hide me when a click outside this element happens
  </div>
</template>

```

HTML

```
<script>
export default {
  data() {
    return {
      show: true
    };
  },

  methods: {
    onClickOutside() {
      this.show = false;
    }
  }
};
</script>

```

**说明：** 你也可以通过双击外部区域来触发，具体用法请参考文档。

## 3\. V-Clipboard

**仓库地址**: [https://github.com/euvl/v-clipboard](https://github.com/euvl/v-clipboard)
**安装**: `npm install --save v-clipboard`

这个简单指令的作者是[Yev Vlasenko](https://github.com/euvl) ，可以用在任何静态或动态元素上。当元素被点击时，指令的值会被复制到剪贴板上。用户需要复制代码片段的时候，这个非常有用。

```
<button v-clipboard="value">
  Copy to clipboard
</button>

```

HTML

## 4\. Vue-ScrollTo

**仓库地址**: [https://github.com/rigor789/vue-scrollTo](https://github.com/rigor789/vue-scrollTo)
**Demo**: [https://vue-scrollto.netlify.com/](https://vue-scrollto.netlify.com/)
**安装**: `npm install --save vue-scrollto`

这个指令监听元素的点击事件，然后滚动到指定位置。我通常用来处理文章目录跳转和导航跳转。

```
<span v-scroll-to="{
  el: '#element',          // 滚动的目标位置元素
  container: '#container', // 可滚动的容器元素
  duration: 500,           // 滚动动效持续时长（毫秒）
  easing: 'linear'         // 动画曲线
  }"
>
  Scroll to #element by clicking here
</span>

```

**说明：** 也可以通过代码动态设置，具体看文档。

## 5\. Vue-Lazyload

**仓库地址**: [https://github.com/hilongjw/vue-lazyload](https://github.com/hilongjw/vue-lazyload)
**Demo**: [http://hilongjw.github.io/vue-lazyload/](http://hilongjw.github.io/vue-lazyload/)
**安装**: `npm install --save vue-lazyload`
图片懒加载，非常方便。

```
<img v-lazy="https://www.domain.com/image.jpg">
```

## 6\. V-Tooltip

**仓库地址**: [v-tooltip](https://github.com/Akryum/v-tooltip)
**Demo**: [available here](https://akryum.github.io/v-tooltip/#/)
**安装**: `npm install --save v-tooltip`
几乎每个项目都会用到 tooltip。这个指令可以给元素添加响应式的tooltip，并可控制显示位置、触发方式和监听事件。

```
<button v-tooltip="'You have ' + count + ' new messages.'">

```

**说明：** 还有一个比较流行的tooltip插件[vue-directive-tooltip](https://app.asana.com/0/0/1114593780251778).

## 7\. V-Scroll-Lock

**仓库地址**: [https://github.com/phegman/v-scroll-lock](https://github.com/phegman/v-scroll-lock)
**Demo**: [https://v-scroll-lock.peterhegman.com/](https://v-scroll-lock.peterhegman.com/)
**安装**: `npm install --save v-scroll-lock`

基于 [body-scroll-lock](https://github.com/willmcpo/body-scroll-lock) 开发，这个指令的作用是在打开模态浮层的时候防止下层的元素滚动。

```
<template>
  <div class="modal" v-if="opened">
    <button @click="onCloseModal">X</button>
    <div class="modal-content" v-scroll-lock="opened">
      <p>A bunch of scrollable modal content</p>
    </div>
  </div>
</template>

```


```
<script>
export default {
  data () {
    return {
      opened: false
    }
  },
  methods: {
    onOpenModal () {
      this.opened = true
    },

    onCloseModal () {
      this.opened = false
    }
  }
}
</script>

```


## 8\. V-Money

**仓库地址**: [https://github.com/vuejs-tips/v-money](https://github.com/vuejs-tips/v-money)
**Demo**: [https://vuejs-tips.github.io/v-money/](https://vuejs-tips.github.io/v-money/)
**安装**: `npm install --save v-money`
如果你需要在输入框里加上货币前缀或后缀、保留小数点位数或者设置小数点符号——不用找了，就是它！一行代码搞定这些需求：
```
<template>
  <div>
    <input v-model.lazy="price" v-money="money" /> {{price}}
  </div>
</template>

```

```
<script>
export default {
  data () {
    return {
      price: 123.45,
      money: {
        decimal: ',',
        thousands: '.',
        prefix: '$ ',
        precision: 2,
      }
    }
  }
}
</script>

```


## 9\. Vue-Infinite-Scroll

**仓库地址**: [https://github.com/ElemeFE/vue-infinite-scroll](https://github.com/ElemeFE/vue-infinite-scroll)
**安装**: `npm install --save vue-infinite-scroll`

无限滚动指令，当滚动到页面底部时会触发绑定的方法。

```
<template>
  <!-- ... -->
  <div
    v-infinite-scroll="onLoadMore"
    infinite-scroll-disabled="busy"
    infinite-scroll-distance="10"
  ></div>
<template>

```

```
<script>
export default {
  data() {
    return {
      data [],
      busy: false,
      count: 0
    }
  },

  methods: {
    onLoadMore() {
      this.busy = true;

      setTimeout(() => {
        for (var i = 0, j = 10; i < j; i++) {
          this.data.push({ name: this.count++ });
        }
        this.busy = false;
      }, 1000);
    }
  }
}
</script>

```

## 10\. Vue-Clampy

**仓库地址**: [vue-clampy](https://github.com/clampy-js/vue-clampy).
**安装**: `npm install --save @clampy-js/vue-clampy`

这个指令会截断元素里的文本，并在末尾加上省略号。它是用clampy.js实现的。

```
  <p v-clampy="3">Long text to clamp here</p>
  <!-- displays: Long text to...-->

```

## 11\. Vue-InputMask

**仓库地址**: [vue-inputmask](https://github.com/scleriot/vue-inputmask)
**安装**: `npm install --save vue-inputmask`
当你需要在输入框里格式化日期时，这个指令会自动生成格式化文本。基于[Inputmask library](https://github.com/RobinHerbots/Inputmask) 开发。


```
<input type="text" v-mask="'99/99/9999'" />

```

HTML

## 12\. Vue-Ripple-Directive

**仓库地址**: [vue-ripple-directive](https://github.com/PygmySlowLoris/vue-ripple-directive)
**安装**: `npm install --save vue-ripple-directive`

 [Aduardo Marcos](https://github.com/Edujugon) 写的这个指令可以给点击的元素添加波纹动效。

```
<div v-ripple class="button is-primary">This is a button</div>

```

## 13\. Vue-Focus

**仓库地址**: [vue-focus](https://github.com/simplesmiler/vue-focus)
**安装**: `npm install --save vue-focus`
有时候，用户在界面里操作，需要让某个输入框获得焦点。这个指令就是干这个的。

```
<template>
  <button @click="focused = true">Focus the input</button>

  <input type="text" v-focus="focused">
</template>

```

```
<script>
export default {
  data: function() {
    return {
      focused: false,
    };
  },
};
</script>

```


## 14\. V-Blur

**仓库地址**: [v-blur](https://github.com/ndelvalle/v-blur)
**Demo**: [戳这里](https://codesandbox.io/s/823o069zoj?module=%2Fsrc%2Fcomponents%2FHelloWorld.vue)
**安装**: `npm install --save v-blur`
假设你的页面在访客没有注册的时候，有些部分需要加上半透明遮罩。用这个指令可以轻松实现，还可以自定义透明度和过渡效果。

```
<template>
  <button
    @click="blurConfig.isBlurred = !blurConfig.isBlurred"
  >Toggle the content visibility</button>

  <p v-blur="blurConfig">Blurred content</p>
</template>

```

```
<script>
  export default {
      data () {
        return           
          blurConfig: {
            isBlurred: false,
            opacity: 0.3,
            filter: 'blur(1.2px)',
            transition: 'all .3s linear'
          }
        }
      }
    }
  };
</script>

```


## 15\. Vue-Dummy

**仓库地址**: [vue-dummy](https://github.com/paulcollett/vue-dummy)
**Demo**: [available here](https://paulcollett.github.io/vue-dummy/example/)
**安装**: `npm install --save vue-dummy`
开发 app 的时候，偶尔会需要使用假文本数据，或者特定尺寸的占位图片。用这个指令可以轻松实现。

```
<template>
  <!-- the content inside will have 150 words -->
  <p v-dummy="150"></p>

  <!-- Display a placeholder image of 400x300-->
  <img v-dummy="'400x300'" />
</template>

```


## 总结

欢迎补充更多好用的 Vue 自定义指令。

