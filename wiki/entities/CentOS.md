---
type: entity
status: budding
created: 2026-10-27
updated: 2026-10-27
sources:
  - raw/Frontend_Backend/Centos7_Install_MySQL.md
  - raw/Frontend_Backend/CentOS-7-reg.repo.md
tags:
  - bigdata
  - centos
---

**CentOS 7** 是 2014 年发布的 RHEL 7 下游重建版，Linux 发行版，曾是国内大数据/服务端教学环境的事实默认（Hadoop、Hive、MySQL 的教程大都以它为例）；已于 2024-06-30 结束维护（EOL）。

包管理与 [[Arch Linux]] 的路线相反：Arch 用 [[pacman]] 滚动更新、单一仓库；CentOS 7 用 `yum`（dnf 的前身）读 `/etc/yum.repos.d/*.repo` 里的多个仓库，底层是 `rpm`。仓库条目由 `baseurl`（含 `$releasever`/`$basearch` 占位）、`gpgcheck=1` 与 `gpgkey` 组成，素材里 `CentOS-7-reg.repo` 换成华为云镜像、保留 base/updates/extras 三个仓库并禁用 `centosplus`，生效要 `yum clean all` → `yum makecache` → `yum repolist`。

服务管理是它和现代发行版最容易混的地方：CentOS 7 名义上已用 **systemd**（`systemctl`），但 MySQL 官方 RPM 与素材里的写法仍是 SysV 风格（`service mysql start` / `service mysql restart`），两套命令指向同一批 unit，混用时要注意服务名是 `mysqld` 还是 `mysql`（EL7 的 mysql-community-server 装出来的是 `mysqld`）。另外 SELinux 默认 enforcing、`firewalld` 默认开启，两者都会静默拦掉远程连接（MySQL 3306 就是典型），素材里完全没提这两项。

版本上，CentOS 7 的 EOL 意味着镜像站会陆续下线、`$releasever` 解析到的源可能失效，这也是为什么素材必须自己写 `.repo` 兜底；同类替代一般是 Rocky Linux / AlmaLinux，或迁移到仍维护的发行版。它上面装的 MySQL 5.6/5.7 也分别于 2021-02 与 2023-10 EOL，整条链路都是过期技术栈。

## 关联

- 实体：[[Hive]]、[[Apache Zeppelin]]
- 概念：[[数据仓库]]
- 相邻发行版：[[Arch Linux]]、[[pacman]]（对比 yum/rpm 与 systemd 的用法差异）
- 摘要：[[2026-10-27-CentOS7安装MySQL]]

## 参考

- CentOS 生命周期（EOL 日期核对）：https://endoflife.date/centos

来源：[[raw/Frontend_Backend/Centos7_Install_MySQL]]、[[raw/Frontend_Backend/CentOS-7-reg.repo]]
