---
title: "Fedora 使用 KVM 虚拟化技术完整指南"
date: 2026-02-03 17:45:00
tags:
  - "Fedora KVM 安装"
  - "Fedora 虚拟机配置"
  - "KVM libvirt Fedora"
  - "virt-manager Fedora"
categories:
  - Linux
feature: false
comments: false
abstracts: "在 Fedora 上安装 KVM 虚拟化环境的完整步骤：通过 dnf 安装 qemu-kvm、libvirt、virt-install 和 virt-manager，启动 libvirtd 服务，并将用户加入 libvirt/kvm 组以实现无需 sudo 管理虚拟机。"
---

## 1) 安装需要的软件包（Fedora）
```bash
sudo dnf install -y qemu-kvm libvirt virt-install virt-manager bridge-utils
```

## 2) 启用并启动 libvirt（Fedora 用 systemd）
```bash
sudo systemctl enable --now libvirtd
```

检查状态：
```bash
systemctl status libvirtd
```

## 3) 让当前用户无需 sudo 管理虚拟机（推荐）
```bash
sudo usermod -aG libvirt,kvm $USER
```
然后**注销/重新登录**一次（或重启）让组权限生效。

## 4) 验证 KVM 是否可用
```bash
lsmod | grep kvm
sudo virsh list --all
```


---

> 编辑说明：本文在原文基础上经过 AI 编辑优化。
