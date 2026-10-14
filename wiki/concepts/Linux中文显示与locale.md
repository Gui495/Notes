---
type: concept
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
  - locale
---

Linux 中文显示与 locale 的核心矛盾是：TTY 用内核帧缓冲字体，**不支持中文**；图形界面则依赖字体与 locale 设置。解决办法是让系统全局锁定英文、只在图形界面覆盖为中文。

## 分层策略

| 环境 | 设置 | 位置 |
|---|---|---|
| TTY（纯命令行） | `LANG=en_US.UTF-8` | `/etc/locale.conf` |
| 图形界面（X11/Wayland） | `export LANG=zh_CN.UTF-8`、`export LC_CTYPE=zh_CN.UTF-8` | `~/.xprofile`，或 `~/.xinitrc`（`startx` 场景，须写在 `exec` 桌面环境之前） |

`~/.xprofile` 在用户登录桌面**之前**自动加载，因此进入后桌面环境即为中文。

## 字体

```bash
sudo pacman -S noto-fonts-cjk
```

**关键**：`noto-fonts-cjk` 是图形界面字体，**在 TTY 下无效**——这就是「装了中文字体，TTY 里还是方块」的原因。

## 输入法（可选）

```bash
sudo pacman -S fcitx5 fcitx5-chinese-addons fcitx5-qt fcitx5-gtk
```

并在 `~/.xprofile` 追加：

```
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS=@im=fcitx
```

## 验证

```bash
locale              # TTY 下应为 en_US.UTF-8；图形界面终端应为 zh_CN.UTF-8
fc-list :lang=zh    # 应列出已安装的中文字体
```

配置修改后需**重启生效**。若图形界面仍乱码，检查 locale 是否已导出、字体是否安装、以及是否重启过会话。

## 关联

- 输入法工具：[[fcitx5]]
- 安装流程中的位置：[[2026-10-14-ArchLinux安装流程]]
- 发行版背景：[[Arch Linux]]

来源：[[raw/Linux/Installation_Problems]]、[[raw/Linux/ArchLinux Installation Guide]]
