---
title: "使用frp + frp-panel 可视化管理frp穿透映射"
date: 2026-09-15 23:59:00
tags:
  - "frp-panel"
categories:
  - 工具
feature: false
comments: false
abstracts: "这篇还是安装草稿。下面只保留 frp-panel master 的编排骨架，方便以后补完客户端和映射。"
---

这篇还是安装草稿。下面只保留 frp-panel master 的编排骨架，方便以后补完客户端和映射。

原文 compose 里写了真实的 `APP_GLOBAL_SECRET` 和公网 IP，这里改成占位符。密钥请自己生成一串足够长的随机字符串，IP 换成这台 master 的公网地址。

```yaml
services:
  frpp-master:
    image: vaalacat/frp-panel:latest
    container_name: frpp-master
    network_mode: host
    restart: unless-stopped
    command: master
    volumes:
      - ./data:/data
    environment:
      APP_GLOBAL_SECRET: "换成一串足够长的随机字符串"
      MASTER_RPC_HOST: "你的服务器公网IP"
      MASTER_RPC_PORT: 9001
      MASTER_API_HOST: "你的服务器公网IP"
      MASTER_API_PORT: 9000
      CLIENT_RPC_URL: "grpc://你的服务器公网IP:9001"
      CLIENT_API_URL: "http://你的服务器公网IP:9000"
      APP_ENABLE_REGISTER: "false"
```

客户端怎么登记、每条穿透怎么在面板里建，原文停在 TODO，这里也不补没有记过的步骤。


---

> 编辑说明：本文在原文基础上经过 AI 编辑优化。
