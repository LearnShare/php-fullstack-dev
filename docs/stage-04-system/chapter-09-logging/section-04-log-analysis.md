# 4.9.4 日志分析与监控

## 概述

日志分析是从日志中提取有用信息的过程，用于问题排查、性能监控、安全审计等。在实际应用中，我们需要分析日志以了解系统行为、发现问题、优化性能。理解日志分析的方法、工具、监控策略等，对于构建可观测的系统至关重要。

本节介绍日志分析的概念、方法、工具，以及监控和告警的实现，帮助零基础学员掌握日志分析和监控的方法。

**主要内容**：
- 日志分析概述（什么是日志分析、为什么需要日志分析）
- 日志聚合
- 日志搜索和过滤
- 日志统计和分析
- 监控和告警
- 日志可视化
- 实际应用场景和最佳实践

## 特性

- **信息提取**：从日志中提取有用信息
- **问题定位**：快速定位问题
- **趋势分析**：分析系统趋势
- **监控告警**：及时发现和通知问题

## 语法/定义

### 日志分析要点

日志分析没有单一的语法定义，而是通过工具和方法实现。

## 基本用法

### 示例 1：简单的日志分析工具

```php
<?php
declare(strict_types=1);

class LogAnalyzer
{
    private string $logFile;
    
    public function __construct(string $logFile)
    {
        $this->logFile = $logFile;
    }
    
    /**
     * 统计日志级别
     */
    public function countByLevel(): array
    {
        $counts = [];
        $handle = fopen($this->logFile, 'r');
        
        if ($handle === false) {
            return $counts;
        }
        
        while (($line = fgets($handle)) !== false) {
            $entry = json_decode(trim($line), true);
            if ($entry === null || !isset($entry['level'])) {
                continue;
            }
            
            $level = $entry['level'];
            $counts[$level] = ($counts[$level] ?? 0) + 1;
        }
        
        fclose($handle);
        return $counts;
    }
    
    /**
     * 搜索日志
     */
    public function search(string $keyword, ?string $level = null, ?int $limit = null): array
    {
        $results = [];
        $handle = fopen($this->logFile, 'r');
        
        if ($handle === false) {
            return $results;
        }
        
        while (($line = fgets($handle)) !== false) {
            if ($limit !== null && count($results) >= $limit) {
                break;
            }
            
            $entry = json_decode(trim($line), true);
            if ($entry === null) {
                continue;
            }
            
            // 级别过滤
            if ($level !== null && ($entry['level'] ?? '') !== $level) {
                continue;
            }
            
            // 关键词搜索
            $message = $entry['message'] ?? '';
            $jsonLine = json_encode($entry, JSON_UNESCAPED_UNICODE);
            
            if (stripos($message, $keyword) !== false || stripos($jsonLine, $keyword) !== false) {
                $results[] = $entry;
            }
        }
        
        fclose($handle);
        return $results;
    }
    
    /**
     * 获取错误日志
     */
    public function getErrors(?int $limit = 100): array
    {
        return $this->search('', 'ERROR', $limit);
    }
    
    /**
     * 统计时间段内的日志
     */
    public function countByTimeRange(string $startTime, string $endTime): int
    {
        $count = 0;
        $start = strtotime($startTime);
        $end = strtotime($endTime);
        
        $handle = fopen($this->logFile, 'r');
        if ($handle === false) {
            return $count;
        }
        
        while (($line = fgets($handle)) !== false) {
            $entry = json_decode(trim($line), true);
            if ($entry === null || !isset($entry['timestamp'])) {
                continue;
            }
            
            $timestamp = strtotime($entry['timestamp']);
            if ($timestamp >= $start && $timestamp <= $end) {
                $count++;
            }
        }
        
        fclose($handle);
        return $count;
    }
}

// 使用
$analyzer = new LogAnalyzer('/tmp/app.log');

// 统计日志级别
$counts = $analyzer->countByLevel();
echo "日志级别统计:\n";
print_r($counts);

// 搜索日志
$results = $analyzer->search('数据库', 'ERROR', 10);
echo "搜索结果:\n";
print_r($results);

// 获取错误日志
$errors = $analyzer->getErrors(20);
echo "错误日志数量: " . count($errors) . "\n";
```

**说明**：
- 实现简单的日志分析功能
- 支持搜索、过滤、统计等操作

### 示例 2：日志监控和告警

```php
<?php
declare(strict_types=1);

class LogMonitor
{
    private string $logFile;
    private int $errorThreshold;
    private int $timeWindow;  // 时间窗口（秒）
    
    public function __construct(string $logFile, int $errorThreshold = 10, int $timeWindow = 60)
    {
        $this->logFile = $logFile;
        $this->errorThreshold = $errorThreshold;
        $this->timeWindow = $timeWindow;
    }
    
    /**
     * 检查错误率
     */
    public function checkErrorRate(): array
    {
        $endTime = time();
        $startTime = $endTime - $this->timeWindow;
        
        $totalCount = 0;
        $errorCount = 0;
        
        $handle = fopen($this->logFile, 'r');
        if ($handle === false) {
            return ['status' => 'unknown', 'message' => 'Cannot open log file'];
        }
        
        // 从文件末尾读取（假设日志是追加写入的）
        fseek($handle, -min(1024 * 1024, filesize($this->logFile)), SEEK_END);
        
        while (($line = fgets($handle)) !== false) {
            $entry = json_decode(trim($line), true);
            if ($entry === null || !isset($entry['timestamp'])) {
                continue;
            }
            
            $timestamp = strtotime($entry['timestamp']);
            if ($timestamp < $startTime) {
                continue;
            }
            
            $totalCount++;
            $level = $entry['level'] ?? '';
            if (in_array($level, ['ERROR', 'CRITICAL'], true)) {
                $errorCount++;
            }
        }
        
        fclose($handle);
        
        $errorRate = $totalCount > 0 ? ($errorCount / $totalCount) * 100 : 0;
        
        return [
            'status' => $errorCount >= $this->errorThreshold ? 'alert' : 'ok',
            'total_count' => $totalCount,
            'error_count' => $errorCount,
            'error_rate' => $errorRate,
            'threshold' => $this->errorThreshold,
        ];
    }
    
    /**
     * 发送告警
     */
    public function sendAlert(string $message, array $details = []): void
    {
        // 实际实现中，可以发送邮件、Slack 通知等
        echo "告警: {$message}\n";
        if (!empty($details)) {
            print_r($details);
        }
        
        // 记录告警日志
        error_log("ALERT: {$message} " . json_encode($details, JSON_UNESCAPED_UNICODE));
    }
    
    /**
     * 监控并告警
     */
    public function monitor(): void
    {
        $result = $this->checkErrorRate();
        
        if ($result['status'] === 'alert') {
            $this->sendAlert('错误率过高', $result);
        }
    }
}

// 使用
$monitor = new LogMonitor('/tmp/app.log', 10, 60);  // 60秒内10个错误触发告警
$monitor->monitor();
```

**说明**：
- 实现日志监控功能
- 检查错误率并发送告警

### 示例 3：日志统计报告

```php
<?php
declare(strict_types=1);

class LogReport
{
    private string $logFile;
    
    public function __construct(string $logFile)
    {
        $this->logFile = $logFile;
    }
    
    /**
     * 生成统计报告
     */
    public function generateReport(?string $startTime = null, ?string $endTime = null): array
    {
        $start = $startTime ? strtotime($startTime) : strtotime('-1 day');
        $end = $endTime ? strtotime($endTime) : time();
        
        $stats = [
            'period' => [
                'start' => date('Y-m-d H:i:s', $start),
                'end' => date('Y-m-d H:i:s', $end),
            ],
            'total' => 0,
            'by_level' => [],
            'by_hour' => [],
            'errors' => [],
        ];
        
        $handle = fopen($this->logFile, 'r');
        if ($handle === false) {
            return $stats;
        }
        
        while (($line = fgets($handle)) !== false) {
            $entry = json_decode(trim($line), true);
            if ($entry === null || !isset($entry['timestamp'])) {
                continue;
            }
            
            $timestamp = strtotime($entry['timestamp']);
            if ($timestamp < $start || $timestamp > $end) {
                continue;
            }
            
            $stats['total']++;
            
            // 按级别统计
            $level = $entry['level'] ?? 'UNKNOWN';
            $stats['by_level'][$level] = ($stats['by_level'][$level] ?? 0) + 1;
            
            // 按小时统计
            $hour = date('H', $timestamp);
            $stats['by_hour'][$hour] = ($stats['by_hour'][$hour] ?? 0) + 1;
            
            // 记录错误
            if (in_array($level, ['ERROR', 'CRITICAL'], true)) {
                $stats['errors'][] = [
                    'timestamp' => $entry['timestamp'],
                    'message' => $entry['message'] ?? '',
                    'context' => $entry['context'] ?? [],
                ];
            }
        }
        
        fclose($handle);
        
        // 只保留最近100个错误
        $stats['errors'] = array_slice($stats['errors'], -100);
        
        return $stats;
    }
    
    /**
     * 打印报告
     */
    public function printReport(?string $startTime = null, ?string $endTime = null): void
    {
        $report = $this->generateReport($startTime, $endTime);
        
        echo "=== 日志统计报告 ===\n\n";
        echo "时间范围: {$report['period']['start']} 至 {$report['period']['end']}\n";
        echo "总日志数: {$report['total']}\n\n";
        
        echo "按级别统计:\n";
        foreach ($report['by_level'] as $level => $count) {
            echo "  {$level}: {$count}\n";
        }
        echo "\n";
        
        echo "按小时统计:\n";
        ksort($report['by_hour']);
        foreach ($report['by_hour'] as $hour => $count) {
            echo "  {$hour}:00 - {$count} 条\n";
        }
        echo "\n";
        
        echo "错误日志数量: " . count($report['errors']) . "\n";
        if (!empty($report['errors'])) {
            echo "最近错误:\n";
            foreach (array_slice($report['errors'], -5) as $error) {
                echo "  [{$error['timestamp']}] {$error['message']}\n";
            }
        }
    }
}

// 使用
$report = new LogReport('/tmp/app.log');
$report->printReport();
```

**说明**：
- 生成日志统计报告
- 包含按级别、按时间等维度的统计

### 示例 4：实时日志监控

```php
<?php
declare(strict_types=1);

class RealTimeLogMonitor
{
    private string $logFile;
    private int $lastPosition;
    
    public function __construct(string $logFile)
    {
        $this->logFile = $logFile;
        $this->lastPosition = filesize($logFile);
    }
    
    /**
     * 监控新日志
     */
    public function monitor(callable $callback, int $interval = 1): void
    {
        while (true) {
            $currentSize = filesize($this->logFile);
            
            if ($currentSize > $this->lastPosition) {
                // 读取新内容
                $handle = fopen($this->logFile, 'r');
                if ($handle !== false) {
                    fseek($handle, $this->lastPosition);
                    
                    while (($line = fgets($handle)) !== false) {
                        $entry = json_decode(trim($line), true);
                        if ($entry !== null) {
                            $callback($entry);
                        }
                    }
                    
                    fclose($handle);
                    $this->lastPosition = $currentSize;
                }
            }
            
            sleep($interval);
        }
    }
}

// 使用
$monitor = new RealTimeLogMonitor('/tmp/app.log');

$monitor->monitor(function (array $entry) {
    $level = $entry['level'] ?? '';
    $message = $entry['message'] ?? '';
    
    echo "[{$level}] {$message}\n";
    
    // 如果是错误，发送告警
    if ($level === 'ERROR') {
        echo "⚠️  错误检测到: {$message}\n";
    }
}, 1);  // 每秒检查一次
```

**说明**：
- 实现实时日志监控
- 监控新产生的日志并处理

## 使用场景

### 场景 1：问题排查

通过日志分析定位问题。

**示例**：

```php
<?php
declare(strict_types=1);

$analyzer = new LogAnalyzer('/tmp/app.log');

// 搜索相关错误
$errors = $analyzer->search('database', 'ERROR');
foreach ($errors as $error) {
    echo "错误: {$error['message']}\n";
    echo "时间: {$error['timestamp']}\n";
}
```

### 场景 2：性能监控

通过日志监控系统性能。

**示例**：

```php
<?php
declare(strict_types=1);

$monitor = new LogMonitor('/tmp/app.log');
$result = $monitor->checkErrorRate();

if ($result['status'] === 'alert') {
    echo "警告: 错误率过高 ({$result['error_rate']}%)\n";
}
```

## 注意事项

### 日志文件大小

大日志文件的分析可能较慢，考虑使用日志轮转或工具优化。

**示例**：

```php
<?php
declare(strict_types=1);

// 对于大文件，考虑使用流式读取
// 或使用专门的日志分析工具（如 ELK Stack）
```

### 实时监控性能

实时监控可能影响性能，应该合理设置监控间隔。

**示例**：

```php
<?php
declare(strict_types=1);

// 合理设置监控间隔
$monitor->monitor($callback, 5);  // 5秒检查一次，而不是1秒
```

## 常见问题

### 问题 1：如何分析日志？

**回答**：使用日志分析工具搜索、过滤、统计日志，提取有用信息。

**示例**：见"示例 1：简单的日志分析工具"

### 问题 2：如何实现日志监控？

**回答**：定期检查日志，统计错误率、数量等指标，超过阈值时发送告警。

**示例**：见"示例 2：日志监控和告警"

### 问题 3：如何处理大日志文件？

**回答**：使用日志轮转、流式读取、专门的日志分析工具（如 ELK Stack）等。

**示例**：

```php
<?php
declare(strict_types=1);

// 使用流式读取处理大文件
$handle = fopen($logFile, 'r');
while (($line = fgets($handle)) !== false) {
    // 处理每一行
}
fclose($handle);
```

## 最佳实践

### 1. 使用结构化日志

使用结构化日志便于分析和查询。

**示例**：

```php
<?php
declare(strict_types=1);

// 结构化日志便于分析
$logger->info('操作', ['operation' => 'create', 'resource' => 'user', 'user_id' => 123]);
```

### 2. 实现日志监控

实现日志监控，及时发现和通知问题。

**示例**：见"示例 2：日志监控和告警"

### 3. 定期分析日志

定期分析日志，了解系统状况和趋势。

**示例**：见"示例 3：日志统计报告"

### 4. 使用专业工具

对于复杂场景，使用专业的日志分析工具（如 ELK Stack、Grafana 等）。

**示例**：

```php
<?php
declare(strict_types=1);

// 将日志发送到 ELK Stack 等专业工具
// 使用这些工具进行分析和可视化
```

## 对比分析

### 日志分析工具对比

| 工具       | 易用性 | 功能 | 性能 | 适用场景     |
|:-----------|:-------|:-----|:-----|:-------------|
| **简单脚本** | ✅ 高  | ⚠️ 低| ✅ 高| 小规模日志   |
| **ELK Stack**| ⚠️ 中  | ✅ 高| ⚠️ 中| 大规模日志   |
| **Grafana**| ⚠️ 中  | ✅ 高| ✅ 高| 可视化监控   |

## 练习任务

1. **日志分析工具**：实现一个完整的日志分析工具。

2. **日志监控系统**：创建一个系统，监控日志并发送告警。

3. **日志报告生成工具**：编写一个工具，生成日志统计报告。

4. **实时日志监控工具**：实现一个工具，实时监控日志。

5. **日志分析最佳实践指南**：创建一个指南，总结日志分析的最佳实践。

## 相关章节

- **[4.9.1 日志体系设计](section-01-logging-system.md)**：了解日志体系设计的相关内容
- **[4.9.2 结构化日志](section-02-structured-logs.md)**：了解结构化日志的相关内容
- **[4.9.3 链路追踪](section-03-tracing.md)**：了解链路追踪的相关内容
