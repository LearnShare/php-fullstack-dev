# 6.5.2 OPcache 配置

## 概述

OPcache 是 PHP 的操作码缓存扩展，可以显著提升 PHP 应用性能。本节介绍 OPcache 的工作原理、配置优化、缓存管理，以及性能监控方法。

## OPcache 原理

### 什么是操作码缓存

PHP 执行流程：
1. 源代码 → 词法分析 → 语法分析 → 操作码（Opcode）
2. 操作码 → 执行 → 结果

OPcache 缓存操作码，避免重复编译：

```
源代码 → 编译 → 操作码 → [缓存] → 执行
```

### OPcache 工作流程

1. **首次请求**：编译 PHP 文件，生成操作码，存入共享内存
2. **后续请求**：直接从共享内存读取操作码，跳过编译步骤

## 配置优化

### 基础配置

```ini
; php.ini
[opcache]
; 启用 OPcache
opcache.enable=1

; CLI 模式也启用（可选）
opcache.enable_cli=0

; 共享内存大小（MB）
opcache.memory_consumption=128

; 最大缓存文件数
opcache.max_accelerated_files=10000

; 重新验证文件时间间隔（秒）
opcache.revalidate_freq=2

; 不验证文件时间戳（生产环境）
opcache.validate_timestamps=0
```

### 生产环境配置

```ini
; php.ini - 生产环境
[opcache]
opcache.enable=1
opcache.enable_cli=0
opcache.memory_consumption=256
opcache.interned_strings_buffer=16
opcache.max_accelerated_files=20000
opcache.max_wasted_percentage=5
opcache.use_cwd=1
opcache.validate_timestamps=0
opcache.revalidate_freq=0
opcache.save_comments=1
opcache.fast_shutdown=1
opcache.enable_file_override=1
```

### 开发环境配置

```ini
; php.ini - 开发环境
[opcache]
opcache.enable=1
opcache.enable_cli=1
opcache.memory_consumption=128
opcache.max_accelerated_files=10000
opcache.validate_timestamps=1
opcache.revalidate_freq=0
```

## 缓存管理

### 检查 OPcache 状态

```php
<?php
declare(strict_types=1);

function getOpcacheStatus(): array
{
    if (!function_exists('opcache_get_status')) {
        return ['enabled' => false];
    }
    
    $status = opcache_get_status();
    
    return [
        'enabled' => $status['opcache_enabled'] ?? false,
        'cache_full' => $status['cache_full'] ?? false,
        'memory_usage' => $status['memory_usage'] ?? [],
        'opcache_statistics' => $status['opcache_statistics'] ?? [],
        'scripts' => count($status['scripts'] ?? []),
    ];
}

// 使用
$status = getOpcacheStatus();
print_r($status);
```

### 重置 OPcache

```php
<?php
declare(strict_types=1);

// 重置整个缓存
if (function_exists('opcache_reset')) {
    opcache_reset();
    echo "OPcache reset successfully\n";
}

// 使特定文件失效
if (function_exists('opcache_invalidate')) {
    opcache_invalidate('/path/to/file.php', true);
    echo "File invalidated\n";
}
```

### OPcache 管理类

```php
<?php
declare(strict_types=1);

class OpcacheManager
{
    /**
     * 获取 OPcache 状态
     */
    public static function getStatus(): ?array
    {
        if (!function_exists('opcache_get_status')) {
            return null;
        }
        
        return opcache_get_status();
    }
    
    /**
     * 获取配置信息
     */
    public static function getConfig(): ?array
    {
        if (!function_exists('opcache_get_configuration')) {
            return null;
        }
        
        return opcache_get_configuration();
    }
    
    /**
     * 重置缓存
     */
    public static function reset(): bool
    {
        if (!function_exists('opcache_reset')) {
            return false;
        }
        
        return opcache_reset();
    }
    
    /**
     * 使文件失效
     */
    public static function invalidate(string $file, bool $force = false): bool
    {
        if (!function_exists('opcache_invalidate')) {
            return false;
        }
        
        return opcache_invalidate($file, $force);
    }
    
    /**
     * 编译文件
     */
    public static function compileFile(string $file): bool
    {
        if (!function_exists('opcache_compile_file')) {
            return false;
        }
        
        return opcache_compile_file($file);
    }
    
    /**
     * 获取缓存统计信息
     */
    public static function getStatistics(): array
    {
        $status = self::getStatus();
        
        if (!$status) {
            return [];
        }
        
        $memory = $status['memory_usage'] ?? [];
        $stats = $status['opcache_statistics'] ?? [];
        
        return [
            'memory_used' => $memory['used_memory'] ?? 0,
            'memory_free' => $memory['free_memory'] ?? 0,
            'memory_wasted' => $memory['wasted_memory'] ?? 0,
            'hits' => $stats['hits'] ?? 0,
            'misses' => $stats['misses'] ?? 0,
            'hit_rate' => self::calculateHitRate($stats),
            'cached_scripts' => count($status['scripts'] ?? []),
        ];
    }
    
    private static function calculateHitRate(array $stats): float
    {
        $hits = $stats['hits'] ?? 0;
        $misses = $stats['misses'] ?? 0;
        $total = $hits + $misses;
        
        if ($total === 0) {
            return 0.0;
        }
        
        return round(($hits / $total) * 100, 2);
    }
}

// 使用
$manager = new OpcacheManager();
$stats = $manager->getStatistics();
print_r($stats);
```

## 性能监控

### 监控脚本

```php
<?php
declare(strict_types=1);

// opcache-monitor.php
$status = opcache_get_status();
$config = opcache_get_configuration();

if (!$status || !$config) {
    die("OPcache is not enabled\n");
}

echo "OPcache Status\n";
echo str_repeat("=", 50) . "\n";
echo "Enabled: " . ($status['opcache_enabled'] ? 'Yes' : 'No') . "\n";
echo "Cache Full: " . ($status['cache_full'] ? 'Yes' : 'No') . "\n\n";

// 内存使用
$memory = $status['memory_usage'];
echo "Memory Usage\n";
echo str_repeat("-", 50) . "\n";
echo "Used: " . formatBytes($memory['used_memory']) . "\n";
echo "Free: " . formatBytes($memory['free_memory']) . "\n";
echo "Wasted: " . formatBytes($memory['wasted_memory']) . "\n\n";

// 统计信息
$stats = $status['opcache_statistics'];
echo "Statistics\n";
echo str_repeat("-", 50) . "\n";
echo "Hits: " . number_format($stats['hits']) . "\n";
echo "Misses: " . number_format($stats['misses']) . "\n";
$hitRate = ($stats['hits'] / ($stats['hits'] + $stats['misses'])) * 100;
echo "Hit Rate: " . number_format($hitRate, 2) . "%\n";
echo "Cached Scripts: " . count($status['scripts']) . "\n";

function formatBytes(int $bytes): string
{
    $units = ['B', 'KB', 'MB', 'GB'];
    $i = 0;
    while ($bytes >= 1024 && $i < count($units) - 1) {
        $bytes /= 1024;
        $i++;
    }
    return round($bytes, 2) . ' ' . $units[$i];
}
```

### 定期监控

```php
<?php
declare(strict_types=1);

// 定期检查 OPcache 状态
class OpcacheMonitor
{
    private string $logFile;
    
    public function __construct(string $logFile = '/var/log/opcache-monitor.log')
    {
        $this->logFile = $logFile;
    }
    
    public function check(): void
    {
        $status = opcache_get_status();
        
        if (!$status) {
            $this->log("OPcache is not enabled");
            return;
        }
        
        $memory = $status['memory_usage'];
        $memoryUsage = ($memory['used_memory'] / ($memory['used_memory'] + $memory['free_memory'])) * 100;
        
        // 内存使用率过高
        if ($memoryUsage > 90) {
            $this->log("WARNING: OPcache memory usage is high: " . round($memoryUsage, 2) . "%");
        }
        
        // 缓存已满
        if ($status['cache_full']) {
            $this->log("WARNING: OPcache cache is full");
        }
        
        // 命中率过低
        $stats = $status['opcache_statistics'];
        $hitRate = ($stats['hits'] / ($stats['hits'] + $stats['misses'])) * 100;
        
        if ($hitRate < 80) {
            $this->log("WARNING: OPcache hit rate is low: " . round($hitRate, 2) . "%");
        }
    }
    
    private function log(string $message): void
    {
        $timestamp = date('Y-m-d H:i:s');
        $logMessage = "[{$timestamp}] {$message}\n";
        file_put_contents($this->logFile, $logMessage, FILE_APPEND);
    }
}

// 使用 cron 定期执行
// */5 * * * * php /path/to/opcache-monitor.php
```

## 完整示例

```php
<?php
declare(strict_types=1);

// OPcache 配置和监控示例
class OpcacheService
{
    /**
     * 初始化 OPcache
     */
    public static function initialize(): void
    {
        if (!function_exists('opcache_get_status')) {
            throw new RuntimeException('OPcache is not available');
        }
        
        $status = opcache_get_status();
        if (!$status || !$status['opcache_enabled']) {
            throw new RuntimeException('OPcache is not enabled');
        }
    }
    
    /**
     * 预热缓存
     */
    public static function warmup(array $files): void
    {
        foreach ($files as $file) {
            if (file_exists($file)) {
                opcache_compile_file($file);
            }
        }
    }
    
    /**
     * 清理缓存
     */
    public static function clear(): void
    {
        opcache_reset();
    }
    
    /**
     * 获取性能报告
     */
    public static function getReport(): array
    {
        $status = opcache_get_status();
        $config = opcache_get_configuration();
        
        if (!$status || !$config) {
            return ['error' => 'OPcache is not available'];
        }
        
        $memory = $status['memory_usage'];
        $stats = $status['opcache_statistics'];
        
        $totalMemory = $memory['used_memory'] + $memory['free_memory'];
        $memoryUsage = ($memory['used_memory'] / $totalMemory) * 100;
        $hitRate = ($stats['hits'] / ($stats['hits'] + $stats['misses'])) * 100;
        
        return [
            'enabled' => $status['opcache_enabled'],
            'cache_full' => $status['cache_full'],
            'memory_usage_percent' => round($memoryUsage, 2),
            'memory_wasted_percent' => round(($memory['wasted_memory'] / $totalMemory) * 100, 2),
            'hit_rate' => round($hitRate, 2),
            'cached_scripts' => count($status['scripts']),
            'max_accelerated_files' => $config['directives']['opcache.max_accelerated_files'],
        ];
    }
}

// 使用
OpcacheService::initialize();
$report = OpcacheService::getReport();
print_r($report);
```

## 最佳实践

1. **生产环境**：禁用 `validate_timestamps`，设置 `revalidate_freq=0`
2. **开发环境**：启用 `validate_timestamps`，实时检测文件变化
3. **内存配置**：根据应用大小设置 `memory_consumption`
4. **文件数量**：设置 `max_accelerated_files` 大于实际文件数
5. **监控**：定期检查缓存状态和命中率

## 注意事项

1. 修改配置后需要重启 PHP-FPM
2. 生产环境不要频繁重置缓存
3. 监控内存使用，避免缓存溢出
4. 使用预热脚本提升首次访问性能

## 练习

1. 配置 OPcache，针对生产环境优化设置。

2. 创建一个 OPcache 管理类，提供状态查询、缓存重置等功能。

3. 实现一个 OPcache 监控脚本，定期检查缓存状态并记录日志。

4. 创建一个缓存预热脚本，在部署后自动预热常用文件。
