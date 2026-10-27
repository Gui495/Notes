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
  - package-manager
  - troubleshooting
---

pacman 故障排查按「症状 → 原因 → 处理」组织，集中处理素材里反复出现的三类问题：GPG 签名与密钥环、换源后仍慢或报错、第三方仓库信任失败。

> 本页只记录**已发生的故障与解法**；pacman 的仓库结构与配置文件分工见 [[pacman]]，两页不重复。

## 一、GPG 签名与密钥环

| 症状 | 原因 | 处理 |
|---|---|---|
| `pacstrap` 报 "corrupted package (PGP signature)" | Live 环境的 `archlinux-keyring` 过旧，验不了新包的签名 | 在 `pacstrap` **之前**先跑 `pacman -Sy archlinux-keyring` |
| 换源后出现签名类报错 | 密钥环未初始化或未填充 | `pacman-key --init` → `pacman-key --populate archlinux` |
| `archlinuxcn` 报 "invalid signature" / "unknown trust" | 本地密钥环尚未信任该仓库维护者的签名密钥 | `pacman -Syu archlinuxcn-keyring`，失败则 `pacman-key --lsign-key "farseerfc@archlinux.org"` 后再装 |

**关键顺序**：签名问题几乎都发生在「先装包、后更新密钥环」这个逆序上。安装流程里把 `archlinux-keyring` 提前，能消掉一大半。

## 二、换源后仍慢或报错

| 症状 | 原因 | 处理 |
|---|---|---|
| `pacman-mirrors: command not found` | 该工具是 **Manjaro 专有**，纯 Arch 没有 | 手改 `/etc/pacman.d/mirrorlist`，或装 `reflector` |
| 已换国内源但仍慢 | 源顺序未置顶；pacman 按 `Server` 出现顺序依次尝试 | 把国内源剪到文件最顶端 |
| 想自动化排序 | —— | `pacman -S reflector` 后 `reflector -c China --sort rate --save /etc/pacman.d/mirrorlist` |

## 三、第三方仓库的安全红线

- 在 `/etc/pacman.conf` **末尾**追加 `[archlinuxcn]` 段；**不要**写进 `mirrorlist`（格式错误），**不要**放在官方仓库之前（核心组件可能被替换）。
- ⚠️ **不要用 `SigLevel = never` 跳过签名验证。** 这是网上流传的「偷懒」做法，代价是失去整条供应链的完整性校验。正确做法是继承 `[options]` 里的全局 `SigLevel`（通常是 `Required DatabaseOptional`）并正确导入密钥。

## 四、验证配置是否真的生效

```bash
sudo pacman -Syyu
```

无报错即说明镜像源与仓库配置完成；随后可试装一个只在 archlinuxcn 里有的包（如 `yay`）做端到端确认。

## 关联

- 工具说明与命令表：[[pacman]]
- 故障现场记录：[[2026-10-14-ArchLinux安装问题排查]]、[[2026-10-14-ArchLinux安装流程]]
- 系统背景：[[Arch Linux]]

来源：[[raw/Linux/ArchLinux Installation Guide]]、[[raw/Linux/Installation_Problems]]
