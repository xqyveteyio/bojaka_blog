---
title: "自制无损音乐播放器"
date: 2026-01-17 16:51:00
tags:
  - "树莓派音乐播放器"
  - "自制无损播放器"
  - "树莓派 Zero 2W DIY"
  - "MPD 音乐播放器"
categories:
  - 硬件
feature: false
comments: false
abstracts: "用树莓派 Zero 2 W + DAC 解码器 + OLED 屏幕打造一款高续航、便携、好音质的 DIY 无损音乐播放器，脱离手机专注音乐。涵盖硬件选型（DAC HIFI 模块、按键模块、3D 打印外壳）与软件方案（MPD + MPC / 自定义 Web 界面）。"
---

目标是一台不靠手机的便携无损播放器：续航长一点，声音走独立 DAC，屏幕只显示正在播的曲子。

## 硬件

- 树莓派 Zero 2 W
- 移动电源
- DAC 解码器，用来把数字信号解成 HIFI 一点的模拟输出
- 按键模块，管播放和音量
- OLED，显示当前曲目
- 外壳，3D 打印或另外定制

## 软件

- 系统用 Raspberry Pi OS Lite
- 播放用 MPD（Music Player Daemon）
- 操作界面用 MPC，或者再写一个简单的 Web 界面


---

> 编辑说明：本文在原文基础上经过 AI 编辑优化。
