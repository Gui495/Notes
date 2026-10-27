## 1. 事务
- 事务就是将一组 SQL 语句放在同一批次内执行。
- 如果一个 SQL 语句出错，则该批次内所有 SQL 都将被取消执行。
- ACID 属性：
  - 原子性 Atomicity
  - 一致性 Consistency
  - 隔离性 Isolation
  - 持久性 Durability

## 2. MySQL 事务实现
### 2.1 自动提交
```sql
SET AUTOCOMMIT = 0; -- 关闭自动提交
SET AUTOCOMMIT = 1; -- 开启自动提交
```
- MySQL 默认自动提交。
- 使用事务时应先关闭自动提交。

### 2.2 事务语句
- `START TRANSACTION;`：开始事务，标记起始点。
- `COMMIT;`：提交事务给数据库。
- `ROLLBACK;`：回滚，数据回到本次事务初始状态。
- `SET AUTOCOMMIT = 1;`：开启自动提交。

### 2.3 事务处理步骤
1. `SET AUTOCOMMIT = 0;`
2. `START TRANSACTION;`
3. 执行 SQL。
4. 成功 `COMMIT;`，失败 `ROLLBACK;`
5. `SET AUTOCOMMIT = 1;`

### 2.4 模拟网上支付
- 顾客 A 购买 500 元商品，A 余额 2000，B 余额 10000。
- 创建 shop 数据库和 account 表：
```sql
INSERT INTO account (name, cash) VALUES ('A', 2000.00);
INSERT INTO account (name, cash) VALUES ('B', 10000.00);
```
- A 成功减少 500，B 应增加 500；若错误导致 B 未增加，则回滚 A 的 500，保证总额平衡。
- 使用事务模拟。

## 3. 数据库索引
### 3.1 作用
- 提高查询速度。
- 确保数据唯一性。
- 加速表和表之间连接，实现参照完整性。
- 使用分组和排序子句检索时减少分组排序时间。
- 全文检索字段搜索优化。

### 3.2 分类
- 主键索引 PRIMARY KEY
- 唯一索引 UNIQUE
- 常规索引 INDEX
- 全文索引 FULLTEXT

### 3.3 主键索引
- 某一个属性或属性组合能唯一标识一条记录。
- 最常见索引类型，确保唯一性，确定记录位置。
```sql
CREATE TABLE Grade (
  GradeID INT(11) AUTO_INCREMENT PRIMARY KEY,
  ...
);
-- 或 PRIMARY KEY(GradeID)
```

### 3.4 唯一索引
- 避免同一个表中某数据列值重复。
- 与主键区别：主键索引只能有一个，唯一索引可有多个。
```sql
CREATE TABLE Grade (
  GradeID INT(11) AUTO_INCREMENT PRIMARY KEY,
  GradeName VARCHAR(32) NOT NULL UNIQUE
);
-- 或 UNIQUE KEY GradeID (GradeID)
```

### 3.5 常规索引
- 快速定位特定数据。
- `index` 和 `key` 都可设置常规索引。
- 应加在查找条件字段。
- 不宜添加太多，影响插入、删除、修改。
```sql
CREATE TABLE result (
  ...
  INDEX/KEY ind (studentNo, subjectNo)
);
ALTER TABLE result ADD INDEX ind (studentNo, subjectNo);
```

### 3.6 全文索引
- 快速定位特定数据。
- 只能用于 MyISAM 类型数据表。
- 只能用于 CHAR、VARCHAR、TEXT 数据列类型。
- 适合大型数据集。
```sql
CREATE TABLE student (
  ...
  FULLTEXT(StudentName)
) ENGINE=MYISAM;
ALTER TABLE employee ADD FULLTEXT(first_name);
```

### 3.7 管理索引
- 创建：建表时添加、建表后追加 `ALTER TABLE 表名 ADD 索引类型(数据列名)`。
- 删除：
  - `DROP INDEX 索引名 ON 表名`
  - `ALTER TABLE 表名 DROP INDEX 索引名`
  - `ALTER TABLE 表名 DROP PRIMARY KEY`
- 查看：`SHOW INDEX(或KEYS) FROM 表名`

### 3.8 索引准则
- 索引不是越多越好。
- 不要对经常变动的数据加索引。
- 小数据量表建议不要加索引。
- 索引一般应加在查找条件字段。

### 3.9 EXPLAIN
```sql
EXPLAIN 表名;
DESC 表名;
EXPLAIN SELECT 语句;
```
- 使用 `\G` 结尾竖排显示。
- 分析 SQL 执行性能。
- 连接类型从好到差：const、eq_reg、ref、range、indexhe、ALL。

## 4. 数据库备份与恢复
### 4.1 备份必要性
- 保证重要数据不丢失。
- 数据转移。

### 4.2 备份方法
- mysqldump 备份工具。
- 数据库管理工具，如 SQLyog。
- 直接拷贝数据库文件和相关配置文件。

### 4.3 mysqldump
```bash
mysqldump -h 主机名 -u 用户名 -p [options] [table1 table2 ...] > path/filename.sql
```
示例：
```bash
mysqldump -u root -p myschool > d:/myschool.sql
```
常用选项：
- `--add-drop-table`：导出脚本加 DROP TABLE IF EXISTS，默认打开。
- `--add-locks`：INSERT 语句捆绑 LOCK TABLE 和 UNLOCK TABLE，默认打开。
- `-t` 或 `--no-create-info`：忽略重新创建每个转储表的 CREATE TABLE 语句。
- `-c` 或 `--complete-insert`：每个 INSERT 语句列上加字段名。
- `-d` 或 `--no-data`：不写表行信息，只转储表结构。
- `--where "where-condition", -w`：只转储给定 WHERE 条件选择的记录。
- `--opt`：速记，等同指定多个选项。

### 4.4 恢复
方法一：SOURCE
```sql
SOURCE /path/db_name.sql;
```
- 绝对路径，mysql 运行用户有权限读取。
- 在 MySQL 命令行执行。
- 先创建或选择数据库。

方法二：mysql 客户端
```bash
mysql -u root -p dbname < /path/db_name.sql
```

### 4.5 SQL 语句导出导入
导出：
```sql
SELECT * INTO OUTFILE 'file_name' FROM tbl_name;
```
- 输出文件不能先存在，否则报错。

导入：
```sql
LOAD DATA INFILE 'file_name' INTO TABLE tbl_name[FIELDS];
```

## 5. 小结
- 事务 ACID。
- 关闭/开启自动提交。
- 事务处理流程。
- 索引类型与使用场合。
- 备份方法、mysqldump、SOURCE 导入。