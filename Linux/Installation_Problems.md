## 1. 启动选项说明

从 U 盘启动后，会出现以下选项：

| 启动选项 | 含义 |
|---------|------|
| **Arch Linux install medium (x86_64, UEFI)** | 标准安装模式，绝大多数人选择此项 |
| **... with speech** | 语音辅助模式，为视障人士设计 |
| **... Copy to RAM** | 将整个 ISO 加载到内存运行，可拔掉 U 盘，但需 ≥4GB 内存 |
| **EFI Shell** | UEFI 交互式命令行环境，用于维护引导项 |
| **Reboot Info Firmware Interface** | 重启进入 BIOS/UEFI 设置界面 |

---

## 2. 网络连接问题

- **遇到的问题**：
Arch Live 环境使用 `iwd`（iNet Wireless Daemon）管理无线网络。执行以下命令时失败：
```bash
[iwd]# device name set-property Powered on
```
**报错原因**：命令中的 `name` 是**占位符**，需替换为实际设备名；且需先开启适配器（adapter）而非直接操作设备。

- **解决方法**：
**步骤一：查看设备列表**
```bash
[iwd]# device list
```
记下无线设备名，如 `wlan0`。

**步骤二：开启适配器**
```bash
[iwd]# adapter phy0 set-property Powered on
```

**步骤三：开启设备**
```bash
[iwd]# device wlan0 set-property Powered on
```

**步骤四：扫描并连接 Wi-Fi**
```bash
[iwd]# station wlan0 scan
[iwd]# station wlan0 get-networks
[iwd]# station wlan0 connect 热点名称
```

**关于校园网**：校园网为开放网络（无密码），但需 Web 认证。本方案先使用手机热点完成系统安装，后续再配置校园网自动登录脚本，以避开 Live 环境下的认证难题。

### 2.3 验证网络
```bash
# ping 测试
ping 8.8.8.8
```
连通后按 `Ctrl+C` 停止。

---

## 3. 南邮校园网连接(脚本)
- 下载编译登录脚本：
  ```bash
  git clone https://github.com/lux-QAQ/NJUPT_Fastlogin.git
  cd NJUPT_Fastlogin
  vim version2_ASMcross.cpp   # 修改学号和密码
  g++ -O3 -no-pie -flto -s version2_ASMcross.cpp -o njupt_login
  ./njupt_login   # 测试
  ```
- 设置开机自动执行（systemd 服务或 crontab `@reboot`）。

---

## 4. 看不到预留空间

**现象**：在 Windows 中从 365GB 分区划出了 100GB 未分配空间，但进入 Arch 后执行 `lsblk` 看不到这 100GB。

**原因**：`lsblk` **默认只显示已分配的分区**，未分配的空白区域不会显示。需要用 `fdisk -l` 查看完整的分区表信息。

**解决方法**：

```bash
fdisk -l
```

查看输出，重点关注磁盘 `/dev/nvme0n1`（用户实际设备名为 `/dev/mem0n1`，即 Samsung SSD 980 1TB）：

```
Device          Start        End        Sectors    Size Type
/dev/mem0n1p2   32768       767062015  767029248  365.7G Microsoft basic data
/dev/mem0n1p3   976777216   1953521663 976744448  465.7G Microsoft basic data
```

**p2 结束于 767062015，p3 开始于 976777216**，中间这 209,715,200 个扇区（约 100GB）就是 Windows 中划分出来的未分配空间。

---

## 5. cfdisk操作步骤

**操作步骤**：
1. 在分区列表中，找到 p2 和 p3 之间的 **“Free space”**（100GB）
2. 选中该空闲区域，选择 `[ New ]`，输入 `512M`，回车
3. 选择 `[ Type ]`，改为 **EFI System**
4. 再次选中剩余的空闲区域（约 99.5GB），选择 `[ New ]`，直接回车使用全部空间
5. 类型保持默认的 **Linux filesystem**
6. 选择 `[ Write ]`，输入 `yes` 确认，然后 `[ Quit ]` 退出

> **特别注意**：本次操作中，磁盘 `/dev/mem1`（476GB）是 Windows 系统盘，**千万不能动**。要操作的是 `/dev/mem0n1`（1TB Samsung SSD）。

---

## 6. mkdir参数

**mkdir -p 是什么参数：**
创建 `/mnt/boot` 目录用于挂载 EFI 分区：
```bash
mkdir -p /mnt/boot
```
**`-p` 参数的作用**：
- **自动创建父目录**：如果 `/mnt` 不存在，会先创建 `/mnt`，再创建 `/mnt/boot`
- **目录已存在时不报错**：如果目录已存在，直接跳过，不会中断脚本

**其他常用参数：**

| 参数 | 作用 |
|------|------|
| `-v` | 显示创建过程（打印日志） |
| `-m` | 创建时直接指定权限（如 `mkdir -m 700 secret`） |

---

## 7. PGP签名损

*   **现象**：在执行 `pacstrap` 安装 `base` 等软件包时，出现类似 “corrupted package (PGP signature)” 的错误。
*   **原因**：Live CD中的 `archlinux-keyring` 软件包过旧，无法验证最新软件包的签名。
*   **解决方法**：
    1.  在运行 `pacstrap` **之前**，先在Live环境中更新密钥环：
        `pacman -Sy archlinux-keyring`
    2.  更新完成后，再重新执行 `pacstrap` 命令。

---

## 8. 重启后无法进入系统

- 情况一
	*   **现象**：系统重启后找不到启动设备。
	*   **原因**：引导程序未正确安装或EFI分区未正确挂载。
	*   **解决方法**：
	    1.  **重新进入Live环境**，并挂载根分区和EFI分区到 `/mnt`。
	    2.  **检查挂载点**：使用 `lsblk` 确认EFI分区是否挂载到了 `/mnt/boot` 或 `/mnt/efi`。
	    3.  **重新安装引导程序**：`arch-chroot` 进入 `/mnt`，然后根据你的引导程序（如GRUB）的官方文档，重新执行安装和配置命令。

- 情况二
系统在启动时进入 **emergency mode（紧急模式）**，并提示：
```
Timed out waiting for device /dev/gpt-auto-root.
Cannot open access to console, the root account is locked.
```
根本原因：**NVMe 设备名在两次启动间发生变化**（`nvme0n1p5` → `nvme1n1p5`），导致系统无法通过设备名找到根分区。Linux 中设备名（如 `/dev/nvme0n1`、`/dev/nvme1n1`）的分配顺序是任意的，多次启动后可能互换。

**解决方法：** 获取分区 UUID（对应官方指南“持久化块设备命名”章节）
**操作步骤：** 使用 `blkid` 命令获取分区的 **UUID（通用唯一标识符）** ：
```bash
[root@archiso ~]# blkid /dev/nvme1n1p5
/dev/nvme1n1p5: UUID="91f157ae-71eb-4297-aae6-80e3cab36805" BLOCK_SIZE="4096" TYPE="ext4" PARTUUID="63a07efd-62da-4019-ac95-ccef3204839"
```
**知识点：** 使用 UUID 而非设备名（如 `/dev/nvme0n1p5`）可以避免设备名变化导致的挂载失败。

---

## 9. 修复 root 账户锁定

**遇到的问题：**
系统进入紧急模式时提示 `Cannot open access to console, the root account is locked`——这是系统为安全考虑在 initramfs 中锁定了 root 账户。

**解决方法：**
在 chroot 环境中重置 root 密码：
```bash
[root@archiso ~]# passwd root
New password:
Retype new password:
passwd: password updated successfully
```

---

## 10. 双系统启动efi配置

- **1.装引导程序并遇到第一个问题**

**操作**：在 chroot 环境中，使用 `bootctl install` 命令安装 systemd-boot 作为引导器。安装完成后，按照常规流程创建了 Arch 的启动条目 `arch.conf` 和 Windows 的启动条目 `windows.conf`。

**遇到的问题**：重启后，系统直接进入了 Windows，BIOS 中明明已将 “Linux Boot Manager” 设为第一启动项，但并未出现 systemd-boot 的菜单。

**原因分析**：
1.  `loader.conf` 配置文件几乎为空，只包含注释行，没有设置默认启动项。
2.  systemd-boot 在没有明确默认项时，会尝试加载它找到的第一个有效条目。由于某种原因（例如 NVRAM 记忆了上一次的启动项），它可能默认选择了 Windows。
3.  此外，`windows.conf` 文件中存在拼写错误（`bottmgfu.efi` 应为 `bootmgfw.efi`），导致该条目本身是无效的。

**解决方法**：
1.  **使用 Live USB 启动并 chroot**：由于系统无法正常进入，需要使用 Arch Linux 的 Live USB 启动，挂载根分区和 EFI 分区后，通过 `arch-chroot` 进入已安装的系统环境进行修复。
2.  **创建/编辑 `loader.conf`**：在 chroot 环境中，编辑 `/boot/loader/loader.conf` 文件，明确指定默认启动项并设置菜单超时时间：
    ```conf
    default arch.conf
    timeout 5
    ```
3.  **检查并修正 `arch.conf`**：使用 `ls /boot/loader/entries/` 确认 `arch.conf` 文件存在。如果不存在，则需要手动创建，并确保 `options root=...` 一行中的 `PARTUUID` 与你 Arch 根分区的实际值一致。
4.  **修正 `windows.conf`**：将 `windows.conf` 中的 `bottmgfu.efi` 更正为 `bootmgfw.efi`。

- **2.解决“两个 Windows”问题**

**操作**：修复并重启后，systemd-boot 菜单成功出现，但显示了两个 Windows 条目：
*   `Windows 11 (windows.conf)`：你手动创建的。
*   `Windows 11 (auto-windows)`：systemd-boot 自动扫描到的。

**原因分析**：systemd-boot 默认会扫描所有 EFI 系统分区（ESP）中的可引导条目。由于你的 Arch 和 Windows 分别安装在不同的硬盘上，各自拥有独立的 ESP，因此 systemd-boot 既找到了你手动创建的条目，也自动发现了 Windows 自己 ESP 中的引导文件。

**解决方法**：
编辑 `/boot/loader/loader.conf` 文件，添加一行配置以禁用自动条目扫描功能：
```conf
auto-entries 0
```
保存文件并执行 `sudo bootctl update` 刷新配置后重启，菜单中便只保留了你手动定义的条目。

---

## 11. 命令行出现方块字

>命令行出现方块字（核心问题）

**现象**：在 tty（纯命令行界面）下执行某些命令时，中文显示为方块。
**原因**：Arch Linux 的 tty 默认使用内核帧缓冲字体，**不支持中文显示**。之前推荐的 `noto-fonts-cjk` 是图形界面（X11/Wayland）专用字体，在 tty 下无效。
**解决方案**：分两步走 —— 命令行永久使用英文，图形界面单独使用中文。

-  **第 1 步：系统全局锁定英文（TTY 无方块）**

编辑 `/etc/locale.conf`：
```bash
sudo vim /etc/locale.conf
```
写入：
```
LANG=en_US.UTF-8
```
保存后退出，**重启生效**。此后所有 TTY 命令输出均为英文，彻底告别方块字。

- **第 2 步：图形界面覆盖为中文**

**方式一：使用显示管理器（SDDM/GDM/LightDM）**

创建或编辑 `~/.xprofile` 文件：
```bash
vim ~/.xprofile
```
写入：
```
export LANG=zh_CN.UTF-8
export LC_CTYPE=zh_CN.UTF-8
```
该文件在用户登录桌面**之前**自动加载，进入后桌面环境即为中文。

**方式二：使用 `startx` 手动启动**

编辑 `~/.xinitrc`，在启动桌面环境的命令（如 `exec kde-plasma-desktop`）**之前**添加：
```
export LANG=zh_CN.UTF-8
export LC_CTYPE=zh_CN.UTF-8
```

* **第 3 步：安装图形界面中文字体**

进入图形界面后，安装中文字体：
```bash
sudo pacman -S noto-fonts-cjk
```

- **第 4 步：（可选）配置中文输入法**

安装 fcitx5：
```bash
sudo pacman -S fcitx5 fcitx5-chinese-addons fcitx5-qt fcitx5-gtk
```
在 `~/.xprofile` 中追加：
```
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS=@im=fcitx
```

- **验证配置**

重启后：
- **在 TTY 下**：执行 `locale`，应显示 `LANG=en_US.UTF-8`
- **在图形界面终端**：执行 `locale`，应显示 `LANG=zh_CN.UTF-8`
- **检查中文字体**：`fc-list :lang=zh` 应列出已安装的中文字体

---

## 12. pacman镜像理解设置

- 📖 **第一步：理解核心概念**

Arch Linux 的软件包管理由 `pacman` 负责，其配置主要涉及两个文件：
*   **`/etc/pacman.d/mirrorlist`**：这是**官方仓库的“服务器地址列表”**。Pacman 会按此文件中的顺序依次尝试连接，所以将最快的镜像（如国内的）放在最前面可以显著提升下载速度。
*   **`/etc/pacman.conf`**：这是 Pacman 的**主配置文件**。它定义了所有仓库（包括官方和第三方）以及 pacman 的全局行为。官方仓库通过 `Include = /etc/pacman.d/mirrorlist` 来引用镜像列表，而第三方仓库则直接在此文件中定义。

- 🚀 **第二步：配置官方仓库镜像源 (`mirrorlist`)**

这一步的目标是让 Pacman 从国内高速镜像下载官方软件包（如 Linux 内核）。

- **操作方法：**
手动编辑 `/etc/pacman.d/mirrorlist` 文件，将你选择的国内镜像源地址剪切到文件的最顶端。
```bash
sudo vim /etc/pacman.d/mirrorlist
```
例如，将中科大或清华的镜像源置于文件顶部：
```
## China
Server = https://mirrors.ustc.edu.cn/archlinux/$repo/os/$arch
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch
```
> **官方指南参考**：Arch Wiki 明确指出，Pacman 以 `mirrorlist` 中 `Server` 的顺序作为优先级，国内用户推荐将国内镜像放在最顶端。

- **遇到的问题与解决**
	*   **问题**：我尝试运行 `pacman-mirrors -c China`，但系统提示 `command not found`。
	*   **原因**：`pacman-mirrors` 是 **Manjaro** 系统的专属工具，在纯正的 Arch Linux 中并不存在。
	*   **解决方法**：Arch Linux 官方推荐手动编辑 `mirrorlist` 文件，这完全正确且有效。

如果你希望有自动化的工具，可以安装 `reflector`。它能自动获取最新的镜像列表并按速度排序。
```bash
sudo pacman -S reflector
# 例如：自动检测中国镜像并按速度排序，直接覆盖 mirrorlist
sudo reflector -c China --sort rate --save /etc/pacman.d/mirrorlist
```
> **官方指南参考**：Reflector 是一个 Python 脚本，可以从 Arch Linux 镜像状态页面检索最新的镜像列表，按速度排序并覆盖 `/etc/pacman.d/mirrorlist` 文件。

- 📦 **第三步：添加第三方仓库 (`pacman.conf`)**

这一步的目标是添加 `archlinuxcn`（Arch Linux 中文社区仓库），以安装官方仓库中没有的软件（如 `yay`、`google-chrome` 等）。

- **操作方法**：
编辑 `/etc/pacman.conf` 文件，在**文件的最末尾**添加以下内容：
```ini
[archlinuxcn]
Server = https://mirrors.ustc.edu.cn/archlinuxcn/$arch
```
> **官方指南参考**：Arch Linux CN 仓库的官方 Wiki 明确指示，在 `/etc/pacman.conf` 文件末尾添加以上两行。

**关键提醒**：
*   不要将 `archlinuxcn` 的地址写到 `/etc/pacman.d/mirrorlist` 里，这会导致格式错误。
*   不要将第三方仓库写在官方仓库前面，以免系统核心组件被错误替换。

- **遇到的问题与解决**
	*   **问题**：我看到有的博主在配置中添加了 `SigLevel = never`。
	*   **原因**：这是为了跳过 GPG 签名验证，从而避免因未导入密钥而报错，是一种“偷懒”的做法。
	*   **解决方法**：**强烈不推荐使用 `never`**。这会带来严重的安全风险。正确做法是不添加这一行，让该仓库继承 `pacman.conf` 顶部 `[options]` 中设置的全局 `SigLevel`（通常是 `Required DatabaseOptional`）。

- 🔑 **第四步：导入 GPG 密钥**

为了安全地使用 `archlinuxcn` 仓库，必须导入其维护者的 GPG 公钥。

- **操作方法**：
1.  **刷新 pacman 数据库并导入密钥**：
    ```bash
    sudo pacman -Syu archlinuxcn-keyring
    ```
    > **官方指南参考**：安装 `archlinuxcn-keyring` 包以导入 GPG 密钥。

2.  **（可选）如果报错，手动信任密钥**：
    如果上述命令报错，可以尝试手动信任维护者的密钥：
    ```bash
    sudo pacman-key --lsign-key "farseerfc@archlinux.org"
    ```
    然后再重新安装 `archlinuxcn-keyring`。

- **遇到的问题与解决**
	*   **问题**：安装 `archlinuxcn-keyring` 时可能遇到 “invalid signature” 或 “unknown trust” 错误。
	*   **原因**：系统的本地密钥环尚未信任 `archlinuxcn` 维护者的签名密钥。
	*   **解决方法**：执行上述 `pacman-key --lsign-key` 命令来本地签署并信任该密钥。

- ✅ **最终验证**

完成以上所有步骤后，运行以下命令更新系统并验证配置是否成功：
```bash
sudo pacman -Syyu
```
如果没有报错，说明你的镜像源和仓库配置已全部完成。现在你可以尝试从 `archlinuxcn` 仓库安装软件了，例如：
```bash
sudo pacman -S yay
```