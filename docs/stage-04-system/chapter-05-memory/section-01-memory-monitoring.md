# 4.5.1 内存使用监控

## 概述

内存使用监控是优化程序性能的基础。在实际应用中，我们经常需要了解程序的内存使用情况，以便发现内存泄漏、优化内存占用、诊断性能问题等。PHP 提供了 `memory_get_usage()` 和 `memory_get_peak_usage()` 等函数来监控内存使用。

理解内存监控的方法、函数的使用、内存值的含义，以及如何进行内存分析，对于开发高效的 PHP 应用至关重要。

**主要内容**：
- 内存监控概述（为什么需要监控内存）
- `memory_get_usage()` 函数（获取当前内存使用）
- `memory_get_peak_usage()` 函数（获取峰值内存使用）
- 内存使用分析（趋势分析、泄漏检测）
- 内存分析工具简介
- 实际应用场景和最佳实践

## 特性

- **实时监控**：可以实时获取当前内存使用情况
- **峰值监控**：可以获取脚本执行过程中的峰值内存使用
- **详细分析**：支持详细的内存使用分析
- **工具支持**：可以使用专业工具进行深度分析

## 语法/定义

### memory_get_usage() 函数

**语法**：`memory_get_usage(bool $real_usage = false): int`

**参数**：
- `$real_usage`：可选，如果为 `true`，返回系统分配的实际内存；如果为 `false`（默认），返回 PHP 内部使用的内存

**返回值**：返回当前内存使用量（字节数）。

### memory_get_peak_usage() 函数

**语法**：`memory_get_peak_usage(bool $real_usage = false): int`

**参数**：
- `$real_usage`：可选，如果为 `true`，返回系统分配的实际内存；如果为 `false`（默认），返回 PHP 内部使用的内存

**返回值**：返回峰值内存使用量（字节数）。

## 基本用法

### 示例 1：基本内存监控

```php
<?php
declare(strict_types=1);

function formatBytes(int $bytes): string
{
    $units = ['B', 'KB', 'MB', 'GB'];
    $bytes = max($bytes, 0);
    $pow = floor(($bytes ? log($bytes) : 0) / log(1024));
    $pow = min($pow, count($units) - 1);
    $bytes /= (1 << (10 * $pow));
    
    return round($bytes, 2) . ' ' . $units[$pow];
}

// 获取当前内存使用
$currentMemory = memory_get_usage();
echo "当前内存使用: " . formatBytes($currentMemory) . "\n";

// 执行一些操作
$data = range(1, 100000);
$afterMemory = memory_get_usage();
echo "操作后内存使用: " . formatBytes($afterMemory) . "\n";
echo "内存增加: " . formatBytes($afterMemory - $currentMemory) . "\n";

// 获取峰值内存
$peakMemory = memory_get_peak_usage();
echo "峰值内存使用: " . formatBytes($peakMemory) . "\n";
```

**说明**：
- `memory_get_usage()` 返回当前内存使用量
- `memory_get_peak_usage()` 返回峰值内存使用量
- 返回值是字节数，需要格式化显示

### 示例 2：真实内存使用 vs 内部内存使用

```php
<?php
declare(strict_types=1);

// 内部内存使用（PHP 内部跟踪的内存）
$internalMemory = memory_get_usage(false);
echo "内部内存使用: " . formatBytes($internalMemory) . "\n";

// 真实内存使用（系统实际分配的内存）
$realMemory = memory_get_usage(true);
echo "真实内存使用: " . formatBytes($realMemory) . "\n";

// 真实内存使用通常大于或等于内部内存使用
$difference = $realMemory - $internalMemory;
echo "差异: " . formatBytes($difference) . "\n";
```

**说明**：
- `false`（默认）：返回 PHP 内部跟踪的内存使用
- `true`：返回系统实际分配的内存
- 真实内存使用通常大于内部内存使用

### 示例 3：内存使用监控工具类

```php
<?php
declare(strict_types=1);

class MemoryMonitor
{
    private array $snapshots = [];
    
    public function snapshot(string $label = ''): void
    {
        $this->snapshots[] = [
            'label' => $label,
            'timestamp' => microtime(true),
            'current' => memory_get_usage(),
            'peak' => memory_get_peak_usage(),
        ];
    }
    
    public function getCurrentUsage(): array
    {
        return [
            'current' => memory_get_usage(),
            'peak' => memory_get_peak_usage(),
            'limit' => $this->parseMemoryLimit(ini_get('memory_limit')),
        ];
    }
    
    public function printSnapshots(): void
    {
        echo "内存使用快照:\n";
        echo str_repeat("-", 60) . "\n";
        printf("%-20s %15s %15s\n", "标签", "当前内存", "峰值内存");
        echo str_repeat("-", 60) . "\n";
        
        foreach ($this->snapshots as $snapshot) {
            printf(
                "%-20s %15s %15s\n",
                $snapshot['label'],
                $this->formatBytes($snapshot['current']),
                $this->formatBytes($snapshot['peak'])
            );
        }
    }
    
    public function detectLeak(float $thresholdMB = 10.0): ?array
    {
        if (count($this->snapshots) < 2) {
            return null;
        }
        
        $first = $this->snapshots[0];
        $last = end($this->snapshots);
        
        $memoryIncrease = ($last['current'] - $first['current']) / (1024 * 1024);
        
        if ($memoryIncrease > $thresholdMB) {
            return [
                'leak_detected' => true,
                'memory_increase_mb' => $memoryIncrease,
                'first_snapshot' => $first,
                'last_snapshot' => $last,
            ];
        }
        
        return null;
    }
    
    private function formatBytes(int $bytes): string
    {
        $units = ['B', 'KB', 'MB', 'GB'];
        $bytes = max($bytes, 0);
        $pow = floor(($bytes ? log($bytes) : 0) / log(1024));
        $pow = min($pow, count($units) - 1);
        $bytes /= (1 << (10 * $pow));
        
        return round($bytes, 2) . ' ' . $units[$pow];
    }
    
    private function parseMemoryLimit(string $limit): int
    {
        $limit = trim($limit);
        if ($limit === '-1') {
            return -1;
        }
        
        $last = strtolower($limit[strlen($limit) - 1]);
        $value = (int)$limit;
        
        return match ($last) {
            'g' => $value * 1024 * 1024 * 1024,
            'm' => $value * 1024 * 1024,
            'k' => $value * 1024,
            default => $value,
        };
    }
}

// 使用
$monitor = new MemoryMonitor();
$monitor->snapshot('初始化');

// 执行操作
$data1 = range(1, 50000);
$monitor->snapshot('创建数组1');

$data2 = range(1, 100000);
$monitor->snapshot('创建数组2');

unset($data1);
$monitor->snapshot('释放数组1');

$monitor->printSnapshots();

// 检测内存泄漏
$leak = $monitor->detectLeak();
if ($leak !== null) {
    echo "检测到可能的内存泄漏: " . round($leak['memory_increase_mb'], 2) . " MB\n";
}
```

**说明**：
- 创建内存监控工具类，记录快照
- 支持检测内存泄漏
- 格式化显示内存使用情况

### 示例 4：监控内存使用趋势

```php
<?php
declare(strict_types=1);

function monitorMemoryUsage(callable $callback, string $label): void
{
    $before = memory_get_usage();
    $peakBefore = memory_get_peak_usage();
    
    // 执行回调
    $callback();
    
    $after = memory_get_usage();
    $peakAfter = memory_get_peak_usage();
    
    $used = $after - $before;
    $peakUsed = $peakAfter - $peakBefore;
    
    echo "{$label}:\n";
    echo "  内存使用: " . formatBytes($used) . "\n";
    echo "  峰值增加: " . formatBytes($peakUsed) . "\n";
    echo "\n";
}

function formatBytes(int $bytes): string
{
    $units = ['B', 'KB', 'MB', 'GB'];
    $bytes = max($bytes, 0);
    $pow = floor(($bytes ? log($bytes) : 0) / log(1024));
    $pow = min($pow, count($units) - 1);
    $bytes /= (1 << (10 * $pow));
    
    return round($bytes, 2) . ' ' . $units[$pow];
}

// 监控不同操作的内存使用
monitorMemoryUsage(function () {
    $array = range(1, 10000);
}, '创建小数组');

monitorMemoryUsage(function () {
    $array = range(1, 100000);
}, '创建大数组');

monitorMemoryUsage(function () {
    $objects = [];
    for ($i = 0; $i < 10000; $i++) {
        $objects[] = new stdClass();
    }
}, '创建对象数组');
```

**说明**：
- 监控特定操作的内存使用
- 比较不同操作的内存占用

### 示例 5：内存使用百分比

```php
<?php
declare(strict_types=1);

function getMemoryUsagePercent(): float
{
    $current = memory_get_usage();
    $limitStr = ini_get('memory_limit');
    
    if ($limitStr === '-1') {
        return 0;  // 无限制
    }
    
    $limit = parseMemoryLimit($limitStr);
    if ($limit <= 0) {
        return 0;
    }
    
    return ($current / $limit) * 100;
}

function parseMemoryLimit(string $limit): int
{
    $limit = trim($limit);
    if ($limit === '-1') {
        return -1;
    }
    
    $last = strtolower($limit[strlen($limit) - 1]);
    $value = (int)$limit;
    
    return match ($last) {
        'g' => $value * 1024 * 1024 * 1024,
        'm' => $value * 1024 * 1024,
        'k' => $value * 1024,
        default => $value,
    };
}

$percent = getMemoryUsagePercent();
echo "当前内存使用: " . number_format($percent, 2) . "%\n";

if ($percent > 80) {
    echo "警告: 内存使用超过 80%\n";
}
```

**说明**：
- 计算当前内存使用相对于内存限制的百分比
- 可以用于监控内存使用是否接近限制

## 使用场景

### 场景 1：性能优化

在性能优化过程中，监控内存使用找出内存占用大的操作。

**示例**：见"示例 4：监控内存使用趋势"

### 场景 2：内存泄漏检测

通过监控内存使用趋势，检测可能的内存泄漏。

**示例**：见"示例 3：内存使用监控工具类"

### 场景 3：资源使用分析

分析脚本的资源使用情况。

**示例**：

```php
<?php
declare(strict_types=1);

$monitor = new MemoryMonitor();
$monitor->snapshot('脚本开始');

// 执行脚本逻辑
// ...

$monitor->snapshot('脚本结束');
$monitor->printSnapshots();
```

## 注意事项

### 内存值的含义

`memory_get_usage()` 返回的是 PHP 内部跟踪的内存使用，不是系统实际分配的内存。

**示例**：

```php
<?php
declare(strict_types=1);

// 内部内存使用（PHP 跟踪的）
$internal = memory_get_usage(false);

// 真实内存使用（系统分配的）
$real = memory_get_usage(true);

// 真实内存通常 >= 内部内存
```

### 监控开销

频繁调用内存监控函数有轻微的性能开销。

**示例**：

```php
<?php
declare(strict_types=1);

// 不要在循环中频繁调用
// for ($i = 0; $i < 1000000; $i++) {
//     $memory = memory_get_usage();  // 开销较大
// }

// 在关键点监控即可
$before = memory_get_usage();
// 执行操作
$after = memory_get_usage();
```

### 不同环境下的差异

不同运行环境（CLI、FPM 等）的内存使用可能有差异。

**示例**：

```php
<?php
declare(strict_types=1);

// 不同 SAPI 的内存使用可能不同
$sapi = php_sapi_name();
$memory = memory_get_usage();
echo "SAPI: {$sapi}, 内存: " . formatBytes($memory) . "\n";
```

## 常见问题

### 问题 1：memory_get_usage() 返回什么值？

**回答**：返回当前 PHP 脚本使用的内存量（字节数）。如果 `$real_usage` 为 `false`（默认），返回 PHP 内部跟踪的内存；如果为 `true`，返回系统实际分配的内存。

**示例**：

```php
<?php
declare(strict_types=1);

$internal = memory_get_usage(false);  // PHP 内部跟踪的内存
$real = memory_get_usage(true);       // 系统实际分配的内存
```

### 问题 2：如何计算实际内存使用？

**回答**：使用 `memory_get_usage(true)` 获取系统实际分配的内存。

**示例**：见"示例 2：真实内存使用 vs 内部内存使用"

### 问题 3：如何检测内存泄漏？

**回答**：通过监控内存使用趋势，如果在不应该增长的地方内存持续增长，可能存在内存泄漏。

**示例**：见"示例 3：内存使用监控工具类"的 `detectLeak()` 方法

### 问题 4：内存监控的性能影响？

**回答**：内存监控函数有轻微的性能开销，应该避免在循环中频繁调用。

**示例**：见"监控开销"部分

## 最佳实践

### 1. 在关键点监控内存

在关键操作前后监控内存使用。

**示例**：

```php
<?php
declare(strict_types=1);

$before = memory_get_usage();
// 执行关键操作
$after = memory_get_usage();
$used = $after - $before;
```

### 2. 记录内存使用趋势

记录多个时间点的内存使用，分析趋势。

**示例**：见"示例 3：内存使用监控工具类"

### 3. 使用工具进行深度分析

对于复杂的内存问题，使用专业工具（如 Xdebug、Blackfire）进行深度分析。

**示例**：

```php
<?php
declare(strict_types=1);

// 使用 Xdebug 进行内存分析
// xdebug_start_trace();
// // 执行代码
// xdebug_stop_trace();
```

### 4. 对比不同实现的内存使用

对比不同实现方式的内存使用，选择内存效率高的实现。

**示例**：见"示例 4：监控内存使用趋势"

## 对比分析

### memory_get_usage() vs memory_get_peak_usage()

| 特性         | memory_get_usage()           | memory_get_peak_usage()      |
|:-------------|:-----------------------------|:-----------------------------|
| **返回值**   | 当前内存使用                 | 峰值内存使用                 |
| **变化**     | 实时变化                     | 只增不减（直到脚本结束）     |
| **使用场景** | 监控当前内存状态             | 了解脚本最大内存需求         |

### 内部内存 vs 真实内存

| 特性         | 内部内存（false）            | 真实内存（true）             |
|:-------------|:-----------------------------|:-----------------------------|
| **含义**     | PHP 内部跟踪的内存           | 系统实际分配的内存           |
| **大小**     | 通常较小                     | 通常较大                     |
| **准确性**   | ⚠️ 可能不包含所有内存        | ✅ 更准确                    |
| **使用场景** | 了解 PHP 使用的内存          | 了解系统实际分配的内存       |

## 练习任务

1. **内存监控工具类**：创建一个完整的内存监控工具类，支持快照、趋势分析、泄漏检测。

2. **内存使用分析工具**：实现一个工具，分析脚本的内存使用情况并生成报告。

3. **内存泄漏检测工具**：创建一个工具，检测可能的内存泄漏。

4. **内存使用对比工具**：编写一个工具，对比不同实现方式的内存使用。

5. **内存监控集成**：将内存监控集成到现有的 CLI 工具中。

## 相关章节

- **[4.5.2 内存限制设置](section-02-memory-limits.md)**：了解内存限制的设置
- **[4.5.4 内存优化实践](section-04-memory-optimization.md)**：学习内存优化方法
- **[4.2.10 大文件处理](../chapter-02-filesystem/section-10-large-files.md)**：了解大文件处理中的内存优化
