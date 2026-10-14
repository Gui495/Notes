---
type: summary
status: budding
created: 2026-10-14
updated: 2026-10-14
sources:
  - raw/Linux/Installation_Problems.md
tags:
  - linux
  - archlinux
  - troubleshooting
---

对 [[raw/Linux/Installation_Problems]] 的提炼：安装过程中实际遇到的 12 个问题的现象、原因与解决方法，可直接当排错手册用。

## 问题速查

| # | 现象 | 原因 | 解决 |
|---|---|---|---|
| 1 | 认不清启动选项 | —— | 见下方「启动选项」 |
| 2 | `device name set-property Powered on` 报错 | `name` 是占位符未替换；且未先开适配器 | 先 `device list` 查名，再 `adapter phy0 set-property Powered on`、`device wlan0 set-property Powered on` |
| 3 | 校园网无法认证 | 校园网是开放网络但需 Web 认证，Live 环境难处理 | 先用手机热点装完系统，后续用登录脚本 |
| 4 | `lsblk` 看不到 Windows 预留的 100GB | `lsblk` 不显示未分配区域 | 用 `fdisk -l` 看完整分区表（p2 结束 767062015，p3 开始 976777216，中间约 100GB） |
| 5 | 不确定怎么建分区 | —— | 用 `cfdisk` 在 Free space 建 512M 并改 Type 为 EFI System，余下全给 Linux filesystem |
| 6 | 不懂 `mkdir -p` | —— | `-p` 自动创建父目录且目录已存在不报错；另有 `-v`、`-m` |
| 7 | `pacstrap` 报 PGP 签名错误 | Live 中 `archlinux-keyring` 过旧 | 先 `pacman -Sy archlinux-keyring` |
| 8 | 重启后无法进系统 | 引导未装好 / ESP 未挂对；或设备名变化 | 重装引导；或改用 UUID，见 [[持久化块设备命名]] |
| 9 | 紧急模式下 root 被锁 | initramfs 因挂载失败锁定 root | Live 中 chroot 后 `passwd root` 重置 |
| 10 | 直接进 Windows / 两个 Windows | `loader.conf` 无默认项、条目拼写错误、自动扫描 | 见下方「双系统引导」 |
| 11 | TTY 中文显示方块 | 内核帧缓冲字体不支持中文 | 全局锁定英文，图形界面单独覆盖，见 [[Linux中文显示与locale]] |
| 12 | 换源后仍慢或报错 | 不理解 mirrorlist 与 pacman.conf 的分工 | 见下方「pacman 源与仓库」 |

## 启动选项（问题 1）

| 启动选项 | 含义 |
|---|---|
| Arch Linux install medium (x86_64, UEFI) | 标准安装模式，绝大多数人选它 |
| ... with speech | 语音辅助，为视障人士设计 |
| ... Copy to RAM | 整个 ISO 载入内存运行，可拔 U 盘，需 ≥4GB 内存 |
| EFI Shell | UEFI 交互式命令行，用于维护引导项 |
| Reboot Into Firmware Interface | 重启进入 BIOS/UEFI 设置 |

## 双系统引导（问题 10）

- **直接进 Windows**：`loader.conf` 几乎为空、未设默认项；且 `windows.conf` 把 `bootmgfw.efi` 拼成了 `bottmgfu.efi`，条目本身无效。修法：写好 `default arch.conf`、`timeout 5`，确认 `arch.conf` 的 `options root=` 用 PARTUUID/UUID，并改正拼写。
- **菜单出现两个 Windows**：systemd-boot 默认扫描所有 ESP，Arch 与 Windows 各在独立硬盘、各有 ESP 时就会重复。修法：`loader.conf` 加 `auto-entries 0`，再 `bootctl update`。
- 系统进不去时只能靠 Live U 盘启动、挂载后 `arch-chroot` 进去修。

## pacman 源与仓库（问题 12）

| 文件 | 作用 |
|---|---|
| `/etc/pacman.d/mirrorlist` | 官方仓库的服务器地址列表，**按顺序尝试**，国内源置顶提速 |
| `/etc/pacman.conf` | 主配置文件，定义所有仓库与全局行为；官方仓库通过 `Include` 引用 mirrorlist，第三方仓库直接在此定义 |

- `pacman-mirrors` 是 **Manjaro 专有**工具，Arch 中不存在；Arch 用手改 mirrorlist 或装 `reflector` 自动排序。
- 添加 `archlinuxcn`：在 `pacman.conf` **末尾**追加 `[archlinuxcn]` 段（**不要**写进 mirrorlist，也不要放在官方仓库之前），再 `pacman -Syu archlinuxcn-keyring` 导入密钥；报签名错误时 `pacman-key --lsign-key "farseerfc@archlinux.org"`。
- ⚠️ **不要用 `SigLevel = never`** 跳过签名验证——那是「偷懒」做法，会带来安全风险；应继承全局 `SigLevel`。

## 校园网脚本（问题 3）

```bash
git clone https://github.com/lux-QAQ/NJUPT_Fastlogin.git
cd NJUPT_Fastlogin
vim version2_ASMcross.cpp   # 改学号和密码
g++ -O3 -no-pie -flto -s version2_ASMcross.cpp -o njupt_login
./njupt_login
```
详见 [[NJUPT_Fastlogin]]。

## 素材完整性问题

同一素材内部的设备名写法不一致（§4/§5 用 `/dev/mem0n1`、`/dev/mem1`，§8 用 `/dev/nvme*`），已在 [[持久化块设备命名]] 中标注为矛盾待裁决。

## 关联

- 概念：[[持久化块设备命名]]、[[UEFI双系统引导]]、[[Linux中文显示与locale]]、[[磁盘分区与挂载]]
- 实体：[[pacman]]、[[iwd]]、[[systemd-boot]]、[[NJUPT_Fastlogin]]
- 同科素材：[[2026-10-14-ArchLinux安装流程]]

来源：[[raw/Linux/Installation_Problems]]
