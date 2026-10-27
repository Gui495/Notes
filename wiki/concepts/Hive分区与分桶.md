---
type: concept
status: budding
created: 2026-10-27
updated: 2026-10-27
provenance: external
sources:
  - https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL
  - https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties
  - https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL+BucketedTables
  - https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Sampling
  - https://cwiki.apache.org/confluence/display/Hive/LanguageManual+JoinOptimization
  - https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/HdfsDesign.html
tags:
  - bigdata
---

Hive 的分区（partition）与分桶（bucket）是同一件事的两个粒度：分区把「列的取值」映射成 HDFS 目录，让查询能整目录跳过；分桶把「列的哈希」映射成目录内的固定数量文件，让 join 能整文件归并。

> 📌 来源性质：本页无 raw 素材，依据外部资料编写，待有 raw 素材时再收编。

## 两种切分的物理形态

Hive 官方 DDL 文档对分区的定义是：用 `PARTITIONED BY` 建表后，「a separate data directory is created for each distinct value combination in the partition columns」——即每个不同的分区值组合对应一个独立数据目录。所以 `PARTITIONED BY (dt STRING, country STRING)` 落到 HDFS 上就是 `.../dt=2026-10-27/country=us/` 这样的路径层级。

分桶则进一步在目录内部切文件：`CLUSTERED BY (col) INTO n BUCKETS` 把同一分区内的行按 `hash_function(col) mod n` 分到 n 个文件里。

```sql
CREATE TABLE access_log (
  uid BIGINT, url STRING, ts BIGINT
)
PARTITIONED BY (dt STRING, country STRING)
CLUSTERED BY (uid) INTO 32 BUCKETS;
```

一句话记住分工：**分区决定「去哪个目录找」，分桶决定「目录里读哪几个文件」**。

## 分区：把谓词变成路径

### 静态分区与动态分区

静态分区在写入时把分区值写死，编译器直接知道要写哪个目录：

```sql
INSERT OVERWRITE TABLE access_log PARTITION (dt='2026-10-27', country='cn')
SELECT uid, url, ts FROM ods_log WHERE dt='2026-10-27' AND country='cn';
```

动态分区让分区值来自查询结果的最后一列，由运行时决定写多少个目录：

```sql
INSERT OVERWRITE TABLE access_log PARTITION (dt, country)
SELECT uid, url, ts, dt, country FROM ods_log;   -- 分区列必须放在 SELECT 末尾
```

动态分区默认是关的（0.9.0 之前 `hive.exec.dynamic.partition` 默认 `false`，0.9.0 起默认 `true`），而且默认的 `strict` 模式要求语句里至少保留一个静态分区列，防止一条语句误写全表：

| 配置项 | 默认值 | 机制含义 |
|---|---|---|
| `hive.exec.dynamic.partition` | 0.9.0 前 `false`，0.9.0 起 `true` | 总开关 |
| `hive.exec.dynamic.partition.mode` | `strict` | `strict` 要求至少一个静态分区列；`nonstrict` 放开 |
| `hive.exec.max.dynamic.partitions` | 1000 | 单条语句允许创建的最大分区数 |
| `hive.exec.max.dynamic.partitions.pernode` | 100 | 每个 mapper/reducer 允许创建的最大分区数 |
| `hive.exec.max.created.files` | 100000 | 单个作业允许创建的最大文件数 |
| `hive.exec.default.partition.name` | `__HIVE_DEFAULT_PARTITION__` | 分区列为 NULL 时落到的兜底分区 |

**判据**：动态分区适合「按天回刷历史数据」这类一次性产出大量分区的场景；如果源数据本身是流式小批量写入，动态分区会把文件数放大到「分区数 × 任务数」，应改用静态分区 + 定时合并。

### 分区裁剪为什么能省扫描

裁剪发生在编译期，不发生在读数据的时候。流程是：解析 SQL → 从 Metastore 取该表的分区列表 → 用 WHERE 里的分区谓词逐分区求值 → 只保留求值为真的分区 → 只有这些分区的目录参与生成输入分片。因为分区目录本身是可枚举的元数据，Hive 不需要打开任何数据文件就能判断「这个目录不可能命中」。

**判据**：看 `EXPLAIN` 输出里 `TableScan` 的 `partition` 字段，或对比 `Map Input Records` 与全表行数。如果谓词命中的分区数与总分区数相同，说明裁剪没生效。

**易错点**（都是把分区列包进函数或做隐式转换，使谓词无法在分区枚举阶段求值）：

- `WHERE from_unixtime(unix_timestamp(dt)) >= '2026-01-01'`：分区列被函数包裹，退化为全分区扫描。
- `WHERE dt > 20260101`（`dt` 是 STRING）：Hive 会把两侧都转成 double 再比，分区枚举阶段失去可比性。
- 分区列做 `LIKE '%...%'` 或 `OR` 混入非分区列条件：只有能拆成独立合取项的部分才参与裁剪。

## 分桶：把 join 变成归并

### 桶的编号规则

官方的分桶文档给出：桶号等于 `hash_function(bucketing_column) mod num_buckets`（对 `INT` 型列直接用该整数值参与取模，官方示例用 `0x7FFFFFFF` 掩码保证非负）。桶文件按 `000000_0`、`000001_0` 之类命名，编号从 0 开始；而 `TABLESAMPLE` 语法里的桶编号从 1 开始，这是两处编号约定的差异，容易数错一桶。

```sql
-- 取 32 个桶中的第 3 桶
SELECT * FROM access_log TABLESAMPLE (BUCKET 3 OUT OF 32 ON uid);
-- 每 4 个桶取 1 个：OUT OF 8 表示取 32/8 = 4 个桶
SELECT * FROM access_log TABLESAMPLE (BUCKET 3 OUT OF 8 ON uid);
```

`ON colname` 必须是分桶列或分区列，省略时默认用分桶列；也可以写 `ON rand()` 做随机抽样，但那时就不能按桶对齐了。

### SMB join 的意义

Sort-Merge-Bucket join 的前提是：两张表按**同一个键**分桶、桶数相同或成整数倍、且桶内已按该键排序（`SORTED BY`）。满足时 Hive 不再做全表 shuffle，而是让第 i 个 map 任务只读两边的第 i 个桶，「the join boils down to just merging the already sorted tables」——每个桶内部是两条有序流，归并即可，代价从「全量 shuffle + 建哈希表」降到「顺序读 + 归并」。

相关开关（三者需同时开）：

```properties
hive.auto.convert.sortmerge.join=true
hive.optimize.bucketmapjoin=true
hive.optimize.bucketmapjoin.sortedmerge=true
```

**判据**：只有当 join 键与分桶键一致、且两表桶数成倍数关系时 SMB 才成立；任一条不满足，优化器会回退成普通 join，此时桶只带来了「更均匀的输入分片」，没有省掉 shuffle。

**版本差异**：`hive.enforce.bucketing`（写入时强制按桶数并发写）在 Hive 0.6–1.x 需要手动设为 `true`，Hive 2.x 起文档明确标注不再需要；老版本教程里仍然保留这一行。

## 分区与分桶的分工对照

| 维度 | 分区 | 分桶 |
|---|---|---|
| 物理形态 | HDFS 目录（一个值组合一个目录） | 目录内的固定数量文件 |
| 键的选择 | 低基数、常出现在 WHERE 里（`dt`、`country`） | 高基数、常出现在 JOIN/去重键上（`uid`、`order_id`） |
| 命中方式 | 谓词求值 → 整目录跳过 | 取模定位 → 整文件跳过，或整桶归并 |
| 元数据位置 | Metastore 的分区表（键值对） | 表属性（桶列 + 桶数） |
| 主要收益 | 减少扫描的数据量 | 减少 shuffle 的数据量 |
| 写入代价 | 每个分区至少一个文件 | 每次写入要按桶数并发 |
| 失效条件 | 谓词被函数包裹/类型不匹配 | join 键与分桶键不一致 |

一句话：**分区省 I/O，分桶省网络**。真正的宽表往往两者叠加——按天分区、按用户 ID 分桶。

## 过度分区的代价

分区不是越细越好，因为分区的最小存储单元是「目录 + 至少一个文件」，而 NameNode 的内存占用与文件/块的数量成正比。HDFS 架构文档的原话是「The NameNode keeps an image of the entire file system namespace and file Blockmap in memory」——命名空间对象常驻堆内存，文件数就是内存开销。

算一笔：按 `dt + hour + country` 三级分区，假设每天 24 小时 × 200 个国家 = 4800 个分区，一年就是约 175 万个目录，若每个目录只放一个几 MB 的小文件，NameNode 上就是百万级的 inode 对象；同时每个 Hive 任务都要为这些小文件开一个 map，任务启动开销会盖过数据本身。

**判据**（工程经验，非官方硬指标，标注为经验值）：

- 单个分区文件目标 ≥ 一个 HDFS 块（默认 128 MB），至少不要长期低于几十 MB。
- 分区总数控制在数千到数万量级；超过就先聚合到更粗的粒度（月分区 + 桶），再靠桶和排序处理细粒度。
- 已经产生小文件时，用 `INSERT OVERWRITE` 重写 + `DISTRIBUTE BY`/`CLUSTER BY` 合并，或对 ORC 表用 `ALTER TABLE ... CONCATENATE`。

## 元数据不一致：MSCK REPAIR TABLE

分区是「Metastore 里的一条记录」+「HDFS 上的一个目录」，两者可以脱节。用 `hadoop fs -put` 或 `hadoop distcp` 直接把目录塞进表路径下，Metastore 不会知道，`SELECT` 就查不到这些数据。官方文档对此的描述是：直接往 HDFS 增删分区后，Metastore 不会感知，除非对每个分区手动执行 `ADD/DROP PARTITION`。

```sql
MSCK REPAIR TABLE access_log;                    -- 默认 ADD PARTITIONS，只补不存在的
MSCK REPAIR TABLE access_log SYNC PARTITIONS;    -- 等价于 ADD + DROP，双向对齐
MSCK TABLE access_log;                           -- 不带 REPAIR：只报告不一致，不修
```

**易错点**：

- `MSCK REPAIR` 只认 `key=value` 结构的目录名，非等值路径（如 `dt=2026-10-27/part-00000` 之外的自定义布局）不会被识别。
- Hive 1.3 起遇到分区值里含非法字符会直接抛异常，需用 `hive.msck.path.validation=skip`（跳过）或 `ignore`（照建，行为同旧版）处理。
- 分区数量巨大时会 OOM，用 `hive.msck.repair.batch.size` 分批（默认 0 表示一次性全做）。
- Hive 4.0 起可以在表属性里设 `discover.partitions=true`，由 HMS 后台线程周期性执行 MSCK（默认 300s），不必再手动跑——这是版本差异，不是默认行为。

## 易错点清单

- 把桶编号的 0 起点（文件 `000000_0`）与 `TABLESAMPLE` 的 1 起点混用，抽错桶。
- 动态分区一次刷出几万个分区，作业最后卡在「生成小文件」而不是计算。
- `strict` 模式下动态分区语句报 `Dynamic partition strict mode requires at least one static partition column`，不是权限问题。
- 分区列与数据列同名导致 `SELECT *` 多出一列错位；分区列不是表定义里的普通列。
- 认为「分桶一定加速 join」——只有 SMB 的三个开关都开、且键与桶数对齐时才成立。

## 关联

- [[Hive]]：本页是 Hive 建模范式的展开，语法与配置都以 Hive 为准。
- [[HDFS]]：分区目录与桶文件最终落在 HDFS 的命名空间上，NameNode 内存约束是分区粒度的上限来源。
- [[数据仓库]]：分区对应「时间维度切片」，分桶对应「事实表的分布键」，两者是数仓分层里的物理设计手段。
- [[MapReduce]]：裁剪减少的是 map 输入分片数，分桶减少的是 shuffle 阶段的网络与排序开销。
- [[YARN]]：动态分区写入时的并发度由 YARN 容器数决定，桶数超过可用容器数会导致写入排队。
- [[2026-10-27-Hive]]：本机安装记录，本页补的是它缺失的建模手段。

## 来源

- Apache Hive, LanguageManual DDL（`PARTITIONED BY`、`CLUSTERED BY`、`MSCK REPAIR TABLE`、动态分区入口）：https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL ，访问日期 2026-10-27。
- Apache Hive, Configuration Properties（`hive.exec.dynamic.partition*`、`hive.exec.max.*`、`hive.msck.*` 默认值）：https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties ，访问日期 2026-10-27。
- Apache Hive, BucketedTables（桶号取模规则、`hive.enforce.bucketing` 版本差异）：https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL+BucketedTables ，访问日期 2026-10-27。
- Apache Hive, Sampling（`TABLESAMPLE (BUCKET x OUT OF y)` 语义与桶编号起点）：https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Sampling ，访问日期 2026-10-27。
- Apache Hive, JoinOptimization（SMB join 的三个开关与前提）：https://cwiki.apache.org/confluence/display/Hive/LanguageManual+JoinOptimization ，访问日期 2026-10-27。
- Apache Hadoop, HDFS Architecture（NameNode 将命名空间与 Blockmap 常驻内存）：https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/HdfsDesign.html ，访问日期 2026-10-27。

> 说明：`cwiki.apache.org` 的旧链接现会 301 跳转到 `confluence.apache.org` 的新页面地址，内容一致；上表保留旧地址以便与历史笔记对照。分区粒度的具体阈值（如「单文件 ≥ 128 MB」）属于工程经验而非官方规定，标为经验值。
