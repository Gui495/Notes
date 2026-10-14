---
type: concept
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

UEFI 双系统引导指在同一台机器上让 Linux 与 Windows 各自可启动的配置工作，核心是 ESP（EFI 系统分区）上的引导程序与启动条目。

## 两个方案

### 方案一：GRUB
```bash
pacman -S grub efibootmgr
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg
```
检测 Windows 需安装 `os-prober`，并在 `/etc/default/grub` 中取消注释 `GRUB_DISABLE_OS_PROBER=false` 后重新生成配置。

### 方案二：systemd-boot（更轻量）
```bash
bootctl install
```
手动创建 `/boot/loader/loader.conf`（`default arch.conf`、`timeout 5`）与 `/boot/loader/entries/arch.conf`；多系统再补 `windows.conf`，其中 `efi /EFI/Microsoft/Boot/bootmgfw.efi`。

## 双系统四个高频坑

| 现象 | 原因 | 解决 |
|---|---|---|
| 重启后只有 "Reboot Into Firmware Interface" | 缺 `arch.conf`，或内核/initramfs 不在 ESP 中 | 确认 ESP 挂在 `/boot`，补建 `arch.conf`，`root=` 用 UUID |
| 重启后直接进 Windows，无菜单 | `loader.conf` 未设默认项；或 `windows.conf` 里 `efi` 路径拼错 | 设 `default` 与 `timeout`；检查 `bootmgfw.efi` 拼写 |
| 菜单里出现两个 Windows | systemd-boot 自动扫描了其他 ESP 的引导文件 | `loader.conf` 加 `auto-entries 0`，再 `bootctl update` |
| chroot 中 `bootctl install` 无法创建启动项 | `arch-chroot` 阻止修改 UEFI 变量 | 退出 chroot，在 Live 环境执行 `bootctl --esp-path=/mnt/boot install` |

「两个 Windows」的成因值得记住：Arch 与 Windows 装在不同硬盘、各有独立 ESP 时，systemd-boot 既会列出你手写的条目，也会自动发现 Windows 自己 ESP 中的引导文件。

## 装完后

重启后拔掉 U 盘，进入 BIOS 把 `Linux Boot Manager`（或 GRUB）设为第一启动项。找不到启动设备时，重进 Live 环境挂载根分区与 ESP，确认挂载点后重装引导程序。

## 关联

- 分区前提：[[磁盘分区与挂载]]
- ESP 中的路径要用 UUID：[[持久化块设备命名]]
- 涉及工具：[[systemd-boot]]、[[GRUB]]、[[Arch Linux]]

来源：[[raw/Linux/ArchLinux Installation Guide]]、[[raw/Linux/Installation_Problems]]
