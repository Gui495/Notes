---
type: entity
status: budding
created: 2026-10-27
updated: 2026-10-27
sources:
  - raw/Frontend_Backend/大数据安装.md
tags:
  - bigdata
---

**Apache Hadoop** 是开源的高可靠、可横向扩展的分布式大数据计算框架：单机可以平滑扩到数千节点，每个节点同时承担存储与计算。

它由四个模块组成：`hadoop-common`（公共库与配置）、[[HDFS]]（存储）、[[YARN]]（资源调度）、[[MapReduce]]（批处理计算）。这个「存储与计算同址」的设计是 Hadoop 早期的核心卖点——把计算搬到数据旁边，避免跨机搬数据；代价是计算与存储耦合、集群很难只扩其中一边。

运行模式有两档。**Standalone**：所有进程跑在同一个 JVM 里、没有守护进程，用于单元测试与本地调试；**Pseudo-distributed（伪分布式）**：单机上把 NameNode/DataNode/ResourceManager/NodeManager 全起一遍，用来在装真集群前验证配置。素材 [[2026-10-27-大数据安装]] 走的就是伪分布式：`core-site.xml` 的 `fs.defaultFS=hdfs://192.168.220.129:9000`、`hdfs-site.xml` 的 `dfs.replication=1`、`mapred-site.xml` 的 `mapreduce.framework.name=yarn`、`yarn-site.xml` 的 `yarn.resourcemanager.hostname=localhost` + `yarn.nodemanager.aux-services=mapreduce_shuffle`。

版本现状：素材用的是 3.3.6（配 JDK 17），但该版本的官方 Java 支持范围是 8/11，能跑 JDK 17 靠的是 `HADOOP_OPTS` 里加 `--add-opens` 打开模块访问；截至 2026 年官方发布页已有 3.4.3（2026-02-24）与 3.5.0（2026-04-02）。也就是说素材给的是「能跑但不在官方支持矩阵内」的组合，升级时要留意这一点。

与生态其它组件的关系：[[Hive]] 4.1.0 官方配套的是 Hadoop 3.4.1（而非 3.3.6），[[Apache Zeppelin]] 通过 `HADOOP_HOME`/`HADOOP_CONF_DIR` 读取本集群配置、并以代理用户身份提交作业，需要在 `core-site.xml` 里放行 `hadoop.proxyuser.*`。

## 关联

- 概念：[[HDFS]]、[[YARN]]、[[MapReduce]]、[[大数据4V]]、[[分布式系统]]
- 实体：[[Hive]]、[[Apache Zeppelin]]
- 摘要：[[2026-10-27-大数据安装]]

## 参考

- Apache Hadoop 发布页（版本与日期核对）：https://hadoop.apache.org/releases.html
- Apache Hadoop 3.3.6 文档（Java 支持范围）：https://hadoop.apache.org/docs/r3.3.6/

来源：[[raw/Frontend_Backend/大数据安装]]
