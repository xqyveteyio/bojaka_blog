---
title: "用 Chrome 把局域网网站变成独立 App"
date: 2026-07-17 23:20:00
tags:
  - "Chrome 应用模式"
  - "Chrome --app 快捷方式"
  - "Fedora 创建独立网页 App"
  - "Fedora WebKit WebView 卡顿"
categories:
  - 工具
feature: false
comments: false
abstracts: "Fedora 上 WebKit WebView 需要一堆环境变量才能启动，启动后还有 GPU 卡顿和功能缺失。改用 Chrome 应用模式把任意 URL（包括局域网 HTTP）做成独立窗口，并去掉无证书警告。"
---

家里有些服务是纯网页的，比如局域网里的 Vikunja。本来想直接找个 WebView 套壳做成“独立软件”——Electron、Tauri、Neutralino 这类常规方案都试过。

在 Fedora 上的实际体验是：系统 WebKit 兼容性很差，这些框架经常没法直接启动，得先塞一堆环境变量，比如：

```bash
WEBKIT_DISABLE_COMPOSITING_MODE=1
GDK_BACKEND=x11
WEBKIT_DISABLE_DMABUF_RENDERER=1
```

硬启动起来之后也不好用。界面会卡，感觉和 GPU 加速那一套对不上；拖拽排序这类交互在 WebView 里直接不可用。能开 ≠ 能用。

后来想到很多网站会提示“安装应用”，装完其实就是 Chrome 开一个没有地址栏的独立窗口。既然本机已经有最新的 Chrome，不如直接用它——流畅很多，也少踩 WebKit 那堆坑。

## 先让 Chrome 开出独立窗口

用 Chrome 打开目标网站，右上角三点菜单里找「保存并分享」→「创建快捷方式…」，勾上「作为窗口打开」，确认。

之后桌面或应用菜单里会出现一个图标。双击打开就是一个干净窗口：没有标签栏、没有地址栏，看起来像个普通软件，底层还是你平时那份 Chrome，不会撞上系统 WebKit / WebView 那套兼容性问题。

Fedora 上 Chrome 会在 `~/.local/share/applications/` 里自动生成一个 `.desktop` 文件，名字类似：

```
chrome-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx-Default.desktop
```

我这边 Vikunja 生成出来的内容大概是这样：

```ini
[Desktop Entry]
Version=1.0
Terminal=false
Type=Application
Name=Vikunja
Exec=/opt/google/chrome/google-chrome --profile-directory=Default --app-id=hfmefipklbdolennkgfbmboknaiablkb
Icon=chrome-hfmefipklbdolennkgfbmboknaiablkb-Default
StartupWMClass=crx_hfmefipklbdolennkgfbmboknaiablkb
```

也就是说，它用的是主配置文件（`Default`）加上一个 `--app-id`，并不是直接 `--app=网址`。对于正常 HTTPS 网站，这样已经够用了。

## 局域网 HTTP 会有一条难看的警告

我的 Vikunja 跑在局域网 HTTP 上，大概是 `http://192.168.31.182:3456`，根本没有证书，也不需要证书。问题是 Chrome 不管你是不是内网，照样在窗口上面挂一条“不安全 / 未加密”的提示，独立窗口里看起来特别别扭。

网上常见做法是给启动命令加：

```bash
--unsafely-treat-insecure-origin-as-secure="http://192.168.31.182:3456"
```

但有个坑：这个参数在和主浏览器共用配置目录时经常不生效。Chrome 要求它跑在独立的用户数据目录里才会认。所以还得再加：

```bash
--user-data-dir=$HOME/.config/chrome-vikunja-app
```

可以先在终端验证：

```bash
/opt/google/chrome/google-chrome \
  --app="http://192.168.31.182:3456/" \
  --unsafely-treat-insecure-origin-as-secure="http://192.168.31.182:3456" \
  --user-data-dir=$HOME/.config/chrome-vikunja-app
```

窗口正常打开、警告条没了，就说明参数没问题。首次启动会像新装的浏览器一样问你要不要设默认浏览器之类的，点否就行。登录状态也不会和日常用的 Chrome 共用，相当于单独一份配置。

## 改掉自动生成的 .desktop

终端能跑通之后，把应用菜单里的快捷方式也改成同一套参数。直接编辑那个自动生成的文件：

```bash
nano ~/.local/share/applications/chrome-hfmefipklbdolennkgfbmboknaiablkb-Default.desktop
```

我最后改成这样（主入口和右键快捷动作都带上了信任参数）：

```ini
[Desktop Entry]
Version=1.0
Terminal=false
Type=Application
Name=Vikunja
Exec=/opt/google/chrome/google-chrome --app="http://192.168.31.182:3456/" --unsafely-treat-insecure-origin-as-secure="http://192.168.31.182:3456" --user-data-dir=%h/.config/chrome-vikunja-app
Icon=chrome-hfmefipklbdolennkgfbmboknaiablkb-Default
StartupWMClass=crx_hfmefipklbdolennkgfbmboknaiablkb
Actions=Namespaces-And-Projects-Overview;Overview;Tasks-Next-Month;Tasks-Next-Week;Teams-Overview

[Desktop Action Namespaces-And-Projects-Overview]
Name=Namespaces And Projects Overview
Exec=/opt/google/chrome/google-chrome --app="http://192.168.31.182:3456/namespaces" --unsafely-treat-insecure-origin-as-secure="http://192.168.31.182:3456" --user-data-dir=%h/.config/chrome-vikunja-app

[Desktop Action Overview]
Name=Overview
Exec=/opt/google/chrome/google-chrome --app="http://192.168.31.182:3456/" --unsafely-treat-insecure-origin-as-secure="http://192.168.31.182:3456" --user-data-dir=%h/.config/chrome-vikunja-app

[Desktop Action Tasks-Next-Month]
Name=Tasks Next Month
Exec=/opt/google/chrome/google-chrome --app="http://192.168.31.182:3456/tasks/by/month" --unsafely-treat-insecure-origin-as-secure="http://192.168.31.182:3456" --user-data-dir=%h/.config/chrome-vikunja-app

[Desktop Action Tasks-Next-Week]
Name=Tasks Next Week
Exec=/opt/google/chrome/google-chrome --app="http://192.168.31.182:3456/tasks/by/week" --unsafely-treat-insecure-origin-as-secure="http://192.168.31.182:3456" --user-data-dir=%h/.config/chrome-vikunja-app

[Desktop Action Teams-Overview]
Name=Teams Overview
Exec=/opt/google/chrome/google-chrome --app="http://192.168.31.182:3456/teams" --unsafely-treat-insecure-origin-as-secure="http://192.168.31.182:3456" --user-data-dir=%h/.config/chrome-vikunja-app
```

几个要点：

- 去掉原来的 `--profile-directory=Default --app-id=...`，改成 `--app="网址"` 直连。
- `%h` 是 `.desktop` 里表示家目录的写法，等价于 `$HOME`。
- `Icon=` 继续用 Chrome 生成的那个，菜单里图标不会丢。

改完之后先把已经打开的 Vikunja 窗口全部关掉，再刷一下桌面数据库：

```bash
update-desktop-database ~/.local/share/applications/
```

然后从应用菜单重新打开。警告条没了，右键那些快捷入口也能正常跳转。

## 顺便记一下

- 我这边 Chrome 是 dnf 装的，路径是 `/opt/google/chrome/google-chrome`。如果是 Flatpak 版，命令和沙盒权限会不一样，不能直接照抄。
- `--unsafely-treat-insecure-origin-as-secure` 只适合自己信任的内网地址，别往公网乱加。
- 换了一个独立的 `user-data-dir` 之后，登录态、插件、主题都是单独一份。对我这种只想当客户端用的局域网服务来说刚好合适，不会和日常浏览搅在一起。

WebView 能凑合跑但卡顿又缺功能的时候，用本机最新 Chrome 套一层应用模式会顺很多。不需要自己写 Electron / Tauri，也不用再跟 WebKit 环境变量较劲。


---

> 编辑说明：本文在原文基础上经过 AI 编辑优化。
