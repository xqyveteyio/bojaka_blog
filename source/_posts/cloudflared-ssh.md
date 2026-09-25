---
title: "不支持ProxyCommand的ssh工具连接cloudflareTunnels教程"
date: 2026-09-15 23:59:00
tags:
  - "ProxyCommand"
  - "cloudflare"
  - "Tunnels"
categories:
  - 工具
feature: false
comments: false
abstracts: "有些 SSH 客户端不支持 ProxyCommand，就不能在配置里直接写 cloudflared access tcp。"
---

有些 SSH 客户端不支持 `ProxyCommand`，就不能在配置里直接写 `cloudflared access tcp`。

这种客户端可以先在终端把 Cloudflare Tunnel 映射到本机端口，再让 SSH 去连这个本地端口：

```bash
cloudflared access tcp --hostname self-ssh-1.keyboard2005.net --listener localhost:12345
```

映射起来之后，SSH 的主机填 `localhost`，端口填 `12345`。`hostname` 换成你自己的 Tunnel 主机名。


---

> 编辑说明：本文在原文基础上经过 AI 编辑优化。
