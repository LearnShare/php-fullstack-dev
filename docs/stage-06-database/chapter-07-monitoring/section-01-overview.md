# 6.7.1 数据库性能监控概述

## 概述

数据库性能监控是数据库运维的重要工作，用于识别性能瓶颈、优化查询性能、预防系统故障。通过监控数据库的性能指标，可以及时发现问题、优化系统、提高用户体验。

本节详细介绍数据库性能监控的概念、重要性、监控指标、监控工具、监控策略等，帮助零基础学员理解数据库性能监控的价值和实现方式。

**主要内容**：
- 性能监控的概念和重要性
- 监控指标（QPS、TPS、连接数、慢查询等）
- 监控工具和方法
- 监控策略和最佳实践
- 性能分析和优化
- 完整示例和最佳实践

---

## 特性

- **实时监控**：实时监控数据库性能
- **问题识别**：及时识别性能问题
- **性能优化**：指导性能优化
- **预防故障**：预防系统故障
- **数据驱动**：基于数据的决策

---

## 性能监控概念

### 什么是性能监控

性能监控是指持续收集和分析数据库的性能指标，以识别性能问题和优化机会。

**监控的目标**：

- **识别瓶颈**：识别性能瓶颈
- **优化性能**：指导性能优化
- **预防故障**：预防系统故障
- **提高体验**：提高用户体验

### 为什么需要性能监控

性能监控的重要性：

- **及时发现问题**：及时发现问题，快速响应
- **优化性能**：基于数据优化性能
- **预防故障**：预防系统故障
- **提高可靠性**：提高系统可靠性

---

## 监控指标

### QPS（每秒查询数）

QPS（Queries Per Second）是每秒执行的查询数。

**监控方法**：

```sql
-- 查看当前 QPS
SHOW GLOBAL STATUS LIKE 'Questions';

-- 计算 QPS
-- QPS = (当前 Questions - 上次 Questions) / 时间间隔
```

### TPS（每秒事务数）

TPS（Transactions Per Second）是每秒执行的事务数。

**监控方法**：

```sql
-- 查看当前事务数
SHOW GLOBAL STATUS LIKE 'Com_commit';
SHOW GLOBAL STATUS LIKE 'Com_rollback';

-- 计算 TPS
-- TPS = (Com_commit + Com_rollback) / 时间间隔
```

### 连接数

监控数据库连接数。

**监控方法**：

```sql
-- 查看当前连接数
SHOW STATUS LIKE 'Threads_connected';

-- 查看最大连接数
SHOW VARIABLES LIKE 'max_connections';

-- 查看历史最大连接数
SHOW STATUS LIKE 'Max_used_connections';
```

### 慢查询

监控慢查询数量和执行时间。

**监控方法**：

```sql
-- 查看慢查询数量
SHOW STATUS LIKE 'Slow_queries';

-- 查看慢查询日志
-- 需要启用慢查询日志
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 1;
```

### 缓存命中率

监控查询缓存命中率。

**监控方法**：

```sql
-- 查看查询缓存统计
SHOW STATUS LIKE 'Qcache%';

-- 计算命中率
-- 命中率 = Qcache_hits / (Qcache_hits + Com_select)
```

---

## 监控工具

### MySQL 内置工具

MySQL 提供了内置的监控工具。

**SHOW STATUS**：

```sql
-- 查看所有状态变量
SHOW STATUS;

-- 查看特定状态
SHOW STATUS LIKE 'Threads%';
```

**SHOW PROCESSLIST**：

```sql
-- 查看当前连接和查询
SHOW PROCESSLIST;

-- 查看详细信息
SHOW FULL PROCESSLIST;
```

**Performance Schema**：

```sql
-- 启用 Performance Schema
-- 在 my.cnf 中设置
performance_schema = ON

-- 查询性能数据
SELECT * FROM performance_schema.events_statements_summary_by_digest;
```

### 第三方监控工具

可以使用第三方监控工具。

**工具**：

- **MySQL Workbench**：MySQL 官方工具
- **phpMyAdmin**：Web 管理工具
- **Grafana + Prometheus**：监控系统
- **New Relic**：APM 工具

---

## 监控策略

### 监控频率

根据重要性确定监控频率。

**建议**：

- **关键指标**：实时监控
- **一般指标**：每分钟监控
- **历史数据**：每小时或每天监控

### 告警策略

设置合理的告警阈值。

**建议**：

- **连接数**：超过最大连接数的 80% 告警
- **慢查询**：慢查询数量突然增加告警
- **QPS**：QPS 异常波动告警

---

## 完整示例

### 性能监控脚本

```php
<?php
declare(strict_types=1);

class DatabaseMonitor
{
    private PDO $pdo;
    private Redis $redis;
    
    public function __construct(PDO $pdo, Redis $redis)
    {
        $this->pdo = $pdo;
        $this->redis = $redis;
    }
    
    public function collectMetrics(): array
    {
        $metrics = [];
        
        // QPS
        $questions = $this->getStatus('Questions');
        $metrics['qps'] = $this->calculateRate('questions', $questions);
        
        // 连接数
        $metrics['connections'] = $this->getStatus('Threads_connected');
        $metrics['max_connections'] = $this->getVariable('max_connections');
        
        // 慢查询
        $metrics['slow_queries'] = $this->getStatus('Slow_queries');
        
        // 缓存命中率
        $qcacheHits = $this->getStatus('Qcache_hits');
        $comSelect = $this->getStatus('Com_select');
        if ($qcacheHits + $comSelect > 0) {
            $metrics['cache_hit_rate'] = $qcacheHits / ($qcacheHits + $comSelect);
        }
        
        return $metrics;
    }
    
    private function getStatus(string $name): int
    {
        $stmt = $this->pdo->query("SHOW STATUS LIKE '{$name}'");
        $result = $stmt->fetch(PDO::FETCH_ASSOC);
        return (int)($result['Value'] ?? 0);
    }
    
    private function getVariable(string $name): int
    {
        $stmt = $this->pdo->query("SHOW VARIABLES LIKE '{$name}'");
        $result = $stmt->fetch(PDO::FETCH_ASSOC);
        return (int)($result['Value'] ?? 0);
    }
    
    private function calculateRate(string $key, int $current): float
    {
        $last = $this->redis->get("metrics:{$key}");
        $lastTime = $this->redis->get("metrics:{$key}:time");
        
        if ($last === false || $lastTime === false) {
            $this->redis->set("metrics:{$key}", $current);
            $this->redis->set("metrics:{$key}:time", time());
            return 0;
        }
        
        $timeDiff = time() - (int)$lastTime;
        if ($timeDiff == 0) {
            return 0;
        }
        
        $rate = ($current - (int)$last) / $timeDiff;
        
        $this->redis->set("metrics:{$key}", $current);
        $this->redis->set("metrics:{$key}:time", time());
        
        return $rate;
    }
    
    public function checkAlerts(array $metrics): array
    {
        $alerts = [];
        
        // 连接数告警
        if ($metrics['connections'] > $metrics['max_connections'] * 0.8) {
            $alerts[] = '连接数超过 80%';
        }
        
        // 慢查询告警
        if ($metrics['slow_queries'] > 100) {
            $alerts[] = '慢查询数量过多';
        }
        
        return $alerts;
    }
}

// 使用
$monitor = new DatabaseMonitor($pdo, $redis);
$metrics = $monitor->collectMetrics();
$alerts = $monitor->checkAlerts($metrics);

if (!empty($alerts)) {
    // 发送告警
    foreach ($alerts as $alert) {
        error_log("数据库告警: {$alert}");
    }
}
```

---

## 使用场景

### 性能优化

通过监控识别性能瓶颈，指导优化。

### 容量规划

通过监控数据规划系统容量。

### 故障预防

通过监控预防系统故障。

---

## 注意事项

### 监控开销

注意监控本身的开销，不要影响系统性能。

### 告警阈值

设置合理的告警阈值，避免误报。

### 数据保留

合理保留监控数据，便于分析。

---

## 常见问题

### 如何监控数据库性能？

使用 MySQL 内置工具或第三方监控工具。

### 需要监控哪些指标？

- QPS、TPS
- 连接数
- 慢查询
- 缓存命中率

### 如何设置告警？

根据业务需求设置合理的告警阈值。

---

## 最佳实践

### 全面监控

监控所有关键指标。

### 及时告警

设置及时告警，快速响应。

### 数据分析

定期分析监控数据，优化系统。

---

## 练习任务

1. **监控指标收集**
   - 实现指标收集
   - 实现指标计算
   - 测试监控功能
   - 验证数据准确性

2. **告警实现**
   - 实现告警机制
   - 设置告警阈值
   - 测试告警功能
   - 验证告警效果

3. **监控可视化**
   - 实现监控数据可视化
   - 创建监控面板
   - 测试可视化功能
   - 优化展示效果

4. **性能分析**
   - 分析监控数据
   - 识别性能瓶颈
   - 提出优化建议
   - 编写分析报告

5. **综合应用**
   - 创建一个完整的监控系统
   - 实现所有功能
   - 测试各种场景
   - 编写最佳实践文档

---

**相关章节**：

- [6.7.2 慢查询分析](section-02-slow-query.md)
- [6.7.3 性能指标监控](section-03-metrics.md)
- [6.6 数据库运维与管理](../chapter-06-operations/readme.md)
