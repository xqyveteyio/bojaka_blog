---
title: "Fedora 安装英伟达（NVIDIA）显卡驱动"
date: 2026-02-02 00:01:00
tags:
  - "Fedora 安装 NVIDIA 驱动"
  - "Fedora 英伟达显卡驱动"
  - "RPM Fusion NVIDIA 安装"
  - "akmod-nvidia Fedora"
categories:
  - Linux
feature: true
comments: false
abstracts: "在 Fedora Workstation（GNOME）上通过 RPM Fusion 安装 NVIDIA 专有驱动（akmod-nvidia）的完整步骤：启用 RPM Fusion 源、安装 akmod-nvidia 和 CUDA 库、重启等待模块自动编译；附 Secure Boot 关闭方案、双显卡（Optimus）说明及 nvidia-smi 验证与常见报错排查。"
---

下面以 **Fedora Workstation（GNOME）** 为例，教你在 Fedora 上安装 **NVIDIA（英伟达）专有驱动**（最常用、也最稳定的方式：RPM Fusion）。命令基本通用到 Fedora 39/40/41。

---

## 0）先确认你的显卡型号（可选但建议）
```bash
lspci | grep -E "VGA|3D"
```
如果看到 `NVIDIA`，继续。

---

## 1）更新系统并启用 RPM Fusion 源（关键）
1. 更新系统：
```bash
sudo dnf upgrade --refresh -y
```

2. 启用 RPM Fusion（free + nonfree）：
把下面两条都执行（会自动匹配你的 Fedora 版本）：
```bash
sudo dnf install -y \
  https://download1.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm \
  https://download1.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm
```

3. 再刷新一次：
```bash
sudo dnf upgrade --refresh -y
```

---

## 2）安装 NVIDIA 驱动（RPM Fusion 的 akmod 方案，推荐）
### 方案 A：大多数用户（推荐）
```bash
sudo dnf install -y akmod-nvidia
```

然后（强烈建议）装上 CUDA 库与视频解码相关组件（可选但常用）：
```bash
sudo dnf install -y xorg-x11-drv-nvidia-cuda xorg-x11-drv-nvidia-cuda-libs
```

> 说明：`akmod-nvidia` 会在你当前内核上**自动编译**对应的驱动模块，后续内核升级也能自动跟上（比手动装 .run 安装包省心很多）。

---

## 3）重启并等待模块构建完成
安装后请 **重启**：
```bash
sudo reboot
```

重启进系统后，第一次会自动编译模块，可能需要 1–5 分钟。你可以用下面命令确认驱动是否就绪：

1. 看模块是否加载：
```bash
lsmod | grep -i nvidia
```

2. 看 nvidia-smi：
```bash
nvidia-smi
```

正常会显示显卡型号、驱动版本、显存占用等信息。

---

## 4）（可选）Wayland / X11 说明
- Fedora 默认是 **Wayland**，NVIDIA 新驱动一般没问题。
- 如果你遇到登录黑屏/花屏/扩展屏异常，可临时在登录界面齿轮里选 **“GNOME on Xorg”** 测试。
- Secure Boot（安全启动）开启时，常见问题是 **驱动模块无法加载**（见下方排错）。

---

# 常见问题排错（很实用）

## A）开了 Secure Boot，驱动装了但加载不了
症状：`nvidia-smi` 报错、`lsmod` 没有 nvidia、系统日志提示 module verification failed。

解决思路有两种（二选一）：
1) **最简单**：进 BIOS/UEFI 把 **Secure Boot 关闭**。  
2) **更规范**：给 NVIDIA 内核模块做 **MOK 签名**（步骤相对繁琐，不同机器略有差异）。签名流程跟机器差异比较大，这篇先不展开。多数情况下关掉 Secure Boot 就能让模块加载。

---

## B）内核刚升级后进不去 / 驱动没编译好
可以先确认 akmods 编译状态：
```bash
sudo systemctl status akmods
```

强制重建一次（常用于升级后）：
```bash
sudo akmods --force
sudo dracut --force
sudo reboot
```

---

## C）你是很老的显卡（需要 470xx/390xx 等旧分支）
新卡通常直接用 `akmod-nvidia`；但 **较老**的卡可能要用特定分支包。把 `lspci` 里的型号和上面的 Fedora 版本对照 RPM Fusion 的分支说明，老卡再改用 470xx 或 390xx。

---

## D）笔记本双显卡（Intel/AMD + NVIDIA，Optimus）
Fedora 上通常也能直接用 `akmod-nvidia`，但“只在需要时启用独显”会涉及 PRIME offload 等。笔记本双显卡如果要省电或外接显示器，通常还要再配 PRIME 或后面的 `supergfxctl`，不在这篇的安装步骤里。

---

## 判断该装哪个驱动分支

如果新驱动对不上老显卡，先自己跑下面几条命令，对照型号和 Secure Boot 状态再选分支（例如 470xx）：

```bash
rpm -E %fedora
uname -r
mokutil --sb-state 2>/dev/null || echo "mokutil not installed"
lspci | grep -E "VGA|3D"
```


---

> 编辑说明：本文在原文基础上经过 AI 编辑优化。
