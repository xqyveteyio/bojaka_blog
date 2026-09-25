---
title: "Hugo 静态网站生成器用法指南"
date: 2025-10-27 15:00:00
tags:
  - "Hugo 用法教程"
  - "Hugo 静态网站搭建"
  - "Hugo 命令大全"
  - "hugo new site 创建站点"
categories:
  - 笔记
feature: false
comments: false
abstracts: "Hugo 静态网站生成器常用命令速查：创建站点与文章、启动开发服务器（hugo server）、生成静态文件、主题子模块管理，附 config.yaml 配置文件示例和 front matter 格式参考，帮助快速上手 Hugo 博客搭建。"
---

## Hugo 简介

Hugo 是一个用 Go 语言编写的静态网站生成器，以其速度快、配置简单而闻名。它可以快速将 Markdown 文件转换为静态 HTML 网站。

## 最常用的 Hugo 命令

### 1. 创建新站点
```bash
hugo new site mysite
cd mysite
```

### 2. 创建新文章
```bash
# 在 content/posts/ 目录下创建新文章
hugo new posts/my-first-post.md

# 创建带有自定义路径的文章
hugo new content/blog/my-article.md
```

### 3. 启动开发服务器
```bash
# 启动本地开发服务器（默认端口 1313）
hugo server

# 启动服务器并包含草稿文章
hugo server -D

# 指定端口启动
hugo server --port 8080

# 绑定到所有网络接口
hugo server --bind 0.0.0.0
```

### 4. 生成静态网站
```bash
# 生成静态文件到 public/ 目录
hugo

# 生成时包含草稿文章
hugo -D

# 指定输出目录
hugo --destination /path/to/output
```

### 5. 主题管理
```bash
# 添加主题（使用 git submodule）
git submodule add https://github.com/theme-author/theme-name.git themes/theme-name

# 初始化主题子模块
git submodule update --init --recursive

# 更新主题
git submodule update --remote
```

### 6. 其他常用命令
```bash
# 查看 Hugo 版本
hugo version

# 检查配置文件
hugo config

# 列出所有内容
hugo list all

# 清理生成的文件
hugo mod clean
```

## 配置文件示例

Hugo 支持多种配置文件格式，最常用的是 `config.yaml`：

```yaml
baseURL: 'https://example.com'
languageCode: 'zh-cn'
title: '我的博客'
theme: 'your-theme-name'

params:
  description: '这是我的个人博客'
  author: '作者姓名'

menu:
  main:
    - name: '首页'
      url: '/'
      weight: 1
    - name: '文章'
      url: '/posts/'
      weight: 2
```

## Front Matter 示例

文章头部的元数据格式：

```yaml
---
title: "文章标题"
date: 2024-01-15T10:00:00+08:00
draft: false
tags: ["标签1", "标签2"]
categories: ["分类"]
description: "文章描述"
---
```

## 实用技巧

1. **使用 `--navigateToChanged` 参数**：开发时自动跳转到修改的页面
   ```bash
   hugo server --navigateToChanged
   ```

2. **快速原型设计**：使用 `--templateMetrics` 查看模板性能
   ```bash
   hugo --templateMetrics
   ```

3. **部署前检查**：使用 `--minify` 压缩输出文件
   ```bash
   hugo --minify
   ```

Hugo 的强大之处在于其简洁性和速度，掌握这些基本命令就能高效地管理静态网站了。


---

> 编辑说明：本文在原文基础上经过 AI 编辑优化。
