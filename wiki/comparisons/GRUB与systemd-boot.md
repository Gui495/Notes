---
type: comparison
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
  - comparison
---

GRUB 与 systemd-boot 都能在 UEFI 下引导 Arch，差别主要在配置方式、Windows 探测机制，以及各自暴露过的故障模式。

## 逐项对照

| 维度 | [[GRUB]] | [[systemd-boot]] |
|---|---|---|
| 安装命令 | `pacman -S grub efibootmgr` → `grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB` | `bootctl install` |
| 配置生成 | `grub-mkconfig -o /boot/grub/grub.cfg` 自动生成 | 手写 `/boot/loader/loader.conf` 与 `entries/*.conf` |
| 配置载体 | 生成的单一 `grub.cfg` | 若干纯文本条目文件 |
| Windows 探测 | 需装 `os-prober`，并在 `/etc/default/grub` 打开 `GRUB_DISABLE_OS_PROBER=false`，再重新生成 | 默认自动扫描所有 ESP；也可手写 `windows.conf` 指定 `efi /EFI/Microsoft/Boot/bootmgfw.efi` |
| chroot 内安装 | 素材未记录限制 | **有限制**：`arch-chroot` 阻止改 UEFI 变量，需退出到 Live 环境执行 `bootctl --esp-path=/mnt/boot install` |
| 重复条目问题 | 素材未记录 | 有：自动扫描会把另一块硬盘 ESP 里的 Windows 也列出来，菜单出现两个 Windows |

## 各自踩过的坑（来自素材实证）

**GRUB**
- 不装 `os-prober` 或没打开 `GRUB_DISABLE_OS_PROBER=false` 时，菜单里不会出现 Windows。

**systemd-boot**
- `loader.conf` 为空（只有注释）时没有默认项，可能直接进 Windows。
- 条目里 `efi` 路径拼错（如 `bottmgfu.efi`）会让该条目失效。
- 缺 `arch.conf` 或内核/initramfs 不在 ESP 中时，重启只看到 "Reboot Into Firmware Interface"。
- 多 ESP 场景下重复扫描 Windows，需在 `loader.conf` 加 `auto-entries 0` 并 `bootctl update`。

## 怎么选

素材本身没有给出「哪个更好」的结论，只提供了两条路径的完整命令与故障记录。可据以下事实自行取舍：

- 想要**自动生成配置、少写文件** → GRUB 省事，但要额外处理 os-prober 开关。
- 想要**配置可读、条目即文件** → systemd-boot 直观，但要接受手写条目与自动扫描带来的重复条目风险。
- 装系统时若只能在 `chroot` 内操作 → systemd-boot 需要退回 Live 环境，GRUB 素材未提及此限制。

## 关联

- 场景与分区前提：[[UEFI双系统引导]]
- 相关工具：[[GRUB]]、[[systemd-boot]]、[[pacman]]
- 发行版：[[Arch Linux]]

来源：[[raw/Linux/ArchLinux Installation Guide]]、[[raw/Linux/Installation_Problems]]
