## 1. DQL 概述
DQL（Data Query Language，数据查询语言）：
- 查询数据库数据，如 SELECT 语句。
- 简单的单表查询或多表复杂查询和嵌套查询。
- 数据库语言中最核心、最重要、使用频率最高的语句。

## 2. SELECT 语法
```sql
SELECT [ALL | DISTINCT]
{ * | table.* | [ table.field1 [as alias1] [, table.field2 [as alias2]][, ...]] }
FROM table_name [as table_alias]
[ left|out|inner join table_name2 ]  # 联合查询
[ WHERE ... ]                       # 指定结果需满足的条件
[ GROUP BY ... ]                    # 指定结果按照哪几个字段分组
[ HAVING ... ]                      # 过滤分组记录必须满足的次要条件
[ ORDER BY ... ]                    # 指定查询记录按一个或多个条件排序
[ LIMIT { [offset,] row_count | row_count OFFSET offset } ];
```
- `[]` 可选，`{}` 必须。
- `#` 或 `/* */` 注释。

## 3. 指定查询字段
- 查询所有列：`select * from student;` 效率低，不推荐。
- 指定列：`SELECT StudentNo, StudentName, Phone FROM student;`
- 多表同名字段指定表：`SELECT student.StudentNo, StudentName, StudentResult FROM student, result;`

## 4. AS 子句
- 给数据列取别名。
- 给表取别名。
- 给计算/总结结果取新名称。
```sql
SELECT StudentNo AS "学号" FROM student;
SELECT a.StudentNo FROM student AS a;
SELECT Phone+1 AS Tel FROM student;
```
- AS 可省略。

## 5. DISTINCT
- 去掉 SELECT 返回记录中重复的记录（所有返回列值都相同），只返回一条。
```sql
SELECT DISTINCT SubjectNo FROM result;
```
- ALL 默认，返回所有记录。

## 6. 表达式列
- 表达式由文本值、列值、NULL、函数和操作符组成。
- 应用场景：SELECT 结果列、ORDER BY/HAVING 子句、DML where 条件。
```sql
SELECT version(), 100*3;
SELECT SubjectName "课程名称", ClassHour+10 AS "新学时" FROM subject;
SELECT version() as MySQL_V, 123.44*100 AS EXPRESSION;
```

## 7. WHERE 条件查询
- 检索数据表中符合条件的记录。
- 搜索条件可由一个或多个逻辑表达式组成，结果一般为真或假。
- 逻辑操作符：
  - `AND` 或 `&&`：同时为真才真。
  - `OR` 或 `||`：一个为真则真。
  - `NOT` 或 `!`：操作数为假则真。
- 比较操作符：
  - `IS NULL`、`IS NOT NULL`
  - `BETWEEN a AND b`
  - `LIKE`
  - `IN (a1,a2,...)`
- 注意：数值数据类型记录之间才能算术运算；相同数据类型才能比较。

## 8. BETWEEN AND
```sql
SELECT * FROM subject WHERE ClassHour BETWEEN 110 AND 120;
-- 等同于
SELECT * FROM subject WHERE ClassHour >= 110 AND ClassHour <= 120;
```

## 9. LIKE 模糊查询
- `%`：匹配 0 或任意多个字符。
- `_`：匹配单个字符。
```sql
SELECT * FROM subject WHERE SubjectName LIKE "%数学%";
SELECT StudentNo, StudentName FROM student WHERE StudentName LIKE "李_";
```

## 10. IN 范围查询
```sql
SELECT * FROM subject WHERE ClassHour IN (100,110,120);
```
- 比多个 OR 更简洁，效率更高。

## 11. NULL 查询
- NULL 代表“无值”，区别于 0 和空字符串。
- 只能出现在定义允许为 NULL 的字段。
- 使用 `IS NULL` 或 `IS NOT NULL` 比较。

## 12. 连接查询
- 内连接 INNER JOIN：至少一个匹配时返回记录。
```sql
SELECT SubjectName, GradeName
FROM subject INNER JOIN grade
ON subject.GradeID = grade.GradeID;
```
- 等值和非等值连接：
  - 非等值：`FROM subject, grade`
  - 等值：`WHERE subject.GradeID = grade.GradeID`
  - 等效于内连接。
- 左外连接 LEFT JOIN：从左表返回所有记录，即便右表无匹配。
- 右外连接 RIGHT JOIN：从右表返回所有记录，即便左表无匹配。
- JOIN 对比：
  - INNER JOIN：至少一个匹配返回行。
  - LEFT JOIN：返回左表所有行。
  - RIGHT JOIN：返回右表所有行。
- 自连接：数据表与自身连接。
  - 示例：category 表包含 categoryId、categoryName、pid，查询父栏目名称和子栏目名称。

## 13. ORDER BY 排序
```sql
SELECT ... ORDER BY 字段 [ASC|DESC];
```
- 默认 ASC。
- 示例：查询《数据库结构-1》所有考试结果，按成绩由高到低排列。

## 14. LIMIT 分页
```sql
SELECT * FROM result LIMIT 5;      -- 前5条
SELECT * FROM result LIMIT 5,10;   -- 第6-15条
```
- m 不指定则偏移量为 0，从第一条开始返回前 n 条。
- 常用子分页显示。

## 15. 子查询
- WHERE 条件子句中嵌套另一个查询语句。
- 查询课程为《高等数学-2》且分数不小于80分的学生的学号和姓名。
- 嵌套查询可由多个子查询组成，求解由里及外。
- 子查询返回结果一般是集合，建议使用 IN。

## 16. 统计函数
- `COUNT()`：满足 SELECT 条件的记录总和数。
- `SUM()`：数字字段或表达式列统计，返回一列总和。
- `AVG()`：平均值。
- `MAX()`：最大值。
- `MIN()`：最小值。
- 不建议使用 `*`，效率低。

## 17. GROUP BY 与 HAVING
- 对查询结果分组。
- 分组依据字段可有多个，并依次分组。
- 与 HAVING 结合，进行分组后的数据筛选。
```sql
SELECT 课程名, MAX(StudentResult) 最高分, MIN(StudentResult) 最低分, AVG(StudentResult) 平均分
FROM result
GROUP BY 课程名
HAVING AVG(StudentResult) >= 60;
```

## 18. 小结
- SELECT 基本语法。
- 模糊查询 LIKE。
- 连接查询场合：内连接、左外连接、右外连接、自连接。
- 子查询、排序、分页、统计、分组。