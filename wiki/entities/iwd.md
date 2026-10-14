---
type: entity
status: budding
created: 2026-10-14
updated: 2026-10-14
sources:
  - raw/Linux/Installation_Problems.md
  - raw/Linux/ArchLinux Installation Guide.md
tags:
  - linux
  - archlinux
  - network
---

iwd（iNet Wireless Daemon）是 Linux 上的无线网络守护进程，Arch 安装镜像默认用它连接 Wi-Fi。

## 交互式用法

```bash
iwctl
[iwd]# device list                          # 查看无线网卡名称，如 wlan0
[iwd]# adapter phy0 set-property Powered on # 先开适配器
[iwd]# device wlan0 set-property Powered on # 再开设备
[iwd]# station wlan0 scan
[iwd]# station wlan0 get-networks
[iwd]# station wlan0 connect "SSID"         # 输入密码
[iwd]# exit
```

## 常见错误

```bash
[iwd]# device name set-property Powered on   # ✗ 失败
```

原因有两点：命令里的 `name` 是**占位符**，必须换成实际设备名；且必须**先开启 adapter，再开启 device**，不能直接操作设备。

## 安装后的注意点

新系统若只装了 NetworkManager（命令为 `nmcli`），重启后会遇到 `iwctl: command not found`。此时可用 `nmcli` 连网，或在 chroot 中补装 `iwd`。详见 [[2026-10-14-ArchLinux安装问题排查]]。

## 关联

- 宿主系统：[[Arch Linux]]
- 安装流程中的位置：[[2026-10-14-ArchLinux安装流程]]
- 包管理：[[pacman]]

来源：[[raw/Linux/Installation_Problems]]、[[raw/Linux/ArchLinux Installation Guide]]
