---
title: "conda 默认不激活 base 环境"
date: 2026-01-31 03:18:00
tags:
  - "conda 关闭自动激活 base"
  - "conda auto_activate_base false"
  - "conda 不激活 base 环境"
  - "conda 启动不进 base"
categories:
  - 工具
feature: false
comments: false
abstracts: "一行命令关闭 conda 每次启动终端时自动激活 base 环境的行为，避免进入不需要的 Python 环境，适用于 Anaconda / Miniconda 用户。"
---

Anaconda 和 Miniconda 默认会在每次打开终端时激活 `base`。不需要的话，关掉自动激活：

```bash
conda config --set auto_activate_base false
```

新开一个终端就不会再自动进入 `base`。需要时再手动 `conda activate`。


---

> 编辑说明：本文在原文基础上经过 AI 编辑优化。
