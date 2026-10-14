---
type: entity
status: budding
created: 2026-10-14
updated: 2026-10-14
sources:
  - raw/Linux/ArchLinux Installation Guide.md
  - raw/Linux/Installation_Problems.md
tags:
  - linux
  - archlinux
  - package-manager
---

pacman 是 Arch Linux 的包管理器，负责官方仓库与第三方仓库的软件安装、更新与依赖解析。

## 两个配置文件的分工

| 文件 | 作用 |
|---|---|
| `/etc/pacman.d/mirrorlist` | 官方仓库的**服务器地址列表**；pacman 按文件中的顺序依次尝试，因此最快的（国内）镜像要放在最顶端 |
| `/etc/pacman.conf` | pacman 的**主配置文件**；定义所有仓库（官方与第三方）及全局行为。官方仓库通过 `Include = /etc/pacman.d/mirrorlist` 引用镜像列表，第三方仓库直接在此定义 |

## 常用命令

| 命令 | 用途 |
|---|---|
| `pacman -Sy archlinux-keyring` | 更新密钥环（`pacstrap` 报 PGP 签名错误时先做这步） |
| `pacstrap -K /mnt base linux linux-firmware` | 安装基础系统（在 Live 环境执行） |
| `pacman -S networkmanager` | 安装网络管理器 |
| `pacman -S reflector` | 安装镜像自动排序工具 |
| `pacman -Syu archlinuxcn-keyring` | 导入 archlinuxcn 仓库密钥 |
| `pacman -Syyu` | 全量刷新数据库并更新系统 |
| `pacman-key --init` / `--populate archlinux` | 初始化密钥环 |
| `pacman-key --lsign-key "farseerfc@archlinux.org"` | 本地签署并信任密钥 |

## 镜像加速

- 手动编辑 `mirrorlist`，把 `mirrors.ustc.edu.cn`、`mirrors.tuna.tsinghua.edu.cn` 等置顶。
- 或用 `reflector` 自动排序：
  ```bash
  sudo reflector -c China --sort rate --save /etc/pacman.d/mirrorlist
  ```
- **`pacman-mirrors` 是 Manjaro 专有工具**，纯 Arch 中不存在，不要照搬。

## 第三方仓库：archlinuxcn

在 `/etc/pacman.conf` **末尾**追加：

```ini
[archlinuxcn]
Server = https://mirrors.ustc.edu.cn/archlinuxcn/$arch
```

- 之后用 `pacman -S yay` 等安装中文社区软件。
- **不要**把该地址写进 `/etc/pacman.d/mirrorlist`（会格式错误）。
- **不要**把第三方仓库写在官方仓库之前，以免系统核心组件被替换。
- ⚠️ **不要用 `SigLevel = never`** 跳过签名验证，应继承全局 `SigLevel` 并正确导入密钥。

## 关联

- 宿主系统：[[Arch Linux]]
- 使用场景：[[2026-10-14-ArchLinux安装流程]]、[[2026-10-14-ArchLinux安装问题排查]]

来源：[[raw/Linux/ArchLinux Installation Guide]]、[[raw/Linux/Installation_Problems]]
