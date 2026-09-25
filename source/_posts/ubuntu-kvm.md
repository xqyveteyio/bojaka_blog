---
title: "Ubuntu 使用 KVM 虚拟化技术完整指南"
date: 2025-10-20 23:41:40
tags:
  - "Ubuntu KVM 安装"
  - "Ubuntu 虚拟机搭建"
  - "KVM libvirt Ubuntu"
  - "virt-manager Ubuntu"
categories:
  - Linux
cover: /images/ubuntu-kvm/image.png
feature: false
comments: false
abstracts: "在 Ubuntu 上安装 KVM 虚拟化环境的完整步骤：通过 apt 安装 qemu-kvm、libvirt-daemon-system、virt-manager 等组件，启动 libvirtd 服务，并配置用户组权限，使用 virt-manager 图形界面创建和管理虚拟机。"
---

![alt text](/images/ubuntu-kvm/image.png)

KVM（Kernel-based Virtual Machine）是 Linux 内核的虚拟化模块，它允许 Linux 主机运行多个虚拟机。本文将详细介绍在 Ubuntu 系统上安装和配置 KVM 的完整步骤。

## 🔧 二、安装 KVM 及管理工具

首先更新软件包列表，然后安装必要的 KVM 组件：

```bash
sudo apt update
sudo apt install -y qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virt-manager
```

**安装的组件说明：**
- `qemu-kvm`: KVM 虚拟化核心组件
- `libvirt-daemon-system`: libvirt 守护进程
- `libvirt-clients`: libvirt 客户端工具
- `bridge-utils`: 网络桥接工具
- `virt-manager`: 图形化管理界面

## 🚀 三、启用并启动服务

安装完成后，启用并启动 libvirtd 服务：

```bash
sudo systemctl enable --now libvirtd
```

检查服务是否正常运行：

```bash
sudo systemctl status libvirtd
```

如果服务运行正常，您应该看到 `Active: active (running)` 状态。

## 🧑‍💻 四、使用 virt-manager 图形界面

### 启动图形管理界面

运行以下命令启动 virt-manager：

```bash
virt-manager
```

### 配置用户权限

为了能够正常使用 virt-manager，需要将当前用户添加到 libvirt 组：

```bash
sudo usermod -aG libvirt $(whoami)
```

**重要提示：** 执行权限配置后，需要重新登录或重启系统才能生效。

## 📋 总结

通过以上步骤，您已经成功在 Ubuntu 系统上安装了 KVM 虚拟化环境。现在您可以：

1. 使用 `virt-manager` 创建和管理虚拟机
2. 通过命令行工具管理虚拟化资源
3. 享受完整的 Linux 虚拟化体验

KVM 提供了高性能的虚拟化解决方案，特别适合开发和测试环境使用。

---

*如果您在安装过程中遇到任何问题，欢迎在评论区留言讨论！*


---

> 编辑说明：本文在原文基础上经过 AI 编辑优化。
