---
type: entity
status: budding
created: 2026-10-14
updated: 2026-10-14
sources:
  - raw/Linux/Installation_Problems.md
  - raw/Linux/ArchLinux Installation Guide.md
tags:
  - linux
  - archlinux
  - i18n
---

fcitx5 是 Linux 上的输入法框架，在 Arch 中通过 [[pacman]] 安装，并依靠图形环境变量接入应用程序。

## 安装

```bash
sudo pacman -S fcitx5 fcitx5-chinese-addons fcitx5-qt fcitx5-gtk
```

## 环境变量

在 `~/.xprofile` 中追加：

```
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS=@im=fcitx
```

## 注意点

- fcitx5 **只在图形界面生效**，TTY 下无效。TTY 中文显示为方块的根因是内核帧缓冲字体，与输入法无关，详见 [[Linux中文显示与locale]]。
- 需要先装中文字体（`noto-fonts-cjk`）才能正常显示候选词。

## 关联

- 环境配置：[[Linux中文显示与locale]]
- 宿主系统：[[Arch Linux]]、[[2026-10-14-ArchLinux安装流程]]

来源：[[raw/Linux/Installation_Problems]]、[[raw/Linux/ArchLinux Installation Guide]]
