---
type: concept
status: budding
created: 2026-10-27
updated: 2026-10-27
sources:
  - raw/Frontend_Backend/大数据安装.md
  - raw/Frontend_Backend/Hive.md
tags:
  - bigdata
---

**MapReduce** 是 [[Hadoop]] 的批处理计算模型，把一次计算拆成「Map 分散处理」与「Reduce 聚合」两段，中间靠分区、排序、分组把相同 key 的数据送到同一个地方。

它的执行链路是：输入先切成多个分片（split）→ 每个分片交给一个 MapTask，产出若干 KV 对 → 这些 KV 先按 key 分区（Partition，决定去哪个 ReduceTask）、再在分区内排序（Sort）、再按 key 分组（Group）→ 相同 key 的值落到同一个 ReduceTask 里做聚合。三步各有各的必要性：分区解决「同一个 key 必须同处一地」，排序解决「Reduce 端不用全量建哈希」，分组解决「一次 Reduce 调用拿到这个 key 的全部值」。

理解这条链路就能理解它的定位边界。MapReduce 是「磁盘式」批处理：中间结果落盘、任务粒度粗、启动开销大，适合一次扫完全量、容忍分钟级延迟的场景，不适合迭代计算与实时流。素材里 [[Hive]] 默认引擎就是 `hive.execution.engine=mr`——SQL 被翻译成一串 MR 作业，所以 Hive 的每个查询都有可感知的启动延迟；Hive 4 官方建议迁到 Tez 正是为了消掉中间落盘。

它和 [[YARN]] 的分工是「算」与「调度」分离：MapReduce 只描述任务与数据流，资源由 ResourceManager 分配，容器里跑 Map/Reduce 任务时需要 `yarn.nodemanager.aux-services=mapreduce_shuffle` 才拿得到 shuffle 服务（这条在 [[2026-10-27-Apache Zeppelin]] 的补丁里也是同一个原理：容器里缺 `HADOOP_MAPRED_HOME` 就找不到库）。

## 关联

- 概念：[[HDFS]]、[[YARN]]、[[分布式系统]]、[[数据仓库]]
- 实体：[[Hadoop]]、[[Hive]]
- 摘要：[[2026-10-27-大数据安装]]、[[2026-10-27-Hive]]

来源：[[raw/Frontend_Backend/大数据安装]]、[[raw/Frontend_Backend/Hive]]
