---
title: 周末实验：给旧笔记本做一次轻量改造
date: 2026-09-20 16:40:00
tags:
  - 工具
  - 终端
  - 周末
categories:
  - 动手
cover: https://images.unsplash.com/photo-1517694712202-14dd9538aa97?auto=format&fit=crop&w=1600&q=80
feature: true
comments: false
abstracts: 一台开机要四十秒、风扇像吹风机的旧笔记本，周末被我拆开清灰、收起开机启动项，再配上一组只为自己服务的命令别名。改造不大，但周一早上打开它时，手感完全不同。
---

这台笔记本是 2019 年买的。屏幕还算清楚，键盘的空格键略微偏软，电池从「能撑一个下午」变成「合上盖子像在赌它会不会自己醒来」。我原本想换新的。周末把报价单看了两遍，发现自己真正受不了的不是性能，而是每次开机后的十分钟：风扇狂转、启动项一个个弹出来、桌面上堆着三个月没关的安装包。

所以实验目标改成一句话：**让它在二十分钟内变成一台可以开始写字的机器。**

## 先测量，再动手

没测量就清灰，很容易产生「我已经努力过了」的错觉。我先记了四项，用的都是机器上本来就有的办法。

| 项目 | 改造前 | 改造后 | 我怎么看 |
| --- | --- | --- | --- |
| 冷启动到桌面 | 约 48 秒 | 约 22 秒 | 启动项砍掉一半就够了 |
| 静置温度 | 风扇持续转 | 只在编译时转 | 出风口灰比硅脂更值得先处理 |
| 桌面图标 | 37 个 | 6 个 | 剩下的进了「归档」文件夹 |
| 写第一句话的时间 | 经常超过 10 分钟 | 打开编辑器即可 | 这才是这次实验的真正指标 |

温度是用手心贴在出风口估计的，不算实验室数据。但改造前手心发烫、改造后只是温热，这个差别不需要传感器也能复述。

## 清灰时我只做了三步

1. 关机、拔电源、等了十分钟，让自己没有「再看一眼网页」的借口。
2. 后盖螺丝按拆下来的顺序排在桌上，用纸条写了左上、右下。上次有颗螺丝装错孔，盖子翘了一条缝。
3. 出风口和风扇用软毛刷顺着叶片清，没有用吸尘器直接对着风扇吸。风扇叶反过来转，可能伤轴承。

硅脂没换。拆散热模组要动更多螺丝，而这次的目标是轻量。灰清掉之后，编译博客时风扇仍会响，但平时写 Markdown 已经安静下来。知道边界在哪里，比一次做到「彻底」更重要。

## 一组只为自己写的别名

系统里最拖时间的是重复输入。我把周末会用到的几件事收成短命令，放在 PowerShell 的配置里。原则是：别名必须短到不用想，而且不能覆盖系统原有命令。

```powershell
function blog {
  Set-Location "$HOME\Documents\GitHub\bojaka_blog"
}

function preview {
  blog
  npx hexo clean
  npx hexo server
}

function post {
  param([Parameter(Mandatory = $true)][string]$Name)
  blog
  npx hexo new $Name
}
```

`blog` 负责到达现场，`preview` 负责看见结果，`post` 负责新开一篇。三件事分开，是因为我经常只想进目录改一个错别字，并不想顺手把服务重启。把它们绑成一个「万能命令」，下次改错别字也会等上半分钟。

## 一个很小的草稿检查脚本

文章写到第三篇时，我开始漏 `cover`。首页卡片就会空一块。与其靠眼睛扫，不如在生成前跑一个只检查 front matter 的脚本。它不修改文件，只把缺字段的稿子列出来。

```javascript
const fs = require("fs");
const path = require("path");

const dir = path.join(process.cwd(), "source", "_posts");
const required = ["title", "date", "cover", "categories", "tags", "abstracts"];

for (const file of fs.readdirSync(dir).filter((name) => name.endsWith(".md"))) {
  const text = fs.readFileSync(path.join(dir, file), "utf8");
  const head = text.split("---")[1] || "";
  const missing = required.filter((key) => !new RegExp(`^${key}:`, "m").test(head));
  if (missing.length) {
    console.log(`${file} -> ${missing.join(", ")}`);
  }
}
```

脚本故意写得很笨。正则只认行首的字段名，遇到折叠写法会误报，但对我现在这种手写的 front matter 够用。误报比漏报便宜：多看一眼文件，总好过首页缺一张图。

## 周一早上的手感

改造没有让这台机器变成新电脑。导出一组照片时风扇还是会叫，电池还是劝我插电。可是桌面干净了，命令记得住，第一句话可以在打开盖子后马上写。

我把实验结论记在笔记本贴膜的内侧，用铅笔，方便以后擦掉：

> 先修你每天都会碰到的摩擦，再决定要不要换一台新的。

如果下个月导出照片这件事每周都发生，再考虑内存和硬盘。在那之前，这台旧机器已经足够当一台博客编辑器。
