# 6.7.3 分析执行计划 (EXPLAIN)

## 概述

通过上一节的慢查询日志，我们能**找出**哪些 SQL 语句是低效的。而 `EXPLAIN` 命令则能告诉我们**为什么**这些 SQL 语句是低效的。

**执行计划 (Execution Plan)** 是 MySQL 数据库在执行一条 SQL 查询时，内部优化器所决定的“行动方案”。这个方案包括了如何访问表、使用哪个索引、以何种顺序连接表等一系列决策。

`EXPLAIN` 命令就是让 MySQL 把这个内部的“行动方案”展示给我们看。通过解读这份方案，我们可以洞悉查询的性能瓶颈所在。它是 SQL 优化中**最核心、最重要**的工具。

## 如何使用 EXPLAIN

使用方法非常简单，只需在你想要分析的 `SELECT` 语句前加上 `EXPLAIN` 关键字即可。

**语法**：
`EXPLAIN SELECT ... FROM ... WHERE ...`

`EXPLAIN` 命令并**不会真正执行**查询，而只是返回其执行计划。执行计划以一个表格的形式呈现，包含了多个列，每一列都揭示了查询执行过程中的一部分信息。

## 解读 EXPLAIN 的输出

下面我们将逐一介绍 `EXPLAIN` 输出结果中最关键的几列。

**一个 `EXPLAIN` 示例**:
```sql
EXPLAIN SELECT * FROM users WHERE id = 1;
```
**输出**:
| id  | select_type | table | partitions | type  | possible_keys | key     | key_len | ref   | rows | filtered | Extra |
|---|---|---|---|---|---|---|---|---|---|---|---|
| 1   | SIMPLE      | users | NULL       | const | PRIMARY       | PRIMARY | 4       | const | 1    | 100.00   | NULL  |

---

### `type` (访问类型)

**这是 `EXPLAIN` 结果中最重要的一列**。它描述了 MySQL 是如何查找表中所需的数据的。其值的性能从最优到最差依次为：

-   `system`: 表中只有一行数据，是 `const` 的特例。
-   `const`: **(最优)** 通过主键或唯一索引，最多只匹配一行数据。查询在开始时就被优化为常量。
-   `eq_ref`: 在 `JOIN` 查询中，对于前一个表的每一行，后一个表只有一行数据与之匹配。通常发生在 `JOIN` 条件是主键或唯一索引时。
-   `ref`: 非唯一性索引的查找。返回所有匹配某个特定值的行。
-   `range`: **(较好)** 使用索引来检索一个给定范围内的行。例如 `WHERE id > 10` 或 `WHERE age BETWEEN 20 AND 30`。
-   `index`: **(较差)** 对整个**索引**进行扫描来查找数据。虽然比扫描整个表要快（因为索引通常比表数据小），但仍然是低效的。
-   `ALL`: **(最差)** **全表扫描 (Full Table Scan)**。MySQL 不得不遍历整张表来查找匹配的行。如果数据量大，这是巨大的性能灾难。**`type` 列为 `ALL` 是优化的首要目标。**

### `key` (实际使用的索引)

这一列显示了 MySQL 在查询中**实际决定使用**的索引。如果为 `NULL`，则表示 MySQL 没有使用任何索引。
-   `possible_keys`: 显示 MySQL *认为可能*可以使用的索引列表。
-   **分析要点**：如果 `possible_keys` 中有索引，但 `key` 却为 `NULL`，这说明 MySQL 因为某种原因（如数据分布、查询写法）最终放弃了使用索引，需要我们深入分析。

### `rows` (估算的扫描行数)

这一列给出了 MySQL **估算**为了找到所需数据而需要读取的行数。这是一个估算值，不一定精确，但极具参考价值。
-   **分析要点**：这个数字越小越好。如果一个查询最终只返回几行数据，但 `rows` 列的值却非常大，这通常意味着查询的效率很低。

### `Extra` (额外信息)

这一列包含了执行计划的额外信息，其中一些是判断性能好坏的关键标志。

-   `Using index`: **(好)** 表示查询所需的数据仅通过读取索引树就能全部获得，无需回表查询（即访问实际的数据行）。这称为**覆盖索引 (Covering Index)**，是性能极好的一种情况。
-   `Using where`: 表示 MySQL 服务器将在存储引擎返回行后，再应用 `WHERE` 子句进行过滤。这是正常现象。
-   `Using temporary`: **(差)** 表示 MySQL 为了处理查询（通常是 `ORDER BY` 或 `GROUP BY`），需要创建一个临时表。这通常发生在内存中，但如果临时结果集太大，则会在磁盘上创建，性能会急剧下降。
-   `Using filesort`: **(差)** 表示 MySQL 无法利用索引来完成排序操作，只能在内存或磁盘中进行额外的排序（文件排序）。这是一个非常消耗 CPU 和 I/O 的操作，是优化的重点对象。

## 实际分析工作流：一个前后对比

假设我们有如下查询，且 `email` 字段没有索引：
`EXPLAIN SELECT id, username FROM users WHERE email = 'test@example.com';`

**优化前**:
| id | select_type | table | type | possible_keys | key  | rows  | Extra       |
|----|-------------|-------|------|---------------|------|-------|-------------|
| 1  | SIMPLE      | users | ALL  | NULL          | NULL | 50000 | Using where |

**解读**：
-   `type` 是 `ALL`：全表扫描，灾难。
-   `key` 是 `NULL`：没有使用任何索引。
-   `rows` 是 `50000`：MySQL 预计需要扫描 5 万行数据。

**优化步骤**：为 `email` 字段添加索引。
```sql
ALTER TABLE users ADD INDEX idx_email (email);
```

**优化后**:
再次执行 `EXPLAIN`：
| id | select_type | table | type | possible_keys | key       | rows | Extra       |
|----|-------------|-------|------|---------------|-----------|------|-------------|
| 1  | SIMPLE      | users | ref  | idx_email     | idx_email | 1    | Using where |

**解读**：
-   `type` 变成了 `ref`：通过索引查找，非常高效。
-   `key` 显示 `idx_email`：成功用上了我们创建的索引。
-   `rows` 变成了 `1`：MySQL 预计只需扫描 1 行。

通过这个简单的“前后对比”，我们可以清晰地看到索引带来的巨大性能提升。

---

## 总结

-   `EXPLAIN` 是透视 MySQL 查询性能的 X 光机。
-   **分析重点**：
    -   检查 `type` 列，避免出现 `ALL`。
    -   检查 `key` 列，确保用上了正确的索引。
    -   检查 `rows` 列，看估算的扫描行数是否在一个合理的范围。
    -   检查 `Extra` 列，警惕 `Using temporary` 和 `Using filesort`。

掌握 `EXPLAIN` 是从“知其然”到“知其所以然”的飞跃，它是你从一名普通开发者迈向高级开发者的必备技能。

---

## 练习任务

1.  **分析现有查询**：
    找一个你之前项目中比较复杂的 `SELECT` 查询（例如，带有多表 `JOIN` 和 `WHERE` 条件的查询），在它前面加上 `EXPLAIN` 并执行。尝试解读输出结果中的 `type`, `key`, `rows` 和 `Extra` 列。

2.  **索引选择实验**：
    在一个有多个索引的表上，编写一个 `WHERE` 条件，使其 `possible_keys` 中出现多个索引。观察 MySQL 最终在 `key` 列中选择了哪一个，并思考其原因。

3.  **触发 `filesort`**：
    在一个有索引的字段（如 `username`）和一个没有索引的字段（如 `login_count`）的 `users` 表上：
    -   执行 `EXPLAIN SELECT id, username FROM users ORDER BY username;`。
    -   执行 `EXPLAIN SELECT id, login_count FROM users ORDER BY login_count;`。
    对比两次 `EXPLAIN` 结果中 `Extra` 列的差异，观察 `Using filesort` 是如何出现的。
