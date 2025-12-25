# 6.5.2 OPcache 配置

## 概述

OPcache 是 PHP 的内置操作码缓存扩展，通过缓存已编译的 PHP 代码，可以显著提升应用性能。在生产环境中，OPcache 是必不可少的性能优化工具。

### 为什么需要 OPcache

**没有 OPcache 的情况**：
- 每次请求都需要重新编译 PHP 文件
- 重复进行词法分析、语法分析、生成操作码
- 消耗大量 CPU 资源
- 响应时间较长

**使用 OPcache 后**：
- 首次请求编译并缓存操作码
- 后续请求直接使用缓存的操作码
- 大幅减少 CPU 使用率
- 显著提升响应速度

### 性能提升效果

- **响应时间**：减少 30-50%
- **CPU 使用率**：降低 40-60%
- **吞吐量**：提升 2-3 倍
- **内存开销**：增加约 100-256MB（用于缓存）

## OPcache 原理

### 什么是操作码缓存

**PHP 执行流程**（无缓存）：

1. **源代码**：读取 PHP 文件内容
2. **词法分析**：将源代码转换为标记（Token）
3. **语法分析**：将标记转换为抽象语法树（AST）
4. **编译**：生成操作码（Opcode）
5. **执行**：Zend 引擎执行操作码
6. **结果**：输出执行结果

**使用 OPcache 后的流程**：

```
首次请求：源代码 → 编译 → 操作码 → [存入缓存] → 执行
后续请求：源代码 → [从缓存读取操作码] → 执行
```

### 操作码（Opcode）的概念

**操作码**是 PHP 代码编译后的中间表示形式，类似于 Java 的字节码。它包含了执行代码所需的所有信息，但比源代码更接近机器指令。

**示例**：

```php
<?php
// 源代码
$a = 1 + 2;
```

对应的操作码（简化表示）：
```
ASSIGN $a, ADD(1, 2)
```

### OPcache 工作流程详解

#### 1. 首次请求

1. **检查缓存**：OPcache 检查文件是否已缓存
2. **未缓存**：执行完整编译流程
   - 词法分析
   - 语法分析
   - 生成操作码
3. **存入缓存**：将操作码存入共享内存
4. **执行**：Zend 引擎执行操作码

#### 2. 后续请求

1. **检查缓存**：OPcache 检查文件是否已缓存
2. **已缓存**：直接从共享内存读取操作码
3. **验证文件**（可选）：根据配置检查文件是否更新
4. **执行**：Zend 引擎执行缓存的操作码

#### 3. 文件更新处理

当 PHP 文件被修改时：

- **`validate_timestamps=1`**（开发环境）：
  - 每次请求检查文件时间戳
  - 如果文件更新，重新编译并更新缓存
  - 确保代码更改立即生效

- **`validate_timestamps=0`**（生产环境）：
  - 不检查文件时间戳
  - 需要手动重置缓存（重启 PHP-FPM 或调用 `opcache_reset()`）
  - 性能最优，但需要部署流程配合

### 共享内存机制

OPcache 使用**共享内存**存储缓存的操作码，这意味着：

1. **多进程共享**：所有 PHP-FPM 进程共享同一份缓存
2. **内存效率**：避免每个进程都存储一份操作码
3. **持久化**：缓存在 PHP-FPM 重启前一直有效
4. **快速访问**：内存访问速度远快于磁盘 I/O

### 缓存命中率

**命中（Hit）**：请求的文件已在缓存中，直接使用缓存
**未命中（Miss）**：请求的文件不在缓存中，需要编译

**命中率** = 命中次数 / (命中次数 + 未命中次数) × 100%

- **理想命中率**：> 95%
- **可接受命中率**：> 80%
- **低命中率**：< 80%（需要检查配置）

## 配置优化

### 配置位置

OPcache 配置在 `php.ini` 文件中，通常位于 `[opcache]` 部分：

```ini
; php.ini
[opcache]
; 配置项
```

修改配置后需要**重启 PHP-FPM** 才能生效。

### 基础配置参数详解

#### opcache.enable

**作用**：启用或禁用 OPcache

**值**：
- `1`：启用（推荐）
- `0`：禁用

**说明**：这是最基础的配置，必须设置为 `1` 才能使用 OPcache。

#### opcache.enable_cli

**作用**：是否在 CLI 模式下启用 OPcache

**值**：
- `1`：启用（开发环境推荐）
- `0`：禁用（生产环境推荐）

**说明**：
- CLI 模式通常用于命令行脚本、定时任务等
- 开发环境可以启用，提升命令行脚本性能
- 生产环境通常禁用，因为 CLI 脚本通常只执行一次，缓存意义不大

#### opcache.memory_consumption

**作用**：分配给 OPcache 的共享内存大小（MB）

**值**：整数，单位 MB

**推荐值**：
- 小型项目：64-128 MB
- 中型项目：128-256 MB
- 大型项目：256-512 MB

**说明**：
- 内存不足会导致缓存溢出，部分文件无法被缓存
- 可以通过 `opcache_get_status()` 监控内存使用情况
- 建议设置为项目所有 PHP 文件总大小的 1.5-2 倍

**计算方法**：
```
估算内存 = (项目 PHP 文件总数 × 平均文件大小) × 1.5
```

#### opcache.max_accelerated_files

**作用**：OPcache 可以缓存的最大文件数

**值**：整数

**推荐值**：
- 小型项目：5000-10000
- 中型项目：10000-20000
- 大型项目：20000-50000

**说明**：
- 应设置为略大于项目中的 PHP 文件总数
- 如果文件数超过此值，部分文件将无法被缓存
- 可以通过 `find . -name "*.php" | wc -l` 统计项目文件数

**注意事项**：
- 此值必须是质数，OPcache 会自动选择最接近的质数
- 如果设置为 10000，实际可能是 10007（最接近的质数）

#### opcache.revalidate_freq

**作用**：重新验证文件时间戳的时间间隔（秒）

**值**：整数，单位秒

**推荐值**：
- 开发环境：`0`（立即检查）
- 生产环境：`0`（配合 `validate_timestamps=0` 使用）

**说明**：
- 仅在 `validate_timestamps=1` 时有效
- `0` 表示每次请求都检查文件时间戳
- 大于 0 表示每隔指定秒数检查一次
- 生产环境通常设置为 `0`，配合 `validate_timestamps=0` 使用

#### opcache.validate_timestamps

**作用**：是否验证文件时间戳

**值**：
- `1`：验证（开发环境）
- `0`：不验证（生产环境）

**说明**：
- `1`：每次请求（或根据 `revalidate_freq`）检查文件是否更新
- `0`：不检查文件更新，性能最优，但需要手动重置缓存
- 生产环境必须设置为 `0` 以获得最佳性能

### 基础配置示例

```ini
; php.ini
[opcache]
; 启用 OPcache
opcache.enable=1

; CLI 模式禁用（生产环境）
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

### 高级配置参数详解

#### opcache.interned_strings_buffer

**作用**：字符串驻留（String Interning）缓冲区大小（MB）

**值**：整数，单位 MB

**推荐值**：8-16 MB

**说明**：
- 字符串驻留可以减少重复字符串的内存占用
- 对于大量使用相同字符串的应用效果明显
- 通常设置为 `memory_consumption` 的 5-10%

#### opcache.max_wasted_percentage

**作用**：允许浪费内存的最大百分比

**值**：整数，0-100

**推荐值**：5-10

**说明**：
- 当浪费内存超过此百分比时，OPcache 会重启并清理缓存
- 浪费内存通常由文件更新、缓存失效等原因造成
- 设置过低可能导致频繁重启，设置过高会浪费内存

#### opcache.use_cwd

**作用**：是否使用当前工作目录区分同名文件

**值**：
- `1`：使用（推荐）
- `0`：不使用

**说明**：
- `1`：不同目录下的同名文件会被分别缓存
- `0`：同名文件只缓存一个，可能导致冲突
- 建议始终设置为 `1`

#### opcache.save_comments

**作用**：是否保存注释和文档块

**值**：
- `1`：保存（推荐）
- `0`：不保存

**说明**：
- `1`：保存注释，支持反射、文档生成等功能
- `0`：不保存注释，节省内存，但会影响反射功能
- 如果使用依赖注入、ORM 等框架，必须设置为 `1`

#### opcache.fast_shutdown

**作用**：快速关闭模式

**值**：
- `1`：启用（推荐）
- `0`：禁用

**说明**：
- `1`：脚本执行完毕后快速清理，提升性能
- `0`：正常清理，可能稍慢
- 建议始终设置为 `1`

#### opcache.enable_file_override

**作用**：是否允许文件覆盖函数

**值**：
- `1`：允许
- `0`：不允许

**说明**：
- 某些框架可能使用文件覆盖功能
- 如果不确定，可以保持默认值或设置为 `1`

### 生产环境配置

**配置目标**：最大化性能，最小化资源消耗

```ini
; php.ini - 生产环境
[opcache]
; 基础配置
opcache.enable=1
opcache.enable_cli=0

; 内存配置
opcache.memory_consumption=256
opcache.interned_strings_buffer=16
opcache.max_accelerated_files=20000

; 性能优化
opcache.max_wasted_percentage=5
opcache.use_cwd=1
opcache.validate_timestamps=0
opcache.revalidate_freq=0
opcache.save_comments=1
opcache.fast_shutdown=1
opcache.enable_file_override=1
```

**配置说明**：
- `validate_timestamps=0`：不检查文件更新，性能最优
- `revalidate_freq=0`：配合 `validate_timestamps=0` 使用
- 代码更新后需要手动重置缓存（重启 PHP-FPM）

### 开发环境配置

**配置目标**：代码更改立即生效，便于调试

```ini
; php.ini - 开发环境
[opcache]
; 基础配置
opcache.enable=1
opcache.enable_cli=1  ; CLI 模式也启用

; 内存配置
opcache.memory_consumption=128
opcache.max_accelerated_files=10000

; 开发优化
opcache.validate_timestamps=1  ; 检查文件更新
opcache.revalidate_freq=0      ; 立即检查
```

**配置说明**：
- `validate_timestamps=1`：检查文件更新，代码更改立即生效
- `revalidate_freq=0`：每次请求都检查，确保实时性
- `enable_cli=1`：CLI 模式也启用，提升命令行脚本性能

## 缓存管理

### 缓存管理的概念

**缓存管理**包括：
1. **状态查询**：检查 OPcache 是否启用、内存使用情况、缓存文件数等
2. **缓存重置**：清空所有缓存，强制重新编译所有文件
3. **文件失效**：使特定文件的缓存失效，强制重新编译该文件
4. **缓存预热**：提前编译常用文件，提升首次访问性能

### 为什么需要缓存管理

1. **代码更新**：生产环境代码更新后，需要重置缓存
2. **性能监控**：了解缓存使用情况，优化配置
3. **问题排查**：检查缓存是否正常工作
4. **性能优化**：预热常用文件，提升首次访问速度

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

**重置缓存**会清空所有缓存的操作码，强制所有文件重新编译。

**使用场景**：
- 代码部署后
- 配置修改后
- 缓存出现问题时

**方法 1：使用函数**

```php
<?php
declare(strict_types=1);

// 重置整个缓存
if (function_exists('opcache_reset')) {
    opcache_reset();
    echo "OPcache reset successfully\n";
}
```

**方法 2：重启 PHP-FPM**

```bash
# 重启 PHP-FPM（推荐）
sudo systemctl restart php-fpm

# 或使用 service 命令
sudo service php-fpm restart
```

**注意事项**：
- 重置缓存会导致所有文件重新编译，首次请求可能较慢
- 生产环境应避免频繁重置缓存
- 建议在低峰期或维护窗口期进行

### 使文件失效

**使文件失效**会清除特定文件的缓存，强制该文件重新编译。

**使用场景**：
- 单个文件更新后
- 调试特定文件时
- 部分文件缓存异常时

```php
<?php
declare(strict_types=1);

// 使特定文件失效
if (function_exists('opcache_invalidate')) {
    // 参数 1：文件路径（绝对路径）
    // 参数 2：是否强制失效（true = 立即失效，false = 等待下次检查）
    opcache_invalidate('/path/to/file.php', true);
    echo "File invalidated\n";
}
```

**参数说明**：
- **文件路径**：必须是绝对路径
- **强制失效**：
  - `true`：立即失效，即使 `validate_timestamps=0` 也会失效
  - `false`：等待下次时间戳检查时失效

**示例**：

```php
<?php
declare(strict_types=1);

// 使多个文件失效
$files = [
    __DIR__ . '/config.php',
    __DIR__ . '/app/bootstrap.php',
];

foreach ($files as $file) {
    if (function_exists('opcache_invalidate')) {
        opcache_invalidate($file, true);
    }
}
```

### 编译文件（预热）

**编译文件**会提前编译文件并存入缓存，提升首次访问性能。

**使用场景**：
- 部署后预热常用文件
- 提升首次访问速度
- 确保关键文件已缓存

```php
<?php
declare(strict_types=1);

// 编译单个文件
if (function_exists('opcache_compile_file')) {
    $result = opcache_compile_file('/path/to/file.php');
    if ($result) {
        echo "File compiled successfully\n";
    } else {
        echo "Failed to compile file\n";
    }
}
```

**预热脚本示例**：

```php
<?php
declare(strict_types=1);

// 预热常用文件
function warmupOpcache(array $files): void
{
    foreach ($files as $file) {
        $realPath = realpath($file);
        if ($realPath && function_exists('opcache_compile_file')) {
            opcache_compile_file($realPath);
        }
    }
}

// 使用
$files = [
    __DIR__ . '/vendor/autoload.php',
    __DIR__ . '/config/app.php',
    __DIR__ . '/app/bootstrap.php',
];

warmupOpcache($files);
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

### 为什么需要性能监控

**性能监控**可以帮助我们：
1. **了解缓存状态**：是否启用、内存使用情况、缓存文件数
2. **评估配置效果**：命中率、内存利用率、缓存效率
3. **发现问题**：缓存溢出、命中率低、内存不足
4. **优化配置**：根据实际使用情况调整配置参数

### 关键监控指标

#### 1. 缓存状态

- **是否启用**：`opcache_enabled`
- **缓存是否已满**：`cache_full`
- **缓存文件数**：`scripts` 数组长度

#### 2. 内存使用

- **已使用内存**：`memory_usage['used_memory']`
- **空闲内存**：`memory_usage['free_memory']`
- **浪费内存**：`memory_usage['wasted_memory']`
- **内存使用率**：已使用 / (已使用 + 空闲) × 100%

#### 3. 缓存统计

- **命中次数**：`opcache_statistics['hits']`
- **未命中次数**：`opcache_statistics['misses']`
- **命中率**：命中次数 / (命中次数 + 未命中次数) × 100%
- **缓存脚本数**：`scripts` 数组长度

#### 4. 性能指标

- **理想命中率**：> 95%
- **可接受命中率**：> 80%
- **内存使用率**：< 90%
- **浪费内存率**：< 10%

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

**定期监控**可以自动检查 OPcache 状态，发现问题并及时告警。

**监控频率**：
- 生产环境：每 5-15 分钟
- 开发环境：每 30 分钟或按需

**告警阈值**：
- 内存使用率 > 90%
- 缓存已满
- 命中率 < 80%
- 浪费内存率 > 10%

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
        $totalMemory = $memory['used_memory'] + $memory['free_memory'];
        $memoryUsage = ($memory['used_memory'] / $totalMemory) * 100;
        
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
        $total = $stats['hits'] + $stats['misses'];
        if ($total > 0) {
            $hitRate = ($stats['hits'] / $total) * 100;
            if ($hitRate < 80) {
                $this->log("WARNING: OPcache hit rate is low: " . round($hitRate, 2) . "%");
            }
        }
        
        // 浪费内存过多
        $wastedPercent = ($memory['wasted_memory'] / $totalMemory) * 100;
        if ($wastedPercent > 10) {
            $this->log("WARNING: OPcache wasted memory is high: " . round($wastedPercent, 2) . "%");
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

**设置定时任务**：

```bash
# 编辑 crontab
crontab -e

# 每 5 分钟执行一次监控
*/5 * * * * php /path/to/opcache-monitor.php
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
