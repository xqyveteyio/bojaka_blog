---
title: "Fedora 打开任何软件卡顿数秒"
date: 2026-02-02 00:10:00
tags:
  - "Fedora 打开软件卡顿"
  - "Fedora 软件启动慢数秒"
  - "Fedora 混合显卡卡顿"
  - "Linux 显卡驱动卡顿"
categories:
  - Linux
feature: false
comments: false
abstracts: "Fedora 系统（尤其是混合显卡笔记本）打开任何应用时卡顿数秒随后恢复正常，根本原因通常是缺少独立显卡驱动。安装 NVIDIA 专有驱动后卡顿问题消失，本文附官方驱动安装教程链接。"
---

混合显卡笔记本在 Fedora 上打开任何软件，都会先卡几秒，然后恢复正常。查下来是独显驱动没装上，装好 NVIDIA 专有驱动之后这个停顿就消失了。

台式机如果核显和独显也是混用，可能遇到同样的现象。驱动安装步骤写在 [Fedora 安装英伟达驱动](/post/fedora-nvidia-driver/)。


---

> 编辑说明：本文在原文基础上经过 AI 编辑优化。
