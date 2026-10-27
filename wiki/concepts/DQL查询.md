---
type: concept
status: budding
created: 2026-10-27
updated: 2026-10-27
sources:
  - raw/Frontend_Backend/DQL_Command.md
tags:
  - mysql
  - dql
---

DQL（Data Query Language，数据查询语言）是 SQL 中专门负责「把数据读出来」的分支，核心语句只有一条 `SELECT`，但它的可组合子句最多。

`SELECT` 的骨架是一条**固定顺序**的子句链：`SELECT` 选列 → `FROM` 定表 → `JOIN` 连表 → `WHERE` 过滤行 → `GROUP BY` 分组 → `HAVING` 过滤组 → `ORDER BY` 排序 → `LIMIT` 分页。书写顺序与执行顺序不同（`WHERE` 实际早于 `SELECT` 的列计算），这是理解「为什么 `WHERE` 里不能直接用列别名、而 `ORDER BY` 里可以」的关键，素材未点明这层。

几个子句的分工必须分清：`WHERE` 过滤**行**，`HAVING` 过滤**分组后的组**；`DISTINCT` 只有在所有返回列都相同时才算重复；`LIKE` 的 `%` 匹配任意长度、`_` 匹配恰好一个字符；`BETWEEN a AND b` 是闭区间；`IN` 是多个 `OR` 的简洁写法且效率更高；`NULL` 表示「无值」，既不是 0 也不是空串，只能用 `IS NULL` 判断。

**多表查询**三种连接：`INNER JOIN` 取双方都匹配的行，`LEFT JOIN` 保左表、`RIGHT JOIN` 保右表（另一侧缺失处补 NULL）；另有**自连接**——用同一张表玩两次、靠别名区分，典型用途是查树形结构的父子关系。

**分页**靠 `LIMIT m,n`：`m` 是跳过的条数、`n` 是取的条数，`LIMIT 5,10` 即第 6~15 条。这是 Web 列表分页的底层机制。

**统计与分组**：`COUNT` / `SUM` / `AVG` / `MAX` / `MIN` 五个聚合函数把一组行压成一个值，`GROUP BY` 决定「按什么分组后再压」，两者几乎总是成对出现。

DQL 与 [[DML数据管理]] 构成 [[MySQL]] 的两条基本操作线：DML 改变数据，DQL 读取数据；而 DQL 语句的写法直接决定了 [[2026-10-27-MySQL事务索引与备份]] 里 `EXPLAIN` 给出的性能判读结果。

## 关联

- [[DML数据管理]]、[[MySQL]]
- 素材提炼：[[2026-10-27-DQL查询]]
