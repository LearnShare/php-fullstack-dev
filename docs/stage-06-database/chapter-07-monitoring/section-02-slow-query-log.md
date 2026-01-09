# 6.7.2 慢查询日志 (Slow Query Log)

## 概述

**慢查询日志 (Slow Query Log)** 是 MySQL 提供的一个非常强大的性能诊断工具。它是一个文本文件，专门用来记录那些执行时间超过了预设阈值的 SQL 语句。

对于开发者来说，慢查询日志是定位和优化低效 SQL 的**首要武器**。通过分析这个日志，你可以准确地知道哪些查询正在拖慢你的应用，从而有针对性地进行优化。它直接回答了性能优化中最关键的问题：“我应该优化什么？”

## 如何开启和配置慢查询日志

慢查询日志的功能由一系列 MySQL 服务器变量来控制。

### 核心配置参数

-   `slow_query_log`
    -   **作用**：慢查询日志的总开关。`ON` 为开启，`OFF` 为关闭。
-   `slow_query_log_file`
    -   **作用**：指定慢查询日志文件的存放路径和名称。
-   `long_query_time`
    -   **作用**：设置时间的阈值（单位：秒）。执行时间**超过**这个值的查询才会被记录。
    -   **建议**：默认值通常是 10 秒，对于 Web 应用来说太高了。生产环境建议设置为 `1` 或 `2` 秒。开发环境中可以设置得更低，如 `0.5`，以便捕获更多潜在的优化点。
-   `log_queries_not_using_indexes`
    -   **作用**：一个非常有用的选项。如果设置为 `ON`，MySQL 会将那些**没有使用索引**的查询也记录到慢查询日志中，**无论**它们的执行时间有多长。
    -   **重要性**：这能帮助我们主动发现那些因为缺少索引而进行全表扫描的查询，提前进行优化，防患于未然。

### 配置方法

**方法一：动态配置 (用于临时测试)**
你可以通过 SQL 命令在 MySQL 运行时动态修改这些变量。这种方式无需重启服务，但设置会在 MySQL 重启后丢失。

```sql
-- 开启慢查询日志
SET GLOBAL slow_query_log = 'ON';

-- 设置慢查询时间为 1 秒
SET GLOBAL long_query_time = 1;

-- 记录没有使用索引的查询
SET GLOBAL log_queries_not_using_indexes = 'ON';

-- 查看当前配置
SHOW VARIABLES LIKE '%slow%';
SHOW VARIABLES LIKE 'long_query_time';
```
**注意**：`SET GLOBAL` 对当前已经存在的连接无效，只对新建立的连接生效。

**方法二：永久配置 (推荐)**
为了让配置在 MySQL 重启后依然生效，需要将其写入 MySQL 的配置文件中（Linux 系统中通常是 `/etc/my.cnf` 或 `/etc/mysql/my.cnf`，Windows 中是 `my.ini`）。

在配置文件的 `[mysqld]` 区块下添加或修改以下行：
```ini
[mysqld]
# 开启慢查询日志
slow_query_log = 1

# 指定日志文件路径
slow_query_log_file = /var/log/mysql/mysql-slow.log

# 设置慢查询时间为 1 秒
long_query_time = 1

# 记录未使用索引的查询
log_queries_not_using_indexes = 1
```
修改配置文件后，需要**重启 MySQL 服务**才能生效。

## 解读慢查询日志

慢查询日志是一个纯文本文件，你可以直接打开查看。每一条慢查询记录通常包含以下几个部分：

**一个典型的慢查询日志条目**:
```
# Time: 2026-01-10T15:30:05.123456Z
# User@Host: myapp[myapp] @ localhost []  Id: 12345
# Query_time: 1.821560  Lock_time: 0.000150 Rows_sent: 10  Rows_examined: 500000
SET timestamp=1768020605;
SELECT * FROM posts p LEFT JOIN users u ON p.user_id = u.id WHERE u.city = 'New York';
```

**关键信息解读**：
-   `# Time`: 查询发生的时间。
-   `# User@Host`: 执行查询的用户和主机。
-   `# Query_time`: **查询执行的总时间**。这是最重要的信息，直接反映了查询的耗时。
-   `# Lock_time`: 等待锁（表锁、行锁）的时间。如果这个值很高，说明存在锁竞争问题。
-   `# Rows_sent`: 发送给客户端的**行数**。
-   `# Rows_examined`: 数据库引擎为了找到结果，**实际扫描和检查的总行数**。

> **性能分析的关键**：
> 对比 `Rows_examined` 和 `Rows_sent` 的值。如果 `Rows_examined` 远远大于 `Rows_sent`（如上例中，扫描了 50 万行才返回 10 行），这通常是一个**强烈的危险信号**，意味着这个查询效率极低，很可能进行了全表扫描或没有用上最优的索引。

## 使用工具分析日志

当慢查询日志文件很大时，手动阅读变得非常困难。我们需要借助工具来汇总和分析。

### `mysqldumpslow`

这是 MySQL 自带的一个命令行工具，可以对慢查询日志进行聚合和排序。

**常用命令**：
-   **显示摘要信息**:
    `mysqldumpslow /var/log/mysql/mysql-slow.log`
-   **按查询时间排序，显示最慢的 5 条查询**:
    `mysqldumpslow -s t -t 5 /var/log/mysql/mysql-slow.log`
-   **按扫描行数排序，显示扫描行数最多的 5 条查询**:
    `mysqldumpslow -s r -t 5 /var/log/mysql/mysql-slow.log`
-   **按查询次数排序，显示执行最频繁的 5 条慢查询**:
    `mysqldumpslow -s c -t 5 /var/log/mysql/mysql-slow.log`

**参数说明**：
-   `-s`: 排序方式 (`t`: query time, `r`: rows examined, `c`: count)
-   `-t`: 返回前 N 条记录

### 更高级的工具
对于更复杂的分析，可以使用 Percona 公司的开源工具 `pt-query-digest`，它能生成非常详细和专业的分析报告。

## 优化的标准工作流程

1.  **开启日志**：确保在开发、测试和生产环境中都开启了慢查询日志。
2.  **定期审查**：定期（如每天）使用 `mysqldumpslow` 等工具分析日志。
3.  **定位元凶**：找出那些执行时间最长、扫描行数最多或执行频率最高的查询。
4.  **分析计划**：将找出的慢查询语句，拿到数据库客户端中，在前面加上 `EXPLAIN` 来分析其执行计划。这是下一节的重点。
5.  **实施优化**：根据 `EXPLAIN` 的结果，采取优化措施，例如添加合适的索引、改写 SQL 语句等。
6.  **验证效果**：部署优化后，持续监控慢查询日志，确认问题 SQL 不再出现或性能已有显著提升。

---

## 练习任务

1.  **开启慢查询**：
    连接到你的本地 MySQL，使用 `SET GLOBAL` 命令动态地开启慢查询日志，并将 `long_query_time` 设置为 0.1。

2.  **制造一条慢查询**：
    在一个有较多数据（可以自己虚构并插入几万行）的表上，执行一个没有使用索引的 `WHERE` 查询，并使用 `LIKE '%...%'`。然后去查看你的慢查询日志文件，找到你刚刚执行的这条记录。

3.  **使用 `mysqldumpslow`**：
    对你生成的慢查询日志文件，练习使用 `mysqldumpslow` 命令，并尝试按不同的方式（执行时间、扫描行数）进行排序，看看结果有何不同。
