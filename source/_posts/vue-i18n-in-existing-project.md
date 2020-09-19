---
title: 现有 Vue.js 项目快速实现多语言切换的一种思路
date: 2020-09-16 21:55:11
tags: Vue.js
---
Web 项目多语言（i18n，即国际化）是比较常见的需求，常规的做法大概有以下几种：
1. 每种语言单独开发页面，适用于 CMS 之类的网站
2. 多语言文本和页面结构分离，运行时动态替换。适用于单页应用（SPA）
3. 直接用网页翻译插件，机器翻译。这种效果不太理想，同时有一些局限性（后面会讲到）
## 问题
每一种方案都有各自的优点和局限性，具体项目应该根据实际情况选择。最近在工作中碰到的需求是要在现有的项目基础上快速推出多语言版本。项目是基于 Vue.js 开发的，已经迭代过很多版本了。其实一开始是有规划多语言的，也引进了 `vue-i18n` 插件。这个插件就是上面第二种方案，用 JSON 文件管理多语言的文本资源，在 Vue 组件模板里通过键名引用文本。但是要管理这些英文键名比较麻烦，命名就很头疼。而且阅读代码的时候也很难从键名快速识别出对应的中文。后面发现 VS Code 有相关的插件，可以显示出对应的中文，但是代码找起来还是有点麻烦。再加上产品的多语言版本一直没有提上日程，时间久了就嫌麻烦，慢慢地就直接在模板里写中文了。
<!-- more -->
结果，该来的还是来了。老板突然说最近要推出英文版，后续还有其他语言。一开始的想法是直接用 Chrome 浏览器自带的 Google 翻译功能，怎么快怎么来。但经过一番测试，发现了不少问题。首先机翻的效果肯定是要打折扣的，但这还在接受范围内。最关键的是会影响到功能使用。什么问题呢？由于项目是用 Vue.js 开发的单页应用，页面内容完全是用 JS 动态渲染的。有些对话框内的文字 Google 翻译就忽略了。另外，Google 翻译只处理了 DOM 文本节点，`input`输入框内的文字（包括`placeholder`）被忽略了。最严重的问题是，经过 Google 翻译处理后的 DOM 元素，竟然失去了 Vue 响应式特性，数据变化后 DOM 内的文字不会更新了！

如果要继续采用浏览器 Google 翻译的方案，就要解决这几个问题。通过调试发现 Google 翻译用的 JS 脚本是嵌入到浏览器 VM 里的，通过 HTTP 调用翻译服务，然后修改 DOM 元素。JS 脚本是压缩混淆过的，格式化后也很难看。想要找到更新 DOM 的代码，然后用自己的逻辑去覆盖？眼睛都看瞎了，还是算了。
![Google 翻译JS代码](https://ww1.sinaimg.cn/large/6bb8ee92gy1giwazyzs2dj20lg0i8n6r.jpg)

鉴于以上原因，浏览器自带的 Google 翻译方案基本不考虑了。

现在只剩下第二种方案了，语言配置文件和页面结构分离。前面提过，`vue-i18n`用得不彻底，如果把所有组件重新规范化，工作量太大了。有没有办法不修改现有代码，也能实现文本翻译呢？很自然地就想到了 Google 翻译的思路，直接对页面渲染结果进行翻译。自己翻译的优势就是，可以精细地控制 DOM 操作，比如可以把输入框里的文本和`placeholder`也翻译出来。同时，经过研究发现，Vue 组件通过数据绑定渲染出来的 DOM 元素，包含的文本内容不能直接通过 `innerHTML`或者`innerText`修改，这样会导致响应式失效。解决办法是操作它的子元素，也就是文本节点（`nodeType`为3的节点），修改它的 `textContent`属性。

## 多语言配置映射表
跟 Google 翻译不同之处在于，我们采用静态翻译，也就是通过多语言配置文件映射。 `vue-i18n` 是每种语言准备一个 JSON 文件，属性名用英文，用命名空间（多层级对象）的方式避免命名冲突。我直接简化了，用一个 JS  对象存储所有语言版本，键名就是页面用到的中文。随着日积月累的开发迭代，这些中文散落在几百个文件里……我的做法是用 VS Code 全局正则搜索，把查找结果复制出来，写一个 JS 方法把这些字符串处理成 JS 对象。
![搜索中文](https://ww1.sinaimg.cn/large/6bb8ee92gy1giwb0au85pj20u00ch11c.jpg)
匹配中文的正则（不够全面，有些还夹杂了其他符号）：
```
[A-Z]*[\u4e00-\u9fa5][，,!！ 0-9a-zA-Z\u4e00-\u9fa5]*
```
将结果复制到翻译工具翻译，再写一个函数把这些文本合并成对象，并保存到`labels.js`文件中备用。
```
var kv = dist.reduce((acc,cur, index) => {
acc[cur]=en[index] || cur;return acc;
},{})
```
对象的结构大致如下：
```
// labels.js
export default {
  客户性名: {
    en: 'Customer Name',
  },
 // 动态文本，后面会讲到
 '剩余{0}台矿机未登记': {
    en: '{0} unregistered',
  },
  xxxx: {
    en: 'XXX',
  }
}
```
## 操作 DOM
跟 Google 翻译类似，我们也采取事后更新 DOM 的方式来进行翻译。由于是单页应用，随着用户的操作，会不停地更新 DOM。一开始的想法是监听整个 `body`的变化，在回调里再更新 DOM。监听 DOM  变化有一个原生的 API 可用，就是 `MutationObserver`。
```
mounted() {
  this.observeDOM(document.body);
},
methods: {
  observeDOM(el) {
    let mutationTimer;
    const vm = this;
    const observer = new MutationObserver(() => {
      // 类似于 debounce 的效果，多次调用合并为一次
      clearTimeout(mutationTimer);
      mutationTimer = setTimeout(() => {
        if (!vm.mutationFromTrans) {
          translate();
          vm.mutationFromTrans = true;
          setTimeout(() => {
            vm.mutationFromTrans = false;
          }, 300);
        }
      }, 100);
    });
    const options = {
      childList: true, // 监视node直接子节点的变动
      subtree: true, // 监视node所有后代的变动
      attributes: true, // 监视node属性的变动
      characterData: true, // 监视指定目标节点或子节点树中节点所包含的字符数据的变化。
    };
    if (this.language === 'en') {
      observer.observe(el, options);
    }
  },
},
```
但是试过之后发现这会导致无线循环，因为没有判断 DOM 的变化来自用户操作还是翻译本身。所以代码里后面加了判断，但是结果依然不理想。这种操作代价太大了，页面性能受了很大影响。而且还有个很明显的问题，就是进入到新的界面会闪一下，从中文变成英文。这个体验太糟糕了。后面有改进办法。
## 翻译
先来来看下翻译的过程。翻译就是从多语言配置对象里查找匹配的属性名，获取对应语言的属性值。这对于静态文本来说比较简单，直接用属性名就好了。但是对于动态的文本怎么处理呢？由于中英文表达方式不一样，这种文本不能简单地拆分成多个部分单独处理，而是要在英文的表达方式里替换动态数据。我的做法是使用带格式的键名，比如`{0}`这样的占位符。在查找的时候，优先匹配固定文本。因为大部分情况是固定文本，而且这种匹配是O(1)时间复杂度的，优先判断会提高性能。匹配失败的时候才去提前构造好的正则列表里遍历匹配，成功则提取正则匹配的`group`用于替换动态数据。如果失败，说明没有对应的翻译，直接返回原始字符串就行了。
```
const keys = Object.keys(words);
// 提前缓存正则，避免重复执行消耗性能
const regExps = keys.reduce((acc, key) => {
  // 模板型键名
  if (key.indexOf('{0}') > -1) {
    const reg = new RegExp(key.replace('{0}', '(.+)'));
    acc.push({
      expression: reg,
      key,
    });
  }
  return acc;
}, []);
export function translate(el = document.body, lang = 'en') {
  const kv = words;
  if (!el.querySelectorAll) {
    return;
  }
  const _trans = label => {
    const text = label?.trim?.();
    if (!text) {
      return label;
    }
    if (kv[text]?.[lang]) {
      return kv[text]?.[lang];
    }
    for (let index = 0; index < regExps.length; index++) {
      const regItem = regExps[index];
      const m = text.match(regItem.expression);
      if (m) {
        return kv[regItem.key][lang].replace('{0}', m[1]);
      }
    }
    return text;
  };
  [...el.querySelectorAll('*')].forEach(node => {
    // 不能直接修改node.innerText，会导致Vue响应式失效
    // node.innerText = kv[node.innerText?.trim?.()] || node.innerText;
    if (node.nodeName === 'INPUT' && node.type === 'text') {
      node.value = _trans(node.value);
      node.placeholder = _trans(node.placeholder);
    }
    const textNodes = [...node.childNodes].filter(n => n.nodeType === 3);
    textNodes.forEach(textNode => {
      textNode.textContent = _trans(textNode.textContent);
    });
  });
}

```
## 改进后的 DOM 操作
前面提过，如果在 DOM 渲染后再执行翻译，页面性能非常差。于是想到了 Vue 本身的渲染过程，能不能拦截 Vue 组件渲染过程，插入一些额外的逻辑呢？通过扒源码发现，Vue 原型上有个`__patch__`方法，每次更新 DOM 的时候都会执行。就从这里入手， 重写这个方法，对还没挂载到文档树的 DOM 元素执行翻译操作。

```
const __patch__ = Vue.prototype.__patch__;
Vue.prototype.__patch__ = function() {
  const elm = __patch__.apply(this, arguments);
  if (this.$store?.getters?.language) {
    translate(elm, this.$store?.getters?.language);
  }
  return elm;
};

```
至此，基本完成了多语言翻译。经过权衡对比，这个方案算是比较省时省力又能完成需求的了。当然，这种方案或多或少对页面性能有一定影响，毕竟增加了 DOM 更新的时间。尤其是动态文本较多的情况，涉及到遍历正则匹配，比较耗时。如果大家有更好的方案，欢迎留言！
