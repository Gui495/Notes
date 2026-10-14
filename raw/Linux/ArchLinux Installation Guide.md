根据 [Arch Linux 官方安装指南](https://wiki.archlinux.org/title/Installation_guide) 的流程，结合您提供的实际操作记录，现将完整的安装、配置与修复步骤按**时间顺序**重新整理。**所有操作命令与步骤独立呈现，问题和解决方案统一汇总至文末**，便于对照查阅。

---

# Arch Linux 完整安装与配置流程

## 📑 目录

- [[#一、安装前准备|一、安装前准备]]
- [[#二、Live 环境基础配置|二、Live 环境基础配置]]
- [[#三、磁盘分区与格式化|三、磁盘分区与格式化]]
- [[#四、安装基础系统|四、安装基础系统]]
- [[#五、进入新系统并配置|五、进入新系统并配置]]
- [[#六、安装引导程序|六、安装引导程序]]
- [[#七、完成安装并重启|七、完成安装并重启]]
- [[#八、安装后配置|八、安装后配置]]
- [[#九、参考资料|九、参考资料]]
- [[#十、问题汇总|十、问题汇总]]
## 一、安装前准备

### 1. 获取安装镜像
- 访问 [Arch Linux 下载页面](https://archlinux.org/download) 下载 ISO 镜像（如 `archlinux-YYYY.MM.DD-x86_64.iso`）及其 PGP 签名文件（`.sig`）。

### 2. 验证签名（推荐）
```bash
pacman-key -v archlinux-版本-x86_64.iso.sig
```
[[Installation_Problems#7. PGP签名损|PGP签名损]]

### 3. 制作启动 U 盘
- 使用 `dd`（Linux/macOS）或 Rufus/balenaEtcher（Windows）将 ISO 写入 U 盘。  
  **示例（Linux）**：
  ```bash
  sudo dd bs=4M if=./archlinux-YYYY.MM.DD-x86_64.iso of=/dev/sdX status=progress oflag=direct
  ```
  （`/dev/sdX` 替换为实际 U 盘设备，**注意是整个设备而非分区**）

### 4. 准备硬盘空间
- 在 Windows 中使用“磁盘管理”压缩分区，为 Arch 留出空闲空间。

### 5. 关闭 Secure Boot
- 进入 BIOS/UEFI 设置，将 **Secure Boot** 设为 **Disabled** 或 **Setup Mode**。

### 6. 启动 Live 环境
- 从 U 盘启动，选择 **Arch Linux install medium (x86_64, UEFI)**（若为 UEFI 模式）。

---

## 二、Live 环境基础配置

### 1. 启动选项说明
- **本次选择**：第一个标准模式。
- **其他选择**：[[Installation_Problems#1. 启动选项说明|参照]]
### 2. 验证启动模式
```bash
cat /sys/firmware/efi/fw_platform_size
```
- 返回 `64` → UEFI 64 位；返回 `32` → UEFI 32 位；报错 → BIOS 模式。

### 3. 连接网络
#### 有线网络
- 通常自动通过 DHCP 获取 IP，用 `ping archlinux.org` 测试。

#### 无线网络（使用 iwd）
```bash
iwctl
[iwd]# device list                     # 查看无线网卡名称，如 wlan0
[iwd]# adapter phy0 set-property Powered on   # 开启适配器（若需要）
[iwd]# device wlan0 set-property Powered on   # 开启设备
[iwd]# station wlan0 scan
[iwd]# station wlan0 get-networks
[iwd]# station wlan0 connect "SSID"    # 输入密码
[iwd]# exit
```

[[Installation_Problems#2. 网络连接问题|遇到网络连接问题请戳这]]

### 4. 同步系统时钟

```bash
# 时间同步服务 systemd-timesyncd 默认已启用，确认
timedatectl

# 未启用
timedatectl set-ntp true
timedatectl status
```

---

## 三、磁盘分区与格式化

### 1. 查看磁盘布局
```bash
lsblk          # 显示分区概览
fdisk -l       # 显示完整分区表（含未分配空间）
```
>看不到[[Installation_Problems#4. 看不到预留空间|请戳这]]

### 2. 创建分区（UEFI + GPT 示例）

- #### 2.1 分区方案（UEFI + 双系统）
| 分区       | 大小     | 类型    | 挂载点               |
| -------- | ------ | ----- | ----------------- |
| EFI 系统分区 | ≥260MB | FAT32 | `/boot`（或 `/efi`） |
| Root 分区  | ≥50GB  | ext4  | `/mnt`            |
| Home 分区  | 剩余空间   | ext4  | `/mnt/home`       |

可直接使用 Windows 已有的 EFI 分区。

- #### 2.2 创建分区（以 /dev/nvme0n1 为例）

```bash
# parted /dev/nvme0n1
(parted) print
(parted) mkpart primary fat32 起始 结束
(parted) set 分区号 esp on
(parted) mkpart primary ext4 起始 结束
(parted) mkpart primary ext4 起始 结束
(parted) quit
```
- 或使用 [[Installation_Problems#5. cfdisk操作步骤|cfdisk]]（或 `gdisk`）操作磁盘（如 `/dev/nvme0n1`）：

### 3. 格式化分区
```bash
mkfs.fat -F32 /dev/nvme0n1pX      # EFI 分区（X 为实际编号）
mkfs.ext4 /dev/nvme0n1pY          # 根分区
mkfs.ext4 /dev/nvme0n1pZ          # Home 分区（若有）
```
- **Swap（交换空间）** 
	相当于虚拟内存——当物理内存不足时，系统将暂时不用的数据移到硬盘上腾出空间。
	同时，Swap 也是**系统休眠（挂起至磁盘）** 的必需品。
	若不创建 Swap 分区，可后续使用 Swap 文件。

> **警告**：如果磁盘上已有 EFI 分区，**不要重新格式化**，否则会破坏其他系统的引导。本次是在新磁盘上创建全新的 EFI 分区，所以可以格式化。

### 4. 挂载分区
```bash
mount /dev/nvme0n1pY /mnt                     # 挂载根分区
mount --mkdir /dev/nvme0n1pX /mnt/boot       # 挂载 EFI 分区到 /boot
mount --mkdir /dev/nvme0n1pZ /mnt/home       # 挂载 Home（若有）
```
- [[Installation_Problems#6. mkdir参数|mkdir参数]]

---

## 四、安装基础系统

### 1. 优化镜像源（可选）
- 编辑 `/etc/pacman.d/mirrorlist`，将国内镜像（如 `mirrors.ustc.edu.cn`）移至顶部。

### 2. 执行安装
```bash
pacstrap -K /mnt base linux linux-firmware
```
- 可附加安装常用包：`base-devel`、`intel-ucode`/`amd-ucode`、`networkmanager`、`vim`/`nano` 等。
    *   `base`：**唯一必需的**软件包组。
    *   `linux`：Linux内核。
    *   `linux-firmware`：通用硬件固件。
    *   **CPU微码**：`intel-ucode` 或 `amd-ucode`。
    *   **特定固件**：如 `sof-firmware` (音频)。
    *   **文件系统工具**：如 `btrfs-progs`、`xfsprogs` 等。
    *   **网络管理器**：如 `networkmanager`。
    *   **文本编辑器**：如 `nano` 或 `vim`。
    *   **文档工具**：如 `man-db`、`man-pages`。

### 3. 生成 fstab
```bash
genfstab -U /mnt >> /mnt/etc/fstab
cat /mnt/etc/fstab   # 检查是否正确
```

---

## 五、进入新系统并配置

### 1. Chroot
```bash
arch-chroot /mnt
```

### 2. 设置时区
```bash
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
hwclock --systohc
```

### 3. 本地化设置
- 编辑 `/etc/locale.gen`，取消注释 `en_US.UTF-8 UTF-8`（以及 `zh_CN.UTF-8 UTF-8` 可选）：
```bash
locale-gen
echo 'LANG=en_US.UTF-8' > /etc/locale.conf
```

### 4. 设置主机名
```bash
echo "myarch" > /etc/hostname
```

### 5. 设置 root 密码
```bash
passwd
```
- [[Installation_Problems#9. 修复 root 账户锁定|root 账户锁定]]

### 6. 安装网络管理工具（重要）
```bash
pacman -S networkmanager   
systemctl enable NetworkManager

# 或 iwd + dhcpcd
# pacman -S iwd dhcpcd
# systemctl enable iwd
# systemctl enable dhcpcd
```
>**重要**：这一步必须在 chroot 中完成，否则重启后新系统将无法联网。

### 7. 创建普通用户(可选|建议)
```bash
useradd -m -G wheel -s /bin/bash 用户名
passwd 用户名
```
- 后续通过 `visudo` 启用 `wheel` 组的 sudo 权限。

---

## 六、安装引导程序

### 方案一：GRUB（UEFI）
```bash
pacman -S grub efibootmgr
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg
```
- 如需检测 Windows，安装 `os-prober`，并编辑 `/etc/default/grub` 取消注释 `GRUB_DISABLE_OS_PROBER=false`，再重新生成配置。
```bash
pacman -S os-prober
# 编辑 /etc/default/grub，取消注释 GRUB_DISABLE_OS_PROBER=false
grub-mkconfig -o /boot/grub/grub.cfg
```

### 方案二：systemd-boot（UEFI，更轻量）
```bash
bootctl install
```
- 创建 `/boot/loader/loader.conf`，设置 `default arch.conf` 和 `timeout 5`。
- 创建 `/boot/loader/entries/arch.conf`：
  ```
  title   Arch Linux
  linux   /vmlinuz-linux
  initrd  /initramfs-linux.img
  options root=UUID=你的根分区UUID rw
  ```
  （UUID 用 `blkid /dev/根分区` 获取）
- 若多系统，手动添加 Windows 条目 `/boot/loader/entries/windows.conf`：
  ```
  title Windows 11
  efi /EFI/Microsoft/Boot/bootmgfw.efi
  ```
- 可选禁用自动扫描：在 `loader.conf` 加 `auto-entries 0`。

---

## 七、完成安装并重启

```bash
exit
umount -R /mnt
reboot
```
- 重启后拔掉 U 盘，进入 BIOS 将 `Linux Boot Manager`（或 GRUB）设为第一启动项。
- [[Installation_Problems#8. 重启后无法进入系统|找不到设备]]
- [[Installation_Problems#10. 双系统启动efi配置|双系统启动efi配置]]

---

## 八、安装后配置（图形界面、中文环境、软件源等）

[配置教程可供参考](https://wiki.archlinux.org/title/General_recommendations)
### 1. 连接网络（若使用 NetworkManager/iwd）
```bash
nmcli device wifi list
nmcli device wifi connect "SSID" password "密码"

$ iwctl
[iwd]# device list
[iwd]# station wlan0 scan
[iwd]# station wlan0 connect 热点名称
# 输入密码
$ ping 8.8.8.8  # 验证
```
>南邮这种**校园网**需要登陆，[[Installation_Problems#3. 南邮校园网连接(脚本)|请戳这]]

### 2. 安装桌面环境（以 KDE Plasma 为例）(可选)
```bash
sudo pacman -S plasma-desktop sddm
sudo systemctl enable sddm
```

### 3. 配置中文环境（TTY 英文 + 图形中文）
- **保持系统 LANG 为英文**（[[Installation_Problems#11. 命令行出现方块字|避免 TTY 方块]]）：
  ```bash
  sudo vim /etc/locale.conf
  LANG=en_US.UTF-8
  ```
- **图形界面覆盖为中文**：在 `~/.xprofile`（或 `~/.xinitrc`）中：
  ```bash
  export LANG=zh_CN.UTF-8
  export LC_CTYPE=zh_CN.UTF-8
  ```
- **安装中文字体**：
  ```bash
  sudo pacman -S noto-fonts-cjk
  ```

### 4. 安装中文输入法（fcitx5）
```bash
sudo pacman -S fcitx5 fcitx5-chinese-addons fcitx5-qt fcitx5-gtk
```
- 在 `~/.xprofile` 添加：
  ```bash
  export GTK_IM_MODULE=fcitx
  export QT_IM_MODULE=fcitx
  export XMODIFIERS=@im=fcitx
  ```

### 5. 配置官方镜像源（加速 [[Installation_Problems#12. pacman镜像理解设置|pacman]]）
- 编辑 `/etc/pacman.d/mirrorlist`，将中国镜像源置顶：
  ```
  Server = https://mirrors.ustc.edu.cn/archlinux/$repo/os/$arch
  Server = https://mirrors.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch
  ```
- 可使用 `reflector` 自动排序：
  ```bash
  sudo pacman -S reflector
  sudo reflector -c China --sort rate --save /etc/pacman.d/mirrorlist
  ```

### 6. 添加 archlinuxcn 仓库（中文社区软件）
- 在 `/etc/pacman.conf` 末尾添加：
  ```ini
  [archlinuxcn]
  Server = https://mirrors.ustc.edu.cn/archlinuxcn/$arch
  ```
- 导入 GPG 密钥：
  ```bash
  sudo pacman -Syu archlinuxcn-keyring
  # 若报错，手动签署密钥：
  sudo pacman-key --lsign-key "farseerfc@archlinux.org"
  ```
- 现在可安装 `yay` 等 AUR 助手：
  ```bash
  sudo pacman -S yay
  ```

## 九、参考资料

- [Arch Linux Installation Guide (英文)](https://wiki.archlinux.org/title/Installation_guide)
- [Arch Linux 安装指南 (简体中文)](https://wiki.archlinux.org.cn/)
- [iwd - ArchWiki](https://wiki.archlinux.org/title/Iwd)
- [UEFI 双系统快速安装指南](https://wiki.archlinux.org/title/User:Soloturn/Quick_Installation_guide_UEFI)
- [General recommendations(系统配置)](https://wiki.archlinux.org/title/General_recommendations)
- [Persistent block device naming](https://wiki.archlinux.org/title/Persistent_block_device_naming)
- [systemd-boot](https://wiki.archlinux.org/title/Systemd-boot)

## 十、问题汇总

以下汇总了您在安装过程中遇到的所有问题、原因与解决方法，按出现顺序排列。

|**问题现象**|**根本原因**|**解决方法**|
|---|---|---|
|**1. Live 环境无线连接失败**：`device set-property Powered on` 报错|命令中 `name` 是占位符，未替换为实际设备名；且适配器未开启|先用 `device list` 查看设备名，再执行 `adapter phy0 set-property Powered on` 和 `device wlan0 set-property Powered on`|
|**2. `lsblk` 看不到 Windows 预留的 100GB 空闲空间**|`lsblk` 默认不显示未分配区域|使用 `fdisk -l` 查看完整分区表，找到空闲扇区范围|
|**3. 不确定应操作哪个磁盘**|多块硬盘易混淆|用 `fdisk -l` 根据容量和型号仔细辨认，避免误操作 Windows 系统盘|
|**4. 分区后新分区编号不确定**|分区后编号可能变化|用 `lsblk` 或 `fdisk -l` 再次确认，再格式化|
|**5. `mkdir -p` 参数含义**|不了解 `-p` 作用|`-p` 自动创建父目录，目录存在时不报错|
|**6. 是否必须创建 Swap 分区**|不清楚 Swap 作用|物理内存 ≥8GB 时可暂不建分区，后续用 Swap 文件代替（更灵活）|
|**7. `pacstrap` 报 PGP 签名错误**|Live 环境中 `archlinux-keyring` 过旧|在 `pacstrap` 前执行 `pacman -Sy archlinux-keyring` 更新密钥环|
|**8. 更换国内镜像后仍下载慢或报错**|镜像源列表未更新，或未初始化密钥|手动将中国源移顶；若换源后报错，执行 `pacman-key --init` 和 `pacman-key --populate archlinux`|
|**9. Chroot 中 `bootctl install` 无法创建 UEFI 启动项**|`arch-chroot` 阻止修改 UEFI 变量|退出 chroot，在 Live 环境中执行 `bootctl --esp-path=/mnt/boot install`|
|**10. 重启后只有 "Reboot Into Firmware Interface"**|systemd-boot 缺少 `arch.conf` 配置文件，或内核/initramfs 不在 ESP 中|确认 ESP 挂载在 `/boot`，创建 `/boot/loader/entries/arch.conf`，并确保 `root=` 参数使用 UUID|
|**11. 重启后直接进入 Windows，无引导菜单**|`loader.conf` 未设置默认项，或 Windows 条目配置错误|设置 `default arch.conf` 和 `timeout`；检查 `windows.conf` 中 `efi` 路径拼写（如 `bootmgfw.efi`）|
|**12. systemd-boot 菜单出现两个 Windows**|自动扫描到其他 ESP 中的引导文件|在 `loader.conf` 添加 `auto-entries 0` 禁用自动扫描|
|**13. 紧急模式：`Timed out waiting for device /dev/gpt-auto-root`**|NVMe 设备名（`nvme0n1` ↔ `nvme1n1`）发生改变|在 `/etc/fstab` 和引导配置中将设备名全部替换为 **UUID**（用 `blkid` 获取）|
|**14. 紧急模式下 root 账户被锁定**|initramfs 因挂载失败锁定了 root|在 Live 环境中 chroot 后执行 `passwd root` 重置密码|
|**15. `/etc/fstab` 格式错误（挂载点、类型、选项错误）**|手动编辑时误写|按规范重写，根分区挂载到 `/`，文件系统为 `ext4`，选项 `rw,relatime`|
|**16. `arch.conf` 中路径拼写错误（`/unlinux-linux`）**|误将 `vmlinuz` 写成 `unlinux`|修正为 `/vmlinuz-linux`，并确保 `options` 为复数形式|
|**17. 重启后无网络（`iwctl: command not found`）**|新系统未安装 `iwd`，只装了 NetworkManager|使用 `nmcli` 连接 Wi-Fi，或 chroot 补装 `iwd`|
|**18. TTY 下中文显示为方块**|内核帧缓冲字体不支持中文|系统全局 LANG 设为 `en_US.UTF-8`，图形界面单独覆盖为 `zh_CN.UTF-8`|
|**19. 安装中文字体后图形界面仍乱码**|未正确设置 locale 或未重启会话|在 `~/.xprofile` 中导出 `LANG` 和 `LC_CTYPE`，并确保字体已安装（`fc-list :lang=zh` 检查）|
|**20. `pacman-mirrors: command not found`**|该工具为 Manjaro 专有，Arch 无此命令|手动编辑 `mirrorlist`，或安装 `reflector` 自动化排序|
|**21. `archlinuxcn` 仓库报 "invalid signature"**|未导入该仓库维护者的 GPG 公钥|安装 `archlinuxcn-keyring`；若失败，执行 `pacman-key --lsign-key "farseerfc@archlinux.org"`|
|**22. 使用 `SigLevel = never` 跳过验证是否安全**|有人推荐此“偷懒”做法|**强烈不推荐**，应继承全局 `SigLevel` 或显式设置 `Required DatabaseOptional`，并正确导入密钥|

> **核心原则**：始终使用 **UUID** 代替设备名（如 `/dev/nvme0n1pX`）在 `/etc/fstab` 和引导配置中，这是避免设备名变化导致启动失败的最佳实践。
