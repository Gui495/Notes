---
type: concept
status: budding
created: 2026-10-27
updated: 2026-10-27
provenance: external
sources:
  - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/binary-log.html
  - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/innodb-redo-log.html
  - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/innodb-undo-logs.html
  - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/point-in-time-recovery.html
  - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/point-in-time-recovery-binlog.html
  - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/mysqldump.html
  - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/mysqlbinlog.html
  - https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/replication-options-gtids.html
tags:
  - mysql
---

binlog（二进制日志）是 MySQL 按提交顺序记录「改变数据或可能改变数据」的事件流，它与一次全量备份配合，就能把数据库恢复到备份之后的任意时间点——这套流程叫增量恢复（PITR，point-in-time recovery）。

> 📌 来源性质：本页无 raw 素材，依据外部资料编写，待有 raw 素材时再收编。

素材里的「用 mysqldump 备份」只到「把数据导出成 SQL」这一步，没有回答更关键的问题：备份之后到故障之间那几小时的数据怎么办、怎么证明恢复出来的库是对的、误删了一条数据怎么只回滚那一条。这三个问题都由 binlog 回答。

## 三种日志的分工

初学者最容易把三种日志混成一件事，实际它们处在不同层，解决不同的问题：

| 日志 | 归属 | 记录什么 | 解决什么 | 崩溃后 |
| --- | --- | --- | --- | --- |
| redo log | InnoDB 引擎 | 物理页的修改请求，以单调递增的 LSN 编号 | 崩溃恢复：把「已提交但数据页还没写完」的修改重放回来 | 启动时从最新 checkpoint LSN 开始重放 |
| undo log | InnoDB 引擎 | 如何撤销对聚簇索引记录的最近一次更改 | 事务回滚；同时为一致性读提供旧版本 | 未提交事务被回滚 |
| binlog | MySQL Server 层 | 逻辑事件：语句（STATEMENT）或行变更（ROW） | 归档、复制、恢复到任意时间点 | 只保留完整事件，尾部半截事件被丢弃 |

官方对 redo 的定义可以当作判据：「The redo log is a disk-based data structure used during crash recovery to correct data written by incomplete transactions.」意外宕机后，那些「did not finish updating data files」的修改会在初始化阶段、接受连接之前被自动重放。它的关键局限是**循环覆盖**：checkpoint 推进时最旧的数据被截断，所以 redo 不能当归档，也不足以支撑「恢复到上周三 10:00」。undo log 则承担 MVCC 的历史版本职责——「if another transaction needs to see the original data as part of a consistent read, the unmodified data is retrieved from the undo log records」，这条把 undo 与隔离级别绑在一起，详见 [[事务隔离级别]]。

三种日志还有一个必须点明的耦合点：InnoDB 与 binlog 的一致性靠两阶段提交（XA）保证，8.0 中该机制对 InnoDB 始终启用。若没有它，崩溃点恰好落在「binlog 已写、数据未提交」之间，就会出现「副本/恢复重放了不存在的事务」或「已提交事务在 binlog 中缺失」——前者造成重复数据，后者造成永久丢失。这一点解释了为什么「先写 binlog 再 COMMIT」的顺序在源码级是硬要求。

## binlog 的记录机制

binlog 写的是 **events**，内容涵盖库结构变更与数据变更，也包括「可能造成变更的语句」（例如匹配 0 行的 DELETE）——但在行格式下这个描述不成立，行格式只记真正的行变化。它不记录 SELECT、SHOW 这类纯读语句。官方给的两大用途就是复制与「还原备份后重放 events 追平」，本章余下内容都围绕后者。

写时机是理解重放安全性的基础：事件在「语句或事务完成之后、释放锁与提交之前」写入，因此官方表述为「logged in commit order」。同时「The binary log is resilient to unexpected halts. Only complete events or transactions are logged or read back.」——一个事务要么整块出现在 binlog 里，要么完全不出现，绝不会只写一半；而且事务不会被拆分到两个文件（`max_binlog_size` 只是轮转触发条件，遇到大事务会等它写完）。这决定了「重放到某个 binlog 文件末尾」是安全的边界。

几个与运维直接相关的机制事实：默认文件基名是 `binlog`（可用 `--log-bin` 改），索引文件记录现有文件清单（`--log-bin-index`）；8.0 中 binlog **默认开启**（`log_bin=ON`），例外是用 `--initialize`/`--initialize-insecure` 手工初始化数据目录时默认关闭，关闭用 `--skip-log-bin`/`--disable-log-bin`。事务缓存超过 `binlog_cache_size` 会落到磁盘临时文件（`Binlog_cache_disk_use` 计数上升就是信号），`max_binlog_cache_size` 默认 4GB 且同时是上限。写入失败时 `binlog_error_action` 的行为默认是 `ABORT_SERVER`（另一值是 `IGNORE_ERROR`，会让服务器继续跑但 binlog 断裂）。`sync_binlog=1` 在 8.0 是默认值，意味着每次提交都 fsync binlog——最安全也最慢，这是「为什么 binlog 开启后写入变慢」的主因之一。

还有一组随事件一起写入、并被副本采用的会话变量值得记住：`sql_mode`、`foreign_key_checks`、`unique_checks`、`character_set_client`、`collation_connection`、`collation_database`、`collation_server`、`sql_auto_is_null`。它们的存在说明「重放一条 SQL」在 STATEMENT 格式下并不等价于「执行同一段文本」——上下文不同，结果就可能不同，这也是后文推荐 ROW 的底层理由。

## 三种格式的取舍

| 格式 | 记录内容 | 体积 | 确定性 | 主要风险 |
| --- | --- | --- | --- | --- |
| STATEMENT | SQL 文本 + 部分上下文 | 小 | 低 | 非确定性函数、依赖扫描顺序、快照语义差异导致主从不一致 |
| ROW | 每一行的变更前后值 | 大（与行数成正比） | 高 | 批量更新日志膨胀；需要 `binlog_row_image` 控制记录哪些列 |
| MIXED | 默认 STATEMENT，遇不确定语句自动转 ROW | 中 | 中 | 判定规则不覆盖全部上下文依赖 |

STATEMENT 的失效场景可以归纳成一条判据：**只要「副本重放时的执行环境」可能与源库不同，STATEMENT 就不安全**。典型的有 `NOW()`、`RAND()`、`UUID()`、`USER()` 这类非确定性函数；`UPDATE ... LIMIT n`（无 `ORDER BY`）这类依赖扫描顺序的语句；以及按「当前读到的行集」决定改哪些行的 `UPDATE`/`DELETE`。此时即便主从数据最初一致，重放也会改错行。

隔离级别在这里不是旁观者，而是硬约束：官方规定 **READ COMMITTED 下只支持 row-based 二进制日志**，若 `binlog_format=MIXED` 却在 RC 下运行，服务器会自动改用 row-based 记录。原因正是 RC 每条 SELECT 都取新快照、不锁间隙，源库与副本命中的行集可能不同，STATEMENT 无从保证一致。8.0 的 `binlog_format` 默认值本次未取到原文（待核，5.7.7 起业界普遍为 ROW），但无论默认如何，判断顺序都应是：**能接受日志体积就用 ROW；用 RC 就必须是 ROW；只有在确认语句确定性、且能用 MIXED 兜底时才考虑 STATEMENT**。

## GTID：让「重放到哪」不再靠猜

GTID（全局事务标识符）给每个事务一个全局唯一编号（`server_uuid:事务号`），于是副本不再需要记「文件 + 偏移」，而是记「哪些 GTID 已执行」。这对恢复同样重要：从备份重建实例时，只要知道备份点对应的 GTID 集合，就能精确表达「从哪之后开始重放」。

配置项的关键约束是**不能一步到位**。`gtid_mode` 默认 `OFF`，合法值只有 `OFF`、`OFF_PERMISSIVE`、`ON_PERMISSIVE`、`ON`，切换时必须逐级移动（例如 `OFF_PERMISSIVE` 只能退回 `OFF` 或前进到 `ON_PERMISSIVE`）；并且必须先把 `enforce_gtid_consistency` 设为 `ON` 才允许 `gtid_mode=ON`——它会禁止那些无法用单一 GTID 表示的事务，包括事务内 `CREATE/DROP TEMPORARY TABLE`、同一语句同时修改事务性与非事务性表，以及 8.0.21 之前的 `CREATE TABLE ... SELECT`。这两条是「开 GTID 要先改业务写法」的真正原因，不是配置洁癖。

`gtid_executed` 是已执行的全部 GTID 集合（等同 `SHOW MASTER STATUS` 里的 `Executed_Gtid_Set`）；`gtid_purged` 是它的子集，指「已提交但已经不在任何 binlog 文件里」的部分。`RESET MASTER` 会把两者清空。这个语义有一处恢复时的高频陷阱：`mysqldump` 默认会在输出里写 `SET @@GLOBAL.GTID_PURGED`（由 `--set-gtid-purged` 控制），把备份点的 GTID 登记进新实例；此后重放 binlog 时，凡「GTID 已登记」的事务会被**静默跳过**。它既是防重复的保护，也可能在你「以为重放了、其实被跳过」时造成数据缺口。要绕开这一行为，可用 `mysqlbinlog --skip-gtids` 让输出不带原 GTID，从而按新事务重新生成编号。

## 全量 + 增量恢复的标准流程

标准流程五步，每一步都有对应的判据，缺一步都会让后面的步骤变得不可验证：

**第一步，全量基线。** 官方给出的在线备份方式是 `mysqldump --single-transaction`，它在不施加任何表锁的前提下导出一致快照（`--single-transaction` 的选项说明是「Issue a BEGIN SQL statement before dumping data from server」，也就是在 dump 前开启一个事务，让 InnoDB 用一致性读给出快照）。两个必须配合的参数：`--flush-logs`（「Flush MySQL server log files before starting dump」）在导出前轮转 binlog，使「备份结束之后」的所有 binlog 都只含备份点之后的事务，从而得到一个干净的起点；`--source-data`（旧名 `--master-data`，8.0 已标记弃用）把当时的 binlog 文件名与位置写进 dump 的输出里，省掉手工记位置这一步。权限上要注意：**不使用 `--single-transaction` 时需要 `LOCK TABLES` 权限**，8.0.21 起未加 `--no-tablespaces` 还需要 `PROCESS`；若 `gtid_mode=ON` 且 `gtid_purged` 的设置为 `ON`/`AUTO`，用 `--single-transaction` 还需 `RELOAD` 或 `FLUSH_TABLES`。还有一个判据必须记住：`--single-transaction` 的保护只对事务引擎成立，MyISAM 表在导出期间仍可能被改，备份不可信。

**第二步，记录时间点。** 用 `SHOW BINARY LOGS;` 列出文件，用 `SHOW MASTER STATUS;` 看当前文件与位置；dump 头部的注释里也会带上这些信息。把它与「事故发生的墙钟时间」一起记下来，两边都要，因为重放既可按时间、也可按位置截断。

**第三步，恢复全量。** 把 dump 灌回实例。注意此时全量恢复本身也是写入，会进 binlog——通常用 `--disable-log-bin`（mysqlbinlog 的对应选项会插入 `SET sql_log_bin = 0`）或手工 `SET sql_log_bin=OFF` 避免污染，否则后续重放的边界会被自己的恢复动作搅乱。

**第四步，重放增量。** 官方给的两种用法：直接把 mysqlbinlog 的输出管道给 mysql（`mysqlbinlog binlog_files | mysql -u root -p`），或先落盘再执行（`mysqlbinlog binlog_files > tmpfile` 然后 `mysql -u root -p < tmpfile`）。多个文件要一次交给 mysqlbinlog（`mysqlbinlog binlog.000001 binlog.000002 | mysql`），这样 GTID 与位点的连续性由工具自己维持。截断用时间或位置：`--start-datetime` 的语义是「从第一个时间戳大于等于该参数的事件开始读」，`--stop-datetime` 是「读到第一个时间戳大于等于该参数的事件就停」——也就是区间取 `[start, stop)`，想停在某条 DELETE 之前，把 `stop-datetime` 设成那条语句的时间戳即可。两个额外约束：binlog 若开启加密（8.0.14+，`binlog_encryption=ON`），mysqlbinlog 不能直读，须加 `--read-from-remote-server` 从服务器读取；binlog 内容含 `\0` 时 mysql 客户端需 `--binary-mode`。`--base64-output` 保持默认 `AUTO`——官方明确「Automatic BINLOG display is the only safe behavior if you intend to use the output of mysqlbinlog to re-execute binary log file contents」。

**第五步，验证。** 行数、校验和、关键业务表对比、外键与自增序列连续性。没有验证的恢复不能算恢复。

落盘再执行比直接管道更值得推荐，理由很实在：管道是一次性通过，你看不到将要执行什么；落盘后可以先 grep 出 DROP/DELETE/UPDATE 关键语句、确认边界，再决定是否执行。这一步的成本是几分钟，收益是避免把误删动作再执行一遍。

## 时间点定位的判据

| 场景 | 该用什么 | 理由 |
| --- | --- | --- |
| 知道「几点几分出的错」 | `--stop-datetime` | 语义直白，区间是 `[start, stop)`，停在第一条 >= 该时间的事件前 |
| 已知具体语句的位置 | `--stop-position` | 位置是 binlog 内偏移，绝对精确，不受时区与时钟漂移影响 |
| 只恢复某个库 | `--database`（行格式下按实际被改的表过滤）或 `--rewrite-db` | 行格式不看 `USE`，语句格式才看默认库 |
| 按事务挑 | `--include-gtids` / `--exclude-gtids` | 需要 GTID 开启 |
| 想在别处演练 | `--rewrite-db` + 影子库 | 不改原库即可验证重放结果 |

从 mysqlbinlog 的文本输出里定位时间很直接：每个事件前有头部注释，第一行 `# at 141` 表示该事件在文件中的偏移，第二行形如 `#100309 9:28:36 server id 123 end_log_pos 245` 给出该语句在源库开始执行的时间、源库 `server_id` 与「下一个事件的起始位置」（`end_log_pos` 是本事件结束位置 + 1）。注意使用事件组时多行注释会被合并，不要把它误当成空偏移。

## 误删数据之后：跳过危险语句的两种手法与它们的风险

恢复误删的核心难点不是「怎么重放」，而是「在哪一条之前停」。手法有两类，风险等级完全不同。

**第一类是精确截断**：`--stop-datetime` 或 `--stop-position`。优点是语义明确——重放的是一个连续前缀，之后的事一条都不做；缺点是要求你知道准确的边界，且出错边界往往比你以为的早一条（同一秒内多条语句、或事务跨多个事件）。

**第二类是过滤**：`--skip-gtids`、`--include-gtids`/`--exclude-gtids`、`--database`、`--rewrite-db`。`--skip-gtids` 的官方定义是「Do not include the GTIDs from the binary log files in the output dump file」，它让重放出的事务以新 GTID 身份执行，从而绕过「GTID 已执行 → 静默跳过」的保护。这正是它危险的地方：**同一个事务因此可以被重复应用**。若你为了跳过一条 DELETE 而做了过滤，紧接着又把整段 binlog 重放两遍，重复插入与重复扣款就会真实发生。

**第三类更该警惕：让错误静默。** `mysqlbinlog --idempotent` 会让服务器抑制重复键（1062）与键不存在（1032）错误。官方对该选项的说明是「causes suppression of any duplicate-key or key-not-found errors」，作用域仅当前 mysqlbinlog 客户端与会话；本页读取的选项表把它的默认值标为 `true`，而其他来源常见说法是「不显式提供则不生效」，此处**待核**。无论默认如何，用它意味着「本来该报错的中断点被吞掉了」——数据不一致会以「命令执行成功」的形式伪装过去。另一个反例用法是 `--disable-log-bin`：它插入 `SET sql_log_bin = 0`，若只想避免恢复动作污染 binlog 是对的，但它同时也让这次执行不留痕，事后无法从 binlog 复盘做了什么。

因此真正的判据不是「用哪个参数跳过」，而是：**先在影子库演练**。原因有四条，都是不可逆性带来的：重放是写操作，在生产上做错一次就没有第二次机会；binlog 里 DDL 与 DML 交错，顺序敏感，跳过一条可能让后面所有语句报错；时间边界经常差一条语句，而这一条的差别只能靠跑一遍发现；`--idempotent` 这类选项会把错误变成静默成功，只有在对比行数时才会暴露。演练的具体做法是把 binlog 与全量备份恢复到一台独立实例（或另一个库名，用 `--rewrite-db`），

1. 先恢复全量，再重放截断后的增量；
2. 对比关键表的行数与业务不变量（例如「账户余额之和」这类可校验的等式）；
3. 在演练库上确认边界语句确实停在目标之前；
4. 才在生产执行，并在执行前**先把当前（哪怕是坏掉的）实例做一次物理或逻辑备份**——否则「恢复」本身成为不可逆操作。

## 版本差异（5.7 vs 8.0）

| 项目 | 5.7 | 8.0 | 影响 |
| --- | --- | --- | --- |
| `log_bin` 默认 | `OFF` | `ON` | 升级后磁盘与写入开销都会变；也意味着默认就有 PITR 能力 |
| `server_id` 默认 | `0` | `1` | 5.7 开 binlog 而未设 `server_id` 会直接起不来 |
| `log-slave-updates` 默认 | `OFF` | `ON` | 级联复制行为变化 |
| `expire_logs_days` 默认 | `0`（不过期） | `30` | binlog 会被自动清理，保留期不够就丧失 PITR 能力 |
| `max_allowed_packet` 默认 | `4194304` | `67108864` | 大事务/大 dump 的截断问题减少 |
| `gtid_executed_compression_period` 默认 | `1000` | `0` | GTID 表压缩策略变化 |
| 备份参数名 | `--master-data` | `--source-data`（`--master-data` 已弃用） | 老脚本仍可用但会告警 |
| 建用户语法 | `GRANT ... IDENTIFIED BY` 可用（已弃用） | 8.0.11 移除 | 恢复账号脚本时直接失败，须改 `CREATE USER` |
| 默认字符集 / 排序 | `latin1` / `latin1_swedish_ci` | `utf8mb4` / `utf8mb4_0900_ai_ci` | 重放含中文的语句时字符集假设变化 |
| 默认认证插件 | `mysql_native_password` | `caching_sha2_password` | 旧版 `mysql`/`mysqlbinlog` 客户端可能连不上，需升级客户端或走安全连接 |

此外还有两条 8.0 才有的能力：8.0.14 起 binlog 可加密（`binlog_encryption=ON`，此时 mysqlbinlog 须 `--read-from-remote-server`）；8.0.20 起支持事务负载压缩（`binlog_transaction_compression=ON`，同版本及更新的 mysqlbinlog 会自动解压）。5.7 与 8.0 的手册章节号不同（binlog 在 8.0 是 §7.4.4，在 5.7 是 §5.4.4），引用时须注明版本。

## 判据速查

| 你的目标 | 必要配置/做法 | 常见反例 |
| --- | --- | --- |
| 已提交事务不因宕机丢失 | `sync_binlog=1` + InnoDB 的 `innodb_flush_log_at_trx_commit=1`（后者默认值本页未核，待核） | 为吞吐把 `sync_binlog` 调大，却又要求「一条都不能丢」 |
| 能恢复到任意时间点 | `log_bin=ON` + 保留期足够（`expire_logs_days` 8.0 默认仅 30 天）+ 定期全量 | 有 binlog 但从没做过全量，恢复无从起步 |
| 复制/重放结果与源一致 | ROW 格式 + GTID | 用 RC 却保留 STATEMENT |
| 备份可在线做且一致 | `mysqldump --single-transaction --flush-logs --source-data=2` | 表里有 MyISAM 却以为 `--single-transaction` 保护了它 |
| 误删后只回滚到故障前 | 影子库演练 + `--stop-datetime`/`--stop-position` 精确截断 | 直接用 `--skip-gtids` 全量重放，重复应用事务 |
| 重放前能看清将执行什么 | mysqlbinlog 先落盘、grep 关键语句再执行 | 直接管道进 mysql，看不到执行内容 |

## 关联

- [[MySQL]] —— 上位条目。
- [[2026-10-27-MySQL事务索引与备份]] —— 素材摘要，其中「用 mysqldump 备份」只有结论，本页补的是 redo/undo/binlog 分工与增量恢复流程。
- [[事务隔离级别]] —— RC 与 ROW 格式的强制绑定、undo log 作为 MVCC 旧版本来源都在该页展开。
- [[B+树索引]] —— 索引数量与写入量直接影响 binlog 体积与重放时长。
- [[DML数据管理]] —— 误删/误改的语句形态决定了如何在 binlog 中定位边界。
- [[2026-10-27-MySQL8安装]] —— 8.0 的 `log_bin` 默认开启、`caching_sha2_password` 默认插件都会影响备份与恢复工具链。
- [[CentOS]] —— 备份文件落地、`mysqlbinlog` 落盘与磁盘空间规划的实际环境。

## 来源

主源为 Oracle 官方托管的 MySQL 8.0 手册镜像 `docs.oracle.com/cd/E17952_01/mysql-8.0-en/`（`dev.mysql.com` 同页面对本机抓取返回 HTTP 403，故用镜像；内容为同一手册）。访问日期均为 2026-10-27。

1. Section 7.4.4「The Binary Log」（事件、写时机、完整性、格式、缓存与 fsync 相关变量） —— `https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/binary-log.html`
2. Section 17.6.5「Redo Log」 —— `https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/innodb-redo-log.html`
3. Section 17.6.6「Undo Logs」 —— `https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/innodb-undo-logs.html`
4. Section 9.5「Point-in-Time (Incremental) Recovery」 —— `https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/point-in-time-recovery.html`
5. Section 9.5.1「Point-in-Time Recovery Using Binary Log」 —— `https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/point-in-time-recovery-binlog.html`
6. Section 6.5.4「mysqldump — A Database Backup Program」（`--single-transaction`／`--flush-logs`／`--source-data`／`--set-gtid-purged`／权限要求） —— `https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/mysqldump.html`
7. Section 6.6.9「mysqlbinlog — Utility for Processing Binary Log Files」（`--start-datetime`／`--stop-datetime`／`--skip-gtids`／`--idempotent`／`--base64-output`／`--disable-log-bin`） —— `https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/mysqlbinlog.html`
8. Section 19.1.6.5「Replication and Binary Logging Options and Variables」（`gtid_mode`、`enforce_gtid_consistency`、`gtid_executed`、`gtid_purged`） —— `https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/replication-options-gtids.html`
9. Section 9.2「Database Backup Methods」（`--single-transaction` 在线备份、`FLUSH LOGS` 轮转后拷贝 binlog 的增量做法） —— `https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/backup-methods.html`
10. Section 3.5「Changes in MySQL 8.0」（Changed Server Defaults 表；移除 `GRANT ... IDENTIFIED BY`；`caching_sha2_password` 与 `utf8mb4` 默认值） —— `https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/upgrading-from-previous-series.html`

待核三项：`binlog_format` 在 8.0 的默认值（本次未取到该变量默认值原文）；`mysqlbinlog --idempotent` 的默认值（本页选项表标为 `true`，其他来源说法不一）；`innodb_flush_log_at_trx_commit` 的默认值。
