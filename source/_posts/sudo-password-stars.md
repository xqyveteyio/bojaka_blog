---
title: "Linux 终端输入密码时显示星号"
date: 2026-06-11 14:00:00
tags:
  - "Linux sudo 显示星号"
  - "Linux 终端密码反馈"
  - "visudo pwfeedback 配置"
  - "sudo 密码输入显示星号"
categories:
  - Linux
feature: false
comments: false
abstracts: "Linux 终端执行 sudo 命令时默认不显示任何密码字符。本文介绍通过在 visudo 中为 Defaults 添加 pwfeedback 选项，让密码输入时显示星号（*）反馈，增强输入体感，并附恢复默认的方法。"
---

Linux 默认在终端输入密码时不显示任何字符，这是为了防止旁人根据输入长度猜密码。如果你希望输入时能看到 `*` 提示，改一下 sudo 配置就行。

## 操作步骤

打开 sudo 配置文件：

```bash
sudo visudo
```

找到这一行：

```
Defaults env_reset
```

在末尾加上 `,pwfeedback`：

```
Defaults env_reset,pwfeedback
```

如果找不到 `Defaults env_reset`，直接新起一行写：

```
Defaults pwfeedback
```

保存退出（nano 下 `Ctrl+O` 保存，`Ctrl+X` 退出）。配置立即生效，下次 `sudo` 输入密码时就会显示星号了。

## 恢复默认

想改回什么都不显示，再次运行 `sudo visudo`，删掉 `,pwfeedback` 即可。


---

> 编辑说明：本文在原文基础上经过 AI 编辑优化。
