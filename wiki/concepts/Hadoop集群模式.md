---
type: concept
status: budding
created: 2026-10-27
updated: 2026-10-27
provenance: external
sources:
  - https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/SingleCluster.html
  - https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/ClusterSetup.html
  - https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/hdfs-default.xml
  - https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-yarn/yarn-default.xml
  - https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-mapreduce/mapred-default.xml
  - https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/HDFSHighAvailabilityWithQJM.html
  - https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/Federation.html
tags:
  - bigdata
---

Hadoop 的「集群模式」指守护进程在几台机器、以什么身份运行：本地模式全在一个 JVM 里、伪分布式在一台机器上跑全套守护进程、完全分布式把守护进程分散到多台机器并由配置文件区分主从角色。

> 📌 来源性质：本页无 raw 素材，依据外部资料编写，待有 raw 素材时再收编。

## 三种模式的判断标准

判断模式不看启动脚本叫什么名字，看三件事：`jps` 有没有守护进程、`fs.defaultFS` 指向什么、同一份数据存几份。

| 模式 | 守护进程 | `fs.defaultFS` | 副本数 | 典型用途 |
|---|---|---|---|---|
| 本地模式（local / standalone） | 无，`jps` 只有一个 JVM | 不设或 `file:///`（`LocalFileSystem`） | 无 HDFS，谈不上副本 | 单机调试逻辑、跑通 WordCount |
| 伪分布式（pseudo-distributed） | 单机起 NameNode / DataNode / ResourceManager / NodeManager，各自独立 JVM | `hdfs://localhost:9000` | `dfs.replication=1` | 学习、验证配置、跑小数据 |
| 完全分布式（fully distributed） | 多机，master 起 NameNode / ResourceManager，worker 起 DataNode / NodeManager | `hdfs://<master>:8020` | 默认 3 | 真实存储与计算 |

判据细节：本地模式下作业输出会写到本地目录，日志里出现 `LocalJobRunner`；伪分布式下 `jps` 能在同一台机器上看到 `NameNode`、`DataNode`、`ResourceManager`、`NodeManager`、`SecondaryNameNode` 五个进程；完全分布式则要把 `jps` 分别到 master 和每台 worker 上执行，看到的是两组不同的进程集合。

**易错点**：装了 Hadoop 不等于是集群。只要 `fs.defaultFS` 还是 `file:///`、`mapreduce.framework.name` 还是默认的 `local`，`start-dfs.sh` 起多少个进程都没用，作业仍在本地跑。

## 完全分布式必须改的四类配置

配置文件都在 `$HADOOP_HOME/etc/hadoop/` 下。官方 Cluster Setup 文档列出的核心项可以归成四组，它们分别回答「集群入口在哪」「数据放哪、存几份」「资源由谁调度」「作业跑在哪个框架上」。

### core-site.xml：集群入口

```xml
<property>
  <name>fs.defaultFS</name>
  <value>hdfs://master:8020</value>
</property>
```

`fs.defaultFS` 是默认文件系统的 URI，客户端不显式指定 scheme 时就用它。伪分布式官方示例里写的是 `hdfs://localhost:9000`，完全分布式改成 master 的主机名与 RPC 端口（Hadoop 3 默认 RPC 端口 8020，很多发行版和旧教程仍是 9000，属版本/发行版差异，不裁决，以实际 `core-site.xml` 与 `netstat` 为准）。

### hdfs-site.xml：数据放哪、存几份

```xml
<property><name>dfs.replication</name><value>3</value></property>
<property><name>dfs.namenode.name.dir</name><value>/data/hadoop/dfs/name</value></property>
<property><name>dfs.datanode.data.dir</name><value>/data/hadoop/dfs/data</value></property>
```

默认值来自 `hdfs-default.xml`：`dfs.replication` 为 `3`，`dfs.namenode.name.dir` 为 `file://${hadoop.tmp.dir}/dfs/name`，`dfs.datanode.data.dir` 为 `file://${hadoop.tmp.dir}/dfs/data`。默认值把数据放在 `hadoop.tmp.dir` 下，而该目录常被系统 `/tmp` 清理策略波及——这是「重启后集群空了」的常见原因，生产必须显式改成独立磁盘路径。

副本数还要满足一个硬约束：**副本数不能大于 DataNode 台数**，否则文件会长期处于 under-replicated 状态，`hdfs fsck` 报 missing blocks。

### yarn-site.xml：资源由谁调度

```xml
<property><name>yarn.resourcemanager.hostname</name><value>master</value></property>
<property><name>yarn.nodemanager.aux-services</name><value>mapreduce_shuffle</value></property>
```

`yarn.resourcemanager.hostname` 在 `yarn-default.xml` 里的默认值是 `0.0.0.0`，意味着 ResourceManager 只在本地所有网卡上监听、不对外声明自己是谁；集群里必须改成 master 的主机名，否则 worker 上的 NodeManager 找不到 RM。

`yarn.nodemanager.aux-services` 是 NodeManager 上挂载的附属服务，官方说明它是「Shuffle service that needs to be set for Map Reduce applications」——不设成 `mapreduce_shuffle`，MapReduce 作业能提交、容器能起，但 reducer 拉不到 map 输出，表现为卡在 reduce 阶段。

### mapred-site.xml：作业跑在哪个框架

```xml
<property><name>mapreduce.framework.name</name><value>yarn</value></property>
<property>
  <name>mapreduce.application.classpath</name>
  <value>$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/*:$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/lib/*</value>
</property>
```

`mapreduce.framework.name` 的默认值是 `local`（见 `mapred-default.xml`），这是最容易被忽略的一项：只改 HDFS 相关配置、不改这一项，作业不会进 YARN，`yarn application -list` 里永远是空的。

配置项汇总：

| 文件 | 关键项 | 默认值 | 改它的目的 |
|---|---|---|---|
| `core-site.xml` | `fs.defaultFS` | `file:///` | 把默认文件系统指向 HDFS |
| `hdfs-site.xml` | `dfs.replication` | `3` | 控制副本数，需 ≤ DataNode 数 |
| `hdfs-site.xml` | `dfs.namenode.name.dir` | `${hadoop.tmp.dir}/dfs/name` | NameNode 元数据落盘位置 |
| `hdfs-site.xml` | `dfs.datanode.data.dir` | `${hadoop.tmp.dir}/dfs/data` | DataNode 数据块落盘位置 |
| `yarn-site.xml` | `yarn.resourcemanager.hostname` | `0.0.0.0` | 让 NodeManager 找到 RM |
| `yarn-site.xml` | `yarn.nodemanager.aux-services` | 无（需显式设） | 提供 shuffle 服务 |
| `mapred-site.xml` | `mapreduce.framework.name` | `local` | 让作业走 YARN 而不是本地 JVM |

## 主从角色划分与 SSH 免密

官方 Cluster Setup 对角色划分的表述很直接：master 上跑 NameNode 和 ResourceManager，其余机器跑 DataNode 和 NodeManager，这些「其余机器」就是 worker。SecondaryNameNode（Hadoop 3 里也有，用于合并 FsImage 与 EditLog）通常单独放一台或放在 master 上，它不是 NameNode 的热备——这点常被误解，热备是 HA 里的 Standby NameNode。

SSH 免密不是可选项，因为 `start-dfs.sh` / `stop-yarn.sh` 这类脚本是「在 master 上执行、再 ssh 到各 worker 上起进程」的：脚本通过 `workers` 文件拿到主机列表，逐台 ssh 过去执行 `hadoop-daemon.sh`。所以 master 必须能免密 ssh 到每台 worker（以及自身）。

```bash
# master 上
ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa      # 官方单机文档给的正是这组参数
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 0600 ~/.ssh/authorized_keys
ssh worker1 'hostname'                        # 能免密打印主机名才算通
```

**易错点**：`authorized_keys` 权限放宽到 644 也会导致免密失效（sshd 的 `StrictModes` 会拒绝）；第一次 ssh 会问 yes/no，脚本化场景要预先 `ssh-keyscan` 写入 `known_hosts`，否则启动脚本在交互处挂死。

## workers 文件

`$HADOOP_HOME/etc/hadoop/workers`（Hadoop 2 时代叫 `slaves`，3.0 起改名）列出全部 worker 主机名，一行一台：

```
worker1
worker2
worker3
```

它的作用范围要说清：**只有启停脚本读它，Java 进程本身不读它**。DataNode 是自己在 `core-site.xml` 的 `fs.defaultFS` 上向 NameNode 注册的，不需要知道 peers。所以「workers 里少写了一台」的表现是：那台机器上的 DataNode 照常运行并注册成功，但 `start-dfs.sh` 不会去启动它。

**版本差异**：Hadoop 2.x 与 3.x 的目录结构、文件名都不同（`slaves` → `workers`，`hadoop-daemon.sh` → `hdfs --daemon start`），照抄旧教程的脚本名会直接报「command not found」。

## 格式化只能做一次

`hdfs namenode -format` 做的事是在 `dfs.namenode.name.dir` 下生成全新的空命名空间（`VERSION` 文件里带一个随机 `clusterID`）。DataNode 首次启动时把 NameNode 告知的 `clusterID` 写进自己的 `VERSION`，此后两者必须一致。

第二次 format 会生成**新的** `clusterID`，而此时 DataNode 的 `VERSION` 里还是旧的，两边对不上，DataNode 启动即报 `Incompatible clusterIDs`，拒绝注册。恢复代价很大：要么删掉 DataNode 的数据目录重来（数据全丢），要么手工把 `VERSION` 里的 `clusterID` 改成一致。

所以顺序必须是：**先改完所有配置 → 首次 `-format` → 再 `start-dfs.sh`**。反过来（先起进程再 format）在 Hadoop 3 里已经被拦下，`start-dfs.sh` 会检测到未格式化的元数据目录并报错退出，但这条检查只保护「完全没格式化」的情况，不保护「已经格式化过又再格式化一次」。

## 启动顺序与 jps 自检

```bash
# 仅首次，或明确要重建命名空间时
hdfs namenode -format

# 之后每次启动
start-dfs.sh        # 或分别 sbin/hdfs --daemon start namenode / datanode
start-yarn.sh       # ResourceManager + NodeManager
mapred --daemon start historyserver   # 需要看历史作业时

jps                 # 本机进程自检
```

`jps` 自检清单（按角色分两台机器看）：

| 机器 | 期望看到的进程 | 缺失时的含义 |
|---|---|---|
| master | `NameNode` | 元数据目录没格式化或端口被占 |
| master | `ResourceManager` | `yarn-site.xml` 有误，或 `start-yarn.sh` 未执行 |
| master（若部署） | `SecondaryNameNode` | checkpoint 不跑，EditLog 会一直增长 |
| worker | `DataNode` | 无法连上 NameNode，或 `clusterID` 不一致 |
| worker | `NodeManager` | 找不到 RM，或 aux-services 配错 |
| master | `JobHistoryServer` | 只能看运行中作业，看不到历史 |

验证顺序也建议自下而上：`hdfs dfsadmin -report` 看 DataNode 是否全部注册 → `hdfs dfs -mkdir /test && hdfs dfs -put` 看副本是否达标 → `yarn node -list` 看节点与资源 → 最后跑一个 MapReduce 作业确认 shuffle 正常。

**易错点**：`jps` 显示 `Jps` 自己，说明 jps 能跑但可能连的是另一个用户/另一个 JDK；`jps` 输出的进程名是 `DataNode` 但状态是 `zombie`，通常是端口被上一次未退干净的进程占用。

## HA 与 Federation 的一句话定位

- **HA（High Availability）**：解决「一个 NameNode 挂了整个 HDFS 不可用」。做法是两个 NameNode（Active/Standby）共享一份 EditLog，Hadoop 3 的推荐实现是 QJM（Quorum Journal Manager，奇数台 JournalNode），Standby 实时重放 EditLog 以便秒级切换，客户端靠 nameservice 逻辑名而不是具体主机寻址。
- **Federation**：解决「一个 NameNode 的命名空间和内存扛不住规模」。做法是多个**互相独立**的 NameNode 各自管一个命名空间（namespace），客户端通过 `ViewFs` 把它们挂成一棵逻辑目录树。Federation 不提供高可用，HA 也不扩展命名空间容量，两者是正交的两个维度，生产上常常叠加部署。

## 关联

- [[Hadoop]]：三种模式的划分与配置项来自 Hadoop 官方单机/集群安装文档，本页是那套流程的判据化整理。
- [[HDFS]]：`dfs.replication`、`dfs.namenode.name.dir` 直接决定 HDFS 的可靠性与落盘位置。
- [[YARN]]：`yarn.resourcemanager.hostname` 与 aux-services 是「资源调度能不能工作」的两道门槛。
- [[MapReduce]]：`mapreduce.framework.name` 决定作业进 YARN 还是留在本地 JVM，是最容易漏改的一项。
- [[分布式系统]]：主从角色、单点故障与 HA/Federation 的一般性问题在这里具体化。
- [[2026-10-27-大数据安装]]：本机只到伪分布式，本页补的是完全分布式的配置判据。
- [[Hive分区与分桶]]：集群起来之后的建模手段，出现在同一批概念页里。

## 来源

- Apache Hadoop, Hadoop: Setting up a Single Node Cluster（伪分布式配置、SSH 免密命令、`-format` 与 `start-dfs.sh` 的先后）：https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/SingleCluster.html ，访问日期 2026-10-27。
- Apache Hadoop, Hadoop Cluster Setup（master/worker 角色、`workers` 文件、`core-site.xml` / `hdfs-site.xml` / `yarn-site.xml` 关键项与 aux-services 说明）：https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/ClusterSetup.html ，访问日期 2026-10-27。
- Apache Hadoop, hdfs-default.xml（`dfs.replication`=3，`dfs.namenode.name.dir`、`dfs.datanode.data.dir` 默认值）：https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/hdfs-default.xml ，访问日期 2026-10-27。
- Apache Hadoop, yarn-default.xml（`yarn.resourcemanager.hostname` 默认 `0.0.0.0`）：https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-yarn/yarn-default.xml ，访问日期 2026-10-27。
- Apache Hadoop, mapred-default.xml（`mapreduce.framework.name` 默认 `local`）：https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-mapreduce/mapred-default.xml ，访问日期 2026-10-27。
- Apache Hadoop, HDFS High Availability Using the Quorum Journal Manager：https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/HDFSHighAvailabilityWithQJM.html ，访问日期 2026-10-27。
- Apache Hadoop, HDFS Federation：https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/Federation.html ，访问日期 2026-10-27。

> 说明：默认 RPC 端口（8020 还是 9000）、`slaves` 改名 `workers` 的具体版本、`jps` 进程清单的完整性，均随发行版与集群拓扑变化，文中已标注为版本/实践差异；`jps` 清单属于运维实践共识，官方文档未给出统一表格。
