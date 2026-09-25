---
title: "Fedora 华硕笔记本切换显卡模式（supergfxctl）"
date: 2026-06-11 19:55:00
tags:
  - "Fedora 华硕显卡模式切换"
  - "supergfxctl 使用教程"
  - "ASUS Linux 显卡切换"
  - "asus-linux Fedora 安装"
categories:
  - Linux
feature: false
comments: false
abstracts: "华硕游戏本（Intel/AMD 核显 + NVIDIA 独显）在 Fedora 上安装 supergfxctl，实现 Integrated（纯核显省电）、Hybrid（混合模式）、AsusMuxDgpu（独显直连高性能）三种显卡模式的查看与切换，MUX 切换后需重启生效。"
---

华硕游戏本（Intel/AMD 核显 + NVIDIA 独显）在 Fedora 上默认一般是 **Hybrid 混合模式**：日常用核显省电，需要时再调用独显。

如果想 **全部走独显**，或者反过来 **只用核显** 省电，可以用 [ASUS Linux](https://asus-linux.org/) 项目的 `supergfxctl`。

> 前提：已安装 NVIDIA 专有驱动（`akmod-nvidia`）。可参考 [Fedora 安装英伟达驱动](/post/fedora-nvidia-driver/)。

## 安装

```bash
sudo dnf copr enable lukenukem/asus-linux
sudo dnf install asusctl supergfxctl
sudo systemctl enable --now supergfxd.service
```

`asusctl` 可选，用来调风扇、性能模式等；切换显卡只需要 `supergfxctl`。

## 查看支持的模式

```bash
supergfxctl --supported
```

常见输出：

```
[Integrated, Hybrid, AsusMuxDgpu]
```

| 模式 | 说明 |
|------|------|
| `Integrated` | 只用核显，关闭独显，最省电 |
| `Hybrid` | 混合模式，默认推荐 |
| `AsusMuxDgpu` | 独显直连，全部走 NVIDIA，性能最好 |

## 查看当前模式

```bash
supergfxctl --get
```

也可以用下面命令确认实际在用的显卡：

```bash
glxinfo | grep "OpenGL renderer"
```

- 显示 **AMD / Intel** → 核显在渲染
- 显示 **NVIDIA** → 独显直连

## 切换模式

**切到独显模式：**

```bash
sudo supergfxctl --mode AsusMuxDgpu
sudo reboot
```

**切回混合模式：**

```bash
sudo supergfxctl --mode Hybrid
sudo reboot
```

**只用核显：**

```bash
sudo supergfxctl --mode Integrated
```

> MUX 切换是硬件级的，改完 **必须重启** 才能生效。

## 其他常用命令

```bash
supergfxctl --status       # 独显电源状态
supergfxctl --pend-mode    # 是否有待生效的模式切换
```

## 参考

- [supergfxctl 官方文档](https://asus-linux.org/manual/supergfxctl-manual/)


---

> 编辑说明：本文在原文基础上经过 AI 编辑优化。
