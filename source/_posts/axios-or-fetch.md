---
title: axios 和 fetch，应该用哪个？
date: 2019-10-24 20:04:54
tags: JavaScript
---

在最近的一篇文章[axios专家指南](https://www.jianshu.com/p/50d6c71fa2af)中，我论述了使用Axios库的好处。尽管如此，必须承认Axios并不总是一个理想的方案，有时候可以用更好的方法来发送HTTP请求。

毫无疑问，一些开发人员更喜欢Axios而不是原生API，因为它容易使用。但许多人高估了对这样一个库的需求。`fetch()` API完全能够实现Axios的关键功能，而且它还有一个额外的优点，即可以在所有现代浏览器中使用。

本文将比较`fetch()`和Axios，看看如何利用它们完成不同的任务。希望在本文的最后，你能够更好地理解这两个API。
<!-- more -->

## 基本语法

在深入研究Axios的更高级特性之前，让我们将其基本语法与`fetch()`进行比较。下面介绍如何使用Axios将带有自定义头部的`POST` 请求发送到某个URL。Axios自动将数据转换为JSON，所以不必自己操心。
```
// axios

const options = {
  url: 'http://localhost/test.htm',
  method: 'POST',
  headers: {
    'Accept': 'application/json',
    'Content-Type': 'application/json;charset=UTF-8'
  },
  data: {
    a: 10,
    b: 20
  }
};

axios(options)
  .then(response => {
    console.log(response.status);
  });
```

现在将这段代码与`fetch()`版本进行比较，后者产生了相同的结果:
```
// fetch()

const url = 'http://localhost/test.htm';
const options = {
  method: 'POST',
  headers: {
    'Accept': 'application/json',
    'Content-Type': 'application/json;charset=UTF-8'
  },
  body: JSON.stringify({
    a: 10,
    b: 20
  })
};

fetch(url, options)
  .then(response => {
    console.log(response.status);
  });
```

注意：

* 发送数据时，`fetch()`使用body属性，而Axios使用data属性
* `fetch()`中的数据是字符串化的
* URL作为参数传递给`fetch()`。但是在Axios中，URL是在options对象中设置的
## 向后兼容性

Axios的主要卖点之一是其广泛的浏览器支持。甚至像IE11这样的旧浏览器也可以毫无问题地运行Axios。另一方面，`fetch()`只支持Chrome 42+、Firefox 39+、Edge 14+和Safari 10.1+(你可以在[can I use](https://caniuse.com/#search=Fetch))上查看完整支持表格。

如果使用Axios的惟一原因是向后兼容性，那么实际上并不需要HTTP库。相反，您可以使用`fetch()`和一个类似的[polyfill](https://github.com/github/fetch)来实现类似的功能，如果web浏览器上不支持 `fetch()`的话。要使用fetch polyfill，可以通过npm命令安装：

```npm install whatwg-fetch --save```

然后你就可以这样发请求了：

```
import 'whatwg-fetch'
window.fetch(...)
```

请记住，在一些旧的浏览器中可能还需要一个promise polyfill。
## 响应超时

能够简单地设置超时，是一些开发人员更喜欢使用Axios而不是`fetch()`的原因之一。在Axios中，可以利用配置对象中的可选`timeout`属性设置请求中止之前的毫秒数。例如:
```
axios({
  method: 'post',
  url: '/login',
  timeout: 4000,    // 4 seconds timeout
  data: {
    firstName: 'David',
    lastName: 'Pollock'
  }
})
.then(response => {/* handle the response */})
.catch(error => console.error('timeout exceeded'))
```

`fetch()`通过`AbortController`接口提供了类似的功能。但它不像Axios版本那么简单:
```
const controller = new AbortController();
const options = {
  method: 'POST',
  signal: controller.signal,
  body: JSON.stringify({
    firstName: 'David',
    lastName: 'Pollock'
  })
};  
const promise = fetch('/login', options);
const timeoutId = setTimeout(() => controller.abort(), 4000);

promise
  .then(response => {/* handle the response */})
  .catch(error => console.error('timeout exceeded'));
```

这里，我们使用`AbortController`构造函数创建一个`AbortController`对象，它允许我们稍后中止请求。`signal`是`AbortController`的只读属性，提供了与请求通信或中止请求的方法。如果服务器在4秒内没有响应，则调用`controller.abort()`，并终止操作。
## JSON 数据自动转换

如前所述，Axios在发送请求时自动 stringify 数据(尽管你可以覆盖默认行为并定义不同的转换机制)。但是，在使用 `fetch()`时，你必须手动完成。对比下：
```
// axios
axios.get('https://api.github.com/orgs/axios')
  .then(response => {
    console.log(response.data);
  }, error => {
    console.log(error);
  });

// fetch()
fetch('https://api.github.com/orgs/axios')
  .then(response => response.json())    // one extra step
  .then(data => {
    console.log(data) 
  })
  .catch(error => console.error(error));
```

自动转换数据是一个很好的特性，但还是那句话，它不是`fetch()`做不到的事情。
## HTTP 拦截器

Axios的关键特性之一是它能够拦截HTTP请求。当你需要检查或更改从应用程序到服务器的HTTP请求时，HTTP拦截器就派上用场了，反之亦然(例如，日志记录、身份验证等)。使用拦截器，你不必为每个HTTP请求编写单独的代码。

下面是如何在Axios中声明请求拦截器：
```
axios.interceptors.request.use(config => {
  // log a message before any HTTP request is sent
  console.log('Request was sent');

  return config;
});

// sent a GET request
axios.get('https://api.github.com/users/sideshowbarker')
  .then(response => {
    console.log(response.data);
  });
```

在这段代码中， `axios.interceptors.request.use()`方法用于定义在发送HTTP请求之前要运行的代码。

默认情况下，`fetch()`并没有提供拦截请求的方法，但是找到一个变通方法并不难。你可以覆盖全局获取方法并定义自己的拦截器，像这样:
```
fetch = (originalFetch => {
  return (...arguments) => {
    const result = originalFetch.apply(this, arguments);
      return result.then(console.log('Request was sent'));
  };
})(fetch);

fetch('https://api.github.com/orgs/axios')
  .then(response => response.json())
  .then(data => {
    console.log(data) 
  });
```

## 下载进度指示

进度指示器在加载大型资源时非常有用，特别是对于网速较慢的用户。以前，JavaScript程序员使用 `XMLHttpRequest.onprogress`回调函数来实现进度指示器。

[Fetch API](https://blog.logrocket.com/patterns-for-data-fetching-in-react-981ced7e5c56/)没有`onprogress`处理函数。相反，它通过响应对象的body属性提供了一个`ReadableStream` 实例。

下面的例子说明了`ReadableStream`在图片下载过程中为用户提供即时反馈的用法：
```
// 源码: https://github.com/AnthumChris/fetch-progress-indicators

<div id="progress" src="">progress</div>
<img id="img">

<script>

'use strict'

const element = document.getElementById('progress');

fetch('https://fetch-progress.anthum.com/30kbps/images/sunrise-baseline.jpg')
  .then(response => {

    if (!response.ok) {
      throw Error(response.status+' '+response.statusText)
    }

    // ensure ReadableStream is supported
    if (!response.body) {
      throw Error('ReadableStream not yet supported in this browser.')
    }

    // store the size of the entity-body, in bytes
    const contentLength = response.headers.get('content-length');

    // ensure contentLength is available
    if (!contentLength) {
      throw Error('Content-Length response header unavailable');
    }

    // parse the integer into a base-10 number
    const total = parseInt(contentLength, 10);

    let loaded = 0;

    return new Response(

      // create and return a readable stream
      new ReadableStream({
        start(controller) {
          const reader = response.body.getReader();

          read();
          function read() {
            reader.read().then(({done, value}) => {
              if (done) {
                controller.close();
                return; 
              }
              loaded += value.byteLength;
              progress({loaded, total})
              controller.enqueue(value);
              read();
            }).catch(error => {
              console.error(error);
              controller.error(error)                  
            })
          }
        }
      })
    );
  })
  .then(response => 
    // construct a blob from the data
    response.blob()
  )
  .then(data => {
    // insert the downloaded image into the page
    document.getElementById('img').src = URL.createObjectURL(data);
  })
  .catch(error => {
    console.error(error);
  })

function progress({loaded, total}) {
  element.innerHTML = Math.round(loaded/total*100)+'%';
}

</script>
```

在Axios中实现进度指示器更简单，特别是如果使用[Axios Progress Bar](https://github.com/rikmms/progress-bar-4-axios/)模块。首先，你需要引入以下样式和脚本:
```
<link rel="stylesheet" type="text/css" href="https://cdn.rawgit.com/rikmms/progress-bar-4-axios/0a3acf92/dist/nprogress.css" />

<script src="https://cdn.rawgit.com/rikmms/progress-bar-4-axios/0a3acf92/dist/index.js"></script>
```

然后你可以实现这样的进度条：
```
<img id="img">

<script>

loadProgressBar();

const url = 'https://fetch-progress.anthum.com/30kbps/images/sunrise-baseline.jpg';

function downloadFile(url) {
  axios.get(url, {responseType: 'blob'})
    .then(response => {
      const reader = new window.FileReader();
      reader.readAsDataURL(response.data); 
      reader.onload = () => {
        document.getElementById('img').setAttribute('src', reader.result);
      }
    })
    .catch(error => {
      console.log(error)
    });
}

downloadFile(url);

</script>
```

这段代码使用 `FileReader` API异步读取下载的图片。`readAsDataURL`方法以base64编码的字符串形式返回图片数据，然后将其插入`img` 标签的`src`属性中以显示图片。
## 并发请求

要同时发出多个请求，Axios提供了 `axios.all()`方法。只需将一个请求数组传递给这个方法，然后使用`axios.spread()`将响应数组的属性分配给多个变量:
```
axios.all([
  axios.get('https://api.github.com/users/iliakan'), 
  axios.get('https://api.github.com/users/taylorotwell')
])
.then(axios.spread((obj1, obj2) => {
  // Both requests are now complete
  console.log(obj1.data.login + ' has ' + obj1.data.public_repos + ' public repos on GitHub');
  console.log(obj2.data.login + ' has ' + obj2.data.public_repos + ' public repos on GitHub');
}));
```

你可以通过使用原生的`Promise.all()`方法来实现相同的结果。将所有fetch请求作为一个数组传递给Promise.all()。接下来，通过使用 `async`函数来处理响应，如下所示:
```
Promise.all([
  fetch('https://api.github.com/users/iliakan'),
  fetch('https://api.github.com/users/taylorotwell')
])
.then(async([res1, res2]) => {
  const a = await res1.json();
  const b = await res2.json();
  console.log(a.login + ' has ' + a.public_repos + ' public repos on GitHub');
  console.log(b.login + ' has ' + b.public_repos + ' public repos on GitHub');
})
.catch(error => {
  console.log(error);
});
```

## 总结

Axios在一个小巧的包中提供了一个易于使用的API，可以满足大多数HTTP通信需求。但是，如果你希望坚持使用原生API，那么没有什么可以阻止你实现Axios具有的特性。

正如本文所讨论的，使用web浏览器提供的`fetch()`方法完全可以实现Axios库的关键特性。最终，额外加载一个客户端HTTP API是否值得，取决于你是否愿意使用原生API。

