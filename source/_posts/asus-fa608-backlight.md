---
title: "解决华硕 FA608 键盘背光在 Linux 下不亮的问题"
date: 2026-06-12 22:16:00
tags:
  - "华硕 FA608 键盘背光不亮"
  - "ASUS TUF Linux 键盘灯"
  - "hidapitester Linux 背光"
  - "I2C HID 键盘背光修复"
categories:
  - Linux
feature: false
comments: false
abstracts: "华硕 TUF Gaming A16 FA608（FA608PM/WV/WI/UM）在 Linux 上键盘背光无法点亮的根本原因：键盘是 I2C HID 设备，内核缺 EC 映射，asusctl 无效。本文介绍用 hidapitester 直接发送 feature report 来唤醒背光，并附开机自动执行方案。"
---

华硕 TUF Gaming A16 FA608 系列（FA608PM / WV / WI / UM 等）在 Linux 上有个尴尬问题：**键盘背光怎么都点不亮**。

- `/sys/class/leds/asus::kbd_backlight/brightness` 可以从 0 改到 3，但灯没反应
- `asusctl -k high`、GNOME 设置里的亮度滑块，统统无效

原因是这块键盘是 **I2C HID 设备**（`0B05:19B6`，ITE51368），被通用驱动 `hid-generic` 接管，内核里没有对应的 EC 映射，所以那个亮度节点写了等于没写，asusctl 也管不到。

解决办法：用 [`hidapitester`](https://github.com/todbot/hidapitester) 直接给键盘控制器发一个初始化 feature report，把背光「唤醒」。

## 1. 安装依赖并编译

以 Fedora 为例：

```bash
sudo dnf install -y gcc make git hidapi-devel systemd-devel

cd ~
git clone https://github.com/todbot/hidapitester
cd hidapitester
make
```

> `hidapi-devel` 提供 HID 库和头文件，`systemd-devel` 提供 `libudev`。两者缺一编译都会报错（`hidapi.h` 找不到或 `libudev not found`）。

编译完成后，当前目录会生成可执行文件 `./hidapitester`。

## 2. 执行点亮

```bash
sudo ./hidapitester --vidpid 0B05/19B6 --open --send-feature 70,1
```

执行完键盘灯应立刻亮起。之后再用快捷键或 `asusctl -k high` 调亮度就正常了：

```bash
asusctl -k high     # off / low / med / high
```

## 关于 sudo

**必须 `sudo`。** 这条命令要打开 `/dev/hidrawX` 设备节点，而该节点默认属主是 `root`，普通用户没有读写权限，不加 `sudo` 会直接报打不开设备。

如果实在不想每次都加 `sudo`，可以写一条 udev 规则给这个设备放权，但日常使用直接 `sudo` 最简单。

## 说明

- 关机或休眠后背光会失效，需要重新执行一次该命令。
- 这是内核层面的缺陷，要等给 FA608 加上 EC quirk 才能原生支持，相关讨论见 asusctl 的 [issue #578](https://gitlab.com/asus-linux/asusctl/-/work_items/578)、[#700](https://gitlab.com/asus-linux/asusctl/-/work_items/700)。在那之前，上面这个办法是最可靠的临时方案。


---

> 编辑说明：本文在原文基础上经过 AI 编辑优化。
