---
type: concept
status: budding
created: 2026-10-27
updated: 2026-10-27
provenance: external
sources:
  - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/innodb-index-types.html
  - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/multiple-column-indexes.html
  - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/index-condition-pushdown-optimization.html
  - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/explain-output.html
  - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/verifying-index-usage.html
  - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/innodb-fulltext-index.html
tags:
  - mysql
---

B+ 树索引是 InnoDB 用来把「键值 → 行」的映射组织成多路平衡树的结构，它的设计目标不是减少比较次数，而是把一次点查的磁盘 IO 次数压到与表规模近似无关的常数（通常 3~4 层）。

> 📌 来源性质：本页无 raw 素材，依据外部资料编写，待有 raw 素材时再收编。

素材里的「索引分四类」（主键、唯一、普通、全文）只描述了「约束用途」这一维度，没有描述「数据结构」这一维度——而后者才解释得了为什么联合索引有最左前缀、为什么范围查询后列会失效、为什么索引不是越多越好。本页补的是后者。

## 为什么是 B+ 树

数据库的瓶颈在磁盘随机 IO，而磁盘的读写单位是页而不是字节（InnoDB 的页默认 16KB，`innodb_page_size`）。因此索引结构要优化的量是「访问了多少个页」，比较次数只有在全部命中内存时才是主成本。B+ 树正是围绕这一点设计的：

**高扇出带来低树高。** 一棵多路平衡树的每个内部节点存的是「键 + 子页指针」，一个 16KB 页能容纳几百到上千个这样的条目。扇出约 1000 时，三层就能覆盖十亿量级的行；反过来说，树高就是点查的最坏 IO 次数。这是「索引让查询变快」的准确表述：不是变聪明了，是少读了几次盘。

**与 B 树的分工不同。** B 树的内部节点也存数据行，同页能放的键就少了，于是扇出变小、树变高；B+ 树把数据全部下放到叶子、内部节点只留键和指针，换来更高扇出。更关键的是 B+ 树的叶子节点用链表串联，范围扫描可以沿叶子顺序读，而 B 树要在内部节点之间反复回溯。还有一点常被忽略：B+ 树所有查询路径长度相同，代价可预测，B 树的命中深度则取决于键落在哪一层。

**哈希的边界。** 哈希索引把等值查找做到 O(1)，但完全不保留顺序，因此不支持范围扫描（`BETWEEN`、`>`）、不支持排序、也不支持最左前缀的部分匹配。它只适合「纯等值」的内存型访问模式，这正是 InnoDB 只在特定场景用自适应哈希去加速 B+ 树、而不把主索引换成哈希的原因（自适应哈希的触发条件本页未核，待核）。

**红黑树的边界。** 红黑树、AVL 都是二叉结构，树高约 log₂N；千万行的 log₂ 约 24 层，每层一次随机 IO 就是 24 次，而 B+ 树三层就够。把「二分」换成「多路」是把 IO 次数从 log₂ 降到 log₁₀₀₀ 的唯一办法。

| 结构 | 等值 | 范围/排序 | 层数（千万行） | 磁盘友好度 | 典型问题 |
| --- | --- | --- | --- | --- | --- |
| B+ 树 | 好 | 好（叶子链表） | 3~4 | 最高（页内多键、顺序叶子） | 写入需维护平衡与页分裂 |
| B 树 | 好 | 一般（需回溯） | 略高 | 中（内部节点存数据，扇出小） | 同页容纳键少 |
| 哈希 | 极好 | 不支持 | 1 | 低（随机分布） | 无法范围扫描、无法排序 |
| 红黑树 | 好 | 中 | ~24（千万级） | 低（每层一次 IO） | 树高即 IO 次数 |

## 聚簇索引与二级索引

InnoDB 的表本身就是一棵 B+ 树，这棵树叫**聚簇索引**，叶子节点直接存放整行数据。选取规则是有优先级的：有主键就用主键；没有主键，用第一个「所有列都 NOT NULL 的唯一索引」；两者都没有，InnoDB 会建一个隐藏的 `GEN_CLUST_INDEX`，键是 6 字节单调递增的 row ID——这正是「没有主键的表按插入顺序物理存储」的机制来源。

**二级索引**的叶子节点不存整行，只存「该索引的列 + 主键值」。查询若只用到索引列，返回即可；若还要其他列，就得拿主键值回聚簇索引再查一次，这就是**回表**。两个直接推论：其一，主键越短，所有二级索引越小（因为每个二级索引条目都背负一份主键）；其二，主键的单调性影响写入——随机主键（如 UUID）会造成频繁的页分裂与随机写，官方对此的表述是短主键有利、并推荐单调递增。

这也解释了为什么「给二级索引加一个高区分度主键」的收益是全局的：它同时压低了三样东西——索引体积、回表成本、页分裂概率。

## 联合索引与最左前缀

官方给复合索引的定义是一个非常好用的心智模型：「A multiple-column index can be considered as a sorted array, the rows of which contain values that are created by concatenating the values of the indexed columns.」复合索引上限是 16 列。

既然是「拼接后排序」，那么只有从第一列开始连续的一段前缀才能被用来定位，因为只有那段前缀在拼接结果里是连续的。`INDEX name (last_name, first_name)` 可用于：`last_name='Jones'`、`last_name='Jones' AND first_name='John'`、`last_name='Jones' AND (first_name='John' OR first_name='Jon')`、`last_name='Jones' AND first_name>='M' AND first_name<'N'`；不能用于只有 `first_name='John'`，也不能用于 `last_name='Jones' OR first_name='John'`（OR 要求两侧都能独立走索引）。三列索引 `(col1,col2,col3)` 的可用前缀只有 `(col1)`、`(col1,col2)`、`(col1,col2,col3)`，`(col2)` 与 `(col2,col3)` 都不是最左前缀。

对照表（把「能否用上」与「用上几列」分开看）：

| 查询条件（索引 `(a,b,c)`） | 是否走索引 | 定位用到的列 | 说明 |
| --- | --- | --- | --- |
| `a=1` | 是 | a | 最短可用前缀 |
| `a=1 AND b=2` | 是 | a,b | 最左前缀 |
| `a=1 AND b=2 AND c=3` | 是 | a,b,c | 全列 |
| `a=1 AND c=3` | 是，但只用到 a | a | c 无法定位，退化为过滤（可被 ICP 下推） |
| `b=2` | 否 | — | 跳过前导列，无法组成最左前缀 |
| `a>1 AND b=2` | 是，但只用到 a | a | 范围条件之后的列不再能定位 |
| `a=1 AND b=2 ORDER BY c` | 是 | a,b | 借索引顺序避免额外排序（受范围列影响） |
| `a LIKE 'x%'` | 是 | a | 前缀匹配可走索引 |
| `a LIKE '%x'` | 否 | — | 前导模糊无前缀可用 |
| `DATE(a)='2026-10-27'` | 否 | — | 列参与函数 |
| `a=1 OR d=1`（d 无索引） | 否（全表） | — | OR 一侧不可索引时整条退化 |

两类「看不见的失效」必须单独点名：**列参与函数或表达式**（`WHERE DATE(a)=...`、`WHERE a+1=2`）会让索引列不再以裸列形式出现，优化器无法用二分定位；**隐式类型转换或长度不一致**同理。官方在 EXPLAIN 解读一章给过现成例证：`tt.ActualPC CHAR(10)` 与另一表 `EMPLOYID CHAR(15)` 做连接时，执行计划退化为 `type=ALL` 并附 `Range checked for each record (index map: 0x23)`，文档明确解释「MySQL can use indexes on columns more efficiently if they are declared as the same type and size」，把列改成 `VARCHAR(15)` 后计划改善。判断这类问题的判据很朴素：`EXPLAIN` 的 `type` 掉了、`key` 变 NULL、`rows` 接近全表，就先怀疑类型/函数/字符集不一致，而不是先怀疑索引选错。

## 覆盖索引与 EXPLAIN 的判据

**覆盖索引**指一次查询所需的全部列都能从某个索引里取到，于是不必回表。官方在 EXPLAIN 的 Extra 列里把它写成 `Using index`，原文定义是：「The column information is retrieved from the table using only information in the index tree without having to do an additional seek to read the actual row. This strategy can be used when the query uses only columns that are part of a single index.」注意最后半句——必须落在**同一个**索引里，跨索引拆列不算覆盖。

`type` 与 `Using index` 的组合最容易误读，两者不同层：`type=index` 说的是「扫描方式是索引顺序扫描」，`Using index` 说的是「没有任何回表」。官方在 join type 的 `index` 条目下专门区分了两种情况：若索引是覆盖索引且满足全部数据需求，「only the index tree is scanned. In this case, the `Extra` column says `Using index`」；反之若按索引顺序逐行回表，则 Extra 里不会出现 `Using index`。

| EXPLAIN 线索 | 含义 | 优化方向 |
| --- | --- | --- |
| `Using index` | 覆盖索引，零回表 | 已是最优读法；注意别为覆盖而过宽建索引 |
| `Using index condition` | 索引条件下推（ICP），索引列参与过滤但仍需回表 | 考虑让查询只取索引列以升级为覆盖 |
| 两者都不出现 + `type=ALL` | 全表扫描 | 检查是否有可用最左前缀、是否列被函数包裹 |
| `key_len` 偏小 | 联合索引只用上前几列 | 调整列顺序或补齐条件 |
| `rows` 很大而 `filtered` 很低 | 扫描多、过滤狠 | 说明区分度不足或条件无法定位 |

**索引条件下推（ICP）** 是把「WHERE 里只用索引列就能判定的部分」交给存储引擎先测、通过了才读整行，从而减少回表次数与 server↔engine 交互。它只在 range/ref/eq_ref/ref_or_null 且需要读整行时生效，而且 **InnoDB 只在二级索引上启用**（聚簇索引的整行已经在 buffer 里，省不了 IO）。因此触发 ICP 时 Extra 显示 `Using index condition`，同时**不会**显示 `Using index`——因为逻辑上仍要读整行。它的开关是 `optimizer_switch` 里的 `index_condition_pushdown`，默认开启；子查询、存储函数与触发器产生条件不支持下推。

`key_len` 的用途是反推「实际用了几列」：它由索引列的定义长度累加而来，可空列还会比 NOT NULL 多 1 字节。把它与 `ref` 对照，就能知道「说走联合索引」到底走了几列——这是排查「索引建了但只用到第一列」的最快手段。

## 区分度、基数与列的排序

**区分度**是「不同值的个数 / 行数」的比值，唯一索引为 1。它决定优化器愿不愿意用索引：若某列只有两三种值，通过它定位到的行可能占全表一半，优化器会判断「回表的随机 IO 比顺序全表扫更贵」，从而放弃索引。这解释了「给性别列建索引为什么没用」——不是语法问题，是成本估算问题。

**基数**是索引里不同键的个数，由统计信息维护；EXPLAIN 的 `rows` 在 InnoDB 下本身就是估算值（官方原文：「For InnoDB tables, this number is an estimate, and may not always be exact」），`filtered` 是百分比，`rows × filtered` 才是与下一张表连接的行数估计。统计不准（大批量写入后未更新）会导致选错执行计划，这也是 `ANALYZE TABLE` 在调优里有一席之地的原因。

联合索引的列序是「可用性」与「选择性」的折中：能保证等值条件覆盖的前导列优先；高区分度列放前面能让索引更快收敛；**范围条件列应尽量靠后**，因为它一旦出现，其后的列就失去定位能力（只能靠过滤或 ICP）。没有万能顺序，判据是「实际高频查询的条件形态」。

## 索引不是越多越好

三条成本线同时上升：

**写放大。** 每次 INSERT/UPDATE/DELETE 都要维护所有受影响的索引；二级索引的维护还意味着额外的键值写入与页分裂。写入密集表上，索引数量与写延迟是近似线性关系。

**缓存挤压。** 索引页同样占 buffer pool 的页。多一个索引，就少一份缓存留给数据页与其他索引，命中率下降会转化为更多真实 IO。

**优化器成本。** 候选路径越多，成本估算的误差面越大，优化耗时上升，且更容易选到「估算便宜、实际更慢」的计划。索引过多时还会出现「两个几乎等价的索引，优化器随机挑一个」。

工程判据是「查询收益 × 频率」对「写入成本 × 频率」：读多写少的报表表可以宽一些，高频写入的业务表应把索引收到最小集合，把多列需求合并进一个联合索引（一个 `(a,b,c)` 往往能顶替 `(a)`、`(a,b)`）。另外，加索引本身是 DDL，8.0 的 online DDL 支持在部分场景下并发 DML，但具体算法（INSTANT/INPLACE/COPY）随操作与版本不同，本页未逐项核到原文（待核）。

## 全文索引：一条必须注明版本分界的旧结论

素材里给出的全文索引示例写着 `ENGINE=MYISAM`，并暗示「全文索引要用 MyISAM」。这个结论在 5.5 及更早成立——当时只有 MyISAM 提供 FULLTEXT；InnoDB 从 5.6.4 起才支持 FULLTEXT。**「5.6.4」这个分界号本次只在本库既有序号摘要里见到一条来源，官方 5.6.4 更新页在镜像上返回 404，故标待核**；但「InnoDB 现在支持全文索引」有独立证据：5.7 与 8.0 手册都设有专章「InnoDB Full-Text Indexes」，且写明索引可建于 `CHAR`、`VARCHAR`、`TEXT` 列、用 `MATCH() ... AGAINST` 查询。

现代默认应当是 InnoDB，判据也不只是「版本够了」：MyISAM 没有事务、只有表级锁、崩溃恢复能力弱，把需要与业务数据联查的全文检索留在 MyISAM 上，等于为一个索引把整张表降级。InnoDB 的全文索引机制要点是：采用 inverted index（倒排索引，词 → 文档列表，并为邻近搜索保存字节偏移的位置信息），建索引时会附带 6 张 `fts_*_index_1..6` 辅助表以及若干公共表；而「索引更新在事务提交时才处理」意味着**全文搜索只能看到已提交的数据**——事务内刚插入、尚未提交的行搜不到。删除也不立即回收：只在 `FTS_*_DELETED` 里记下文档 ID，要配合 `OPTIMIZE TABLE` 与 `innodb_optimize_fulltext_only=ON` 才真正清理。缺省情况下 InnoDB 会自动加隐藏列 `FTS_DOC_ID`（伴随 warning 124），自定义该列时必须是 `BIGINT UNSIGNED NOT NULL`，且其值递增间隔有上限：5.7.13 之前为 10000，5.7.13 及以后为 65535。

## 版本差异（5.7 vs 8.0）

索引相关的默认值变化比语法变化更容易在升级时出问题：

| 项目 | 5.7 | 8.0 | 对索引的影响 |
| --- | --- | --- | --- |
| 默认字符集 / 排序 | `latin1` / `latin1_swedish_ci` | `utf8mb4` / `utf8mb4_0900_ai_ci` | 字符列的最大键长、比较与排序规则变化，索引前缀长度上限随之改变 |
| 默认认证插件 | `mysql_native_password` | `caching_sha2_password` | 与索引无关，但升级脚本里常与 `GRANT ... IDENTIFIED BY` 一起失败 |
| 建用户语法 | `GRANT ... IDENTIFIED BY` 可用（已弃用） | 8.0.11 移除 | 权限脚本必须改写；与索引无直接关系，但升级清单里同批出现 |
| `innodb_autoinc_lock_mode` | `1` | `2` | 批量插入的页填充与主键连续性变化 |

还有一处容易误伤：**「全文索引只能用 MyISAM」是 5.5 及更早的限制，不是 8.0 的规则**，把它当规则继续传播会导致新表被建成非事务引擎。

## 判据速查

| 场景 | 建议 | 判据 |
| --- | --- | --- |
| 按主键或唯一键点查 | 聚簇索引/唯一索引，`type` 应为 `const`/`eq_ref` | `eq_ref` 要求索引是 PRIMARY KEY 或 UNIQUE 且「all parts of an index are used」 |
| 多列等值 + 排序 | 联合索引，等值列在前、排序列紧随其后的连续前缀 | `key_len` 应覆盖到排序列 |
| 只取少数字段的高频查询 | 设计成覆盖索引，目标 Extra 出现 `Using index` | 列必须落在同一索引内 |
| 范围条件 + 后列过滤 | 后列靠 ICP（`Using index condition`）而非定位 | InnoDB 仅二级索引启用 ICP |
| 低区分度列单独查询 | 不建索引；或作为联合索引的后列 | 优化器会因回表成本放弃索引 |
| 写入密集表 | 收敛索引数量，合并为联合索引 | 每次写入都需维护全部索引 |
| 全文检索 | InnoDB + `FULLTEXT`，注意提交可见性与 `OPTIMIZE` | 5.6.4 分界待核，但 5.7/8.0 手册已有 InnoDB 专章 |

## 关联

- [[MySQL]] —— 上位条目。
- [[2026-10-27-MySQL事务索引与备份]] —— 素材摘要，其中「索引分四类」「全文索引用 MyISAM」是只有结论的部分，本页补数据结构与版本分界。
- [[事务隔离级别]] —— 间隙锁与 next-key 锁的锁定范围由索引结构决定，无索引则无精确范围锁。
- [[DQL查询]] —— 最左前缀与 EXPLAIN 判据都服务于查询语句的写法。
- [[DML数据管理]] —— 写放大、页分裂与主键单调性属于写入侧代价。
- [[binlog与增量恢复]] —— 索引页数量与写入量会影响 binlog 体积与恢复时长。

## 来源

均取自 Oracle 官方托管的 MySQL 8.0 手册镜像 `docs.oracle.com/cd/E17952_01/mysql-8.0-en/`（`dev.mysql.com` 同页面对本机抓取返回 HTTP 403），另用 5.7 镜像 `mysql-5.7-en/` 交叉核对全文索引。访问日期均为 2026-10-27。

1. Section 17.6.2.1「Clustered and Secondary Indexes」 —— `https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/innodb-index-types.html`
2. Section 10.3.6「Multiple-Column Indexes」 —— `https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/multiple-column-indexes.html`
3. Section 10.2.1.6「Index Condition Pushdown Optimization」 —— `https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/index-condition-pushdown-optimization.html`
4. Section 10.8.2「EXPLAIN Output Format」（`type`/`key_len`/`rows`/`Using index`/`Using index condition`、隐式类型与长度不一致的官方例证） —— `https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/explain-output.html`
5. Section 10.3.7「Verifying Index Usage」 —— `https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/verifying-index-usage.html`
6. Section 17.6.2.4「InnoDB Full-Text Indexes」（8.0） —— `https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/innodb-fulltext-index.html`
7. Section 14.6.2.4「InnoDB Full-Text Indexes」（5.7，交叉核对） —— `https://docs.oracle.com/cd/E17952_01/mysql-5.7-en/innodb-fulltext-index.html`
8. Section 3.5「Changes in MySQL 8.0」（默认字符集、认证插件、移除 `GRANT ... IDENTIFIED BY`） —— `https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/upgrading-from-previous-series.html`

待核三项：InnoDB 支持 FULLTEXT 的起始版本号「5.6.4」（官方 5.6 手册页在镜像上 404，仅有本库既有序号摘要一条来源）；自适应哈希索引的触发条件；8.0 online DDL 对各类索引操作的算法（INSTANT/INPLACE/COPY）逐项对应关系。
