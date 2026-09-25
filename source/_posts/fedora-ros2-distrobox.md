---
title: "Fedora 优雅开发 ROS 2 指南（distrobox）"
date: 2026-02-23 01:38:00
tags:
  - "Fedora ROS2 开发"
  - "distrobox ROS2 安装"
  - "Fedora 容器开发 ROS2"
  - "Podman distrobox Ubuntu"
categories:
  - 开发
feature: false
comments: false
abstracts: "ROS 2 官方不支持 Fedora，本文介绍使用 distrobox（基于 Podman 的容器工具）在 Fedora 上一键运行 Ubuntu 24.04 容器并安装 ROS 2，效果等同 WSL；同时配置 VS Code Remote Container（Podman 后端）和 XWayland GUI 显示，实现无缝开发体验。"
---

ROS 2 官方不支持 Fedora。不管是 Windows，还是非官方支持的 Linux，用虚拟机都更少踩坑。这里说的不是传统虚拟机，而是容器：性能和直接在主机上开发很接近。

Windows 上用 WSL 装 Ubuntu、再装 ROS 2 就够了。我的主力系统是 Fedora，所以用 distrobox。它基于 Podman，可以方便地创建别的发行版容器，体验接近 WSL。

## 安装 distrobox

```bash
sudo dnf install distrobox
```

## 创建并进入 Ubuntu 容器

```bash
distrobox create --name ubuntu-ros --image ubuntu:24.04
distrobox enter ubuntu-ros
```

`distrobox enter` 的感觉和进入 WSL 一样。

## 在容器里安装 ROS 2

按官方文档安装，并让 ROS 2 版本和 Ubuntu 版本对上。优先用预编译的 deb 包，不要自己编译。

## 用 VS Code 打开容器

安装 Remote Containers 插件，把后端从 Docker 改成 Podman，就可以在 VS Code 里直接打开容器中的代码。

## 让 ROS 2 的 GUI 显示出来

Fedora 默认走 Wayland，ROS 2 的 GUI 默认走 X11，所以需要 XWayland：

```bash
sudo dnf install xwayland
```

进容器开发前导出：

```bash
export QT_QPA_PLATFORM=xcb
export DISPLAY=:0
```


---

> 编辑说明：本文在原文基础上经过 AI 编辑优化。
