---
type: summary
status: budding
created: 2026-10-14
updated: 2026-10-14
sources:
  - raw/Linux/ArchLinux Installation Guide.md
tags:
  - linux
  - archlinux
  - install
---

对 [[raw/Linux/ArchLinux Installation Guide]] 的提炼：按时间顺序的完整安装与配置流程，以及原素材汇总的问题表索引。

## 流程总览

### 一、安装前准备
1. 下载 ISO（`archlinux-YYYY.MM.DD-x86_64.iso`）及 `.sig` 签名文件，`pacman-key -v` 验证。
2. 用 `dd`（Linux/macOS）或 Rufus/balenaEtcher（Windows）写盘，**目标是整个设备而非分区**。
3. Windows 磁盘管理中压缩出空闲空间；BIOS 关闭 **Secure Boot**。
4. 从 U 盘启动，选 **Arch Linux install medium (x86_64, UEFI)**。

### 二、Live 环境基础配置
```bash
cat /sys/firmware/efi/fw_platform_size   # 64→UEFI64，32→UEFI32，报错→BIOS 模式
timedatectl set-ntp true                 # 同步时钟（默认已启用）
```
无线用 `iwctl`：`device list` → `adapter phy0 set-property Powered on` → `device wlan0 set-property Powered on` → `station wlan0 scan/get-networks/connect`。

### 三、磁盘分区与格式化
`lsblk` / `fdisk -l` 查看布局 → `parted` 或 `cfdisk` 分区 → `mkfs.fat -F32` / `mkfs.ext4` → `mount`。详见 [[磁盘分区与挂载]]。

### 四、安装基础系统
```bash
pacstrap -K /mnt base linux linux-firmware
genfstab -U /mnt >> /mnt/etc/fstab
```
可在 Live 环境先把国内镜像（`mirrors.ustc.edu.cn`）移到 `/etc/pacman.d/mirrorlist` 顶部加速。`base` 是**唯一必需**的软件包组，其余按需附加（`base-devel`、`intel-ucode`/`amd-ucode`、`networkmanager`、`vim`/`nano`、`man-db` 等）。

### 五、chroot 后配置
```bash
arch-chroot /mnt
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && hwclock --systohc
locale-gen && echo 'LANG=en_US.UTF-8' > /etc/locale.conf
echo "myarch" > /etc/hostname
passwd
pacman -S networkmanager && systemctl enable NetworkManager   # 必须在 chroot 内完成
useradd -m -G wheel -s /bin/bash 用户名 && passwd 用户名      # 再用 visudo 开 sudo
```

### 六、引导程序
两方案见 [[UEFI双系统引导]]：GRUB（`grub-install` + `grub-mkconfig`）或 systemd-boot（`bootctl install` + 手写条目）。

### 七、完成并重启
```bash
exit
umount -R /mnt
reboot
```
重启后拔 U 盘，BIOS 中把 `Linux Boot Manager`（或 GRUB）设为第一启动项。

### 八、安装后配置
连接网络 → 桌面环境（`plasma-desktop` + `sddm`）→ 中文环境与输入法 → 镜像源与第三方仓库。

## 三个最容易踩空的点

1. **chroot 内必须装并启用网络管理器**，否则重启后新系统无法联网。
2. **`pacstrap` 报 PGP 签名错误**：Live 环境的 `archlinux-keyring` 过旧，先执行 `pacman -Sy archlinux-keyring`。
3. **不要重新格式化已有的 EFI 分区**，否则会破坏其他系统的引导。

## 问题表索引

原素材第十节用一张表汇总了 22 个问题及解决方法，逐条展开见 [[2026-10-14-ArchLinux安装问题排查]]。

## 参考资料（原素材列出）

- [Arch Linux Installation Guide](https://wiki.archlinux.org/title/Installation_guide)
- [iwd - ArchWiki](https://wiki.archlinux.org/title/Iwd)
- [systemd-boot - ArchWiki](https://wiki.archlinux.org/title/Systemd-boot)
- [Persistent block device naming](https://wiki.archlinux.org/title/Persistent_block_device_naming)

## 关联

- 概念：[[磁盘分区与挂载]]、[[UEFI双系统引导]]、[[持久化块设备命名]]、[[Linux中文显示与locale]]
- 实体：[[Arch Linux]]、[[pacman]]、[[systemd-boot]]、[[GRUB]]、[[iwd]]、[[fcitx5]]
- 同科素材：[[2026-10-14-ArchLinux安装问题排查]]

来源：[[raw/Linux/ArchLinux Installation Guide]]
