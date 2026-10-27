---
type: concept
status: budding
created: 2026-10-27
updated: 2026-10-27
sources:
  - raw/Frontend_Backend/MySQL_Data_Management.md
  - raw/Frontend_Backend/MySQL_Operations.md
tags:
  - mysql
  - dml
---

DML（Data Manipulation Language，数据操作语言）是 SQL 中负责增删改**表内数据**的分支，三条语句：`INSERT` 添加行、`UPDATE` 修改行、`DELETE` 删除行——它们动的是数据，不是表结构。

三条语句各有必须记住的边界：`INSERT` 省略字段列表时，值的个数与顺序必须与表结构**严格一致**；`UPDATE` 与 `DELETE` 不写 `WHERE` 就作用于**整张表**，这是最常见的生产事故来源。`UPDATE` 的 `SET` 右侧可以引用本列原值做运算（`SET ClassHour = ClassHour - 10`），也可以嵌套 `SELECT` 的结果。

`TRUNCATE TABLE` 常与 `DELETE` 一起讲，但二者性质不同：`TRUNCATE` 速度更快、会**重置 `AUTO_INCREMENT` 计数器**、保留表结构与索引，且不能带 `WHERE`——只能清空整表。素材把 `TRUNCATE` 与 DML 并列，严格按 SQL 功能分类它通常归入 DDL，且是隐式提交、不可回滚的，这一条是本知识库的**编译者补充**，见 [[2026-10-27-MySQL数据管理]] 的标注。

**DML 与事务的关系**是本主题最实用的一条：MySQL 默认自动提交，单条 DML 执行完即落库，`ROLLBACK` 无从撤销；要成批撤销必须先 `SET AUTOCOMMIT = 0` 再 `START TRANSACTION`，最后 `COMMIT` 或 `ROLLBACK`。但事务只能救 `DELETE`，救不了 `TRUNCATE`——因为后者隐式提交。

写进去的数据怎么读出来，见 [[DQL查询]]；两者的共同宿主是 [[MySQL]]。DML 的执行频率也直接影响索引维护成本，取舍见 [[2026-10-27-MySQL事务索引与备份]]。

## 关联

- [[DQL查询]]、[[MySQL]]
- 素材提炼：[[2026-10-27-MySQL数据管理]]
