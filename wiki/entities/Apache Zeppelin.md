---
type: entity
status: budding
created: 2026-10-27
updated: 2026-10-27
sources:
  - raw/Frontend_Backend/Apache_Zeppelin.md
tags:
  - bigdata
---

**Apache Zeppelin** 是基于 Web 的交互式数据分析笔记本，把 SQL、Shell、Markdown 等写在同一份文档里并按段执行，结果直接以表格/图表呈现。

它的核心机制是 **Interpreter（解释器）**：每段代码绑定一个解释器，由解释器进程代为执行，因此同一个笔记本里可以混用不同后端。早期 Zeppelin 有专门的 Hive 解释器，0.10.0 之后改为统一的 **JDBC 解释器**去连 [[Hive]] 的 HiveServer2——素材用的就是这个模式：解释器名 `hive`、组 `jdbc`，四行配置分别是 `default.driver=org.apache.hive.jdbc.HiveDriver`、`default.url=jdbc:hive2://localhost:10000/default`、`default.user`（必须与 HiveServer2 的启动用户一致）、空密码，之后用 `%hive` 或 `%jdbc(hive)` / `%jdbc(mysql)` 指定数据源。

它能读 Hadoop 集群配置，靠的是 `zeppelin-env.sh` 里设 `JAVA_HOME`、`HADOOP_HOME`、`HADOOP_CONF_DIR`、`HIVE_HOME`、`HIVE_CONF_DIR`；但光有配置还不够，素材要求打两处 Hadoop 补丁：一是 `core-site.xml` 加 `hadoop.proxyuser.root.groups=*` 与 `hadoop.proxyuser.root.hosts=*`，因为 Zeppelin 以 root 的代理身份提交作业，不放行会被 NameNode 拒绝；二是 `mapred-site.xml` 把 `HADOOP_MAPRED_HOME` 写进 `yarn.app.mapreduce.am.env` / `mapreduce.map.env` / `mapreduce.reduce.env`，否则 YARN 容器里找不到 Hadoop 的库，任务在容器启动阶段就失败。

版本现状：素材用 0.11.1（2024-04-02 发布，素材正文写「2024 年 3 月」差一个月），当前最新为 0.12.1（2026-06-12），0.12.0 是 2025-01-31。需要特别注意 JDK 17：官方对 JDK 17 的支持到 0.13.0 才落地（ZEPPELIN-6311，2026-09-07 标记 Fixed），0.11.x/0.12.x 的基线仍是 JDK 11——所以素材「JDK 17 + Zeppelin 0.11.1」是自担风险的组合。启动脚本方面，`bin/zeppelin-daemon.sh start` 本身就是后台守护启动，要看前台日志应该用 `bin/zeppelin.sh`。

## 关联

- 概念：[[数据仓库]]、[[MapReduce]]、[[YARN]]
- 实体：[[Hive]]、[[Hadoop]]、[[CentOS]]
- 摘要：[[2026-10-27-Apache Zeppelin]]

## 参考

- Apache Zeppelin 下载页（版本与发布日期）：https://zeppelin.apache.org/download.html
- ZEPPELIN-6311（JDK 17 支持）：https://issues.apache.org/jira/browse/ZEPPELIN-6311

来源：[[raw/Frontend_Backend/Apache_Zeppelin]]
