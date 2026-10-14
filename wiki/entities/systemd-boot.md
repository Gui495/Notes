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
  - boot
---

systemd-boot 是 systemd 提供的轻量 UEFI 引导程序，靠 ESP 上的纯文本配置文件定义启动条目，不使用自动生成的复杂配置。

## 安装

```bash
bootctl install
```

**重要**：`arch-chroot` 内无法创建 UEFI 启动项（chroot 阻止修改 UEFI 变量）。此时应退出 chroot，在 Live 环境执行：

```bash
bootctl --esp-path=/mnt/boot install
```

## 配置文件

`/boot/loader/loader.conf`：
```conf
default arch.conf
timeout 5
auto-entries 0
```

`/boot/loader/entries/arch.conf`：
```conf
title   Arch Linux
linux   /vmlinuz-linux
initrd  /initramfs-linux.img
options root=UUID=你的根分区UUID rw
```

`/boot/loader/entries/windows.conf`（多系统）：
```conf
title Windows 11
efi /EFI/Microsoft/Boot/bootmgfw.efi
```

## 常见故障

| 现象 | 原因 | 解决 |
|---|---|---|
| 只有 "Reboot Into Firmware Interface" | 缺 `arch.conf`，或内核/initramfs 不在 ESP 中 | 确认 ESP 挂在 `/boot`，补建条目，`root=` 用 UUID |
| 直接进 Windows，无菜单 | `loader.conf` 未设默认项；或 `efi` 路径拼错（如 `bottmgfu.efi`） | 补 `default`/`timeout`，改正拼写 |
| 菜单出现两个 Windows | 默认自动扫描所有 ESP 的引导文件 | `loader.conf` 加 `auto-entries 0`，再 `bootctl update` |

## 关联

- 场景与对比：[[UEFI双系统引导]]、[[GRUB]]
- 宿主系统：[[Arch Linux]]
- 故障记录：[[2026-10-14-ArchLinux安装问题排查]]

来源：[[raw/Linux/ArchLinux Installation Guide]]、[[raw/Linux/Installation_Problems]]
