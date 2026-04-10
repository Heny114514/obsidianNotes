# SQL 数据查询
## SELECT 语句一般格式

SQL 数据查询的基本语法（图 3-3 查询块结构）：

```sql
SELECT [DISTINCT | ALL] <目标列表>
FROM <表名或视图名> [<别名>] [, <表名或视图名> [<别名>] ...]
[WHERE <条件>]
[GROUP BY <分组属性列名> [HAVING <分组条件>]]
[ORDER BY <排序属性列名> [ASC | DESC] [, <排序属性列名> [ASC | DESC] ...]]
[LIMIT [<m>,] <n>];
```

- **目标列表**：列名、常量、算术表达式、字符串连接、函数等。可以指定列别名 `AS`。
- **DISTINCT**：消除重复行；**ALL**：保留重复（默认）。
- **FROM**：单表、多表（隐式连接）、子查询（派生表）。
- **WHERE**：元组选择条件。
- **GROUP BY**：分组。
- **HAVING**：分组后筛选。
- **ORDER BY**：排序（默认 ASC）。
- **LIMIT**：限制结果行数（MySQL 等）。

**示例**（基于学生-课程-选课示例表）：

```sql
-- 查询所有学生姓名和平均成绩
SELECT Sname, AVG(Grade) AS AvgGrade
FROM Student S, SC
WHERE S.Sno = SC.Sno
GROUP BY S.Sno, Sname
HAVING AVG(Grade) >= 80
ORDER BY AvgGrade DESC
LIMIT 5;
```

## 单表查询

### 1. 选择列（投影）

- **列名**：`SELECT Sno, Sname FROM Student;`
- **常量**：`SELECT '高分' AS 评级, Sno FROM Student;`
- **算术运算**：`SELECT Sno, 85 + (Grade - 60)*0.5 AS 加权分 FROM SC;`
- **列别名**：`SELECT Sname AS 姓名 FROM Student;`
- **字符串连接**：`SELECT Sname || Smajor AS 学生专业 FROM Student;`（或 `CONCAT(Sname, ' - ', Smajor)`）
- **消除重复**：`SELECT DISTINCT Dept FROM Student;`

**表 3.5 选择列示例**（描述：展示不同投影操作的结果表）。

### 2. 选择元组（限制）

**WHERE 条件谓词**（表 3.5 详细谓词表）：

| 谓词类型 | 语法 | 示例 | 描述 |
|----------|------|------|------|
| 比较 | `A θ B` (θ: =, >, <, ≥, ≤, ≠) | `Sage > 20` | 基本比较 |
| 范围 | `A [NOT] BETWEEN B1 AND B2` | `Grade BETWEEN 60 AND 90` | 等价 `60 ≤ Grade ≤ 90` |
| 集合 | `A [NOT] IN (v1, v2, ...)` | `Cno IN ('C01', 'C02')` | 成员测试 |
| 模式匹配 | `A [NOT] LIKE 模式` | `Sname LIKE '张%'` | % 通配多字符，_ 通配单字符 |
| 空值 | `A IS [NOT] NULL` | `Sage IS NULL` | 空值判断 |
| 存在 | `EXISTS (子查询)` | `EXISTS (SELECT ...)` | 见嵌套查询 |
| 逻辑 | `条件1 AND/OR/NOT 条件2` | `(Sage > 20) OR Dept='CS'` | 复合条件 |

**优先级**：算术 > 比较 > NOT > AND > OR。括号调整。

**示例**：
```sql
SELECT * FROM Student WHERE Smajor = 'CS' AND Sbirthday > '2000-01-01';
```

### 3. 排序 (ORDER BY)

```sql
SELECT * FROM SC ORDER BY Grade DESC, Cno ASC;
```

**表 3.6 ORDER BY 示例**（描述：展示未排序、升序、降序、多列排序结果）。

### 4. 聚集函数

| 函数 | 描述 | 示例 |
|------|------|------|
| `COUNT(*)` / `COUNT(A)` | 计数（* 含空值，列不含） | `COUNT(*) = 5` |
| `SUM(A)` | 求和（数值） | `SUM(Grade)` |
| `AVG(A)` | 平均（数值） | `AVG(Grade)` |
| `MAX(A)` / `MIN(A)` | 最大/最小 | `MAX(Grade)` |

**示例**：`SELECT AVG(Grade) FROM SC WHERE Cno='C01';` → 85.5

### 5. 分组统计 (GROUP BY / HAVING)

- **GROUP BY**：按列分组。
- **HAVING**：筛选组（聚合条件）。

```sql
SELECT Cno, AVG(Grade) AS 平均分
FROM SC
GROUP BY Cno
HAVING AVG(Grade) > 85
ORDER BY 平均分 DESC;
```

### 6. LIMIT 限制结果集

```sql
SELECT * FROM Student LIMIT 10 OFFSET 20;  -- 跳过20行，取10行
```

## 连接查询

隐式连接：`FROM R, S WHERE R.A = S.B;`

显式 JOIN（推荐）：

| 类型 | 语法 | 描述 |
|------|------|------|
| 等值连接 | `R JOIN S ON R.A = S.B` | 标准内连接 |
| 非等值 | `R JOIN S ON R.A > S.B` | 任意比较 |
| 自然连接 | `R NATURAL JOIN S` | 同名列自动等值连接，去重列 |
| 自身连接 | `R AS R1 JOIN R AS R2 ON ...` | 同表连接 |
| 左外连接 | `R LEFT OUTER JOIN S ON ...` | 保留 R 悬挂元组（右填 NULL） |
| 右外连接 | `R RIGHT OUTER JOIN S ON ...` | 保留 S 悬挂元组 |
| 全外连接 | `R FULL OUTER JOIN S ON ...` | 保留双方悬挂 |

**多表连接**：链式 `A JOIN B ON ... JOIN C ON ...`

**示例**（等值连接）：
```sql
SELECT Sname, Cname, Grade
FROM Student S JOIN SC ON S.Sno=SC.Sno
               JOIN Course C ON SC.Cno=C.Cno
WHERE Grade >= 90;
```

**图 3-7 连接查询示意图**（描述：展示两个关系连接过程，悬挂元组）。

**表 3.7/3.8 连接示例**（多表结果）。

## 嵌套查询（子查询）

子查询嵌套在主查询中，可在 SELECT、FROM、WHERE。

### 1. 用 IN 谓词

```sql
SELECT Sname FROM Student
WHERE Sno IN (SELECT Sno FROM SC WHERE Grade > 90 AND Cno='C01');
```

### 2. 比较运算符 + {ANY | ALL | SOME}

```sql
-- 高于所有 C01 成绩的学生
SELECT Sname FROM Student S
WHERE Sno IN (SELECT Sno FROM SC WHERE Cno='C01')
  AND 85 > ALL (SELECT Grade FROM SC WHERE Sno = S.Sno);
```

- `> ALL`：大于所有
- `> ANY`：大于至少一个

### 3. EXISTS 谓词

相关子查询（引用外层变量）：
```sql
SELECT Sname FROM Student S
WHERE EXISTS (SELECT * FROM SC WHERE Sno = S.Sno AND Grade > 90);
```

**NOT EXISTS**：不存在。

**表 3.11/3.12 EXISTS 示例**。

## 集合查询

兼容 UNION 兼容的查询结果：

| 操作 | 语法 | 描述 |
|------|------|------|
| 并 | `Q1 UNION Q2` | 去重并 |
| 交 | `Q1 INTERSECT Q2` | 交集 |
| 差 | `Q1 EXCEPT Q2` | 差集（Q1 - Q2） |

**示例**：
```sql
(SELECT Sno FROM SC WHERE Grade > 90)
UNION
(SELECT Sno FROM SC WHERE Cno = 'C01');
```

**UNION ALL**：保留重复。

## 基于派生表的查询（表子查询）

子查询作为 FROM 的虚拟表：

```sql
SELECT Sno, AVGGrade
FROM (SELECT Sno, AVG(Grade) AS AVGGrade
      FROM SC GROUP BY Sno) AS SC_AVG
WHERE AVGGrade > 85;
```

**示例表 3.15**（描述：派生表计算平均分后筛选）。

## 总结

- **单表查询**：基础，掌握投影、选择、排序、聚合、分组。
- **连接查询**：核心，多表关联，使用 JOIN 语法优于隐式。
- **嵌套/集合/派生**：复杂查询，提高表达力，但注意性能（优化为 JOIN）。
- **最佳实践**：
  - 条件顺序：WHERE > GROUP BY > HAVING > ORDER BY。
  - 避免深嵌套，使用相关子查询慎用。
  - 索引优化查询列。
- **图表参考**：
  - 图 3-3：查询块。
  - 图 3-7：连接。
  - 表 3.5：谓词；3.6：排序；3.10-3.15：查询示例结果。