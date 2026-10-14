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
  - distro
---

Arch Linux 是采用滚动更新模型的 x86-64 Linux 发行版，安装过程完全由命令行手工完成：手工分区、`pacstrap` 装基础系统、`chroot` 配置、最后自行安装引导程序。

## 本库素材覆盖的主线

| 主题 | 页面 |
|---|---|
| 完整安装流程（9 步） | [[2026-10-14-ArchLinux安装流程]] |
| 安装过程 12 个实际问题 | [[2026-10-14-ArchLinux安装问题排查]] |
| 磁盘准备 | [[磁盘分区与挂载]] |
| 引导配置 | [[UEFI双系统引导]] |
| 中文环境 | [[Linux中文显示与locale]] |

## 生态部件

| 部件 | 作用 |
|---|---|
| [[pacman]] | 包管理器，配 `mirrorlist` 与 `pacman.conf` |
| [[systemd-boot]] | 轻量 UEFI 引导程序 |
| [[GRUB]] | 通用引导程序，可探测 Windows |
| [[iwd]] | 无线网络守护进程（安装镜像默认） |
| [[fcitx5]] | 输入法框架 |

## 素材中反复出现的三条经验

1. 在 `/etc/fstab` 和引导配置中一律使用 **UUID** 而非设备名，见 [[持久化块设备命名]]。
2. 网络管理器必须在 `chroot` 内安装并启用，否则重启后无法联网。
3. 已有 EFI 分区时不要重新格式化。

来源：[[raw/Linux/ArchLinux Installation Guide]]、[[raw/Linux/Installation_Problems]]
