# 6.5.4 性能监控

## 概述

性能监控是确保应用稳定运行的关键。本节介绍性能指标、监控工具、性能分析方法，以及优化建议。

## 性能指标

### 1. 响应时间

```php
<?php
declare(strict_types=1);

class ResponseTimeMonitor
{
    private float $startTime;
    
    public function __construct()
    {
        $this->startTime = microtime(true);
    }
    
    public function getElapsedTime(): float
    {
        return microtime(true) - $this->startTime;
    }
    
    public function log(string $endpoint): void
    {
        $time = $this->getElapsedTime();
        error_log("Response time for {$endpoint}: {$time}s");
        
        // 记录到监控系统
        if ($time > 1.0) {
            $this->alertSlowRequest($endpoint, $time);
        }
    }
    
    private function alertSlowRequest(string $endpoint, float $time): void
    {
        // 发送告警
    }
}
```

### 2. 吞吐量

```php
<?php
declare(strict_types=1);

class ThroughputMonitor
{
    private int $requestCount = 0;
    private float $startTime;
    
    public function __construct()
    {
        $this->startTime = microtime(true);
    }
    
    public function increment(): void
    {
        $this->requestCount++;
    }
    
    public function getThroughput(): float
    {
        $elapsed = microtime(true) - $this->startTime;
        return $this->requestCount / $elapsed;
    }
}
```

### 3. 内存使用

```php
<?php
declare(strict_types=1);

function getMemoryUsage(): array
{
    return [
        'current' => memory_get_usage(true),
        'peak' => memory_get_peak_usage(true),
        'limit' => ini_get('memory_limit'),
    ];
}
```

## 监控工具

### 1. APM（应用性能监控）

```php
<?php
declare(strict_types=1);

// 集成 New Relic
if (extension_loaded('newrelic')) {
    newrelic_set_appname("My Application");
    newrelic_add_custom_parameter("environment", "production");
}

// 自定义事务
newrelic_start_transaction("My Transaction");
// ... 业务逻辑
newrelic_end_transaction();
```

### 2. 自定义监控

```php
<?php
declare(strict_types=1);

class PerformanceMonitor
{
    private array $metrics = [];
    
    public function startTimer(string $name): void
    {
        $this->metrics[$name] = [
            'start' => microtime(true),
            'memory_start' => memory_get_usage(true),
        ];
    }
    
    public function endTimer(string $name): void
    {
        if (!isset($this->metrics[$name])) {
            return;
        }
        
        $metric = $this->metrics[$name];
        $metric['duration'] = microtime(true) - $metric['start'];
        $metric['memory_used'] = memory_get_usage(true) - $metric['memory_start'];
        $metric['memory_peak'] = memory_get_peak_usage(true);
        
        $this->logMetric($name, $metric);
    }
    
    private function logMetric(string $name, array $metric): void
    {
        error_log(sprintf(
            "Metric [%s]: Duration=%.4fs, Memory=%s, Peak=%s",
            $name,
            $metric['duration'],
            $this->formatBytes($metric['memory_used']),
            $this->formatBytes($metric['memory_peak'])
        ));
    }
    
    private function formatBytes(int $bytes): string
    {
        $units = ['B', 'KB', 'MB', 'GB'];
        $i = 0;
        while ($bytes >= 1024 && $i < count($units) - 1) {
            $bytes /= 1024;
            $i++;
        }
        return round($bytes, 2) . ' ' . $units[$i];
    }
}

// 使用
$monitor = new PerformanceMonitor();
$monitor->startTimer('user_processing');
// ... 业务逻辑
$monitor->endTimer('user_processing');
```

## 性能分析

### 1. 慢查询日志

```php
<?php
declare(strict_types=1);

class SlowQueryLogger
{
    private float $threshold;
    
    public function __construct(float $threshold = 1.0)
    {
        $this->threshold = $threshold;
    }
    
    public function logQuery(string $sql, float $duration, array $params = []): void
    {
        if ($duration > $this->threshold) {
            error_log(sprintf(
                "Slow query detected: %.4fs\nSQL: %s\nParams: %s",
                $duration,
                $sql,
                json_encode($params)
            ));
        }
    }
}
```

### 2. 性能分析报告

```php
<?php
declare(strict_types=1);

class PerformanceReport
{
    private array $data = [];
    
    public function collect(string $endpoint, float $duration, int $memory): void
    {
        if (!isset($this->data[$endpoint])) {
            $this->data[$endpoint] = [
                'count' => 0,
                'total_duration' => 0,
                'max_duration' => 0,
                'min_duration' => PHP_FLOAT_MAX,
                'total_memory' => 0,
            ];
        }
        
        $this->data[$endpoint]['count']++;
        $this->data[$endpoint]['total_duration'] += $duration;
        $this->data[$endpoint]['max_duration'] = max($this->data[$endpoint]['max_duration'], $duration);
        $this->data[$endpoint]['min_duration'] = min($this->data[$endpoint]['min_duration'], $duration);
        $this->data[$endpoint]['total_memory'] += $memory;
    }
    
    public function generate(): array
    {
        $report = [];
        
        foreach ($this->data as $endpoint => $stats) {
            $report[$endpoint] = [
                'count' => $stats['count'],
                'avg_duration' => $stats['total_duration'] / $stats['count'],
                'max_duration' => $stats['max_duration'],
                'min_duration' => $stats['min_duration'],
                'avg_memory' => $stats['total_memory'] / $stats['count'],
            ];
        }
        
        return $report;
    }
}
```

## 优化建议

### 1. 数据库优化

- 使用索引
- 避免 N+1 查询
- 使用批量操作
- 优化慢查询

### 2. 缓存优化

- 使用 OPcache
- 实现应用层缓存
- 使用 Redis 缓存

### 3. 代码优化

- 避免不必要的计算
- 使用生成器处理大数据
- 优化算法复杂度

## 完整示例

```php
<?php
declare(strict_types=1);

class ApplicationMonitor
{
    private PerformanceMonitor $performance;
    private SlowQueryLogger $queryLogger;
    private array $metrics = [];
    
    public function __construct()
    {
        $this->performance = new PerformanceMonitor();
        $this->queryLogger = new SlowQueryLogger(1.0);
    }
    
    public function monitorRequest(callable $handler): mixed
    {
        $this->performance->startTimer('request');
        
        try {
            $result = $handler();
            return $result;
        } finally {
            $this->performance->endTimer('request');
            $this->collectMetrics();
        }
    }
    
    public function monitorQuery(string $sql, callable $executor, array $params = []): mixed
    {
        $start = microtime(true);
        
        try {
            $result = $executor();
            return $result;
        } finally {
            $duration = microtime(true) - $start;
            $this->queryLogger->logQuery($sql, $duration, $params);
        }
    }
    
    private function collectMetrics(): void
    {
        $this->metrics[] = [
            'timestamp' => time(),
            'memory' => memory_get_usage(true),
            'peak_memory' => memory_get_peak_usage(true),
        ];
    }
    
    public function getMetrics(): array
    {
        return $this->metrics;
    }
}

// 使用
$monitor = new ApplicationMonitor();

$result = $monitor->monitorRequest(function() {
    // 处理请求
    return processRequest();
});

$users = $monitor->monitorQuery(
    "SELECT * FROM users WHERE id = ?",
    fn() => $pdo->prepare("SELECT * FROM users WHERE id = ?")->execute([1]),
    [1]
);
```

## 最佳实践

1. **持续监控**：建立持续监控机制
2. **设置阈值**：为关键指标设置阈值
3. **告警机制**：异常时及时告警
4. **定期分析**：定期分析性能数据
5. **优化迭代**：基于监控数据优化

## 注意事项

1. 监控本身不应影响性能
2. 合理设置采样率
3. 保护监控数据安全
4. 定期清理历史数据

## 练习

1. 实现一个性能监控类，记录请求响应时间和内存使用。

2. 创建一个慢查询日志系统，记录执行时间超过阈值的查询。

3. 实现一个性能报告生成器，分析应用性能数据。

4. 集成 APM 工具，监控应用性能。
