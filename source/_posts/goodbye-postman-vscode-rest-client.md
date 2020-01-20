---
title: 再见了，Postman！HTTP 接口测试出现新杀器
date: 2020-01-14 18:02:50
tags: 开发工具
---


作为 Web 开发人员，开发和调试 REST API 是家常便饭。我们会用一些工具来模拟 HTTP 请求，最常用的可能就是 Postman 了。前不久还冒出一个女版 Postman，那就是 Postwoman，详情可见我之前写的一篇介绍：[还在用 Postman 测试接口？是时候试试 Postwoman 了！](https://www.jianshu.com/p/35efd7324ed6)
这些工具确实很实用，但也存在一些局限性。

通过模拟 HTTP 请求来测试 REST API 是比较常见的做法，但是要编辑接口文档并进行版本管理，或者就是简单地跟团队成员共享，恐怕需要用到更多、更复杂的工具了。

有人说 Postman 付费版本就有这些功能呀。没错，但是要掏钱啊！本着能省则省的原则，有免费工具干嘛不用呢。再者，团队每个人都需要用 Postman 才行，我只想单纯写个代码，不想装那么多工具……

好了，大杀器即将出场。它就是 **REST Client** ！
<!-- more -->
![REST Client 插件](/uploads/1618526-53597424fdd296a3.webp)
[REST Client](https://marketplace.visualstudio.com/items?itemName=humao.rest-client) 是一个 **VS Code 插件**.

装上它，你就可以在 VS Code 里发送 HTTP 请求，查看响应结果。这一切只需要一个文本文件，很容易加入到代码仓库中，方便版本管理。

### 优缺点
它主要的优势就是可以做版本管理和团队共享。

如果你在开发某个 API，你想让其他开发小伙伴们也知道怎么用你的接口，REST Client 就是个很好的帮手。

另外一个优势就是简单。前面说了，它只需要一个**文本文件**。它不但充当了发送请求的参数配置的角色，还可以作为接口文档，当你忘了某个接口怎么用的，直接翻出这个文件看看就行了！

缺点呢？那就是你要用 VS Code……作为 VS Code 铁粉，这根本就不是个事儿！当然，你要是用别的编辑器，那就……话说 VS Code 它不香吗？

### 用法

很简单，只要创建一个扩展名为 `.http` 的文件就行。比如要调用新浪股票的接口，我们新建一个`stock.http`文件：
```
@baseUrl = http://hq.sinajs.cn

GET {{baseUrl}}/list=sh600519
Accept: application/json
```
编辑完后，编辑器里会出现一个 `Send Request` 按钮，点击就可以发送请求了。请求结果会显示在右边的新 tab 里。是不是巨简单？

![image.png](/uploads/1618526-2d68c2b28cab6400.webp?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

通常 REST API 不会这么简单，可能还有其它 `POST`，`PUT`，`DELETE` 等各种请求方法，还会需要请求参数或 Header 等。写法也很简单，其实跟浏览器开发工具里显示的差不多，只要遵循 [RFC 2616](http://www.w3.org/Protocols/rfc2616/rfc2616-sec5.html "http://www.w3.org/Protocols/rfc2616/rfc2616-sec5.html")格式要求，包含请求方法、header和 body 就行了。这里需要注意，header 内容要紧跟请求方法下一行，不能有空行，否则就是无效的。

 ```
POST https://example.com/comments HTTP/1.1
content-type: application/json

{
"name": "sample",
"time": "Wed, 21 Oct 2015 18:27:50 GMT"
}
```
### 进阶

实际项目中可能有更复杂的场景，比如要测试不同环境下的接口。为此，REST Client 提供了各种变量。变量分为两大类：自定义变量和系统变量。顾名思义，自定义变量就是用户自己定义的变量。它又分为环境变量、文件变量和请求变量。系统变量就是插件内置的一些变量，可以直接拿来用。具体用法可以参考官方文档。

举个例子，插件的环境变量是在 VS Code 的配置文件中定义的，也就是 `settings.json`文件：
```
"rest-client.environmentVariables": {
    "$shared": {
        "version": "v1",
        "prodToken": "foo",
        "nonProdToken": "bar"
    },
    "local": {
        "version": "v2",
        "host": "localhost",
        "token": "{{$shared nonProdToken}}",
        "secretKey": "devSecret"
    },
    "production": {
        "host": "example.com",
        "token": "{{$shared prodToken}}",
        "secretKey" : "prodSecret"
    }
}
```
这样，当你切换不同的环境时，就会启用该环境下的变量值。

插件的简单介绍就先到这里，更多功能可以查看插件的[Github 仓库](https://github.com/Huachao/vscode-restclient)。