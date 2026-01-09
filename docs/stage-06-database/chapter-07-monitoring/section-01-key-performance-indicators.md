# 6.7.1 核心性能指标 (Key Performance Indicators)

## 概述

对数据库进行性能监控，首先需要知道应该“看什么”。**性能指标 (Performance Indicators / Metrics)** 就是量化数据库健康状况和工作负载的一系列数据。作为开发者，理解这些核心指标不仅能帮助你与 DBA 或运维团队更有效地沟通，还能让你从宏观上判断应用的数据库访问模式是否健康。

我们将从四个维度来了解这些关键指标：**吞吐量、延迟、并发与连接、资源利用率**。

## 1. 吞吐量指标 (Throughput Metrics)

吞吐量指标衡量的是数据库在单位时间内处理任务的能力，直接反映了数据库的繁忙程度。

### QPS (Queries Per Second)

-   **定义**：每秒执行的 SQL 查询（Query）总数，包括 `SELECT`, `INSERT`, `UPDATE`, `DELETE` 等所有类型的查询。
-   **重要性**：这是衡量数据库总负载最直接的指标。QPS 的突增可能意味着业务流量高峰，但也可能是因为应用中出现了低效的轮询或 N+1 查询。
-   **如何查看 (MySQL)**：
    QPS 需要通过采集两个时间点的 `Questions` 状态值来计算。`Questions` 记录了自服务器启动以来执行的查询总数。
    ```sql
    -- 查看总查询数
    SHOW GLOBAL STATUS LIKE 'Questions';
    ```
    `QPS = (Questions_t2 - Questions_t1) / (t2 - t1)`

### TPS (Transactions Per Second)

-   **定义**：每秒提交的事务数。
-   **重要性**：TPS 通常更能代表业务层面的处理能力，尤其是在写操作密集的应用中。一个事务可能包含多个 SQL 查询，因此 TPS 通常远小于 QPS。
-   **如何查看 (MySQL)**：
    通过 `Com_commit` (提交的事务数) 和 `Com_rollback` (回滚的事务数) 来计算。
    ```sql
    SHOW GLOBAL STATUS LIKE 'Com_commit';
    SHOW GLOBAL STATUS LIKE 'Com_rollback';
    ```
    `TPS = (Com_commit_t2 - Com_commit_t1) / (t2 - t1)`

## 2. 延迟指标 (Latency Metrics)

延迟指标衡量的是操作的响应时间，直接关系到终端用户的体验。

### 慢查询 (Slow Queries)

-   **定义**：执行时间超过了预设阈值（由 `long_query_time` 参数定义，默认为 10 秒）的查询。
-   **重要性**：慢查询是性能问题的“头号公敌”。一条慢查询可能会长时间占用数据库资源，阻塞其他查询，导致应用响应缓慢甚至超时。**监控和消灭慢查询**是数据库优化的核心工作。
-   **如何查看**：主要通过开启和分析**慢查询日志 (Slow Query Log)** 来捕获。我们将在下一节详细讲解。

## 3. 并发与连接指标 (Concurrency & Connection Metrics)

这些指标反映了当前有多少客户端正在与数据库交互。

-   `Threads_connected`
    -   **定义**：当前打开的连接总数。
    -   **重要性**：如果这个值持续增长并接近 `max_connections` 的限制，说明应用可能存在连接泄漏，或者连接池配置不当。

-   `Threads_running`
    -   **定义**：正在**积极处理查询**的线程数（即非睡眠状态的连接）。
    -   **重要性**：这是衡量数据库**当前实时负载**的核心指标。如果这个值持续很高，特别是接近服务器的 CPU 核心数，表明数据库非常繁忙，可能存在大量慢查询或并发瓶颈。

-   `max_connections`
    -   **定义**：MySQL 配置允许的最大并发连接数。
    -   **重要性**：当 `Threads_connected` 达到这个值时，新的连接请求将被拒绝。

**如何查看 (MySQL)**：
```sql
-- 查看所有与线程相关的状态
SHOW GLOBAL STATUS LIKE 'Threads_%';

-- 查看最大连接数配置
SHOW VARIABLES LIKE 'max_connections';
```

## 4. InnoDB 资源利用率指标

对于使用 InnoDB 存储引擎的 MySQL 来说，**缓冲池 (Buffer Pool)** 的性能至关重要。

### 缓冲池命中率 (Buffer Pool Hit Rate)

-   **定义**：查询的数据在内存（Buffer Pool）中直接找到的比例。
-   **重要性**：InnoDB 尝试将最常用的数据和索引都缓存在内存中，以避免昂贵的磁盘 I/O。一个高的命中率（通常应 **> 99%**）表明 Buffer Pool 的大小是充足的，工作效率很高。如果命中率持续走低，说明数据库正在进行大量的磁盘读写，性能会急剧下降，这通常是 `innodb_buffer_pool_size` 配置过小的信号。
-   **如何计算**：
    1.  获取 `Innodb_buffer_pool_read_requests` (从内存读取的逻辑请求总数)。
    2.  获取 `Innodb_buffer_pool_reads` (因内存中没有而必须从磁盘读取的物理请求总数)。
    ```sql
    SHOW GLOBAL STATUS LIKE 'Innodb_buffer_pool_read%';
    ```
    `命中率 = 1 - (Innodb_buffer_pool_reads / Innodb_buffer_pool_read_requests)`

### 缓冲池等待 (Buffer Pool Waits)

-   `Innodb_buffer_pool_wait_free`
    -   **定义**：由于 Buffer Pool 已满，需要等待页面被刷新到磁盘以腾出空间才能继续操作的次数。
    -   **重要性**：如果这个值**持续不为 0 且不断增长**，这是一个非常明确的警报，说明 Buffer Pool 已经严重不足，无法应对当前的写入压力。

## 总结

监控数据库就像给病人做体检。通过持续观察这些核心指标，我们可以建立一个性能基线，并在指标出现异常波动时，及时介入，从而在小问题演变成大故障之前将其扼杀在摇篮里。在接下来的章节中，我们将学习如何使用慢查询日志和 `EXPLAIN` 等工具，来深入诊断由这些指标所暴露出的具体问题。

---

## 练习任务

1.  **获取你本地数据库的指标**：
    连接到你的本地 MySQL 实例，执行 `SHOW GLOBAL STATUS;` 命令。尝试从中找出本节提到的 `Questions`, `Com_commit`, `Threads_connected`, `Threads_running` 等指标，并记录下它们当前的值。

2.  **计算 QPS**：
    执行 `SHOW GLOBAL STATUS LIKE 'Questions';`，记下值。等待 10 秒后，再次执行该命令，记下新的值。用这两个值手动计算出这 10 秒内的平均 QPS 是多少。

3.  **理解命中率**：
    假设你的应用在启动后，`Innodb_buffer_pool_read_requests` 的值是 1,000,000，`Innodb_buffer_pool_reads` 的值是 5,000。请计算出当前的缓冲池命中率是多少。这个命中率是健康的吗？为什么？
