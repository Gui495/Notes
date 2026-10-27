---
type: summary
status: budding
created: 2026-10-27
updated: 2026-10-27
sources:
  - raw/Frontend_Backend/Centos7_Install_MySQL.md
  - raw/Frontend_Backend/CentOS-7-reg.repo.md
tags:
  - bigdata
  - centos
  - mysql
  - install
---

> 📌 **适用范围：仅 MySQL 5.6 / 5.7 + CentOS 7**，三者均已 EOL（CentOS 7：2024-06-30，MySQL 5.6：2021-02，5.7：2023-10）。新部署请走 [[2026-10-27-MySQL8安装]] 的 8.0.30 路线；本页的字符集、改密、授权语法都**只对 5.x 成立**。

对 [[raw/Frontend_Backend/Centos7_Install_MySQL]] 的提炼（含仓库源 [[raw/Frontend_Backend/CentOS-7-reg.repo]] 的并入）：CentOS 7 上从换 yum 源、清 MariaDB、装 MySQL 5.6/5.7，到修中文乱码与开放远程登录的最短路径。

## 顺序

换源（`CentOS-7-reg.repo`）→ 清 MariaDB → 装 MySQL → 文件授权 → 字符集 → root 密码与远程登录。

## 0. yum 源：华为云镜像四仓库

`.repo` 文件放在 `/etc/yum.repos.d/`（名 `CentOS-Base.repo` 或 `CentOS-7-reg.repo`），`gpgcheck=1` + `gpgkey=https://repo.huaweicloud.com/centos/RPM-GPG-KEY-CentOS-7`。

| 仓库 | baseurl 尾段 | 默认 |
|---|---|---|
| `[base]` | `/centos/$releasever/os/$basearch/` | 启用 |
| `[updates]` | `/centos/$releasever/updates/$basearch/` | 启用 |
| `[extras]` | `/centos/$releasever/extras/$basearch/` | 启用 |
| `[centosplus]` | `/centos/$releasever/centosplus/$basearch/` | `enabled=0` |

- `$releasever` / `$basearch` 由 yum 自动展开，不能写死。
- 生效三连：`yum clean all` → `yum makecache` → `yum repolist`。
- `mirrorlist` 被注释掉是有意的：官方镜像列表已失效时靠 `baseurl` 兜底。
- `centosplus` 含增强包、可能替换基础包，默认禁用是对的。

## 1. 清掉 MariaDB

`rpm -qa | grep mariadb` 查包 → `rpm -e --nodeps <包名>` 强删。`--nodeps` 无视依赖关系，可能连带留下坏依赖，属于「先删了再说」的做法。

## 2. 两条安装路径

- **5.6**：`wget http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm` → `rpm -ivh` → `yum install mysql-server -y`。
- **5.7**：`yum -y install mysql57-community-release-el7-10.noarch.rpm` → `rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2022` → `yum install mysql-server -y`。

## 3. 文件授权

`chown -R root:root /var/lib/mysql`、`chown root /var/lib/mysql/`（见勘误第 3 条）。

## 4. 中文乱码：改 `/etc/my.cnf`

三段配置：`[mysql] character-set-server=utf8`、`[client] default-character-set=utf8`、`[mysql] default-character-set=utf8`，然后 `service mysql restart`（见勘误第 2 条：组名写错，服务端字符集实际不生效）。

## 5. root 密码与远程登录

- **5.6**：首次直接登录无密码 → `use mysql;` → `update user set password=password('ok') where user='root';`
- **5.7**：临时密码在日志里 → `ALTER USER 'root'@'localhost' IDENTIFIED BY '你的密码';`；密码策略要求 ≥8 位、含大小写字母与数字。
- **远程**：`GRANT ALL PRIVILEGES ON *.* TO root@"%" IDENTIFIED BY "你的密码";` + `flush privileges;`

## 素材勘误与不一致

1. **临时密码路径**：原文 `grep "password" /var/log/mysql.log`。EL7 的 mysql-community-server 默认 `log-error` 是 **`/var/log/mysqld.log`**。**判定证据**：① MySQL 5.7 官方安装文档给的就是 `grep 'temporary password' /var/log/mysqld.log`；② 同一段又用 `service mysql restart` 这种 SysV 写法，而 EL7 包是 systemd 的 `mysqld`，说明该段命令来自不同来源的混抄。状态：已裁定。
2. **字符集段组名错**：`character-set-server` 是**服务端**变量，必须写在 `[mysqld]` 段；原文写在 `[mysql]` 段，且 `[mysql]` 重复出现两次。`[mysql]` 只被 mysql 命令行客户端读取，所以服务端仍然是默认字符集，「修乱码」实际没修到。状态：已裁定（应改为 `[mysqld]` + `character-set-server=utf8mb4`）。
3. **数据目录属主**：`chown -R root:root /var/lib/mysql` 与 EL7 包默认的 `mysql:mysql` 相反；若 `mysqld` 以 mysql 用户运行，root 属主会让它无法写数据目录（表现为启动失败）。素材没说明为什么要改。状态：**待核实**（疑似某次报错的临时解法被当成通用步骤）。
4. **远程登录 SQL 的版本边界**：`GRANT ... IDENTIFIED BY` 在 MySQL 8.0 已移除（必须先 `CREATE USER`）。对本篇的 5.6/5.7 成立，但与 Hive 那边要的 MySQL 8.0 不兼容。状态：已裁定（页级范围限定见页首横幅）。
5. **整篇基于 EOL 版本**：MySQL 5.6 于 2021-02 EOL、5.7 于 2023-10 EOL，CentOS 7 于 2024-06-30 EOL（`.repo` 素材自己也注明「CentOS 7 已停止维护」）。状态：已裁定——素材过时，仅作历史路径参考，新部署走 8.0。
6. **缺步骤**：没写 `firewalld` 放行 3306、没提 SELinux、没提 `bind-address`，而这三项是远程连不上时的头号原因。

> ✅ **已裁定（2026-10-27，owner 授权）**：与 [[2026-10-27-Hive]] 的冲突按「版本分家」处理，不是二选一。

| 项目 | 本页（yum 路线） | [[2026-10-27-MySQL8安装]]（二进制包路线） |
|---|---|---|
| 版本 | MySQL 5.6 / 5.7 | MySQL 8.0.30 |
| 字符集 | `utf8`（3 字节旧实现） | `utf8mb4`（8.0 默认） |
| 授权 | `GRANT ... IDENTIFIED BY` 一步到位 | 该语法已移除，必须先 `CREATE USER` 再 `GRANT` |
| 适用场景 | 只适用于 5.x 老环境 | [[Hive]] 4.1.0 的 Metastore 要求 8.0，走这条 |

结论：**本页只对应 5.x**，Hive 需要的 8.0 由二进制包路线满足。旁证：另一份素材 [[2026-10-27-MySQL8安装]]（raw/Frontend_Backend/MySQL8.md）用二进制 tar 包装 8.0.30，说明「要 8.0」在本库另有路径。

## 关联

- 概念：[[数据仓库]]（Hive Metastore 的落地库）
- 实体：[[CentOS]]、[[Hive]]
- 相邻主题：[[Linux中文显示与locale]]（同样是「中文乱码」，但一个是 locale/字体层、一个是数据库字符集层，机制不同）
- 索引：[[_Index_Wiki]]

来源：[[raw/Frontend_Backend/Centos7_Install_MySQL]]、[[raw/Frontend_Backend/CentOS-7-reg.repo]]
