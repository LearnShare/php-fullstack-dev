# 6.7.3 性能指标监控

## 概述

性能指标监控是数据库运维的核心工作，通过监控关键性能指标，可以及时发现问题、优化性能、预防故障。

本节详细介绍数据库性能指标监控的概念、关键指标、监控方法、告警机制等，帮助零基础学员理解性能指标监控的重要性和实现方式。

**主要内容**：
- 性能指标的概念和分类
- 关键性能指标（QPS、TPS、连接数、缓存命中率等）
- 监控方法和工具
- 告警机制和阈值设置
- 性能分析和优化
- 完整示例和最佳实践

---

## 特性

- **全面监控**：监控所有关键指标
- **实时监控**：实时监控性能指标
- **及时告警**：及时告警异常情况
- **数据分析**：分析性能数据
- **性能优化**：指导性能优化

---

## 性能指标分类

### 查询性能指标

查询性能指标反映数据库查询的执行情况。

**主要指标**：

- **QPS**：每秒查询数
- **TPS**：每秒事务数
- **平均响应时间**：平均查询响应时间
- **慢查询率**：慢查询占比

### 连接性能指标

连接性能指标反映数据库连接的使用情况。

**主要指标**：

- **当前连接数**：当前活跃连接数
- **最大连接数**：最大允许连接数
- **连接使用率**：连接使用率
- **连接等待数**：等待连接的请求数

### 缓存性能指标

缓存性能指标反映数据库缓存的使用情况。

**主要指标**：

- **缓存命中率**：查询缓存命中率
- **缓存大小**：缓存使用大小
- **缓存淘汰率**：缓存淘汰频率

### 资源使用指标

资源使用指标反映数据库服务器的资源使用情况。

**主要指标**：

- **CPU 使用率**：CPU 使用率
- **内存使用率**：内存使用率
- **磁盘 I/O**：磁盘读写速度
- **网络 I/O**：网络读写速度

---

## 关键性能指标

### QPS（每秒查询数）

QPS 反映数据库的查询负载。

**计算方法**：

```sql
-- 查看当前查询数
SHOW STATUS LIKE 'Questions';

-- 计算 QPS
-- QPS = (当前 Questions - 上次 Questions) / 时间间隔
```

**PHP 实现**：

```php
<?php
declare(strict_types=1);

class QPSMonitor
{
    private PDO $pdo;
    private Redis $redis;
    
    public function __construct(PDO $pdo, Redis $redis)
    {
        $this->pdo = $pdo;
        $this->redis = $redis;
    }
    
    public function getQPS(): float
    {
        $current = $this->getQuestions();
        $last = $this->redis->get('metrics:questions');
        $lastTime = $this->redis->get('metrics:questions:time');
        
        if ($last === false || $lastTime === false) {
            $this->redis->set('metrics:questions', $current);
            $this->redis->set('metrics:questions:time', time());
            return 0;
        }
        
        $timeDiff = time() - (int)$lastTime;
        if ($timeDiff == 0) {
            return 0;
        }
        
        $qps = ($current - (int)$last) / $timeDiff;
        
        $this->redis->set('metrics:questions', $current);
        $this->redis->set('metrics:questions:time', time());
        
        return $qps;
    }
    
    private function getQuestions(): int
    {
        $stmt = $this->pdo->query("SHOW STATUS LIKE 'Questions'");
        $result = $stmt->fetch(PDO::FETCH_ASSOC);
        return (int)($result['Value'] ?? 0);
    }
}
```

### TPS（每秒事务数）

TPS 反映数据库的事务处理能力。

**计算方法**：

```sql
-- 查看提交和回滚数
SHOW STATUS LIKE 'Com_commit';
SHOW STATUS LIKE 'Com_rollback';

-- 计算 TPS
-- TPS = (Com_commit + Com_rollback) / 时间间隔
```

**PHP 实现**：

```php
<?php
declare(strict_types=1);

class TPSMonitor
{
    private PDO $pdo;
    private Redis $redis;
    
    public function __construct(PDO $pdo, Redis $redis)
    {
        $this->pdo = $pdo;
        $this->redis = $redis;
    }
    
    public function getTPS(): float
    {
        $current = $this->getTransactions();
        $last = $this->redis->get('metrics:transactions');
        $lastTime = $this->redis->get('metrics:transactions:time');
        
        if ($last === false || $lastTime === false) {
            $this->redis->set('metrics:transactions', $current);
            $this->redis->set('metrics:transactions:time', time());
            return 0;
        }
        
        $timeDiff = time() - (int)$lastTime;
        if ($timeDiff == 0) {
            return 0;
        }
        
        $tps = ($current - (int)$last) / $timeDiff;
        
        $this->redis->set('metrics:transactions', $current);
        $this->redis->set('metrics:transactions:time', time());
        
        return $tps;
    }
    
    private function getTransactions(): int
    {
        $commit = $this->getStatus('Com_commit');
        $rollback = $this->getStatus('Com_rollback');
        return $commit + $rollback;
    }
    
    private function getStatus(string $name): int
    {
        $stmt = $this->pdo->query("SHOW STATUS LIKE '{$name}'");
        $result = $stmt->fetch(PDO::FETCH_ASSOC);
        return (int)($result['Value'] ?? 0);
    }
}
```

### 连接数监控

监控数据库连接数。

**SQL 查询**：

```sql
-- 查看当前连接数
SHOW STATUS LIKE 'Threads_connected';

-- 查看最大连接数
SHOW VARIABLES LIKE 'max_connections';

-- 查看历史最大连接数
SHOW STATUS LIKE 'Max_used_connections';
```

**PHP 实现**：

```php
<?php
declare(strict_types=1);

class ConnectionMonitor
{
    private PDO $pdo;
    
    public function __construct(PDO $pdo)
    {
        $this->pdo = $pdo;
    }
    
    public function getConnectionMetrics(): array
    {
        return [
            'current' => $this->getStatus('Threads_connected'),
            'max_allowed' => $this->getVariable('max_connections'),
            'max_used' => $this->getStatus('Max_used_connections'),
            'usage_rate' => $this->getUsageRate()
        ];
    }
    
    private function getUsageRate(): float
    {
        $current = $this->getStatus('Threads_connected');
        $max = $this->getVariable('max_connections');
        
        if ($max == 0) {
            return 0;
        }
        
        return ($current / $max) * 100;
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
}
```

### 缓存命中率

监控查询缓存命中率。

**SQL 查询**：

```sql
-- 查看查询缓存统计
SHOW STATUS LIKE 'Qcache%';

-- 计算命中率
-- 命中率 = Qcache_hits / (Qcache_hits + Com_select)
```

**PHP 实现**：

```php
<?php
declare(strict_types=1);

class CacheHitRateMonitor
{
    private PDO $pdo;
    
    public function __construct(PDO $pdo)
    {
        $this->pdo = $pdo;
    }
    
    public function getCacheHitRate(): ?float
    {
        $hits = $this->getStatus('Qcache_hits');
        $selects = $this->getStatus('Com_select');
        
        $total = $hits + $selects;
        if ($total == 0) {
            return null;
        }
        
        return ($hits / $total) * 100;
    }
    
    private function getStatus(string $name): int
    {
        $stmt = $this->pdo->query("SHOW STATUS LIKE '{$name}'");
        $result = $stmt->fetch(PDO::FETCH_ASSOC);
        return (int)($result['Value'] ?? 0);
    }
}
```

---

## 监控系统实现

### 综合监控类

实现一个综合的性能指标监控系统。

```php
<?php
declare(strict_types=1);

class PerformanceMonitor
{
    private PDO $pdo;
    private Redis $redis;
    private array $thresholds;
    
    public function __construct(PDO $pdo, Redis $redis, array $thresholds = [])
    {
        $this->pdo = $pdo;
        $this->redis = $redis;
        $this->thresholds = array_merge([
            'qps_max' => 1000,
            'tps_max' => 100,
            'connection_usage_max' => 80,
            'cache_hit_rate_min' => 80,
            'slow_query_max' => 10
        ], $thresholds);
    }
    
    public function collectAllMetrics(): array
    {
        return [
            'qps' => $this->getQPS(),
            'tps' => $this->getTPS(),
            'connections' => $this->getConnectionMetrics(),
            'cache_hit_rate' => $this->getCacheHitRate(),
            'slow_queries' => $this->getSlowQueryCount(),
            'timestamp' => time()
        ];
    }
    
    public function checkAlerts(array $metrics): array
    {
        $alerts = [];
        
        // QPS 告警
        if ($metrics['qps'] > $this->thresholds['qps_max']) {
            $alerts[] = [
                'level' => 'warning',
                'metric' => 'qps',
                'value' => $metrics['qps'],
                'threshold' => $this->thresholds['qps_max'],
                'message' => "QPS 超过阈值: {$metrics['qps']} > {$this->thresholds['qps_max']}"
            ];
        }
        
        // TPS 告警
        if ($metrics['tps'] > $this->thresholds['tps_max']) {
            $alerts[] = [
                'level' => 'warning',
                'metric' => 'tps',
                'value' => $metrics['tps'],
                'threshold' => $this->thresholds['tps_max'],
                'message' => "TPS 超过阈值: {$metrics['tps']} > {$this->thresholds['tps_max']}"
            ];
        }
        
        // 连接数告警
        if ($metrics['connections']['usage_rate'] > $this->thresholds['connection_usage_max']) {
            $alerts[] = [
                'level' => 'critical',
                'metric' => 'connections',
                'value' => $metrics['connections']['usage_rate'],
                'threshold' => $this->thresholds['connection_usage_max'],
                'message' => "连接使用率超过阈值: {$metrics['connections']['usage_rate']}% > {$this->thresholds['connection_usage_max']}%"
            ];
        }
        
        // 缓存命中率告警
        if ($metrics['cache_hit_rate'] !== null && $metrics['cache_hit_rate'] < $this->thresholds['cache_hit_rate_min']) {
            $alerts[] = [
                'level' => 'warning',
                'metric' => 'cache_hit_rate',
                'value' => $metrics['cache_hit_rate'],
                'threshold' => $this->thresholds['cache_hit_rate_min'],
                'message' => "缓存命中率低于阈值: {$metrics['cache_hit_rate']}% < {$this->thresholds['cache_hit_rate_min']}%"
            ];
        }
        
        // 慢查询告警
        if ($metrics['slow_queries'] > $this->thresholds['slow_query_max']) {
            $alerts[] = [
                'level' => 'warning',
                'metric' => 'slow_queries',
                'value' => $metrics['slow_queries'],
                'threshold' => $this->thresholds['slow_query_max'],
                'message' => "慢查询数量超过阈值: {$metrics['slow_queries']} > {$this->thresholds['slow_query_max']}"
            ];
        }
        
        return $alerts;
    }
    
    public function saveMetrics(array $metrics): void
    {
        $key = 'metrics:' . date('Y-m-d-H-i');
        $this->redis->setex($key, 86400, json_encode($metrics));
        
        // 保存到历史记录
        $historyKey = 'metrics:history';
        $this->redis->lpush($historyKey, json_encode($metrics));
        $this->redis->ltrim($historyKey, 0, 1000); // 保留最近 1000 条
    }
    
    private function getQPS(): float
    {
        $monitor = new QPSMonitor($this->pdo, $this->redis);
        return $monitor->getQPS();
    }
    
    private function getTPS(): float
    {
        $monitor = new TPSMonitor($this->pdo, $this->redis);
        return $monitor->getTPS();
    }
    
    private function getConnectionMetrics(): array
    {
        $monitor = new ConnectionMonitor($this->pdo);
        return $monitor->getConnectionMetrics();
    }
    
    private function getCacheHitRate(): ?float
    {
        $monitor = new CacheHitRateMonitor($this->pdo);
        return $monitor->getCacheHitRate();
    }
    
    private function getSlowQueryCount(): int
    {
        $stmt = $this->pdo->query("SHOW STATUS LIKE 'Slow_queries'");
        $result = $stmt->fetch(PDO::FETCH_ASSOC);
        return (int)($result['Value'] ?? 0);
    }
}

// 使用
$monitor = new PerformanceMonitor($pdo, $redis);

// 收集指标
$metrics = $monitor->collectAllMetrics();

// 保存指标
$monitor->saveMetrics($metrics);

// 检查告警
$alerts = $monitor->checkAlerts($metrics);

if (!empty($alerts)) {
    foreach ($alerts as $alert) {
        error_log("性能告警 [{$alert['level']}]: {$alert['message']}");
    }
}
```

---

## 使用场景

### 性能监控

实时监控数据库性能指标。

### 容量规划

通过监控数据规划系统容量。

### 故障预防

通过监控预防系统故障。

---

## 注意事项

### 监控频率

合理设置监控频率，避免影响性能。

### 告警阈值

设置合理的告警阈值，避免误报。

### 数据保留

合理保留监控数据，便于分析。

---

## 常见问题

### 需要监控哪些指标？

- QPS、TPS
- 连接数
- 缓存命中率
- 慢查询数量

### 如何设置告警阈值？

根据业务需求和历史数据设置合理的阈值。

### 如何分析监控数据？

定期分析监控数据，识别趋势和异常。

---

## 最佳实践

### 全面监控

监控所有关键性能指标。

### 及时告警

设置及时告警，快速响应。

### 数据分析

定期分析监控数据，优化系统。

---

## 练习任务

1. **指标收集**
   - 实现 QPS 监控
   - 实现 TPS 监控
   - 实现连接数监控
   - 实现缓存命中率监控

2. **告警实现**
   - 实现告警机制
   - 设置告警阈值
   - 测试告警功能
   - 验证告警效果

3. **数据存储**
   - 实现指标存储
   - 实现历史数据查询
   - 测试存储功能
   - 验证数据准确性

4. **可视化展示**
   - 实现监控数据可视化
   - 创建监控面板
   - 测试可视化功能
   - 优化展示效果

5. **综合应用**
   - 创建一个完整的性能监控系统
   - 实现所有功能
   - 测试各种场景
   - 编写最佳实践文档

---

**相关章节**：

- [6.7.1 数据库性能监控概述](section-01-overview.md)
- [6.7.2 慢查询分析](section-02-slow-query.md)
- [6.6 数据库运维与管理](../chapter-06-operations/readme.md)
