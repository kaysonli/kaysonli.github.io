---
title: 【技术风向标】Web 性能优化之：Adaptive Loading（自适应加载）
date: 2019-12-07 15:47:08
tags: web
---

在最近的 [Chrome Dev Summit talk](https://www.youtube.com/watch?v=puUPpVrIRkc) 上，Facebook 的 Nate Schloss 和 Chrome 团队的技术经理 Addy Osmani 谈到了一种 Web 性能优化模式，叫 Adaptive Loading（自适应加载）。

### 什么是 Adaptive Loading

简单来说就是，网页应该根据客户端设备的硬件情况分发不同的内容。

以往我们做 Web 性能优化，可能只会根据屏幕尺寸响应不同的内容。但是我们应该考虑到，互联网用户的设备性能千差万别，在高端机上运行良好的页面，可能在低端机型上根本无法正常浏览。能不能根据客户端设备的实际情况，分发不同的网页内容呢？这就是 Adaptive Loading（自适应加载）模式的做法：
<!-- more -->
* 向所有用户（包括低端设备）分发快速的核心体验
* 渐进式地增加只针对高端设备的特性，如果用户的网络和硬件能处理的话
### 具体做法

具体做法包括：
*  在低速网络上提供低质量的图片和视频
*  仅在高速 CPU 上加载用于交互的非关键 JavaScript
*  限制低端设备上的动画帧率
*  避免在低端设备上进行繁重的计算操作
*  在较慢的设备上阻止第三方脚本

[![image](/uploads/1618526-ff9ea37b75bbf925.webp?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)](https://res.cloudinary.com/practicaldev/image/fetch/s---ThChU4G--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://thepracticaldev.s3.amazonaws.com/i/f9jc248z5x8sw1q78z0o.png) 

以下几个信号可以用于 Adaptive Loading：
*   **网络：** 数据传输调优，以占用更少的带宽(通过[`navigator.connection.effectiveType`](https://developer.mozilla.org/en-US/docs/Web/API/Network_Information_API))。我们还可以利用用户的 Data Saver 偏好设置 (通过[`navigator.connection.saveData`](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/save-data#detecting_the_save-data_setting))
*   **内存：**  在低端设备上减少内存消耗 (通过[`navigator.deviceMemory`](https://developer.mozilla.org/en-US/docs/Web/API/Navigator/deviceMemory))
*   **CPU 核心数：** 当设备无法很好地处理时，限制费时的 JavaScript 操作，减少 CPU 密集型逻辑(通过[`navigator.hardwareConcurrency`](https://developer.mozilla.org/en-US/docs/Web/API/NavigatorConcurrentHardware/hardwareConcurrency))。

Facebook 已经在生产环境用上这类技术了。
![Facebook 生产实践](/uploads/1618526-0d9eb89026582584.webp?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 如何使用
Chrome 团队还发布了一系列实验性的 [React Hooks 和工具包](https://github.com/GoogleChromeLabs/react-adaptive-hooks)，帮助 React 项目应用 Adaptive Loading 技术。

包括根据网络状态做适配的 `useNetworkStatus` React hook ：

```
import React from 'react';

import { useNetworkStatus } from 'react-adaptive-hooks/network';

const MyComponent = () => {
  const { effectiveConnectionType } = useNetworkStatus();

  let media;
  switch(effectiveConnectionType) {
    case '2g':
      media = <img src='medium-res.jpg'/>;
      break;
    case '3g':
      media = <img src='high-res.jpg'/>;
      break;
    case '4g':
      media = <video muted controls>...</video>;
      break;
    default:
      media = <video muted controls>...</video>;
      break;
  }

  return <div>{media}</div>;
};

```

根据用户浏览器的 Data Saver 偏好设置做适配的 `useSaveData` 工具：
```
import React from 'react';

import { useSaveData } from 'react-adaptive-hooks/save-data';

const MyComponent = () => {
  const { saveData } = useSaveData();
  return (
    <div>
      { saveData ? <img src='...' /> : <video muted controls>...</video> }
    </div>
  );
};

```

还有根据用户设备 CPU 处理器核心数做适配的 `useHardwareConcurrency`工具：
```
import React from 'react';

import { useHardwareConcurrency } from 'react-adaptive-hooks/hardware-concurrency';

const MyComponent = () => {
  const { numberOfLogicalProcessors } = useHardwareConcurrency();
  return (
    <div>
      { numberOfLogicalProcessors <= 4 ? <img src='...' /> : <video muted controls>...</video> }
    </div>
  );
};

```
未来可能会出现更多的基础设施，能够根据用户网络和设备情况自动分发最优的代码和资源。结合客户端提示和客户端 API，未来的架构可能是这样的：

[![自适应加载架构图](/uploads/1618526-b3a5b9c277c7a241.webp?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)](https://res.cloudinary.com/practicaldev/image/fetch/s--9DmBNhzd--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://thepracticaldev.s3.amazonaws.com/i/1m81metyh0c549lt4xc5.jpg)
