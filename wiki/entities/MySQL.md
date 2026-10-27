---
type: entity
status: evergreen
created: 2026-10-27
updated: 2026-10-27
sources:
  - raw/Frontend_Backend/MySQL8.md
  - raw/Frontend_Backend/MySQL_Data_Management.md
  - raw/Frontend_Backend/MySQL_Operations.md
  - raw/Frontend_Backend/DQL_Command.md
tags:
  - mysql
---

MySQL 是一款开源的关系型数据库管理系统，本知识库覆盖的素材以 MySQL 8.0.30 在 CentOS 7 上的二进制包安装为起点，延伸到 DML 数据操作、事务与索引、备份恢复、DQL 查询四个主题。

## 素材组织（4 篇的分工）

| 篇目 | 主题 | 对应概念页 |
|---|---|---|
| MySQL8 | 8.0.30 二进制 tar 包安装九步、`utf8mb4` 配置、改密与远程访问 | — |
| MySQL_Data_Management | `INSERT`/`UPDATE`/`DELETE`/`TRUNCATE` | [[DML数据管理]] |
| MySQL_Operations | 事务 ACID 与提交回滚、四类索引、`EXPLAIN`、`mysqldump` 备份恢复 | — |
| DQL_Command | `SELECT` 全子句链、连接、分页、子查询、聚合与分组 | [[DQL查询]] |

## 安装要点（MySQL8.md）

- 目标：CentOS 7 + MySQL **8.0.30** 二进制 tar 包（`.tar.xz`），非 rpm、非 yum。
- 九步主干：卸掉系统自带的 `mariadb-libs` → `yum install libaio numactl` → 解压到 `/usr/local` 并改名 `mysql` → `chown -R root:root` 且权限 **755** → `./mysqld --initialize --user=... --datadir=...` 生成初始密码 → 改 `mysql.server` 与 `/etc/my.cnf`（`utf8mb4` 一整套、`datadir`、`socket`、`port`） → 启动并按需 `ln -s` 到 `/usr/bin`、`/etc/init.d` → `ALTER USER` 改密码、`UPDATE user SET Host='%'` 开远程 → `chkconfig` 开机自启。
- **必须用 `utf8mb4`** 而非 `utf8`：MySQL 的老 `utf8` 是每字符最多 3 字节的实现，存不下 emoji 等 4 字节字符；8.0 已把默认字符集改为 `utf8mb4`，素材显式配置一遍是为了兼容旧习惯与旧数据。
- 素材在这一篇里留下了三处前后不一致（socket 路径、服务注册重复、以 root 运行 `mysqld`），已记录在 [[2026-10-27-MySQL8安装]]，未擅自改数值。

## 语言与对象模型

MySQL 的 SQL 按功能分四类，本知识库的素材实际覆盖到两类半：**DQL**（`SELECT`，见 [[DQL查询]]）、**DML**（`INSERT`/`UPDATE`/`DELETE`，见 [[DML数据管理]]）、部分 **DDL**（`CREATE TABLE`、索引的增删、`TRUNCATE`）；**DCL**（`GRANT`/`REVOKE`）素材完全未覆盖。据此本知识库不单独建「SQL语句分类」页——建了也只会是一页没有素材支撑的空壳。

**存储引擎**：素材默认 InnoDB（8.0 的默认引擎），全文索引示例仍写 `ENGINE=MYISAM` 是旧教材残留，实际 5.6.4 起 InnoDB 已支持 `FULLTEXT`，见 [[2026-10-27-MySQL事务索引与备份]] 的订正。

## 事务与索引（MySQL_Operations.md）

- 事务四属性 ACID；MySQL 默认 `AUTOCOMMIT = 1`，因此做事务前必须先关自动提交。
- 四类索引：`PRIMARY KEY`（唯一、每表最多一个）、`UNIQUE`（可多个）、`INDEX`/`KEY`（常规）、`FULLTEXT`（大文本检索）。
- 索引不是越多越好——写入时要维护，小表与高频变动列不宜建。
- `EXPLAIN` 用于分析语句是否走索引；连接类型越好越靠前的顺序是 `const` → `eq_ref` → `ref` → `range` → `index` → `ALL`。
- 备份主力是 `mysqldump`，恢复两条路：`mysql < 文件` 或命令行内 `SOURCE 文件`。

## 版本并存：5.6/5.7 线 vs 8.0 线

本知识库 raw/ 里同时存在两条 MySQL 安装线，结论不可互相套用。两条都踩在已 EOL 的版本上（MySQL 5.6 于 2021-02、5.7 于 2023-10、CentOS 7 于 2024-06-30），新部署以 8.0 为准：

- 本页覆盖的 [[raw/Frontend_Backend/MySQL8]]：**8.0.30**，二进制 tar 包手工装。
- [[2026-10-27-CentOS7安装MySQL]] 覆盖的 [[raw/Frontend_Backend/Centos7_Install_MySQL]]：**5.6 / 5.7**，走 yum 仓库装。

> ✅ **已裁定（2026-10-27，owner 授权）**：两条线的字符集结论不同——CentOS7 线配 `utf8`，8.0 线配 `utf8mb4`。`utf8` 在 MySQL 里是每字符最多 3 字节的旧实现（8.0 已将其别名指向 `utf8mb3`），存不下 4 字节字符；8.0 的默认字符集已是 `utf8mb4`，因此在 8.0 上照抄 `utf8` 是**降级**而非配置。处置：[[2026-10-27-CentOS7安装MySQL]] 已加「仅适用 5.x」的页级限定，两条线互不套用。

> ✅ **已裁定（2026-10-27，owner 授权）**：远程授权写法按版本分开——CentOS7 线用 `GRANT ALL PRIVILEGES ... IDENTIFIED BY`（仅 5.x 成立），8.0 必须先 `CREATE USER` 再授权（本页对应的 8.0 素材用 `ALTER USER` 改密码 + `UPDATE user SET Host='%'` 放开来源）。照 CentOS7 线写法在 8.0 上会直接语法报错，故两条路线**不得互抄**。

## 与 Java 的关系

Java 通过 JDBC 访问 MySQL，驱动加载依赖 [[反射机制]]（`Class.forName("com.mysql.cj.jdbc.Driver")`）。这也是本知识库 Java 线与 MySQL 线唯一的直接交点，见 [[Java]]。

## 素材边界

没有视图、存储过程、触发器、函数；没有用户与权限管理（DCL）；没有主从复制与读写分离；没有锁机制细节（行锁/表锁/间隙锁）与隔离级别；没有性能调优的实战案例。

## 关联

- 概念：[[DQL查询]]、[[DML数据管理]]
- 对照实体：[[Java]]
- 安装与订正：[[2026-10-27-MySQL8安装]]、[[2026-10-27-MySQL事务索引与备份]]

来源：[[raw/Frontend_Backend/MySQL8]]、[[raw/Frontend_Backend/MySQL_Data_Management]]、[[raw/Frontend_Backend/MySQL_Operations]]、[[raw/Frontend_Backend/DQL_Command]]
