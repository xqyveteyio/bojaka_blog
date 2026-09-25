---
title: "GNOME Alt+Tab 切换窗口展开多浏览器"
date: 2026-02-02 00:19:00
tags:
  - "GNOME Alt+Tab 多窗口展开"
  - "GNOME 切换窗口不分组"
  - "gsettings switch-windows"
  - "GNOME 浏览器窗口分开切换"
categories:
  - Linux
feature: false
comments: false
abstracts: "通过 gsettings 将 GNOME 桌面 Alt+Tab 的切换行为从按应用分组改为列出全部独立窗口，让同时打开的多个浏览器/应用窗口在切换时分开显示，而不是合并成一个图标。一条命令搞定，附恢复默认方法。"
---

在 GNOME 桌面环境中，默认的 **Alt+Tab** 切换窗口行为是按应用程序分组的（即同一应用的多个窗口会合并为一个图标）。这对于某些用户来说可能不太方便，尤其是当你同时打开多个浏览器窗口时。


## 用 gsettings 改 Alt+Tab 行为
打开终端执行：

```bash
gsettings set org.gnome.desktop.wm.keybindings switch-windows "['<Alt>Tab']"
gsettings set org.gnome.desktop.wm.keybindings switch-windows-backward "['<Shift><Alt>Tab']"

gsettings set org.gnome.desktop.wm.keybindings switch-applications "[]"
gsettings set org.gnome.desktop.wm.keybindings switch-applications-backward "[]"
```

改完后：
- **Alt+Tab** 会直接列出所有窗口（两个浏览器窗口会分开出现）
- **Shift+Alt+Tab** 反向切换

如需恢复默认：

```bash
gsettings reset org.gnome.desktop.wm.keybindings switch-windows
gsettings reset org.gnome.desktop.wm.keybindings switch-windows-backward
gsettings reset org.gnome.desktop.wm.keybindings switch-applications
gsettings reset org.gnome.desktop.wm.keybindings switch-applications-backward
```


---

> 编辑说明：本文在原文基础上经过 AI 编辑优化。
