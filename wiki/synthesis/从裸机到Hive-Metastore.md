---
type: synthesis
status: budding
created: 2026-10-27
updated: 2026-10-27
sources:
  - raw/Frontend_Backend/大数据安装.md
  - raw/Frontend_Backend/Hive.md
  - raw/Frontend_Backend/Apache_Zeppelin.md
  - raw/Frontend_Backend/MySQL8.md
  - raw/Frontend_Backend/Centos7_Install_MySQL.md
  - raw/Linux/Installation_Problems.md
tags:
  - bigdata
  - linux
  - synthesis
---

本页把「从一台裸 CentOS 7 到能在 Zeppelin 里查 Hive 表」这条链路串起来，重点不在安装步骤（各页已写），而在**组件之间的版本与依赖关系**，因为实际卡住的几乎都是这一层。

## 链路

```
CentOS 7 ── JDK 17 ── Hadoop 3.3.6 ──┬── HDFS（存储）
                                     └── YARN（资源）
                            MySQL 8.0 ── Hive Metastore（元数据）
                                 Hive 4.1.0 ── HiveServer2
                                    Zeppelin 0.11.1（前端）
```

五个组件、四层依赖：JVM → Hadoop → Hive → 前端。上一层起不来时，下一层一定起不来，所以排查永远自下而上。

## 三个真实的卡点

### 1. 版本配套矩阵

素材自己给出的组合是「Hive 4.1.0 + Hadoop 3.3.6 + JDK 17」，但 Hive 4.1.0 官方配套的是 Hadoop 3.4.1，配 3.3.6 的是 4.0.1（见 [[_Index_Wiki]] 裁定第 15 条）。Hive 对 Hadoop 的依赖不是「能跑就行」——RPC 协议与 API 会在小版本间变化，混配的典型症状是 Metastore 起来了但 `beeline` 一连就报 `NoSuchMethodError`。

另一个版本陷阱：Zeppelin 对 JDK 17 的支持要到 0.13.0，0.11.1 跑在 17 上属于未验证组合。

### 2. MySQL 版本分家

Hive 的 Metastore 要一个关系库存元数据，素材选了 MySQL。但本库有两条 MySQL 线：

- [[2026-10-27-CentOS7安装MySQL]]：yum 装 5.6/5.7，`utf8` + `GRANT ... IDENTIFIED BY`。
- [[2026-10-27-MySQL8安装]]：二进制包 8.0.30，`utf8mb4` + 先 `CREATE USER` 再授权。

Hive 4.1.0 的 Metastore 要求 8.0，所以只能走第二条；照第一条的授权语法在 8.0 上会直接语法报错。这条已由 owner 裁定为「版本分家」，见 [[版本生命周期与EOL]]。

`schematool -initSchema` 失败时报的错往往指向连接，实际原因是驱动版本或字符集：8.0 用 `com.mysql.cj.jdbc.Driver`，5.x 用 `com.mysql.jdbc.Driver`，两者不通用。

### 3. 全线 EOL

CentOS 7 于 2024-06-30 终止，MySQL 5.6/5.7 早已 EOL，8.0 的扩展支持到 2026-04。也就是说这套教程描述的是一套**已经退役的环境**。照做可以，但不要用在生产；新环境应换 Rocky/AlmaLinux + MySQL 8.4 LTS + Hadoop 3.4.x。

## 端口与进程对照

| 组件 | 进程 | 端口 | 验证命令 |
|---|---|---|---|
| HDFS NameNode | `NameNode` | 9870（Web）、8020（RPC） | `hdfs dfs -ls /` |
| YARN | `ResourceManager` | 8088 | 浏览器打开 8088 |
| Hive Metastore | `HiveMetaStore` | 9083 | `schematool -dbType mysql -info` |
| HiveServer2 | `HiveServer2` | 10000 | `beeline -u jdbc:hive2://` |
| Zeppelin | `zeppelin` | 8080 | 浏览器打开 8080 |
| MySQL | `mysqld` | 3306 | `mysql -u hive -p` |

`jps` 能一次看到所有 Java 进程，是最快的体检方式——缺哪个进程就往哪层查。

## 推荐的排查顺序

1. `java -version` 与 `echo $JAVA_HOME` 是否一致（素材里路径与版本号就对不上）。
2. `jps` 看 NameNode / ResourceManager 是否都活着。
3. `hdfs dfs -ls /` 确认 HDFS 可用（要能自动退出，说明没进安全模式）。
4. `mysql -u hive -p` 确认元数据库可达。
5. `schematool -info` 确认 schema 已初始化。
6. `beeline` 连 10000，能 `show databases` 才算 Hive 通了。
7. 最后才开 Zeppelin——前端只是展示层，它出错通常是下面某一层没通。

## 关联

- 装机细节：[[2026-10-27-大数据安装]]、[[2026-10-27-CentOS7安装MySQL]]、[[2026-10-27-MySQL8安装]]
- 组件实讲：[[HDFS]]、[[YARN]]、[[MapReduce]]、[[Hive]]、[[Apache Zeppelin]]
- 概念：[[分布式系统]]、[[数据仓库]]、[[大数据4V]]
- 版本风险：[[版本生命周期与EOL]]
- 全库入口：[[_Index_Wiki]]
