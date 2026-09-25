---
title: "Fedora 安装最新版 Docker 并配置用户权限"
date: 2026-01-31 03:22:00
tags:
  - "Fedora 安装 Docker"
  - "Docker Fedora 官方源"
  - "dnf docker-ce 安装"
  - "Fedora Docker 完整教程"
categories:
  - Linux
feature: false
comments: false
abstracts: "在 Fedora 上通过 Docker 官方 RPM 仓库安装最新版 Docker CE 的完整步骤：安装 dnf-plugins-core、用 addrepo 添加官方源、安装 docker-ce，以及将用户加入 docker 组实现无需 sudo 使用 Docker。"
---

### 1) 安装插件
```bash
sudo dnf -y install dnf-plugins-core
```

### 2) 用新命令添加 repo（Fedora 43 适用）
在新版 dnf5 上，通常是用 `addrepo` 子命令，而不是 `--add-repo` 参数：

```bash
sudo dnf config-manager addrepo --from-repofile=https://download.docker.com/linux/fedora/docker-ce.repo
```

如果你的机器上该子命令名字略不同，也可以直接导入 repo 文件（见下面“方案 B”）。

### 3) 刷新缓存并确认仓库存在
```bash
sudo dnf clean all
sudo dnf makecache
sudo dnf repolist | grep -i docker
```

### 4) 再安装 Docker CE
```bash
sudo dnf -y install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### 5) 启动并设置开机自启
```bash
sudo systemctl enable --now docker
```
### 6) 配置普通用户权限
```bash
sudo usermod -aG docker $USER
newgrp docker # 立即生效
```

注销或者重启系统后，普通用户即可使用 Docker 命令而无需 sudo


---

> 编辑说明：本文在原文基础上经过 AI 编辑优化。
