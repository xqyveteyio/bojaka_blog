---
title: "Linux 缺少 .so 动态库依赖的排查与修复"
date: 2026-04-25 12:00:00
tags:
  - "Linux so 动态库缺失"
  - "ldd not found 修复"
  - "dnf provides 反查包"
  - "error while loading shared libraries"
categories:
  - Linux
feature: false
comments: false
abstracts: "从 CuraEngine 报 libpolyclipping.so.22 not found 出发，总结 Linux 下「缺 .so 动态库」问题的完整排查流程：用 ldd 定位缺失库，用 dnf provides / apt-file / pacman -F 反查所属包，安装后验证；还涵盖版本不匹配时做软链、RPATH、LD_LIBRARY_PATH 等进阶解法。"
---

## 起因

运行第三方二进制 `CuraEngine` 时报错：

```
CuraEngine: error while loading shared libraries:
libpolyclipping.so.22: cannot open shared object file: No such file or directory
```

最终通过一条命令解决：

```bash
sudo dnf install polyclipping
```

下面把这次踩坑沉淀成一套**通用流程**，下次再遇到类似问题可以五分钟内搞定。

---

## `libpolyclipping.so.22` 是什么

- **polyclipping**（项目名 *Clipper*）是 Angus Johnson 写的 C++ 多边形布尔运算库，提供：
  - 多边形求交 / 求并 / 求差 / 求异或
  - 多边形偏移（offset / inflate）
- CuraEngine 切片时大量使用它来对每层轮廓做布尔运算、生成壁线偏移、计算填充与支撑等。
- 文件名里的 `.so.22` 中的 `22` 是 **SONAME 版本号（ABI 版本）**，不是包的版本号。当前上游 `polyclipping 6.4.2` 把 SONAME 定为 `22`。

---

## 通用排查流程

### 1. 确认到底缺什么

```bash
ldd 路径/可执行文件 | grep "not found"
```

`ldd` 会列出二进制需要的所有动态库，缺失的显示 `not found`。
报错 `error while loading shared libraries: XXX.so` 表达的是同一件事。

本例输出：

```
libpolyclipping.so.22 => not found
```

### 2. 反查这个 .so 属于哪个包

#### Fedora / RHEL（本机）

```bash
# 关键武器：用文件路径模式反查包，不需要包已安装
dnf provides '*/libpolyclipping.so.22'

# 或按库名搜
dnf search polyclipping
```

#### Debian / Ubuntu

```bash
sudo apt install apt-file
sudo apt-file update
apt-file search libpolyclipping.so.22
```

也可以在线查 <https://packages.ubuntu.com/>（"Search the contents of packages"）。

#### Arch

```bash
pacman -F libpolyclipping.so.22
```

#### 跨发行版兜底

直接 Google `"libpolyclipping.so.22"`，第一页一般就是 [pkgs.org](https://pkgs.org/) 或 Debian/Fedora 的包页。pkgs.org 专做「文件 → 包」反查，对所有主流发行版都管用。

### 3. 安装并验证

```bash
sudo dnf install polyclipping
ldd 路径/可执行文件 | grep -i polyclipping
# 不再出现 not found 即为成功
```

---

## 仓库里没有 / 版本对不上时怎么办

按优先级尝试：

### 1) 版本对不上：做软链

例如系统只有 `libpolyclipping.so.7`，程序要 `.so.22`。**先确认 ABI 兼容**（不兼容硬链会段错误），兼容时可以在**程序自己的 lib 目录**里加软链：

```bash
ln -sf /usr/lib64/libpolyclipping.so.7 \
       程序目录/libpolyclipping.so.22
```

让程序优先加载自己目录里的 .so，有两种方式：

- 启动时设环境变量：`LD_LIBRARY_PATH=程序目录 ./可执行文件`
- 程序自带 `RPATH=$ORIGIN`（CuraEngine 就是这种，把 .so 放它旁边自动生效）

> 怎么看程序的 RPATH：
> ```bash
> readelf -d 可执行文件 | grep -E 'RPATH|RUNPATH'
> ```

### 2) 仓库根本没有

去上游项目 GitHub Release 找预编译，或者源码自编译。

### 3) 不想污染系统

可以用：

- conda / pip 自带的二进制
- Flatpak / AppImage（自带依赖）
- 把所需 .so 全塞到程序目录 + `$ORIGIN` RPATH

---

## 一句话肌肉记忆

> **看到 `error while loading shared libraries: 某.so`**
> **→ `ldd` 确认缺啥**
> **→ `dnf provides '*/某.so'`（或对应发行版的反查命令）找包**
> **→ `sudo dnf install 包名` 装上**
> **→ 再 `ldd` 验证。**

记住这套流程，90% 的「缺 .so」问题都能直接解决。

---

## 附：本次实际命令记录

```bash
# 1. 看缺什么
ldd bin/curaengine/linux/CuraEngine | grep "not found"
# libpolyclipping.so.22 => not found

# 2. 安装（Fedora 43）
sudo dnf install polyclipping
# Installing: polyclipping  x86_64  6.4.2-38.fc43

# 3. 再次验证，已链上 /usr/lib64/libpolyclipping.so.22，问题解决
ldd bin/curaengine/linux/CuraEngine | grep polyclipping
```


---

> 编辑说明：本文在原文基础上经过 AI 编辑优化。
