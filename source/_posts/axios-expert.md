---
title: axios 专家指南
date: 2019-10-20 20:04:54
tags: JavaScript
---


前端程序与服务器通信的最常见方式是通过HTTP协议。你可能熟悉[Fetch API](https://blog.logrocket.com/axios-or-fetch-api/)和`XMLHttpRequest` 接口，用来获取资源并发出HTTP请求。

如果你使用的是JavaScript库，那么它可能附带一个客户端HTTP API。例如，jQuery的 `$.ajax()`函数在前端开发人员中一度特别流行。但随着开发人员从这些库转向原生API，专用HTTP客户端应运而生，填补了这一空白。

在这篇文章中，我们将深入了解Axios，这是一个基于浏览器 `XMLHttpRequest` 接口的客户端HTTP API，并研究导致它在前端开发人员中越来越受欢迎的关键特性。
## 为什么是Axios?

与Fetch一样，Axios也是基于promise的。但是，它提供了一个更强大、更灵活的功能集。
* 请求和响应拦截
* 简化错误处理
* 防止XSRF
* 支持上传进度
* 响应超时
* 取消请求的能力
* 支持旧的浏览器
* 自动JSON数据转换
<!-- more -->
## 安装

你可以使用以下工具安装Axios:

*   npm:
```
$ npm install axios
```

*   Bower 包管理器:

```
$ bower install axios
```

*   或CDN：

```
<script  src="https://unpkg.com/axios/dist/axios.min.js"></script>
```

[](https://logrocket.com/for/angular-error-handling/)

## 发请求

发送HTTP请求很简单，只要将配置对象传递给Axios函数就行。最简单的形式下，对象必须有一个`url`属性；如果没有提供请求方法，则使用`GET`作为默认值。让我们来看一个简单的例子：
```
// send a POST request
axios({
  method: 'post',
  url: '/login',
  data: {
    firstName: 'Finn',
    lastName: 'Williams'
  }
});
```

这对于用过[jQuery](https://blog.logrocket.com/history-and-legacy-jquery/)`$.ajax`函数的人来说应该很熟悉。这段代码只是让Axios向`/login`发送一个POST请求 ，同时带上数据对象键/值对。Axios将自动将数据转换为JSON并将其作为请求体发送。
## 简写方法

Axios还提供了一组简写方法，用于执行不同类型的请求。具体方法如下:
*   `axios.request(config)`
*   `axios.get(url[, config])`
*   `axios.delete(url[, config])`
*   `axios.head(url[, config])`
*   `axios.options(url[, config])`
*   `axios.post(url[, data[, config]])`
*   `axios.put(url[, data[, config]])`
*   `axios.patch(url[, data[, config]])`

例如，下面的代码展示了如何使用`axios.post()`方法编写前面的示例：
```
axios.post('/login', {
  firstName: 'Finn',
  lastName: 'Williams'
});
```

## 处理响应

一旦发出HTTP请求，Axios将返回一个promise ，根据后端服务的响应，promise 将被执行或拒绝。要处理结果，你可以像这样使用 `then()` 方法:
```
axios.post('/login', {
  firstName: 'Finn',
  lastName: 'Williams'
})
.then((response) => {
  console.log(response);
}, (error) => {
  console.log(error);
});
```

如果promise 被解决，则调用`then()` 的第一个参数；如果promise 被拒绝，则调用第二个参数。根据[文档](https://www.npmjs.com/package/axios#response-schema)， fulfillment 值是一个包含以下信息的对象:
```
{
  // `data` is the response that was provided by the server
  data: {},
 
  // `status` is the HTTP status code from the server response
  status: 200,
 
  // `statusText` is the HTTP status message from the server response
  statusText: 'OK',
 
  // `headers` the headers that the server responded with
  // All header names are lower cased
  headers: {},
 
  // `config` is the config that was provided to `axios` for the request
  config: {},
 
  // `request` is the request that generated this response
  // It is the last ClientRequest instance in node.js (in redirects)
  // and an XMLHttpRequest instance the browser
  request: {}
}
```

举个例子，下面是请求GitHub API的数据响应:
```
axios.get('https://api.github.com/users/mapbox')
  .then((response) => {
    console.log(response.data);
    console.log(response.status);
    console.log(response.statusText);
    console.log(response.headers);
    console.log(response.config);
  });

// logs:
// => {login: "mapbox", id: 600935, node_id: "MDEyOk9yZ2FuaXphdGlvbjYwMDkzNQ==", avatar_url: "https://avatars1.githubusercontent.com/u/600935?v=4", gravatar_id: "", …}
// => 200
// => OK
// => {x-ratelimit-limit: "60", x-github-media-type: "github.v3", x-ratelimit-remaining: "60", last-modified: "Wed, 01 Aug 2018 02:50:03 GMT", etag: "W/"3062389570cc468e0b474db27046e8c9"", …}
// => {adapter: ƒ, transformRequest: {…}, transformResponse: {…}, timeout: 0, xsrfCookieName: "XSRF-TOKEN", …}
```

## 发起并发请求

Axios的一个更有趣的特性是，它能够通过向 `axios.all()`方法传递一个参数数组来并行地发出多个请求。此方法返回一个promise对象，该对象仅在参数数组中所有值都解决后才解决。这里有一个简单的例子:
```
// execute simultaneous requests 
axios.all([
  axios.get('https://api.github.com/users/mapbox'),
  axios.get('https://api.github.com/users/phantomjs')
])
.then(responseArr => {
  //this will be executed only when all requests are complete
  console.log('Date created: ', responseArr[0].data.created_at);
  console.log('Date created: ', responseArr[1].data.created_at);
});

// logs:
// => Date created:  2011-02-04T19:02:13Z
// => Date created:  2017-04-03T17:25:46Z
```

这段代码向GitHub API发出两个请求，然后将每个响应的`created_at`属性值记录到控制台。记住，如果数组参数中的任何一个被拒绝，那么整个promise就会立即被拒绝，并将第一个被拒绝的promise的原因传给它。

为了方便起见，Axios还提供了一个名为`axios.spread()` 的方法，将响应数组的属性分配给单独的变量。你可以这样使用这个方法:
```
axios.all([
  axios.get('https://api.github.com/users/mapbox'),
  axios.get('https://api.github.com/users/phantomjs')
])
.then(axios.spread((user1, user2) => {
  console.log('Date created: ', user1.data.created_at);
  console.log('Date created: ', user2.data.created_at);
}));

// logs:
// => Date created:  2011-02-04T19:02:13Z
// => Date created:  2017-04-03T17:25:46Z
```

这段代码的输出与前面的示例相同。惟一的区别是`axios.spread()` 方法用于从响应数组解包值。
## 发送自定义头部

使用Axios发送发送自定义头部非常简单。只需传递一个包含headers 的对象作为最后一个参数。例如:
```
const options = {
  headers: {'X-Custom-Header': 'value'}
};

axios.post('/save', { a: 10 }, options);
```

## 请求和响应数据转换

默认情况下，Axios自动将请求和响应转换为JSON。但是它也允许你覆盖默认行为并定义一个不同的转换机制。这种能力在处理只接受特定数据格式(如XML或CSV)的API时特别有用。

要更改发送到服务器之前的请求数据，请在配置对象中设置`transformRequest` 属性。请注意，此方法只适用于 `PUT`, `POST`, 和`PATCH` 请求方法。你可以这样做:
```
const options = {
  method: 'post',
  url: '/login',
  data: {
    firstName: 'Finn',
    lastName: 'Williams'
  },
  transformRequest: [(data, headers) => {
    // transform the data

    return data;
  }]
};

// send the request
axios(options);
```

要在将数据传递给 `then()`或`catch()`之前修改数据，你可以设置 `transformResponse` 属性:
```
const options = {
  method: 'post',
  url: '/login',
  data: {
    firstName: 'Finn',
    lastName: 'Williams'
  },
  transformResponse: [(data) => {
    // transform the response

    return data;
  }]
};

// send the request
axios(options);
```

## 拦截请求和响应

HTTP拦截是Axios的一个受欢迎的特性。有了这个特性，你可以检查和更改从程序到服务器的HTTP请求，反之亦然，这对于各种隐式任务非常有用，比如日志记录和身份验证。

乍一看，拦截器与转换非常相似，但它们在一个关键方面有所不同:数据转换仅接收数据和头作为参数，拦截器接收整个响应对象或请求配置。
你可以这样在Axios中声明一个请求拦截器:
```
// declare a request interceptor
axios.interceptors.request.use(config => {
  // perform a task before the request is sent
  console.log('Request was sent');

  return config;
}, error => {
  // handle the error
  return Promise.reject(error);
});

// sent a GET request
axios.get('https://api.github.com/users/mapbox')
  .then(response => {
    console.log(response.data.created_at);
  });
```

无论何时发送请求，此代码都会将消息记录到控制台，然后等待，直到从服务器获得响应，然后将在GitHub上创建帐户的时间打印到控制台。使用拦截器的一个优点是你不必再分别实现每个HTTP请求的任务。

Axios还提供了一个响应拦截器，它允许你在从服务器返回应用程序的途中转换响应:
```
// declare a response interceptor
axios.interceptors.response.use((response) => {
  // do something with the response data
  console.log('Response was received');

  return response;
}, error => {
  // handle the response error
  return Promise.reject(error);
});

// sent a GET request
axios.get('https://api.github.com/users/mapbox')
  .then(response => {
    console.log(response.data.created_at);
  });
```
## 防止XSRF的客户端支持

跨站点请求伪造(或简称XSRF)是一种攻击web托管应用程序的方法，在这种方法中，攻击者将自己伪装成合法和受信任的用户，以影响应用程序和用户浏览器之间的交互。有许多方法可以执行这样的攻击，包括`XMLHttpRequest`。

幸运的是，Axios的设计目的是通过允许你在发出请求时嵌入额外的身份验证数据来防止XSRF。这使服务器能够发现来自未授权位置的请求。下面看看Axios如何做到这一点:
```
const options = {
  method: 'post',
  url: '/login',
  xsrfCookieName: 'XSRF-TOKEN',
  xsrfHeaderName: 'X-XSRF-TOKEN',
};

// send the request
axios(options);
```

## 监视 POST 请求进度

Axios的另一个有趣的特性是监视请求进度的能力。这在下载或上传大文件时特别有用。Axios文档提供的[示例](https://github.com/axios/axios/blob/master/examples/upload/index.html)为你展示了如何实现这一功能。但是为了简单和样式，我们将在本教程中使用[Axios Progress Bar](https://github.com/rikmms/progress-bar-4-axios/)模块。

使用这个模块要做的第一件事是引入相关的样式和脚本:
```
<link rel="stylesheet" type="text/css" href="https://cdn.rawgit.com/rikmms/progress-bar-4-axios/0a3acf92/dist/nprogress.css" />

<script src="https://cdn.rawgit.com/rikmms/progress-bar-4-axios/0a3acf92/dist/index.js"></script>
```

然后我们可以实现这样的进度条：
```
loadProgressBar()

const url = 'https://media.giphy.com/media/C6JQPEUsZUyVq/giphy.gif';

function downloadFile(url) {
  axios.get(url)
  .then(response => {
    console.log(response)
  })
  .catch(error => {
    console.log(error)
  })
}

downloadFile(url);
```

要更改进度条的默认样式，我们可以覆盖以下样式规则:
```
#nprogress .bar {
    background: red !important;
}

#nprogress .peg {
    box-shadow: 0 0 10px red, 0 0 5px red !important;
}

#nprogress .spinner-icon {
    border-top-color: red !important;
    border-left-color: red !important;
}
```

## 取消请求

在某些情况下，你可能不再关心结果，并希望取消已发送的请求。这可以通过使用cancel token来完成。在1.5版中，Axios增加了取消请求的功能，它基于[可取消promise提案](https://github.com/tc39/proposal-cancelable-promises)。这里有一个简单的例子:
```
const source = axios.CancelToken.source();

axios.get('https://media.giphy.com/media/C6JQPEUsZUyVq/giphy.gif', {
  cancelToken: source.token
}).catch(thrown => {
  if (axios.isCancel(thrown)) {
    console.log(thrown.message);
  } else {
    // handle error
  }
});

// cancel the request (the message parameter is optional)
source.cancel('Request canceled.');
```

你还可以将执行器函数传递给`CancelToken`构造函数来创建一个cancel token，如下所示:
```
const CancelToken = axios.CancelToken;
let cancel;

axios.get('https://media.giphy.com/media/C6JQPEUsZUyVq/giphy.gif', {
  // specify a cancel token
  cancelToken: new CancelToken(c => {
    // this function will receive a cancel function as a parameter
    cancel = c;
  })
}).catch(thrown => {
  if (axios.isCancel(thrown)) {
    console.log(thrown.message);
  } else {
    // handle error
  }
});
```

## 相关库

Axios在开发人员中越来越受欢迎，因此出现了大量扩展其功能的第三方库。从测试到日志，在使用Axios时你可能需要的几乎所有附加功能都有相应的库。以下是一些目前流行的库：
*   [axios-vcr](https://github.com/nettofarah/axios-vcr): 📼用JavaScript记录和回放请求
*   [axios-response-logger](https://github.com/srph/axios-response-logger): 📣 记录响应的Axios 拦截器
*   [axios-method-override](https://github.com/jacobbuck/axios-method-override): ⛵️ Axios 请求方法重写插件
*   [axios-extensions](https://github.com/kuitos/axios-extensions): 🍱 Axios 扩展库，包括节流和缓存 GET 请求功能
*   [axios-api-versioning](https://weffe.github.io/axios-api-versioning): 向Axios添加易于管理的API版本控制
*   [axios-cache-plugin](https://github.com/jin5354/axios-cache-plugin): 帮助你在使用Axios时缓存GET请求
*   [axios-cookiejar-support](https://github.com/3846masa/axios-cookiejar-support)：为Axios添加tough-cookie支持
*   [react-hooks-axios](https://github.com/use-hooks/react-hooks-axios): Axios 的自定义 React Hooks
*   [moxios](https://github.com/axios/moxios): 模拟Axios请求以便测试
*   [redux-saga-requests](https://github.com/klis87/redux-saga-requests): 简化 AJAX 请求处理的Redux-Saga 插件
*   [axios-fetch](https://github.com/lifeomic/axios-fetch):由 Axios客户端支持的 Web API Fetch 实现
*   [axios-curlirize](https://www.npmjs.com/package/axios-curlirize): 将任何 Axios 请求以 curl 命令的形式记录到控制台
*   [axios-actions](https://github.com/davestewart/axios-actions): 将端点打包为可调用、可重用的服务
*   [mocha-axios](https://github.com/jdrydn/mocha-axios): Mocha 使用 Axios 的HTTP 断言
*   [axios-mock-adapter](https://github.com/ctimmerm/axios-mock-adapter):轻松模拟请求的Axios适配器
*   [axios-debug-log](https://github.com/Gerhut/axios-debug-log): 使用调试库记录请求和响应的Axios拦截器
*   [redux-axios-middleware](https://github.com/svrcekmichal/redux-axios-middleware): 使用Axios HTTP客户端获取数据的Redux中间件
*   [axiosist](https://github.com/Gerhut/axiosist): 基于Axios的超级测试：将Node.js请求处理程序转换为Axios适配器，用于Node.js服务器单元测试

## 浏览器支持情况

在浏览器支持方面，Axios非常可靠。甚至像ie11这样的老浏览器也能很好地与Axios配合使用。
![浏览器支持](/uploads/1618526-7d1eda50cf15a831.webp?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 总结

Axios在开发人员中如此流行是有原因的：它包含了很多有用的特性。在这篇文章中，我们仔细研究了Axios的几个关键特性，并学习了如何在实践中使用它们。但是Axios还有许多方面我们还没有讨论。因此，请务必查看[Axios GitHub页面](https://github.com/axios/axios)以了解更多信息。

你对使用Axios有什么建议吗?请在评论中告诉我!

[原文]([https://blog.logrocket.com/how-to-make-http-requests-like-a-pro-with-axios/#new_tab](https://blog.logrocket.com/how-to-make-http-requests-like-a-pro-with-axios/#new_tab)
)