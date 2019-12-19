---
title: 当你敲完 git commit 命令后，究竟发生了什么？
date: 2019-12-19 10:05:40
tags: git
---

> 作者：Maxence Poutord
来源：dev.to
翻译：1024译站

如今大多数项目都使用 Git 作为版本控制系统，这意味着大多数项目都有一个`.git`文件夹。 但是，你有没有尝试过打开它？

 我试过一次……然后在一分钟内就关掉了！

长期以来，我都是把 Git 当做“黑盒”来用的。
<!-- more -->
[![git explained](/uploads/1618526-3adc1d7dc5eec512.webp?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)](https://res.cloudinary.com/practicaldev/image/fetch/s--FHszxX6n--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://thepracticaldev.s3.amazonaws.com/i/mmjznc6oz0ayl6kajgzp.png) 
> “这是GIT。它通过一个漂亮的分布式图论模型跟踪项目中的协作工作。”
“哇，好酷！怎么用的呢？”
“母鸡。只需记住这些 shell 命令并输入它们来同步。如果出现错误，那就把代码保存到其他地方，删掉项目，再下载一份新的。”
直到一年前。我对这种知其然、不知其所以然的做法感到厌倦。最后我找到了机会，开始学习它。 我看了[《Pro Git》](https://git-scm.com/book/en/v2)这本书，并进行了大量实验。我发现它并不像看起来那么复杂！

因此，如果你：
*   跟上面那张图中的人一样
*   想了解`.git` 文件夹里有什么
*   曾经 "在 git 里弄丢过代码"……

这篇文章就是为你写的！
## 第一步：Git 数据模型

为了便于理解，我会从一个基础项目开始（有2个文件： `index.js` and `README.md`）。
1.  `git init`
2.  `echo "console.log('hi')" >> index.js`
3.  `echo "# Cool project" >> README.md`
4.  `git add . && git commit -m "First commit"`

现在我们来看看`.git` 文件夹里的内容：
```
$ tree .git/objects

.git/objects
├── 2d
│   └── 1a41ebd2d32cb426b6d32e61e063f330aa1af8
├── 66
│   └── 87898e2a0fe2da282efab6b5f7f38b7f788f56
├── c2
│   └── 43f24fb294ebc954b0a7ee6020589245f78315
├── c3
│   └── 0500babf488d06033e8d039cbf64be3edbd089
├── info
└── pack

6 个目录，4 个文件

```

Git 创建了4个文件。为了避免一个文件夹包含太多文件，git 会自动截取前两个字符作为文件夹名。要检索一个 git 对象，必须将文件夹名+文件名拼接起来。

由于这些文件不具备可读性，你可以用命令`git cat-file <sha-1> -p` 查看里面的内容（或者用 `-t` 参数查看类型）。顺便提一下，只能用前 8 个字符。

这些文件是这样互相链接的：
[![git data model](/uploads/1618526-95071bd18ce332ac.webp?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)](https://res.cloudinary.com/practicaldev/image/fetch/s--bQK_5SKc--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://thepracticaldev.s3.amazonaws.com/i/wwspfp8p1s2ruedxhivh.png) 

Git 对象模型有 4 种不同类型：
*   **commit**：包含提交人、日期、消息还有目录树
*   **tree**：引用其他 tree 和（或）blob
*   **blob**：存储文件数据
*   **tag**：存储某个提交的引用（本文未涉及） 

请注意，blob 不存储文件名（和位置）。这就是为什么当你更改文件位置时 git 会丢失历史记录的原因之一。

🤔如果你在本机运行，可能会得到不同的 hash 值（作者和日期不一样）！

## 第二步：第二次提交

现在我们要更新 `index.js` ，给文件再添加一行：
1.  `echo "console.log('world')" >> index.js`
2.  `git add . && git commit -m "Second commit"`

于是又多了 3 个 对象：
```
$ tree .git/objects

.git/objects
├── 11
│   └── 75e42a41f75f4b25bab53db36d581f72387aa9
├── 2d
│   └── 1a41ebd2d32cb426b6d32e61e063f330aa1af8
├── 66
│   └── 87898e2a0fe2da282efab6b5f7f38b7f788f56
├── c2
│   └── 43f24fb294ebc954b0a7ee6020589245f78315
├── c3
│   └── 0500babf488d06033e8d039cbf64be3edbd089
├── ee
│   └── c2cd5e0b771793e03bbd5f8614c567af964a4e
├── fc
│   └── 512af17ca7ec04be6958047648b32629e4b5a5
├── info
└── pack

9 个目录，7 个文件

```

现在我们得到这样的结果：
[![git data model with 2 commits](/uploads/1618526-38bbc1baf3f3c041.webp?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)](https://res.cloudinary.com/practicaldev/image/fetch/s--32eA9DRP--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://thepracticaldev.s3.amazonaws.com/i/0pjlpa3pnkthae89pyf4.png) 

这里有意思了：Git 并没有存储文件之间的差异！幸亏有**packfiles** (位于 `.git/objects/pack`)，Git 在硬盘上保留了一个合理的位置。 
## 第三步：乱改一通

在最后一步，我们将添加一个提交。然后，我们将回到过去以“删除此提交”。
1.  `echo "console.log('')" >> index.js`
2.  `git add . && git commit -m "Third commit"`

你可能猜到了，git 创建了3个新文件。结构跟第二步的类似。
```
.git/objects
├── 00
│   └── ee8c50f8d74eaf1d3a4160e9d9c9eb1c683132
├── 09
│   └── f760de83890e3c363a38e6dc0700b76e782bc1
├── cf
│   └── 81d6f570911938726cff95b62acbf198fd3510
└── ...

12 个目录, 10 个文件

```

现在，我们假装想回退一个提交（`git reset HEAD~1 --hard`）。
[![git reset --hard](/uploads/1618526-004de1687fb56512.webp?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)](https://res.cloudinary.com/practicaldev/image/fetch/s--U-9HQuLc--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://thepracticaldev.s3.amazonaws.com/i/awvu8yb8px54gykql92c.png) 

现在你可能会认为，你搞砸了一切，提交记录再也找不到了。是不是？

可能是。让我们看看还有多少个 git 对象……
```
$ tree .git/objects

.git/objects
└── ...

12 个目录, 10 个文件

```

看到没，我们还有10个文件！没有被删！你猜怎么着？如果我用命令`git cat-file cf81d6f570911938726cff95b62acbf198fd3510 -p`，我还能查看第三次提交的 index.js 文件内容。
> "在 git 里你不可能丢失代码。"
> 
> ——鲁迅

比这更严重的是，我每天使用`git push --force`，`git rebase`和`git reset --hard`，但我从来没有丢失任何东西。但是，我们是人类，人类是容易犯错的。

别担心，如果你想回滚，不用丢弃所有文件。接下来就是见证奇迹的时刻！

## reflog：魔术棒✨

如果你尝试使用`git log`检索历史记录，则不会看到“Third commit” 这个提交。但是，如果加了参数`-g`(代表 `--walk-reflogs`)，就会看到第三个提交。

为了让结果更好看，你可以用 `git log -g --abbrev-commit --pretty=oneline`。

这个超有用的命令有个别名：`git reflog` ❤️
```
$ git reflog

eec2cd5 (HEAD -> master) HEAD@{0}: reset: moving to HEAD~1
00ee8c5 HEAD@{1}: commit: Third commit
eec2cd5 (HEAD -> master) HEAD@{2}: commit: Second commit
c30500b HEAD@{3}: commit (initial): First commit

```

（注意：在 `.git/logs/HEAD`里可以看到类似的结果）

现在，你有了第三个提交的指纹： `00ee8c5`。你可以用 `git reset 00ee8c5 --hard` 来撤销之前的重置。

**注意事项：**

在某些情况下，`git reflog` 对你**没有**帮助：
*   当你拉取别人的代码
*   当你删除仓库并重新克隆
*   当你查找超过90天前的改动（被`git gc`清理掉了 ）。我不知道你们的情况，反正我连一个月前做了啥都不记得了。所以记录保存3个月应该足够了。

另外，如果你像 `ctrl + s`一样使用 git commit，就可能很容易犯迷糊。很抱歉，除了建议你阅读我的那篇有关 [conventional commits](https://www.maxpou.fr/git-conventional-commits) 的文章外，我也无能为力。我认为这是使用 git 最干净的方法。
## 总结 

*   有4种不同类型的 git 对象：`commit`，`tree`，`blob` 和` tag`
*   `blob` 并不存储文件名（这就是为什么移动文件后会丢失历史）
*   Git 不存储文件差异
*   提交后的代码不会丢失。`git reflog` 能帮到你。

今天就到这里，下课！
[](https://dev.to/maxpou/what-s-happens-when-you-git-commit-59n7)

