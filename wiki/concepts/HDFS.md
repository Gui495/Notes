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

**HDFS**（Hadoop Distributed File System）是 [[Hadoop]] 的分布式存储层，用「一份元数据 + 多份数据块」的方式把大文件摊到多台机器上，从而在廉价硬盘上得到高容错的大容量存储。

它把节点分成两个角色。**NameNode** 只存元数据（目录树、文件到数据块的映射、块所在的 DataNode），**DataNode** 存真正的数据块。这个拆分的代价是 NameNode 成为全局单点——所以才有 SecondaryNameNode 定期合并 editlog，但它是辅助快照，不是热备。素材里 [[2026-10-27-大数据安装]] 配置的两个关键项正好对应两个层面：`fs.defaultFS=hdfs://192.168.220.129:9000` 是客户端找 NameNode 的地址（9000 是 RPC 端口，9870 是 Web UI 端口），`dfs.replication=1` 是副本数——单机伪分布式只能设 1，生产默认 3。

副本机制是 HDFS 对抗「部分失败」的核心：任何一个块丢了，还能从别的副本读，NameNode 在 DataNode 心跳超时后会重新复制补齐。也正因为副本要靠 NameNode 的元数据来指路，`hdfs namenode -format` 只能执行一次：重复格式化会在 `hadoop.tmp.dir` 里生成新的 clusterID，老 DataNode 拿着旧 clusterID 就注册不上，表现为 DataNode 起不来。素材给的补救是清空 tmp 目录重新格式化，而这等于丢掉原有数据——这是初始化阶段最容易踩的不可逆坑。

HDFS 的定位只到「存」为止，它不擅长小文件与随机写，这也是 [[Hive]] 的仓库目录（`/hive/warehouse`）直接建在 HDFS 上、由 [[MapReduce]] 做全量扫描而不是逐行更新的原因。

## 关联

- 概念：[[MapReduce]]、[[YARN]]、[[分布式系统]]、[[数据仓库]]
- 实体：[[Hadoop]]、[[Hive]]
- 摘要：[[2026-10-27-大数据安装]]、[[2026-10-27-Hive]]

来源：[[raw/Frontend_Backend/大数据安装]]、[[raw/Frontend_Backend/Hive]]
