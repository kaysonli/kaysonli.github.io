---
title: 用了几十年的浏览器 user-agent 要退出历史舞台了？看看 Google 怎么说
date: 2020-01-19 10:00:09
tags: chrome
---

Google 近日宣布，计划在 Chrome 浏览器上逐步淘汰 user-agent 字符串。

这里稍微解释下，user-agent (UA，用户代理) 字符串是现代 web 和浏览器功能的重要组成部分。

UA 字符串是浏览器建立连接时向网站发送的一段文本。UA 字符串包含了浏览器类型、渲染引擎和操作系统等详细信息。例如，Windows 10 上的 Firefox 浏览器 UA 是这样的：
```
*Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:71.0) Gecko/20100101 Firefox/72.0*
```

UA 在90年代作为 Netscape 浏览器的一部分被开发出来，一直沿用至今。几十年来，各种网站都在利用 UA 字符串，根据访客的客户端情况调整功能特性。

但是现在，谷歌表示这个曾经有用的机制已经成为各种问题的持续来源。
<!-- more -->

首先，UA字符串已经被在线广告商用来跟踪和识别网站访客。
>“这些隐私问题中最严重的是，用户代理嗅探是兼容性问题的多数来源，尤其是小众浏览器，会统一或针对特定网站谎报UA，一些网站（包括谷歌的网站）在某些浏览器上毫无理由的崩溃。”为 Chrome 浏览器工作的谷歌工程师 Yoav Weiss 说到。

为了解决这些问题，谷歌计划通过冻结整个标准来逐步消除 UA 字符串在 Chrome 中的重要性。
### 计划

Google 的计划是停止更新 Chrome 的 UA 字符串内容。

长期的计划是将所有的 Chrome UA 字符串统一为通用值，这样就不会泄露太多用户信息。

这意味着在新的平台上发布的新 Chrome 浏览器，比如在新的智能手机型号或新的操作系统版本上，将使用通用的 UA 字符串，而不是为特定平台定制的。

例如，在未来，一个网站将无法区分使用 Chrome 的访客是在 Windows 7 还是 Windows 11上运行 Chrome，或者 Chrome 移动用户是在使用三星 Galaxy 手机还是 Pixel 9 手机。

网站只能够识别用户是否在运行 Chrome，以及他们是否在桌面或移动设备上，但仅此而已。

为了历史遗留目的，现有的 Chrome UA 字符串将继续工作，所以它们不会破坏运行在整个web上现有的技术和脚本。

下面是谷歌目前弃用 UA 字符串的计划：
*   **Chrome 81** (2020 3月中旬) - 谷歌计划在 Chrome 控制台中为读取 UA 字符串的网页显示警告，这样开发者可以调整网站代码。
*   **Chrome 83** (2020 6月初) - 谷歌将在 UA 字符串中固定 Chrome 浏览器版本并统一操作系统版本
*   **Chrome 85** (2020 9月中旬) - 谷歌将统一桌面操作系统 UA 字符串作为桌面浏览器的通用值。谷歌还将统一移动操作系统/设备字符串作为一个类似的通用值。

### 再见，UA字符串！你好，CLIENT HINTS！

对 UA 字符串机制的弃用是谷歌改善网络隐私的努力的一部分，但不会扼杀网络广告，而广告是当今大多数免费网站的命脉。

Chrome 中的 UA 字符串将被一个名为 [Client hint](https://wicg.github.io/ua-hints/) 的新机制取代。通过这种机制，网站可以请求关于用户的信息，但没有“历史包袱和古老的`User-Agent` 标头暴露的被动指纹信息”，官方标准是这样写的。

 Client Hints 已经被开发为谷歌的[Privacy Sandbox](https://www.chromium.org/home/chromiprivacy/privacy-sandbox)项目的一部分，该项目于[去年8月份宣布](https://blog.chromium.org/2019/08/potential-uses-for-privacy-sandbox.html)。

Privacy Sandbox 技术栈旨在为网站和广告商提供一种方式，使他们能够在浏览器中查询用户详细信息，同时又不会暴露太多用户信息。

通过 Privacy Sandbox，浏览器将分享足够的用户信息，这样广告商就可以将用户分组了，而不是创建详细的个人资料。

弃用 UA 字符串改用 Client Hints 是谷歌实现 Chrome Privacy Sandbox 的第一步，这也是谷歌去年夏天承诺的。

苹果(Safari)、微软(Edge)和 Mozilla (Firefox)也表示支持谷歌冻结和逐步取消用户代理字符串的提议，但在撰写本文时尚未宣布具体计划。

>来源： [zdnet.com](https://www.zdnet.com/article/google-to-phase-out-user-agent-strings-in-chrome/)
翻译整理：1024译站