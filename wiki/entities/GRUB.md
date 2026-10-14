---
type: entity
status: budding
created: 2026-10-14
updated: 2026-10-14
sources:
  - raw/Linux/ArchLinux Installation Guide.md
tags:
  - linux
  - archlinux
  - boot
---

GRUB 是通用引导程序，在 UEFI 双系统场景下通过 `os-prober` 探测并生成 Windows 启动项。

## 安装（UEFI）

```bash
pacman -S grub efibootmgr
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg
```

## 检测 Windows

```bash
pacman -S os-prober
# 编辑 /etc/default/grub，取消注释 GRUB_DISABLE_OS_PROBER=false
grub-mkconfig -o /boot/grub/grub.cfg
```

## 与 systemd-boot 的取舍

| 维度 | GRUB | [[systemd-boot]] |
|---|---|---|
| 配置方式 | 由 `grub-mkconfig` 生成 `grub.cfg` | 手写 `/boot/loader/entries/*.conf` |
| 体积与依赖 | 较重 | 轻量 |
| 探测 Windows | 需 `os-prober` 且显式放开开关 | 默认自动扫描 ESP（可能产生重复条目） |
| 适用 | 需要更复杂的引导菜单/兼容性 | 追求简洁的 UEFI 单/双系统 |

## 关联

- 场景：[[UEFI双系统引导]]
- 宿主系统：[[Arch Linux]]

来源：[[raw/Linux/ArchLinux Installation Guide]]
