---
type: entity
status: budding
created: 2026-10-27
updated: 2026-10-27
sources:
  - raw/Frontend_Backend/Hive.md
  - raw/Frontend_Backend/Apache_Zeppelin.md
tags:
  - bigdata
---

**Apache Hive** 是建在 [[Hadoop]] 之上的数据仓库工具，用类 SQL 的 HiveQL 描述查询，把查询翻译成 [[MapReduce]]（或 Tez/Spark）作业在集群上执行，数据本身存放于 [[HDFS]]。

它由三块构成：**Metastore**（元数据服务，默认后端是 MySQL，thrift 端口 9083）、**HiveServer2**（对外提供 JDBC/ODBC 的查询入口，端口 10000）、**CLI/客户端**。这三者是独立的进程，启动顺序必须是 Hadoop → MySQL → metastore → hiveserver2 → 客户端；顺序错了或 Metastore 没起来，HiveServer2 会因为连不上 9083 而启动失败。表数据落在 `hive.metastore.warehouse.dir`（素材设为 `/hive/warehouse`），这是 HDFS 上的目录而不是本地路径。

安装上的两个高频坑，素材都记录了：一是 **Guava 冲突**，Hive 自带的 guava 版本与 Hadoop 的不一致，运行时抛 `NoSuchMethodError`，解法是删掉 `hive/lib/guava-*.jar`、把 Hadoop 的 `guava-*-jre.jar` 拷进去；二是 **驱动**，MySQL JDBC 驱动必须放进 `hive/lib/`，否则 Metastore 初始化抛 `NoClassDefFoundError`。初始化靠 `schematool -dbType mysql -initSchema`，成功时打印 `schemaTool completed`；重复初始化会因为库已存在而失败，需先 `DROP DATABASE hive` 再以 utf8mb4 重建。

版本现状：素材用 4.1.0，它于 2025-07-31 发布，是首个加入 JDK 17 支持的版本，官方配套 Hadoop 3.4.1 / Tez 0.10.5；而素材配的是 Hadoop 3.3.6（那是 4.0.1 的配套组合），严格说不在一张官方矩阵上。另需注意 4.2.0（2025-11-23）起最低要求 JDK 21，4.2.1 为 2026-08-24；3.x 线已于 2024-10-08 EOL。Hive 4 官方建议把执行引擎从 MR 迁到 Tez，因为 MR 每个作业都要落盘、交互式查询延迟明显。

> ⚠️ 矛盾：与 [[2026-10-27-CentOS7安装MySQL]] 冲突——本篇要求 MySQL 8.0（`com.mysql.cj.jdbc.Driver`、`mysql-connector-java-8.0.33.jar`），而 CentOS 页只覆盖 5.6/5.7 且改密与授权语法是 5.x 写法，待人类裁决实际部署哪个版本。库内另有 [[2026-10-27-MySQL8安装]]（raw/Frontend_Backend/MySQL8.md）走二进制 tar 装 8.0.30，是本篇要求的可行路径。

## 关联

- 概念：[[数据仓库]]、[[HDFS]]、[[MapReduce]]、[[YARN]]、[[Hive分区与分桶]]
- 实体：[[Hadoop]]、[[Apache Zeppelin]]、[[CentOS]]
- 摘要：[[2026-10-27-Hive]]

## 参考

- Apache Hive 下载与版本页：https://hive.apache.org/general/downloads/
- Hive 4.1.0 发布说明（JDK 17 支持、配套版本）：https://hive.apache.org/

来源：[[raw/Frontend_Backend/Hive]]、[[raw/Frontend_Backend/Apache_Zeppelin]]
