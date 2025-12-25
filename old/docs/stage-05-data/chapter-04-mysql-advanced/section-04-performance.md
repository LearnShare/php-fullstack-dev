# 5.4.4 性能优化

## 概述

MySQL 性能优化是数据库应用的关键。本节详细介绍查询优化、索引优化、分区表、执行计划分析，以及性能优化的最佳实践。

## 查询优化

### EXPLAIN 分析

```sql
-- 分析查询执行计划
EXPLAIN SELECT * FROM users WHERE email = 'alice@example.com';

-- 详细分析
EXPLAIN FORMAT=JSON SELECT * FROM users WHERE email = 'alice@example.com';
```

### 执行计划解读

| 列 | 说明 |
| :--- | :--- |
| `type` | 访问类型（ALL、index、range、ref、const） |
| `key` | 使用的索引 |
| `rows` | 扫描的行数 |
| `Extra` | 额外信息（Using index、Using filesort 等） |

### 查询优化技巧

```sql
-- 1. 使用索引
SELECT * FROM users WHERE email = 'alice@example.com';  -- 使用索引

-- 2. 避免 SELECT *
SELECT id, name, email FROM users;  -- 只查询需要的字段

-- 3. 使用 LIMIT
SELECT * FROM users ORDER BY created_at DESC LIMIT 10;

-- 4. 避免函数在 WHERE 子句中使用
-- 不推荐
SELECT * FROM users WHERE YEAR(created_at) = 2024;

-- 推荐
SELECT * FROM users WHERE created_at >= '2024-01-01' AND created_at < '2025-01-01';
```

## 索引优化

### 索引类型选择

```sql
-- 普通索引
CREATE INDEX idx_email ON users(email);

-- 唯一索引
CREATE UNIQUE INDEX uk_email ON users(email);

-- 复合索引
CREATE INDEX idx_user_status ON orders(user_id, status);

-- 覆盖索引（包含查询所需的所有字段）
CREATE INDEX idx_user_cover ON users(id, name, email);
```

### 索引使用原则

1. **最左前缀原则**：复合索引遵循最左前缀
2. **选择性高的字段**：优先为选择性高的字段创建索引
3. **避免过多索引**：索引会降低写入性能

## 分区表

### 分区类型

```sql
-- 范围分区
CREATE TABLE sales (
    id INT PRIMARY KEY,
    date DATE,
    amount DECIMAL(10,2)
) PARTITION BY RANGE (YEAR(date)) (
    PARTITION p2022 VALUES LESS THAN (2023),
    PARTITION p2023 VALUES LESS THAN (2024),
    PARTITION p2024 VALUES LESS THAN (2025),
    PARTITION p_future VALUES LESS THAN MAXVALUE
);

-- 哈希分区
CREATE TABLE users (
    id INT PRIMARY KEY,
    username VARCHAR(255)
) PARTITION BY HASH(id) PARTITIONS 4;
```

## 执行计划分析

### 慢查询日志

```sql
-- 启用慢查询日志
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 1;  -- 1 秒

-- 查看慢查询
SELECT * FROM mysql.slow_log ORDER BY start_time DESC LIMIT 10;
```

### 性能分析

```php
<?php
declare(strict_types=1);

class QueryAnalyzer
{
    private PDO $pdo;

    public function analyzeQuery(string $sql): array
    {
        $stmt = $this->pdo->query("EXPLAIN FORMAT=JSON {$sql}");
        $plan = json_decode($stmt->fetchColumn(), true);
        
        return [
            'cost' => $plan['query_block']['cost_info']['query_cost'] ?? 0,
            'rows' => $plan['query_block']['table']['rows_examined_per_scan'] ?? 0,
            'index' => $plan['query_block']['table']['possible_keys'] ?? [],
        ];
    }
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

class DatabaseOptimizer
{
    private PDO $pdo;

    public function optimizeQuery(string $sql): array
    {
        // 分析执行计划
        $stmt = $this->pdo->query("EXPLAIN {$sql}");
        $plan = $stmt->fetchAll();
        
        $suggestions = [];
        
        foreach ($plan as $row) {
            if ($row['type'] === 'ALL') {
                $suggestions[] = "全表扫描，建议添加索引";
            }
            
            if ($row['Extra'] === 'Using filesort') {
                $suggestions[] = "使用文件排序，考虑添加 ORDER BY 索引";
            }
        }
        
        return $suggestions;
    }
}
```

## 注意事项

1. **索引维护**：定期分析索引使用情况
2. **查询优化**：避免全表扫描
3. **分区使用**：合理使用分区表
4. **监控慢查询**：持续监控和优化

## 练习

1. 分析现有查询的执行计划，优化慢查询。

2. 创建合适的索引，提升查询性能。

3. 实现一个查询优化工具，自动分析查询性能。

4. 设计一个分区表，提升大数据量查询性能。
