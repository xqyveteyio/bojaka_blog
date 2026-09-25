---
title: "Fly-Pie 绑定鼠标侧键为触发键"
date: 2026-08-04 07:20:00
tags:
  - "Fly-Pie"
  - "Input Remapper"
  - "GNOME"
  - "Fedora"
categories:
  - Linux
cover: /images/fly-pie-side-button/1.gif
feature: false
comments: false
abstracts: "在 GNOME 上使用 Input Remapper 将鼠标侧键映射为 Super+右键，实现通过鼠标侧键快速呼出 Fly-Pie 轮盘菜单。"
---

最近发现了 GNOME 上一个非常好用的快捷访问扩展插件，叫 Fly-Pie，可以通过快捷键呼出轮盘。

![Fly-Pie 轮盘菜单效果](/images/fly-pie-side-button/1.gif)

但有个问题：原版扩展的触发快捷键只支持键盘，以及把空格键映射到鼠标右键。但我希望绑定到侧键，这样肯定会非常便利，所以需要一个额外的软件来辅助实现。

软件叫 Input Remapper，Fedora 上可以直接安装：

```bash
sudo dnf install input-remapper
```

然后就可以像下图一样设置绑定，把侧键绑定到 `Super+右键`，实现使用侧键呼出 Fly-Pie 菜单的效果。

![Input Remapper 鼠标侧键映射设置](/images/fly-pie-side-button/2.png)


---

> 编辑说明：本文在原文基础上经过 AI 编辑优化。
