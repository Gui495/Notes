## 1. 数据库数据管理
- 数据库意义：数据存储、数据管理。
- 管理方法：
  - SQLyog 等管理工具。
  - DML 语句管理数据库数据。

## 2. DML 语言
DML（数据操作语言）：用于操作数据库对象中所包含的数据。
包括：
- INSERT：添加数据
- UPDATE：更新数据
- DELETE：删除数据

## 3. 添加数据 INSERT
语法：
```sql
INSERT INTO 表名 [(字段1, 字段2, 字段3, ...)]
VALUES ('值1', '值2', '值3', ...);
```
- 字段或值之间用英文逗号隔开。
- 字段部分可省略，但值必须与表结构数据列顺序对应且数量一致。
- 可同时插入多条数据，values 后用英文逗号隔开。
示例：
```sql
INSERT INTO grade (GradeName) VALUES ('大一'),('大二'),('大三'),('大四');
```
student 表数据示例：学号、姓名、性别、年级、手机号、地址、邮箱、身份证。

## 4. 修改数据 UPDATE
语法：
```sql
UPDATE 表名
SET column_name = value [, column_name2 = value2, ...]
[WHERE condition];
```
- column_name 为要更改的数据列。
- value 可为变量、具体值、表达式或嵌套 SELECT 结果。
- condition 为筛选条件，不指定则修改该表所有列数据。
WHERE 运算符：
- `=`、`!=`、`>`、`<`、`>=`、`<=`
- `BETWEEN ... AND ...`
- `AND`、`OR`
示例：
```sql
UPDATE student SET email='student1013@bdqn.cn', LoginPwd='000000' WHERE StudentNo=1013;
UPDATE subject SET ClassHour = ClassHour - 10 WHERE ClassHour > 110 AND GradeID = 1;
```

## 5. 删除数据 DELETE
语法：
```sql
DELETE FROM 表名 [WHERE condition];
```
- condition 为筛选条件，不指定则删除该表所有列数据。

## 6. TRUNCATE
语法：
```sql
TRUNCATE [TABLE] table_name;
```
- 完全清空表数据，但表结构、索引、约束等不变。
- 与 DELETE 相同：都能删除数据、不删除表结构，TRUNCATE 速度更快。
- 不同：
  - TRUNCATE TABLE 重新设置 AUTO_INCREMENT 计数器。
  - TRUNCATE TABLE 不会对表名有影响。

## 7. 小结
- DML 包括 INSERT、UPDATE、DELETE。
- INSERT 可一次插入多条。
- UPDATE 可同时修改多个数据列。
- UPDATE/DELETE 建议配合 WHERE，避免误操作全表。