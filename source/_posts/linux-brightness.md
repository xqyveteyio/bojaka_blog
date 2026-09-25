---
title: "解决 Ubuntu / Fedora 笔记本不能调节亮度的问题"
date: 2026-06-11 12:00:00
tags:
  - "Linux 笔记本亮度调节失效"
  - "Ubuntu 亮度键无效"
  - "Fedora 亮度调节修复"
  - "acpi_backlight native GRUB"
categories:
  - Linux
feature: false
comments: false
abstracts: "笔记本安装 Ubuntu 或 Fedora 后亮度键无效、亮度滑块调不动，多为内核背光驱动未选对。本文介绍通过在 GRUB 添加 acpi_backlight=native 内核参数来修复亮度控制失效的问题，分别给出 Ubuntu（update-grub）和 Fedora（grubby）的操作步骤。"
---

笔记本在 Ubuntu 或 Fedora 上装好后，亮度键没反应、滑块也调不动，多半是内核背光驱动没选对。加一条内核参数 `acpi_backlight=native` 通常就能解决。

## Ubuntu

编辑 GRUB 配置：

```bash
sudo nano /etc/default/grub
```

找到这一行：

```
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"
```

改成：

```
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash acpi_backlight=native"
```

保存后更新 GRUB 并重启：

```bash
sudo update-grub
sudo reboot
```

## Fedora

Fedora 可以直接用 `grubby` 加参数，不用手改文件：

```bash
sudo grubby --update-kernel=ALL --args="acpi_backlight=native"
sudo reboot
```

如果 `grubby` 不可用，就手动改 `/etc/default/grub`。Fedora 要改的是 **`GRUB_CMDLINE_LINUX`** 这一行（不是 `GRUB_CMDLINE_LINUX_DEFAULT`），在末尾引号前加上 `acpi_backlight=native`，例如：

```
GRUB_CMDLINE_LINUX="rhgb quiet ... acpi_backlight=native"
```

然后：

```bash
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
sudo reboot
```

重启后亮度控制一般就恢复正常了。


---

> 编辑说明：本文在原文基础上经过 AI 编辑优化。
