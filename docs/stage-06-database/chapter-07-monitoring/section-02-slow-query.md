# 6.7.2 慢查询分析

## 概述

慢查询是影响数据库性能的主要因素之一。通过分析慢查询，可以识别性能瓶颈、优化查询语句、提高系统性能。

本节详细介绍慢查询的概念、启用慢查询日志、分析慢查询、优化慢查询等，帮助零基础学员理解慢查询分析和优化方法。

**主要内容**：
- 慢查询的概念和影响
- 启用慢查询日志
- 分析慢查询日志
- 优化慢查询
- 慢查询监控
- 完整示例和最佳实践

---

## 特性

- **问题识别**：识别性能问题
- **查询优化**：优化查询性能
- **性能提升**：提升系统性能
- **问题预防**：预防性能问题
- **数据驱动**：基于数据的优化

---

## 慢查询概念

### 什么是慢查询

慢查询是指执行时间超过设定阈值的查询语句。

**慢查询的影响**：

- **性能下降**：影响系统性能
- **资源占用**：占用系统资源
- **用户体验**：影响用户体验
- **系统稳定性**：影响系统稳定性

### 慢查询阈值

慢查询阈值是指判断查询是否为慢查询的时间阈值。

**默认值**：

- **MySQL 默认**：10 秒
- **建议值**：1-2 秒
- **根据业务调整**：根据业务需求调整

---

## 启用慢查询日志

### 配置慢查询日志

在 MySQL 配置文件中启用慢查询日志。

**配置文件（my.cnf）**：

```ini
# 启用慢查询日志
slow_query_log = 1

# 慢查询日志文件路径
slow_query_log_file = /var/log/mysql/slow-query.log

# 慢查询阈值（秒）
long_query_time = 1

# 记录未使用索引的查询
log_queries_not_using_indexes = 1

# 记录慢查询管理语句
log_slow_admin_statements = 1
```

### 动态启用

可以在运行时动态启用慢查询日志。

**SQL 命令**：

```sql
-- 启用慢查询日志
SET GLOBAL slow_query_log = 'ON';

-- 设置慢查询阈值（秒）
SET GLOBAL long_query_time = 1;

-- 设置慢查询日志文件
SET GLOBAL slow_query_log_file = '/var/log/mysql/slow-query.log';

-- 记录未使用索引的查询
SET GLOBAL log_queries_not_using_indexes = 'ON';

-- 查看当前配置
SHOW VARIABLES LIKE 'slow_query%';
SHOW VARIABLES LIKE 'long_query_time';
```

---

## 分析慢查询日志

### 日志格式

慢查询日志的格式：

```
# Time: 2024-01-01T10:00:00.123456Z
# User@Host: root[root] @ localhost []
# Thread_id: 1
# Query_time: 2.345678  Lock_time: 0.000123  Rows_sent: 100  Rows_examined: 10000
SET timestamp=1704110400;
SELECT * FROM users WHERE email = 'user@example.com';
```

**字段说明**：

- **Time**：查询执行时间
- **User@Host**：用户和主机
- **Thread_id**：线程 ID
- **Query_time**：查询执行时间（秒）
- **Lock_time**：锁等待时间（秒）
- **Rows_sent**：返回行数
- **Rows_examined**：扫描行数

### 使用 mysqldumpslow 分析

MySQL 提供了 `mysqldumpslow` 工具分析慢查询日志。

**基本用法**：

```bash
# 查看最慢的 10 条查询
mysqldumpslow -s t -t 10 /var/log/mysql/slow-query.log

# 按执行次数排序
mysqldumpslow -s c -t 10 /var/log/mysql/slow-query.log

# 按平均执行时间排序
mysqldumpslow -s at -t 10 /var/log/mysql/slow-query.log

# 查看特定用户的慢查询
mysqldumpslow -a -g 'root' /var/log/mysql/slow-query.log
```

**参数说明**：

- **-s**：排序方式（t=时间，c=次数，at=平均时间）
- **-t**：显示前 N 条
- **-a**：不抽象数字和字符串
- **-g**：只显示匹配的查询

### 使用 pt-query-digest 分析

Percona Toolkit 提供了 `pt-query-digest` 工具，功能更强大。

**安装**：

```bash
# 安装 Percona Toolkit
# Ubuntu/Debian
sudo apt-get install percona-toolkit

# CentOS/RHEL
sudo yum install percona-toolkit
```

**基本用法**：

```bash
# 分析慢查询日志
pt-query-digest /var/log/mysql/slow-query.log

# 输出到文件
pt-query-digest /var/log/mysql/slow-query.log > report.txt

# 分析最近 24 小时的慢查询
pt-query-digest --since 24h /var/log/mysql/slow-query.log

# 分析特定时间范围
pt-query-digest --since '2024-01-01 00:00:00' --until '2024-01-02 00:00:00' /var/log/mysql/slow-query.log
```

---

## 优化慢查询

### 识别问题

通过慢查询日志识别问题。

**常见问题**：

- **全表扫描**：没有使用索引
- **索引不当**：索引选择不当
- **JOIN 优化**：JOIN 查询优化
- **子查询优化**：子查询优化

### 添加索引

为慢查询添加合适的索引。

**示例**：

```sql
-- 慢查询
SELECT * FROM users WHERE email = 'user@example.com';

-- 添加索引
CREATE INDEX idx_email ON users(email);

-- 优化后的查询
SELECT * FROM users WHERE email = 'user@example.com';
```

### 优化查询语句

优化查询语句本身。

**示例**：

```sql
-- 慢查询（全表扫描）
SELECT * FROM users WHERE YEAR(created_at) = 2024;

-- 优化后（使用索引）
SELECT * FROM users WHERE created_at >= '2024-01-01' AND created_at < '2025-01-01';
```

### 优化 JOIN

优化 JOIN 查询。

**示例**：

```sql
-- 慢查询（没有索引）
SELECT u.*, p.* 
FROM users u 
JOIN posts p ON u.id = p.user_id 
WHERE u.status = 'active';

-- 优化后（添加索引）
CREATE INDEX idx_user_id ON posts(user_id);
CREATE INDEX idx_status ON users(status);

SELECT u.*, p.* 
FROM users u 
JOIN posts p ON u.id = p.user_id 
WHERE u.status = 'active';
```

---

## 慢查询监控

### 监控慢查询数量

监控慢查询数量。

**SQL 查询**：

```sql
-- 查看慢查询数量
SHOW STATUS LIKE 'Slow_queries';

-- 查看慢查询统计
SELECT * FROM performance_schema.events_statements_summary_by_digest 
WHERE avg_timer_wait > 1000000000000 
ORDER BY avg_timer_wait DESC 
LIMIT 10;
```

### PHP 监控实现

使用 PHP 监控慢查询。

```php
<?php
declare(strict_types=1);

class SlowQueryMonitor
{
    private PDO $pdo;
    private Redis $redis;
    private int $threshold;
    
    public function __construct(PDO $pdo, Redis $redis, int $threshold = 1)
    {
        $this->pdo = $pdo;
        $this->redis = $redis;
        $this->threshold = $threshold;
    }
    
    public function getSlowQueryCount(): int
    {
        $stmt = $this->pdo->query("SHOW STATUS LIKE 'Slow_queries'");
        $result = $stmt->fetch(PDO::FETCH_ASSOC);
        return (int)($result['Value'] ?? 0);
    }
    
    public function analyzeSlowQueries(): array
    {
        $queries = [];
        
        // 从 Performance Schema 获取慢查询
        $sql = "
            SELECT 
                digest_text,
                count_star,
                sum_timer_wait / 1000000000000 as avg_time,
                sum_rows_examined / count_star as avg_rows_examined
            FROM performance_schema.events_statements_summary_by_digest
            WHERE avg_timer_wait > :threshold
            ORDER BY sum_timer_wait DESC
            LIMIT 10
        ";
        
        $stmt = $this->pdo->prepare($sql);
        $stmt->execute(['threshold' => $this->threshold * 1000000000000]);
        
        while ($row = $stmt->fetch(PDO::FETCH_ASSOC)) {
            $queries[] = [
                'query' => $row['digest_text'],
                'count' => (int)$row['count_star'],
                'avg_time' => (float)$row['avg_time'],
                'avg_rows_examined' => (int)$row['avg_rows_examined']
            ];
        }
        
        return $queries;
    }
    
    public function checkAlerts(): array
    {
        $alerts = [];
        $count = $this->getSlowQueryCount();
        
        // 检查慢查询数量
        if ($count > 100) {
            $alerts[] = "慢查询数量过多: {$count}";
        }
        
        // 分析慢查询
        $slowQueries = $this->analyzeSlowQueries();
        if (!empty($slowQueries)) {
            foreach ($slowQueries as $query) {
                if ($query['avg_time'] > $this->threshold * 2) {
                    $alerts[] = "发现严重慢查询: {$query['query']} (平均时间: {$query['avg_time']}秒)";
                }
            }
        }
        
        return $alerts;
    }
}

// 使用
$monitor = new SlowQueryMonitor($pdo, $redis, 1);
$alerts = $monitor->checkAlerts();

if (!empty($alerts)) {
    foreach ($alerts as $alert) {
        error_log("慢查询告警: {$alert}");
    }
}
```

---

## 完整示例

### 慢查询分析工具

```php
<?php
declare(strict_types=1);

class SlowQueryAnalyzer
{
    private string $logFile;
    
    public function __construct(string $logFile)
    {
        $this->logFile = $logFile;
    }
    
    public function parseLog(): array
    {
        $queries = [];
        $content = file_get_contents($this->logFile);
        
        // 解析慢查询日志
        $pattern = '/# Time: (.+?)\n# User@Host: (.+?)\n# Thread_id: (\d+)\n# Query_time: ([\d.]+)\s+Lock_time: ([\d.]+)\s+Rows_sent: (\d+)\s+Rows_examined: (\d+)\nSET timestamp=(\d+);\n(.+?);/s';
        
        preg_match_all($pattern, $content, $matches, PREG_SET_ORDER);
        
        foreach ($matches as $match) {
            $queries[] = [
                'time' => $match[1],
                'user' => $match[2],
                'thread_id' => (int)$match[3],
                'query_time' => (float)$match[4],
                'lock_time' => (float)$match[5],
                'rows_sent' => (int)$match[6],
                'rows_examined' => (int)$match[7],
                'timestamp' => (int)$match[8],
                'query' => trim($match[9])
            ];
        }
        
        return $queries;
    }
    
    public function analyze(): array
    {
        $queries = $this->parseLog();
        $analysis = [
            'total' => count($queries),
            'total_time' => 0,
            'avg_time' => 0,
            'max_time' => 0,
            'top_queries' => []
        ];
        
        if (empty($queries)) {
            return $analysis;
        }
        
        // 计算统计信息
        foreach ($queries as $query) {
            $analysis['total_time'] += $query['query_time'];
            $analysis['max_time'] = max($analysis['max_time'], $query['query_time']);
        }
        
        $analysis['avg_time'] = $analysis['total_time'] / $analysis['total'];
        
        // 找出最慢的查询
        usort($queries, function($a, $b) {
            return $b['query_time'] <=> $a['query_time'];
        });
        
        $analysis['top_queries'] = array_slice($queries, 0, 10);
        
        return $analysis;
    }
    
    public function generateReport(): string
    {
        $analysis = $this->analyze();
        $report = "慢查询分析报告\n";
        $report .= "================\n\n";
        $report .= "总查询数: {$analysis['total']}\n";
        $report .= "总执行时间: " . number_format($analysis['total_time'], 2) . " 秒\n";
        $report .= "平均执行时间: " . number_format($analysis['avg_time'], 2) . " 秒\n";
        $report .= "最大执行时间: " . number_format($analysis['max_time'], 2) . " 秒\n\n";
        $report .= "最慢的 10 条查询:\n";
        $report .= "----------------\n";
        
        foreach ($analysis['top_queries'] as $i => $query) {
            $report .= ($i + 1) . ". 执行时间: " . number_format($query['query_time'], 2) . " 秒\n";
            $report .= "   扫描行数: {$query['rows_examined']}\n";
            $report .= "   返回行数: {$query['rows_sent']}\n";
            $report .= "   查询语句: {$query['query']}\n\n";
        }
        
        return $report;
    }
}

// 使用
$analyzer = new SlowQueryAnalyzer('/var/log/mysql/slow-query.log');
$report = $analyzer->generateReport();
echo $report;
```

---

## 使用场景

### 性能优化

通过分析慢查询优化性能。

### 问题排查

通过慢查询排查性能问题。

### 容量规划

通过慢查询数据规划容量。

---

## 注意事项

### 日志文件大小

注意慢查询日志文件大小，定期清理。

### 性能影响

启用慢查询日志会有一定的性能影响。

### 日志安全

注意慢查询日志的安全性，可能包含敏感信息。

---

## 常见问题

### 如何启用慢查询日志？

在配置文件中设置或使用 SQL 命令动态启用。

### 如何分析慢查询？

使用 `mysqldumpslow` 或 `pt-query-digest` 工具。

### 如何优化慢查询？

添加索引、优化查询语句、优化 JOIN 等。

---

## 最佳实践

### 合理设置阈值

根据业务需求设置合理的慢查询阈值。

### 定期分析

定期分析慢查询日志，识别问题。

### 持续优化

持续优化慢查询，提高性能。

---

## 练习任务

1. **启用慢查询日志**
   - 配置慢查询日志
   - 动态启用慢查询日志
   - 测试慢查询记录
   - 验证日志功能

2. **分析慢查询日志**
   - 使用 mysqldumpslow 分析
   - 使用 pt-query-digest 分析
   - 编写分析脚本
   - 生成分析报告

3. **优化慢查询**
   - 识别慢查询问题
   - 添加索引优化
   - 优化查询语句
   - 验证优化效果

4. **慢查询监控**
   - 实现慢查询监控
   - 设置告警机制
   - 测试监控功能
   - 验证告警效果

5. **综合应用**
   - 创建一个完整的慢查询分析系统
   - 实现所有功能
   - 测试各种场景
   - 编写最佳实践文档

---

**相关章节**：

- [6.7.1 数据库性能监控概述](section-01-overview.md)
- [6.7.3 性能指标监控](section-03-metrics.md)
- [6.1.3 索引设计](../chapter-01-database-design/section-03-index-design.md)
