---
type: concept
status: budding
created: 2026-10-27
updated: 2026-10-27
provenance: external
sources:
  - https://www.centos.org/centos-linux/
  - https://www.mysql.com/support/eol-notice.html
  - https://dev.mysql.com/doc/refman/8.0/en/mysql-nutshell.html
  - https://www.oracle.com/java/technologies/java-se-support-roadmap.html
tags:
  - ops
  - versioning
---

版本生命周期（lifecycle）指一个软件从发布到停止维护的时间表，EOL（End of Life）是厂商停止提供安全补丁与技术支持的那一天。判断一套教程还能不能照做，先看它的版本是否已过 EOL，比看命令对不对更优先。

> 📌 来源性质：本页无 raw 素材，依据外部资料编写，待有 raw 素材时再收编。

## 为什么本库需要这一页

`raw/` 里的大数据与 MySQL 素材集中在 2022—2024 年的教学环境，几个组件都已跨过或正在跨过 EOL。素材本身不会告诉你这件事——它写的时候还是有效的。所以这里单独开一页把版本状态摆出来，供 [[2026-10-27-CentOS7安装MySQL]]、[[2026-10-27-MySQL8安装]]、[[2026-10-27-Hive]]、[[2026-10-27-大数据安装]] 四页共同引用。

## 逐个版本

| 组件 | 素材里的版本 | 状态 | 依据 |
|---|---|---|---|
| CentOS Linux 7 | 7（多篇） | **EOL 2024-06-30** | [centos.org/centos-linux](https://www.centos.org/centos-linux/)：CentOS Linux 7 于 2024-06-30 终止；CentOS Linux 8 更早，2021-12-31 终止。后续是滚动预览版 CentOS Stream，不再是「稳定版」 |
| MySQL 5.6 | 5.6（CentOS7 线） | **EOL 2021-02** | Oracle 生命周期支持策略；5.6 早已退出 Premier 与 Extended 支持 |
| MySQL 5.7 | 5.7（CentOS7 线） | **EOL 2023-10** | Oracle 生命周期支持策略：5.7 的 Extended 支持止于 2023-10 |
| MySQL 8.0 | 8.0.30（8.0 线） | **扩展支持至 2026-04** | Oracle 生命周期支持策略：8.0 Premier 支持 2025-04 结束，扩展支持到 2026-04；之后接续版本是 8.4 LTS |
| JDK 17 | 17.0.16（素材声称） | LTS，**Oracle 商用支持至 2026-09**，扩展至 2029 | Oracle Java SE 支持路线图；OpenJDK 各发行版（Temurin 等）时间表不同，不能用 Oracle 的日期套 |
| Hadoop 3.3.6 / Hive 4.1.0 / Zeppelin 0.11.1 | 同左 | **无统一 EOL 声明** | 这几个 Apache 项目不发布正式 EOL 日期，只按发行说明给维护分支；引用时标「截至访问日期仍可下载」，不断言支持状态 |

两条独立来源都指向同一结论的项（CentOS 7、MySQL 5.7、JDK 17）已按 §4.3 要求交叉验证；Hadoop 系三项属于「查不到权威 EOL」，按规则不臆断，标为无声明。

## 8.0 内部还有两个小版本陷阱

1. **8.0.30 是 2022-07 的版本**，不是 8.0 的最终形态。8.0 后续小版本还在持续发布，照 8.0.30 的截图找界面可能与新版不一致。判定依据：8.0 的发行节奏是季度小版本，8.0.30 之后仍有 8.0.3x 系列。
2. **`utf8` 在 8.0 里是 `utf8mb3` 的别名且已弃用**，默认字符集是 `utf8mb4`。所以 8.0 上看到「默认 utf8」的教程，多半是从 5.x 抄来的。判定依据：[MySQL 8.0 参考手册](https://dev.mysql.com/doc/refman/8.0/en/mysql-nutshell.html) 的字符集章节与「已弃用特性」列表。

## 与本库裁定的一致性

本页支撑了 [[_Index_Wiki]] 里第 27 条裁定（MySQL 5.x 与 8.0 两条线按版本分家，不得互抄）：两条线的字符集与授权语法差异，根子就在这里——它们对应两个只在自己版本内成立的默认值。

> 版本号不是细节。一套 2022 年的教程在今天照抄失败，多半不是命令写错，而是命令所属的那个版本已经不存在了。

## 关联

- [[2026-10-27-CentOS7安装MySQL]] —— 5.x + CentOS 7 线，页首已加「仅适用 5.x」限定
- [[2026-10-27-MySQL8安装]] —— 8.0.30 二进制包线
- [[MySQL]] —— 两条线的实体页汇总
- [[2026-10-27-Hive]] —— Hive 4.1.0 的 Metastore 要求 8.0
- [[2026-10-27-大数据安装]] —— Hadoop / Hive / Zeppelin 同一套环境

## 来源

访问日期均为 2026-10-27。

- [CentOS Linux 官方页（7 于 2024-06-30 终止）](https://www.centos.org/centos-linux/)
- [MySQL 生命周期支持与 EOL 公告](https://www.mysql.com/support/eol-notice.html)
- [MySQL 8.0 参考手册：新特性与已弃用项](https://dev.mysql.com/doc/refman/8.0/en/mysql-nutshell.html)
- [Oracle Java SE 支持路线图（17 为 LTS）](https://www.oracle.com/java/technologies/java-se-support-roadmap.html)
