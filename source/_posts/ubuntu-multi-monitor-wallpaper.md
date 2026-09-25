---
title: "Ubuntu 24 多显示器设置不同壁纸"
date: 2025-03-22 10:00:00
tags:
  - "Ubuntu 多显示器壁纸"
  - "Linux 多屏幕设置不同壁纸"
  - "GNOME 双屏壁纸"
  - "Ubuntu 24 壁纸设置"
categories:
  - Linux
cover: /images/ubuntu-multi-monitor-wallpaper/1.png
feature: false
comments: false
abstracts: "Ubuntu 24 / Linux GNOME 桌面默认不支持多显示器设置不同壁纸。本文介绍如何用 Python PIL 将多张图片横向拼接为一张大图，再通过 dconf 以 spanned 模式设置，实现双屏或多屏各自显示独立壁纸的效果。"
---

Ubuntu 24 并不支持多显示器设置不同壁纸，但是可以通过设置一个大的拉伸壁纸来实现这个效果。

## 效果图
![三块屏幕各自显示不同壁纸](/images/ubuntu-multi-monitor-wallpaper/1.png)


## 设置壁纸方法
写 `dconf` 之前，如果系统里还没有这个工具，先安装：

```bash
sudo apt install dconf-editor
```

```python
import os


def set_wallpaper(path, mode="zoom"):
    cmd = f"""
dconf write /org/gnome/desktop/background/picture-uri-dark "'file://{path}'"
dconf write /org/gnome/desktop/background/picture-uri "'file://{path}'"
dconf write /org/gnome/desktop/background/picture-options "'{mode}'"
"""
    os.system(cmd)
```

## 横向拼接多张图片

```python
from PIL import Image


def horizontal_concatenate(images):
    """
    横向拼接多张图片。
    :param images: 图片文件路径的列表
    :return: 拼接后的图片对象
    """
    # 打开所有图片
    img_list = [Image.open(img) for img in images]

    # 获取图片的宽度和高度
    widths, heights = zip(*(img.size for img in img_list))

    # 计算拼接后图片的总宽度和最大高度
    total_width = sum(widths)
    max_height = max(heights)

    # 创建一个新的空白图片，大小为（总宽度, 最大高度）
    new_img = Image.new("RGB", (total_width, max_height))

    # 将每张图片粘贴到新图片上
    x_offset = 0  # 当前的横向偏移量
    for img in img_list:
        new_img.paste(img, (x_offset, 0))
        x_offset += img.size[0]  # 更新偏移量

    return new_img
```

## 使用示例

```python
def main():
    # 三张图片的路径
    image_paths = [
        "/home/ogumo/文档/GitHub/Multiple-Monitors-Wallpaper/img/1.jpg",
        "/home/ogumo/文档/GitHub/Multiple-Monitors-Wallpaper/img/2.png",
        "/home/ogumo/文档/GitHub/Multiple-Monitors-Wallpaper/img/3.jpg",
    ]

    # 横向拼接图片
    result = horizontal_concatenate(image_paths)

    # 保存结果
    result.save("result.png")
    # result.show()  # 显示拼接后的图片

    path = os.path.abspath("result.png")
    set_wallpaper(path, "spanned")


if __name__ == "__main__":
    main()
```


原文示例里用 `os.path.join(os.getcwd(), "/home/...")` 拼路径。第二个参数已经是绝对路径时，`join` 会丢掉前面的当前目录，壁纸路径会指错。这里改成对刚保存的 `result.png` 取绝对路径，再交给 `set_wallpaper`，模式用 `spanned`。


---

> 编辑说明：本文在原文基础上经过 AI 编辑优化。
