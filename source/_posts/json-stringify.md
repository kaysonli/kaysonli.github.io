---
title: 隐藏实力的 JSON.stringify，原来还可以这么用！
date: 2019-11-16 20:04:54
tags: JavaScript
---
![JSON](/uploads/1618526-e8c92f37fd854ad8.webp)

JavaScript 有许多各司其职的函数。我们每天都在用，但不知道他们的额外功能。直到有一天看了文档之后才发现，它们原来有许多超出我们想象的功能。`JSON.stringify`  就是其中一个。今天我们就来聊聊这个隐藏实力的选手。

### 基本用法

`JSON.stringify` 方法接受一个参数并将其转换成JSON 字符串形式。
```
const firstItem = { 
  title: 'Transformers', 
  year: 2007 
};

JSON.stringify(firstItem);
// {'title':'Transformers','year':2007}

```

当出现一个不能序列化成 JSON 字符串的元素时就会出现问题 。

```
const secondItem = { 
  title: 'Transformers', 
  year: 2007, 
  starring: new Map([[0, 'Shia LaBeouf'],[1, 'Megan Fox']]) 
};

JSON.stringify(secondItem);
// {'title':'Transformers','year':2007,'starring':{}}

```

### 第二个参数
<!-- more -->

`JSON.stringify` 还有第二个参数，叫替换器参数。

你可以传一个字符串数组，作为对象属性白名单，这些属性将会包含在输出结果里。
```
JSON.stringify(secondItem, ['title']);
// {'title':'Transformers'}

```

我们可以筛掉一些不想要的值。这些值要么太大（比如 Error 对象），要么无法转成可读的JSON形式。

替换器参数也可以是一个函数。该函数接受 `JSON.stringify` 方法遍历对象时当前的属性和值作为参数。如果函数不返回任何值或者返回 undefined，当前节点就不会出现在结果里。
```
JSON.stringify(secondItem, (key, value) => {
  if (value instanceof Set) {
    return [...value.values()];
  }
  return value;
});
// {'title':'Transformers','year':2007,'starring':['Shia LaBeouf','Megan Fox']}

```

通过让函数返回 undefined，可以在结果里删除这些属性。
```
JSON.stringify(secondItem, (key, value) => {
  if (typeof value === 'string') {
    return undefined;
  }
  return value;
});
// {"year":2007,"starring":{}}

```
第二个参数还可以用来创建简单的对象哈希函数。但有一点要注意，`JSON.stringify(obj)`不能保证属性的输出顺序，当序列化的结果用于哈希/校验和的时，这点至关重要。为此，我们可以把第二个参数设置为`Object.keys(obj).sort()`，对象将会以这个顺序序列化。
```
function objectHash(obj: object): string {
  const str = JSON.stringify(obj, Object.keys(obj).sort());
  return createHash('sha1').update(str).digest('hex');
}
```

### 第三个参数。

第三个参数设置最终字符串里的空白缩进。如果参数是一个数字，那么序列化的每个层级都会用这个数量的空格符缩进。
```
JSON.stringify(secondItem, null, 2);
//{
//  "title": "Transformers",
//  "year": 2007,
//  "starring": {}
//}

```

如果第三个参数是字符串，就会替代空格符。
```
JSON.stringify(secondItem, null, '🦄');
//{
//🦄"title": "Transformers",
//🦄"year": 2007,
//🦄"starring": {}
//}

```

### toJSON 方法

如果我们序列化的对象有一个`toJSON`方法，它将会采用自定义的序列化过程。你可以在方法里返回一个新的值，这个值将会替换原始对象被序列化。
```
const thirdItem = { 
  title: 'Transformers', 
  year: 2007, 
  starring: new Map([[0, 'Shia LaBeouf'],[1, 'Megan Fox']]),
  toJSON() {
    return { 
      name: `${this.title} (${this.year})`, 
      actors: [...this.starring.values()] 
    };
  }
};

console.log(JSON.stringify(thirdItem));
// {"name":"Transformers (2007)","actors":["Shia LaBeouf","Megan Fox"]}

```

### 最后

查看我们日常使用的函数的文档，有时候会有不少收获。可能会有惊喜，我们也能学到不少东西。保持对知识的渴求，经常读读文档吧！

