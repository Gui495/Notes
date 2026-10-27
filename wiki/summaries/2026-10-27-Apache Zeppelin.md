---
type: summary
status: budding
created: 2026-10-27
updated: 2026-10-27
sources:
  - raw/Frontend_Backend/Apache_Zeppelin.md
tags:
  - bigdata
  - zeppelin
  - install
---

对 [[raw/Frontend_Backend/Apache_Zeppelin]] 的提炼：Zeppelin 0.11.1 用 **JDBC 解释器**连 HiveServer2 的安装、配置与验证流程，外加两处必须打的 Hadoop 补丁。

## 它是什么

基于 Web 的交互式数据分析与协作笔记本，支持 SQL / Scala / Python / R；靠**解释器（Interpreter）**机制接后端引擎，查询结果可直接可视化。

关键版本事实：**0.10.0 起不再提供独立 Hive 解释器**，Hive 功能并入 **JDBC 解释器**，所以整篇的主线是「配 JDBC 解释器」而不是「配 Hive 解释器」。

## 版本核对

| 项 | 素材说法 | 核实（2026-10） |
|---|---|---|
| 0.11.1 发布 | 2024 年 3 月 | **2024-04-02** |
| 是否最新 | 「最新稳定版」 | 当前最新是 **0.12.1**（2026-06-12） |
| JDK 17 支持 | 「对 JDK 17 有较好支持 ✅」 | 官方 JDK 17 支持到 **0.13.0** 才落地（ZEPPELIN-6311），0.11.x / 0.12.x 官方基线仍是 **JDK 11**；素材的「已验证兼容」应理解为**本地实测** |

## 安装步骤

1. 取包：`bin-all`（含全部解释器，开箱即用）优于 `bin-netinst`（只有核心解释器）。
2. 解压到 `/opt/soft/`，改名 `zeppelin-0.11.1`。
3. `zeppelin-site.xml`：`zeppelin.helium.registry=helium`、`zeppelin.server.port=8080`、`zeppelin.server.addr=0.0.0.0`。
4. `zeppelin-env.sh`：`JAVA_HOME`、`HADOOP_HOME`、`HADOOP_CONF_DIR`、`HIVE_HOME`、`HIVE_CONF_DIR`。
5. 给 Hadoop 打两个补丁（下面单独说）。
6. 配 JDBC 解释器，起服务（`bin/zeppelin-daemon.sh start`），访问 `http://<IP>:8080`。

## 两处 Hadoop 补丁（为什么必须打）

- **`core-site.xml`** 加 `hadoop.proxyuser.root.groups=*` 与 `hadoop.proxyuser.root.hosts=*`：Zeppelin 以 root 的代理身份访问 HDFS / 提交作业，不加会被代理用户校验拦掉。
- **`mapred-site.xml`** 加 `yarn.app.mapreduce.am.env`、`mapreduce.map.env`、`mapreduce.reduce.env`，值均为 `HADOOP_MAPRED_HOME=/opt/soft/hadoop-3.3.6`：否则 YARN 容器里找不到 Hadoop 库，作业起不来。

## JDBC 解释器四行参数

| 属性 | 值 |
|---|---|
| `default.driver` | `org.apache.hive.jdbc.HiveDriver` |
| `default.url` | `jdbc:hive2://localhost:10000/default` |
| `default.user` | 需与启动 HiveServer2 的用户一致 |
| `default.password` | 未设认证时留空 |

- 解释器名填 `hive`、组选 `jdbc`，之后 notebook 里用 `%hive` 引用。
- 多数据源用前缀区分：`%jdbc(hive)` / `%jdbc(mysql)`。

## 验证链

`SHOW DATABASES;` → `SHOW TABLES;` → `SELECT * FROM test LIMIT 10;`。能用说明整条链通了：Zeppelin → JDBC → HiveServer2(10000) → [[Hive]] → [[MapReduce]]/[[YARN]] → [[HDFS]]。

## 素材勘误与不一致

1. **安装包体积**：原文 `bin-all ~1.6 GB` / `bin-netinst ~799 MB`。官方 0.11.1 为 **843 MB / 577 MB**（1.5 G 是 0.10.1 的旧数）。**判定证据**：官方下载页按版本逐一列体积，0.10.1 才是 1.5g。状态：已裁定。
2. **启动方式讲反**：`bin/zeppelin-daemon.sh start` 本身就是后台守护式启动，原文把它标成「前台启动（用于调试）」，又给出 `nohup ... daemon.sh start` 作「后台启动（推荐）」，等于给后台进程再套一层后台；前台调试应用 `bin/zeppelin.sh`。状态：已裁定。
3. **驱动放置两套并行**：5.1 节把整个 `$HIVE_HOME/lib/*.jar` 拷进 `interpreter/jdbc/`，第八节又拷 `hive-jdbc-*.jar` / `hive-service-*.jar` 到 `$ZEPPELIN_HOME/lib/`；两处 classpath 不同，素材没说明该用哪条。状态：**待人类裁决**（照 5.1 的整目录拷贝通常够）。
4. **4.3 节 XML 残缺**：第一个 `<property>` 开标签缺失，片段从 `<name>hadoop.proxyuser.root.groups` 直接开始，照抄会写坏 `core-site.xml`。状态：已裁定（素材残缺）。
5. **文件名笔误**：`mapped-site.xml` 应为 **`mapred-site.xml`**。状态：已裁定。
6. **路径三套并存**：正文用 `/opt/soft/...`，第八节用 `/usr/local/softwave/...`，`JAVA_HOME` 又写 `/opt/soft/jdk17`（前一篇是 `/opt/soft/jdk17012`）。状态：已裁定（内部不一致）。
7. **「0.10.0 起移除 Hive 解释器」未核**：本次只核实了版本号与体积，这条按素材原样保留，使用前建议对照 0.11.1 官方文档确认。状态：待核实。

## 关联

- 概念：[[数据仓库]]、[[HDFS]]、[[MapReduce]]
- 实体：[[Apache Zeppelin]]、[[Hive]]、[[Hadoop]]
- 前置素材：[[2026-10-27-Hive]]、[[2026-10-27-大数据安装]]
- 索引：[[_Index_Wiki]]

来源：[[raw/Frontend_Backend/Apache_Zeppelin]]

## 参考

- [Apache Zeppelin 下载页（0.12.1 / 0.11.1 发布时间与包体积）](https://zeppelin.apache.org/download.html)
- [ZEPPELIN-6311 Supports JDK 17（fixVersion 0.13.0）](https://issues.apache.org/jira/browse/ZEPPELIN-6311)
- [Zeppelin JDBC 解释器文档](https://zeppelin.apache.org/docs/latest/interpreter/jdbc.html)
