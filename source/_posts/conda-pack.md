---
title: "conda 导出与迁移环境（conda-pack）"
date: 2026-01-07 19:15:00
tags:
  - "conda 导出环境"
  - "conda-pack 打包"
  - "conda 环境迁移"
  - "conda 离线迁移"
categories:
  - 工具
feature: false
comments: false
abstracts: "使用 conda-pack 将已有 conda 环境打包为 tar.gz 压缩包，拷贝到新机器或离线环境解压后执行 conda-unpack 完成路径修复，实现 conda 环境的快速无网络迁移。"
---

# conda 导出环境（conda-pack）

用 `conda-pack` 将已有环境打包，拷贝到另一台机器后解包并执行 `conda-unpack` 完成修复。

## 1. 安装 conda-pack

```bash
conda install -n base -c conda-forge conda-pack
```

## 2. 打包环境

```bash
conda pack -n openmmlab_env -o openmmlab_env.tar.gz
```

## 3. 迁移并解包

将 `openmmlab_env.tar.gz` 拷贝到目标机器，解压到 `envs/` 目录后执行：

```bash
envs/openmmlab_env/bin/conda-unpack
```


---

> 编辑说明：本文在原文基础上经过 AI 编辑优化。
