---
title: "Ubuntu 安装最新版 Docker 并配置用户权限"
date: 2025-11-02 23:15:00
tags:
  - "Ubuntu 安装 Docker"
  - "Docker Ubuntu 教程"
  - "docker-ce 安装 Ubuntu"
  - "Ubuntu Docker 用户权限"
categories:
  - Linux
feature: false
comments: false
abstracts: "在 Ubuntu 上三步完成 Docker 最新版安装：添加 Docker 官方 apt 源、安装 docker-ce，并将当前用户加入 docker 组，之后无需 sudo 即可直接使用 Docker 命令。附一键安装脚本和国内镜像加速配置。"
---

最简单直接的 Docker 安装教程，三步搞定。

---

## 1. 添加 Docker 官方源

```bash
# 添加 Docker GPG 密钥
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# 添加 Docker 软件源
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

---

## 2. 安装 Docker

```bash
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io
```

安装完成后就可以使用了：

```bash
sudo docker run hello-world
```

---

## 3. 配置普通用户权限（可选）

如果不想每次都输入 `sudo`，执行以下命令：

```bash
sudo usermod -aG docker $USER
```

然后**重启系统**或**注销重新登录**，之后就可以直接用 `docker` 命令了：

```bash
docker run hello-world
```

---

## 验证安装

```bash
docker version
docker ps
```

---

## 常见问题

### 权限错误

如果提示权限被拒绝：

```bash
# 确认用户在 docker 组
groups

# 如果没有 docker 组，重新添加
sudo usermod -aG docker $USER

# 然后重启或注销重新登录
```

### 镜像拉取慢

配置国内镜像加速：

```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<EOF
{
  "registry-mirrors": ["https://mirror.ccs.tencentyun.com"]
}
EOF

sudo systemctl restart docker
```

---

## 一键安装脚本

```bash
#!/bin/bash
# 添加源
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 安装
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io

# 配置权限
sudo usermod -aG docker $USER

echo "安装完成！请重启系统或注销重新登录后生效。"
```

---

就这么简单，不需要那么多步骤。


---

> 编辑说明：本文在原文基础上经过 AI 编辑优化。
