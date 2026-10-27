---
type: concept
status: budding
created: 2026-10-27
updated: 2026-10-27
sources:
  - raw/Frontend_Backend/大数据安装.md
tags:
  - bigdata
---

**YARN**（Yet Another Resource Negotiator）是 [[Hadoop]] 的资源调度层，把「哪台机器有多少 CPU/内存、给哪个作业用」从计算框架里独立出来，使同一集群可以同时跑 [[MapReduce]] 和其他计算框架。

它的核心是两个角色：**ResourceManager**（全局调度，决定资源给谁，Web UI 在 8088）与 **NodeManager**（单机代理，负责在本节点起容器、报心跳）。作业提交后先向 ResourceManager 要资源，拿到容器（container）后在容器里跑任务；资源不够就排队。素材里的 `yarn.resourcemanager.hostname=localhost` 是伪分布式写法，真集群要指到主节点。

与 MapReduce 的耦合点是 `yarn.nodemanager.aux-services=mapreduce_shuffle`：Reduce 阶段要把 Map 的输出按 key 拉到自己这里，这个「拉」由每个 NodeManager 上的 shuffle 服务承担，不配这一项 MapReduce 作业会在 shuffle 阶段失败。同理，容器是一套被裁剪过环境变量的运行环境，所以 [[2026-10-27-Apache Zeppelin]] 里必须额外把 `HADOOP_MAPRED_HOME` 写进 `mapred-site.xml` 的 `yarn.app.mapreduce.am.env` / `mapreduce.map.env` / `mapreduce.reduce.env`，否则容器里的任务找不到 Hadoop 的库。

YARN 解决了 [[分布式系统]] 里「无全局时钟 + 资源竞争」的协调问题：所有分配决策集中到 ResourceManager，用统一的计数器而不是各机器时钟排序。代价是 ResourceManager 成为调度单点，这也是它需要 HA 配置的原因。

## 关联

- 概念：[[MapReduce]]、[[HDFS]]、[[分布式系统]]、[[大数据4V]]
- 实体：[[Hadoop]]
- 摘要：[[2026-10-27-大数据安装]]、[[2026-10-27-Apache Zeppelin]]

来源：[[raw/Frontend_Backend/大数据安装]]
