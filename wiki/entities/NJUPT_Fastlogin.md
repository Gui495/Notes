---
type: entity
status: seedling
created: 2026-10-14
updated: 2026-10-14
sources:
  - raw/Linux/Installation_Problems.md
tags:
  - linux
  - network
  - script
---

NJUPT_Fastlogin 是用于南京邮电大学校园网自动登录的开源脚本（GitHub：`lux-QAQ/NJUPT_Fastlogin`），素材用它解决「校园网需 Web 认证、Live 环境难登录」的问题。

## 使用步骤（原素材）

```bash
git clone https://github.com/lux-QAQ/NJUPT_Fastlogin.git
cd NJUPT_Fastlogin
vim version2_ASMcross.cpp   # 修改学号和密码
g++ -O3 -no-pie -flto -s version2_ASMcross.cpp -o njupt_login
./njupt_login               # 测试
```

之后设置开机自动执行：systemd 服务或 crontab `@reboot`。

## 背景

素材记录：校园网为开放网络（无密码）但需 Web 认证，安装阶段先改用手机热点完成系统安装，装好后再配置该脚本。

## 关联

- 使用场景：[[2026-10-14-ArchLinux安装问题排查]]
- 宿主系统：[[Arch Linux]]

来源：[[raw/Linux/Installation_Problems]]
