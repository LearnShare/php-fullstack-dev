# 6.5.4 性能监控

## 概述

性能监控是确保应用稳定运行的关键。本节介绍性能指标、监控工具、性能分析方法，以及优化建议。

## 性能指标

性能监控需要收集多个维度的指标，全面了解应用的性能状况。

### 1. 响应时间（Response Time）

响应时间是衡量应用性能的核心指标，包括平均响应时间、P95、P99 等分位数。

```php
<?php
declare(strict_types=1);

class ResponseTimeMonitor
{
    private float $startTime;
    private array $timings = [];
    
    public function __construct()
    {
        $this->startTime = microtime(true);
    }
    
    /**
     * 获取已用时间
     */
    public function getElapsedTime(): float
    {
        return microtime(true) - $this->startTime;
    }
    
    /**
     * 记录响应时间
     */
    public function record(string $endpoint, float $time): void
    {
        $this->timings[] = [
            'endpoint' => $endpoint,
            'time' => $time,
            'timestamp' => microtime(true),
        ];
        
        // 记录到日志
        error_log(sprintf(
            "Response time for %s: %.4fs",
            $endpoint,
            $time
        ));
        
        // 慢请求告警
        if ($time > 1.0) {
            $this->alertSlowRequest($endpoint, $time);
        }
    }
    
    /**
     * 获取统计信息
     */
    public function getStatistics(): array
    {
        if (empty($this->timings)) {
            return [];
        }
        
        $times = array_column($this->timings, 'time');
        sort($times);
        $count = count($times);
        
        return [
            'count' => $count,
            'avg' => array_sum($times) / $count,
            'min' => min($times),
            'max' => max($times),
            'p50' => $times[(int)($count * 0.5)],
            'p95' => $times[(int)($count * 0.95)],
            'p99' => $times[(int)($count * 0.99)],
        ];
    }
    
    /**
     * 按端点分组统计
     */
    public function getStatisticsByEndpoint(): array
    {
        $grouped = [];
        
        foreach ($this->timings as $timing) {
            $endpoint = $timing['endpoint'];
            if (!isset($grouped[$endpoint])) {
                $grouped[$endpoint] = [];
            }
            $grouped[$endpoint][] = $timing['time'];
        }
        
        $result = [];
        foreach ($grouped as $endpoint => $times) {
            sort($times);
            $count = count($times);
            
            $result[$endpoint] = [
                'count' => $count,
                'avg' => array_sum($times) / $count,
                'min' => min($times),
                'max' => max($times),
                'p50' => $times[(int)($count * 0.5)],
                'p95' => $times[(int)($count * 0.95)],
                'p99' => $times[(int)($count * 0.99)],
            ];
        }
        
        return $result;
    }
    
    private function alertSlowRequest(string $endpoint, float $time): void
    {
        // 发送告警（邮件、Slack、钉钉等）
        error_log(sprintf(
            "ALERT: Slow request detected - %s took %.4fs",
            $endpoint,
            $time
        ));
    }
}

// 使用示例
$monitor = new ResponseTimeMonitor();
$start = microtime(true);
// ... 处理请求
$monitor->record('/api/users', microtime(true) - $start);

$stats = $monitor->getStatistics();
print_r($stats);
```

### 2. 吞吐量（Throughput）

吞吐量表示单位时间内处理的请求数，通常用 RPS（Requests Per Second）或 QPS（Queries Per Second）表示。

```php
<?php
declare(strict_types=1);

class ThroughputMonitor
{
    private int $requestCount = 0;
    private float $startTime;
    private array $rpsHistory = [];
    
    public function __construct()
    {
        $this->startTime = microtime(true);
    }
    
    /**
     * 增加请求计数
     */
    public function increment(): void
    {
        $this->requestCount++;
        
        // 每秒记录一次 RPS
        $now = microtime(true);
        $elapsed = $now - $this->startTime;
        
        if ($elapsed >= 1.0) {
            $rps = $this->requestCount / $elapsed;
            $this->rpsHistory[] = [
                'timestamp' => $now,
                'rps' => $rps,
            ];
            
            // 重置计数器
            $this->requestCount = 0;
            $this->startTime = $now;
        }
    }
    
    /**
     * 获取当前吞吐量
     */
    public function getThroughput(): float
    {
        $elapsed = microtime(true) - $this->startTime;
        if ($elapsed == 0) {
            return 0.0;
        }
        return $this->requestCount / $elapsed;
    }
    
    /**
     * 获取平均吞吐量
     */
    public function getAverageThroughput(): float
    {
        if (empty($this->rpsHistory)) {
            return $this->getThroughput();
        }
        
        $rpsValues = array_column($this->rpsHistory, 'rps');
        return array_sum($rpsValues) / count($rpsValues);
    }
    
    /**
     * 获取吞吐量历史
     */
    public function getHistory(): array
    {
        return $this->rpsHistory;
    }
}

// 使用示例
$monitor = new ThroughputMonitor();
$monitor->increment();
echo "Current RPS: " . $monitor->getThroughput() . "\n";
```

### 3. 内存使用（Memory Usage）

监控内存使用情况，包括当前使用量、峰值使用量、内存泄漏检测等。

```php
<?php
declare(strict_types=1);

class MemoryMonitor
{
    private array $snapshots = [];
    
    /**
     * 获取当前内存使用情况
     */
    public function getMemoryUsage(): array
    {
        $current = memory_get_usage(true);
        $peak = memory_get_peak_usage(true);
        $limit = $this->parseMemoryLimit(ini_get('memory_limit'));
        
        return [
            'current' => $current,
            'current_formatted' => $this->formatBytes($current),
            'peak' => $peak,
            'peak_formatted' => $this->formatBytes($peak),
            'limit' => $limit,
            'limit_formatted' => $this->formatBytes($limit),
            'usage_percent' => $limit > 0 ? ($current / $limit * 100) : 0,
        ];
    }
    
    /**
     * 记录内存快照
     */
    public function snapshot(string $label = ''): void
    {
        $this->snapshots[] = [
            'label' => $label,
            'timestamp' => microtime(true),
            'memory' => memory_get_usage(true),
            'peak' => memory_get_peak_usage(true),
        ];
    }
    
    /**
     * 检测内存泄漏
     */
    public function detectLeak(): ?array
    {
        if (count($this->snapshots) < 2) {
            return null;
        }
        
        $first = $this->snapshots[0];
        $last = end($this->snapshots);
        
        $memoryIncrease = $last['memory'] - $first['memory'];
        
        if ($memoryIncrease > 10 * 1024 * 1024) { // 超过 10MB
            return [
                'leak_detected' => true,
                'memory_increase' => $memoryIncrease,
                'memory_increase_formatted' => $this->formatBytes($memoryIncrease),
                'first_snapshot' => $first,
                'last_snapshot' => $last,
            ];
        }
        
        return null;
    }
    
    /**
     * 解析内存限制字符串
     */
    private function parseMemoryLimit(string $limit): int
    {
        $limit = trim($limit);
        $last = strtolower($limit[strlen($limit) - 1]);
        $value = (int)$limit;
        
        switch ($last) {
            case 'g':
                $value *= 1024 * 1024 * 1024;
                break;
            case 'm':
                $value *= 1024 * 1024;
                break;
            case 'k':
                $value *= 1024;
                break;
        }
        
        return $value;
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

// 使用示例
$monitor = new MemoryMonitor();
$monitor->snapshot('start');
// ... 执行代码
$monitor->snapshot('end');

$usage = $monitor->getMemoryUsage();
print_r($usage);

$leak = $monitor->detectLeak();
if ($leak) {
    print_r($leak);
}
```

### 4. CPU 使用率

监控 CPU 使用情况，识别 CPU 密集型操作。

```php
<?php
declare(strict_types=1);

class CpuMonitor
{
    /**
     * 获取 CPU 使用率（需要系统命令支持）
     */
    public function getCpuUsage(): ?float
    {
        if (PHP_OS_FAMILY === 'Linux') {
            $load = sys_getloadavg();
            if ($load !== false) {
                // 获取 CPU 核心数
                $cores = (int)shell_exec('nproc');
                if ($cores > 0) {
                    // 1 分钟平均负载 / CPU 核心数
                    return ($load[0] / $cores) * 100;
                }
            }
        }
        
        return null;
    }
    
    /**
     * 获取进程 CPU 使用率
     */
    public function getProcessCpuUsage(): ?float
    {
        if (function_exists('sys_getloadavg')) {
            $pid = getmypid();
            $stat = file_get_contents("/proc/{$pid}/stat");
            $parts = explode(' ', $stat);
            
            // utime + stime（用户时间 + 系统时间）
            $totalTime = $parts[13] + $parts[14];
            
            // 获取系统总时间
            $cpuInfo = file_get_contents('/proc/stat');
            preg_match('/cpu\s+(\d+)\s+(\d+)\s+(\d+)\s+(\d+)/', $cpuInfo, $matches);
            $systemTotal = array_sum(array_slice($matches, 1));
            
            if ($systemTotal > 0) {
                return ($totalTime / $systemTotal) * 100;
            }
        }
        
        return null;
    }
}

// 使用示例
$monitor = new CpuMonitor();
$cpuUsage = $monitor->getCpuUsage();
if ($cpuUsage !== null) {
    echo "CPU Usage: " . round($cpuUsage, 2) . "%\n";
}
```

### 5. 错误率（Error Rate）

监控错误率，包括 HTTP 错误码、异常数量等。

```php
<?php
declare(strict_types=1);

class ErrorRateMonitor
{
    private int $totalRequests = 0;
    private int $errorRequests = 0;
    private array $errorTypes = [];
    
    /**
     * 记录请求
     */
    public function recordRequest(int $httpCode): void
    {
        $this->totalRequests++;
        
        if ($httpCode >= 400) {
            $this->errorRequests++;
            
            $errorType = $this->getErrorType($httpCode);
            if (!isset($this->errorTypes[$errorType])) {
                $this->errorTypes[$errorType] = 0;
            }
            $this->errorTypes[$errorType]++;
        }
    }
    
    /**
     * 获取错误率
     */
    public function getErrorRate(): float
    {
        if ($this->totalRequests == 0) {
            return 0.0;
        }
        
        return ($this->errorRequests / $this->totalRequests) * 100;
    }
    
    /**
     * 获取错误统计
     */
    public function getErrorStatistics(): array
    {
        return [
            'total_requests' => $this->totalRequests,
            'error_requests' => $this->errorRequests,
            'error_rate' => $this->getErrorRate(),
            'error_types' => $this->errorTypes,
        ];
    }
    
    private function getErrorType(int $httpCode): string
    {
        if ($httpCode >= 500) {
            return '5xx';
        } elseif ($httpCode >= 400) {
            return '4xx';
        }
        return 'unknown';
    }
}

// 使用示例
$monitor = new ErrorRateMonitor();
$monitor->recordRequest(200);
$monitor->recordRequest(404);
$monitor->recordRequest(500);

$stats = $monitor->getErrorStatistics();
print_r($stats);
```

## 监控工具

### 1. APM（应用性能监控）工具

APM 工具提供全面的应用性能监控，包括响应时间、吞吐量、错误率、数据库查询等。

#### New Relic

New Relic 是流行的 APM 工具，提供实时性能监控和分析。

```php
<?php
declare(strict_types=1);

// 检查 New Relic 扩展是否加载
if (extension_loaded('newrelic')) {
    // 设置应用名称
    newrelic_set_appname("My Application");
    
    // 添加自定义参数
    newrelic_add_custom_parameter("environment", "production");
    newrelic_add_custom_parameter("version", "1.0.0");
    
    // 开始自定义事务
    newrelic_start_transaction("My Transaction");
    
    // 记录自定义指标
    newrelic_custom_metric("Custom/Metric", 123.45);
    
    // 添加自定义属性
    newrelic_add_custom_attribute("user_id", 12345);
    
    // 业务逻辑
    // ...
    
    // 结束事务
    newrelic_end_transaction();
    
    // 记录异常
    try {
        // 可能抛出异常的代码
    } catch (Exception $e) {
        newrelic_notice_error($e->getMessage(), $e);
        throw $e;
    }
}
```

#### Datadog

Datadog 提供 APM、基础设施监控和日志管理。

```php
<?php
declare(strict_types=1);

// 使用 Datadog APM（需要安装 datadog/dd-trace 包）
// composer require datadog/dd-trace

use DDTrace\GlobalTracer;

// 初始化追踪
$tracer = GlobalTracer::get();

// 创建 span
$span = $tracer->startSpan('my_operation', [
    'tags' => [
        'service' => 'my-service',
        'resource' => 'my-resource',
    ],
]);

try {
    // 业务逻辑
    // ...
    
    $span->setTag('status', 'success');
} catch (Exception $e) {
    $span->setTag('error', true);
    $span->setTag('error.message', $e->getMessage());
    throw $e;
} finally {
    $span->finish();
}
```

#### Sentry

Sentry 专注于错误追踪和性能监控。

```php
<?php
declare(strict_types=1);

// 使用 Sentry（需要安装 sentry/sentry 包）
// composer require sentry/sentry

use Sentry\SentrySdk;
use Sentry\State\Scope;

\Sentry\init([
    'dsn' => 'https://your-dsn@sentry.io/project-id',
    'traces_sample_rate' => 1.0, // 性能追踪采样率
]);

// 记录性能事务
$transaction = \Sentry\startTransaction([
    'name' => 'my_operation',
    'op' => 'http.server',
]);

try {
    // 业务逻辑
    // ...
    
    $transaction->setStatus(\Sentry\TransactionStatus::success());
} catch (Exception $e) {
    $transaction->setStatus(\Sentry\TransactionStatus::internalError());
    \Sentry\captureException($e);
    throw $e;
} finally {
    $transaction->finish();
}
```

#### 自定义 APM 集成类

```php
<?php
declare(strict_types=1);

/**
 * APM 工具统一接口
 */
interface APMInterface
{
    public function startTransaction(string $name, array $context = []): void;
    public function endTransaction(): void;
    public function addCustomMetric(string $name, float $value): void;
    public function addCustomAttribute(string $key, mixed $value): void;
    public function recordException(\Throwable $exception): void;
}

/**
 * New Relic 实现
 */
class NewRelicAPM implements APMInterface
{
    public function startTransaction(string $name, array $context = []): void
    {
        if (extension_loaded('newrelic')) {
            newrelic_start_transaction($name);
            foreach ($context as $key => $value) {
                newrelic_add_custom_parameter($key, $value);
            }
        }
    }
    
    public function endTransaction(): void
    {
        if (extension_loaded('newrelic')) {
            newrelic_end_transaction();
        }
    }
    
    public function addCustomMetric(string $name, float $value): void
    {
        if (extension_loaded('newrelic')) {
            newrelic_custom_metric($name, $value);
        }
    }
    
    public function addCustomAttribute(string $key, mixed $value): void
    {
        if (extension_loaded('newrelic')) {
            newrelic_add_custom_attribute($key, (string)$value);
        }
    }
    
    public function recordException(\Throwable $exception): void
    {
        if (extension_loaded('newrelic')) {
            newrelic_notice_error($exception->getMessage(), $exception);
        }
    }
}

/**
 * APM 管理器
 */
class APMManager
{
    private ?APMInterface $apm = null;
    
    public function __construct(?APMInterface $apm = null)
    {
        $this->apm = $apm ?? $this->detectAPM();
    }
    
    private function detectAPM(): ?APMInterface
    {
        if (extension_loaded('newrelic')) {
            return new NewRelicAPM();
        }
        
        return null;
    }
    
    public function monitor(string $name, callable $callback): mixed
    {
        if ($this->apm) {
            $this->apm->startTransaction($name);
        }
        
        try {
            $result = $callback();
            return $result;
        } catch (\Throwable $e) {
            if ($this->apm) {
                $this->apm->recordException($e);
            }
            throw $e;
        } finally {
            if ($this->apm) {
                $this->apm->endTransaction();
            }
        }
    }
}

// 使用示例
$apm = new APMManager();
$result = $apm->monitor('user_processing', function() {
    // 业务逻辑
    return processUsers();
});
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

## 监控方案

### 1. 实时监控方案

实时监控能够及时发现性能问题，快速响应。

#### 实时监控系统

```php
<?php
declare(strict_types=1);

/**
 * 实时性能监控系统
 */
class RealTimeMonitor
{
    private array $metrics = [];
    private array $alerts = [];
    private float $alertThreshold = 1.0; // 响应时间阈值（秒）
    
    /**
     * 记录性能指标
     */
    public function record(string $endpoint, float $responseTime, int $memoryUsage, int $httpCode): void
    {
        $timestamp = time();
        
        $this->metrics[] = [
            'timestamp' => $timestamp,
            'endpoint' => $endpoint,
            'response_time' => $responseTime,
            'memory_usage' => $memoryUsage,
            'http_code' => $httpCode,
        ];
        
        // 检查是否需要告警
        if ($responseTime > $this->alertThreshold) {
            $this->triggerAlert($endpoint, $responseTime);
        }
        
        // 保持最近 1 小时的数据
        $this->cleanup();
    }
    
    /**
     * 获取实时统计
     */
    public function getRealTimeStats(int $windowSeconds = 60): array
    {
        $cutoff = time() - $windowSeconds;
        $recentMetrics = array_filter(
            $this->metrics,
            fn($m) => $m['timestamp'] >= $cutoff
        );
        
        if (empty($recentMetrics)) {
            return [];
        }
        
        $responseTimes = array_column($recentMetrics, 'response_time');
        $memoryUsages = array_column($recentMetrics, 'memory_usage');
        $httpCodes = array_column($recentMetrics, 'http_code');
        
        $errorCount = count(array_filter($httpCodes, fn($code) => $code >= 400));
        
        return [
            'window_seconds' => $windowSeconds,
            'request_count' => count($recentMetrics),
            'avg_response_time' => array_sum($responseTimes) / count($responseTimes),
            'max_response_time' => max($responseTimes),
            'min_response_time' => min($responseTimes),
            'avg_memory_usage' => array_sum($memoryUsages) / count($memoryUsages),
            'error_rate' => ($errorCount / count($recentMetrics)) * 100,
            'rps' => count($recentMetrics) / $windowSeconds,
        ];
    }
    
    /**
     * 触发告警
     */
    private function triggerAlert(string $endpoint, float $responseTime): void
    {
        $alert = [
            'timestamp' => time(),
            'endpoint' => $endpoint,
            'response_time' => $responseTime,
            'threshold' => $this->alertThreshold,
        ];
        
        $this->alerts[] = $alert;
        
        // 发送告警通知
        $this->sendAlert($alert);
    }
    
    /**
     * 发送告警通知
     */
    private function sendAlert(array $alert): void
    {
        // 实现告警通知（邮件、Slack、钉钉等）
        error_log(sprintf(
            "ALERT: Slow request - %s took %.4fs (threshold: %.4fs)",
            $alert['endpoint'],
            $alert['response_time'],
            $alert['threshold']
        ));
    }
    
    /**
     * 清理旧数据
     */
    private function cleanup(): void
    {
        $cutoff = time() - 3600; // 保留 1 小时
        $this->metrics = array_filter(
            $this->metrics,
            fn($m) => $m['timestamp'] >= $cutoff
        );
    }
    
    /**
     * 获取告警列表
     */
    public function getAlerts(int $limit = 10): array
    {
        return array_slice(array_reverse($this->alerts), 0, $limit);
    }
}

// 使用示例
$monitor = new RealTimeMonitor();

// 记录请求
$monitor->record('/api/users', 0.5, 1024 * 1024, 200);
$monitor->record('/api/users', 1.5, 2048 * 1024, 200); // 触发告警

// 获取实时统计
$stats = $monitor->getRealTimeStats(60);
print_r($stats);
```

### 2. 历史数据分析

历史数据分析帮助识别性能趋势，发现长期问题。

#### 性能数据存储

```php
<?php
declare(strict_types=1);

/**
 * 性能数据存储
 */
class PerformanceDataStore
{
    private string $storagePath;
    
    public function __construct(string $storagePath = '/var/log/performance')
    {
        $this->storagePath = $storagePath;
        
        if (!is_dir($this->storagePath)) {
            mkdir($this->storagePath, 0755, true);
        }
    }
    
    /**
     * 存储性能数据
     */
    public function store(array $data): void
    {
        $date = date('Y-m-d');
        $file = $this->storagePath . "/performance_{$date}.json";
        
        $existing = [];
        if (file_exists($file)) {
            $existing = json_decode(file_get_contents($file), true) ?? [];
        }
        
        $existing[] = [
            'timestamp' => time(),
            'data' => $data,
        ];
        
        file_put_contents($file, json_encode($existing, JSON_PRETTY_PRINT));
    }
    
    /**
     * 获取历史数据
     */
    public function getHistory(string $startDate, string $endDate): array
    {
        $start = strtotime($startDate);
        $end = strtotime($endDate);
        
        $data = [];
        $current = $start;
        
        while ($current <= $end) {
            $date = date('Y-m-d', $current);
            $file = $this->storagePath . "/performance_{$date}.json";
            
            if (file_exists($file)) {
                $dayData = json_decode(file_get_contents($file), true) ?? [];
                $data = array_merge($data, $dayData);
            }
            
            $current = strtotime('+1 day', $current);
        }
        
        return $data;
    }
    
    /**
     * 分析性能趋势
     */
    public function analyzeTrend(string $startDate, string $endDate): array
    {
        $history = $this->getHistory($startDate, $endDate);
        
        if (empty($history)) {
            return [];
        }
        
        $responseTimes = [];
        $memoryUsages = [];
        $errorRates = [];
        
        foreach ($history as $record) {
            $data = $record['data'];
            $responseTimes[] = $data['response_time'] ?? 0;
            $memoryUsages[] = $data['memory_usage'] ?? 0;
            $errorRates[] = $data['error_rate'] ?? 0;
        }
        
        return [
            'period' => [
                'start' => $startDate,
                'end' => $endDate,
            ],
            'response_time' => [
                'avg' => array_sum($responseTimes) / count($responseTimes),
                'min' => min($responseTimes),
                'max' => max($responseTimes),
                'trend' => $this->calculateTrend($responseTimes),
            ],
            'memory_usage' => [
                'avg' => array_sum($memoryUsages) / count($memoryUsages),
                'min' => min($memoryUsages),
                'max' => max($memoryUsages),
                'trend' => $this->calculateTrend($memoryUsages),
            ],
            'error_rate' => [
                'avg' => array_sum($errorRates) / count($errorRates),
                'min' => min($errorRates),
                'max' => max($errorRates),
                'trend' => $this->calculateTrend($errorRates),
            ],
        ];
    }
    
    /**
     * 计算趋势（上升/下降/稳定）
     */
    private function calculateTrend(array $values): string
    {
        if (count($values) < 2) {
            return 'stable';
        }
        
        $firstHalf = array_slice($values, 0, (int)(count($values) / 2));
        $secondHalf = array_slice($values, (int)(count($values) / 2));
        
        $firstAvg = array_sum($firstHalf) / count($firstHalf);
        $secondAvg = array_sum($secondHalf) / count($secondHalf);
        
        $change = (($secondAvg - $firstAvg) / $firstAvg) * 100;
        
        if ($change > 5) {
            return 'increasing';
        } elseif ($change < -5) {
            return 'decreasing';
        } else {
            return 'stable';
        }
    }
}

// 使用示例
$store = new PerformanceDataStore();

// 存储数据
$store->store([
    'endpoint' => '/api/users',
    'response_time' => 0.5,
    'memory_usage' => 1024 * 1024,
    'error_rate' => 0.1,
]);

// 分析趋势
$trend = $store->analyzeTrend('2024-01-01', '2024-01-31');
print_r($trend);
```

### 3. 监控仪表板

创建监控仪表板，可视化性能指标。

#### 监控仪表板数据接口

```php
<?php
declare(strict_types=1);

/**
 * 监控仪表板
 */
class MonitoringDashboard
{
    private RealTimeMonitor $realTimeMonitor;
    private PerformanceDataStore $dataStore;
    
    public function __construct(
        RealTimeMonitor $realTimeMonitor,
        PerformanceDataStore $dataStore
    ) {
        $this->realTimeMonitor = $realTimeMonitor;
        $this->dataStore = $dataStore;
    }
    
    /**
     * 获取仪表板数据
     */
    public function getDashboardData(): array
    {
        return [
            'realtime' => $this->realTimeMonitor->getRealTimeStats(300), // 最近 5 分钟
            'alerts' => $this->realTimeMonitor->getAlerts(10),
            'trends' => $this->dataStore->analyzeTrend(
                date('Y-m-d', strtotime('-7 days')),
                date('Y-m-d')
            ),
            'endpoints' => $this->getEndpointStats(),
        ];
    }
    
    /**
     * 获取端点统计
     */
    private function getEndpointStats(): array
    {
        $trend = $this->dataStore->analyzeTrend(
            date('Y-m-d', strtotime('-1 day')),
            date('Y-m-d')
        );
        
        // 从历史数据中提取端点统计
        // 这里简化处理，实际应从数据存储中查询
        return [
            '/api/users' => [
                'avg_response_time' => 0.5,
                'request_count' => 1000,
                'error_rate' => 0.1,
            ],
            '/api/posts' => [
                'avg_response_time' => 0.3,
                'request_count' => 2000,
                'error_rate' => 0.05,
            ],
        ];
    }
}

// 使用示例（可作为 API 端点）
$dashboard = new MonitoringDashboard($monitor, $store);
$data = $dashboard->getDashboardData();

// 返回 JSON 供前端使用
header('Content-Type: application/json');
echo json_encode($data, JSON_PRETTY_PRINT);
```

## 优化建议

### 1. 数据库优化

- 使用索引：为常用查询字段创建索引
- 避免 N+1 查询：使用批量查询或预加载
- 使用批量操作：减少数据库往返次数
- 优化慢查询：分析慢查询日志，优化 SQL

### 2. 缓存优化

- 使用 OPcache：启用并优化 OPcache 配置
- 实现应用层缓存：缓存计算结果和数据库查询结果
- 使用 Redis 缓存：分布式缓存，提升性能

### 3. 代码优化

- 避免不必要的计算：缓存计算结果
- 使用生成器处理大数据：减少内存占用
- 优化算法复杂度：选择更高效的算法

### 4. 服务配置优化

- PHP-FPM 配置：优化进程池设置
- Nginx 配置：启用缓存、压缩、静态文件优化
- 系统参数：优化文件描述符、TCP 参数等

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
