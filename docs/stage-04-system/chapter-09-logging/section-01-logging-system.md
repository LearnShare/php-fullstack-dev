# 4.9.1 日志体系设计

## 概述

日志系统是应用程序的重要组成部分，用于记录程序运行过程中的重要信息，包括错误、警告、调试信息等。在实际应用中，我们需要设计合理的日志体系，包括日志级别、日志格式、日志存储等，以便于问题排查、性能监控、审计等。

理解日志体系的设计原则、日志级别、日志格式、日志存储策略等，对于构建健壮的应用程序至关重要。

**主要内容**：
- 日志体系概述（什么是日志、为什么需要日志）
- 日志级别（DEBUG、INFO、WARNING、ERROR、CRITICAL）
- 日志格式设计
- 日志存储策略（文件、数据库、远程服务）
- 日志轮转
- 日志分析
- 实际应用场景和最佳实践

## 特性

- **级别分类**：支持多个日志级别
- **格式统一**：统一的日志格式
- **存储灵活**：支持多种存储方式
- **可分析**：便于日志分析和查询

## 语法/定义

### 日志级别

常见的日志级别（从低到高）：
- DEBUG：调试信息
- INFO：一般信息
- WARNING：警告信息
- ERROR：错误信息
- CRITICAL：严重错误

## 基本用法

### 示例 1：简单的日志系统

```php
<?php
declare(strict_types=1);

class SimpleLogger
{
    private const LEVEL_DEBUG = 0;
    private const LEVEL_INFO = 1;
    private const LEVEL_WARNING = 2;
    private const LEVEL_ERROR = 3;
    private const LEVEL_CRITICAL = 4;
    
    private string $logFile;
    private int $minLevel;
    
    public function __construct(string $logFile, int $minLevel = self::LEVEL_INFO)
    {
        $this->logFile = $logFile;
        $this->minLevel = $minLevel;
    }
    
    public function debug(string $message, array $context = []): void
    {
        $this->log(self::LEVEL_DEBUG, 'DEBUG', $message, $context);
    }
    
    public function info(string $message, array $context = []): void
    {
        $this->log(self::LEVEL_INFO, 'INFO', $message, $context);
    }
    
    public function warning(string $message, array $context = []): void
    {
        $this->log(self::LEVEL_WARNING, 'WARNING', $message, $context);
    }
    
    public function error(string $message, array $context = []): void
    {
        $this->log(self::LEVEL_ERROR, 'ERROR', $message, $context);
    }
    
    public function critical(string $message, array $context = []): void
    {
        $this->log(self::LEVEL_CRITICAL, 'CRITICAL', $message, $context);
    }
    
    private function log(int $level, string $levelName, string $message, array $context): void
    {
        if ($level < $this->minLevel) {
            return;
        }
        
        $timestamp = date('Y-m-d H:i:s');
        $contextStr = !empty($context) ? ' ' . json_encode($context, JSON_UNESCAPED_UNICODE) : '';
        $logMessage = "[{$timestamp}] [{$levelName}] {$message}{$contextStr}\n";
        
        file_put_contents($this->logFile, $logMessage, FILE_APPEND | LOCK_EX);
    }
}

// 使用
$logger = new SimpleLogger('/tmp/app.log', SimpleLogger::LEVEL_INFO);

$logger->debug('调试信息');  // 不会记录（级别太低）
$logger->info('应用启动');
$logger->warning('警告信息');
$logger->error('错误信息');
$logger->critical('严重错误');
```

**说明**：
- 实现简单的日志系统
- 支持多个日志级别
- 可以设置最小日志级别

### 示例 2：结构化日志系统

```php
<?php
declare(strict_types=1);

class StructuredLogger
{
    private string $logFile;
    private int $minLevel;
    
    private const LEVEL_DEBUG = 0;
    private const LEVEL_INFO = 1;
    private const LEVEL_WARNING = 2;
    private const LEVEL_ERROR = 3;
    private const LEVEL_CRITICAL = 4;
    
    public function __construct(string $logFile, int $minLevel = self::LEVEL_INFO)
    {
        $this->logFile = $logFile;
        $this->minLevel = $minLevel;
    }
    
    public function log(int $level, string $levelName, string $message, array $context = []): void
    {
        if ($level < $this->minLevel) {
            return;
        }
        
        $logEntry = [
            'timestamp' => date('c'),  // ISO 8601 格式
            'level' => $levelName,
            'message' => $message,
            'context' => $context,
            'memory' => memory_get_usage(),
            'memory_peak' => memory_get_peak_usage(),
        ];
        
        $logLine = json_encode($logEntry, JSON_UNESCAPED_UNICODE) . "\n";
        file_put_contents($this->logFile, $logLine, FILE_APPEND | LOCK_EX);
    }
    
    public function debug(string $message, array $context = []): void
    {
        $this->log(self::LEVEL_DEBUG, 'DEBUG', $message, $context);
    }
    
    public function info(string $message, array $context = []): void
    {
        $this->log(self::LEVEL_INFO, 'INFO', $message, $context);
    }
    
    public function warning(string $message, array $context = []): void
    {
        $this->log(self::LEVEL_WARNING, 'WARNING', $message, $context);
    }
    
    public function error(string $message, array $context = []): void
    {
        $this->log(self::LEVEL_ERROR, 'ERROR', $message, $context);
    }
    
    public function critical(string $message, array $context = []): void
    {
        $this->log(self::LEVEL_CRITICAL, 'CRITICAL', $message, $context);
    }
}

// 使用
$logger = new StructuredLogger('/tmp/app.log');

$logger->info('用户登录', ['user_id' => 123, 'ip' => '192.168.1.1']);
$logger->error('数据库连接失败', ['host' => 'localhost', 'port' => 3306]);
```

**说明**：
- 结构化日志（JSON 格式）
- 包含更多上下文信息
- 便于日志分析和查询

### 示例 3：日志轮转

```php
<?php
declare(strict_types=1);

class RotatingLogger
{
    private string $logDir;
    private string $logFile;
    private int $maxFileSize;
    private int $maxFiles;
    
    public function __construct(string $logDir, string $logFile, int $maxFileSize = 10485760, int $maxFiles = 5)
    {
        $this->logDir = $logDir;
        $this->logFile = $logFile;
        $this->maxFileSize = $maxFileSize;  // 10MB
        $this->maxFiles = $maxFiles;
    }
    
    public function log(string $message): void
    {
        $fullPath = $this->logDir . '/' . $this->logFile;
        
        // 检查文件大小，如果超过限制则轮转
        if (file_exists($fullPath) && filesize($fullPath) >= $this->maxFileSize) {
            $this->rotate();
        }
        
        file_put_contents($fullPath, $message . "\n", FILE_APPEND | LOCK_EX);
    }
    
    private function rotate(): void
    {
        $fullPath = $this->logDir . '/' . $this->logFile;
        
        // 删除最旧的文件
        $oldestFile = $fullPath . '.' . $this->maxFiles;
        if (file_exists($oldestFile)) {
            unlink($oldestFile);
        }
        
        // 轮转文件
        for ($i = $this->maxFiles - 1; $i >= 1; $i--) {
            $oldFile = $fullPath . '.' . $i;
            $newFile = $fullPath . '.' . ($i + 1);
            
            if (file_exists($oldFile)) {
                rename($oldFile, $newFile);
            }
        }
        
        // 将当前文件重命名为 .1
        if (file_exists($fullPath)) {
            rename($fullPath, $fullPath . '.1');
        }
    }
}

// 使用
$logger = new RotatingLogger('/tmp', 'app.log', 1024 * 1024, 5);  // 1MB, 保留5个文件

for ($i = 0; $i < 10000; $i++) {
    $logger->log("Log entry {$i}: " . str_repeat('x', 100));
}
```

**说明**：
- 实现日志轮转
- 当文件达到大小限制时自动轮转
- 保留指定数量的旧日志文件

### 示例 4：日志级别配置

```php
<?php
declare(strict_types=1);

class ConfigurableLogger
{
    private string $logFile;
    private int $minLevel;
    
    private const LEVELS = [
        'DEBUG' => 0,
        'INFO' => 1,
        'WARNING' => 2,
        'ERROR' => 3,
        'CRITICAL' => 4,
    ];
    
    public function __construct(string $logFile, string $minLevel = 'INFO')
    {
        $this->logFile = $logFile;
        $this->minLevel = self::LEVELS[strtoupper($minLevel)] ?? self::LEVELS['INFO'];
    }
    
    public function setMinLevel(string $level): void
    {
        $this->minLevel = self::LEVELS[strtoupper($level)] ?? self::LEVELS['INFO'];
    }
    
    public function debug(string $message, array $context = []): void
    {
        $this->writeLog('DEBUG', $message, $context);
    }
    
    public function info(string $message, array $context = []): void
    {
        $this->writeLog('INFO', $message, $context);
    }
    
    public function warning(string $message, array $context = []): void
    {
        $this->writeLog('WARNING', $message, $context);
    }
    
    public function error(string $message, array $context = []): void
    {
        $this->writeLog('ERROR', $message, $context);
    }
    
    public function critical(string $message, array $context = []): void
    {
        $this->writeLog('CRITICAL', $message, $context);
    }
    
    private function writeLog(string $level, string $message, array $context): void
    {
        $levelValue = self::LEVELS[$level];
        if ($levelValue < $this->minLevel) {
            return;
        }
        
        $timestamp = date('Y-m-d H:i:s');
        $contextStr = !empty($context) ? ' ' . json_encode($context, JSON_UNESCAPED_UNICODE) : '';
        $logMessage = "[{$timestamp}] [{$level}] {$message}{$contextStr}\n";
        
        file_put_contents($this->logFile, $logMessage, FILE_APPEND | LOCK_EX);
    }
}

// 使用
// 开发环境：记录所有级别
$devLogger = new ConfigurableLogger('/tmp/dev.log', 'DEBUG');
$devLogger->debug('调试信息');

// 生产环境：只记录警告及以上
$prodLogger = new ConfigurableLogger('/tmp/prod.log', 'WARNING');
$prodLogger->debug('调试信息');  // 不会记录
$prodLogger->warning('警告信息');  // 会记录
```

**说明**：
- 可配置的日志级别
- 不同环境使用不同的日志级别

## 使用场景

### 场景 1：应用日志记录

记录应用程序运行过程中的重要信息。

**示例**：

```php
<?php
declare(strict_types=1);

$logger = new SimpleLogger('/tmp/app.log');

$logger->info('应用启动');
$logger->info('用户登录', ['user_id' => 123]);
$logger->error('数据库连接失败', ['host' => 'localhost']);
```

### 场景 2：错误追踪

记录错误信息，便于问题排查。

**示例**：

```php
<?php
declare(strict_types=1);

$logger = new StructuredLogger('/tmp/error.log');

try {
    riskyOperation();
} catch (Exception $e) {
    $logger->error('操作失败', [
        'exception' => get_class($e),
        'message' => $e->getMessage(),
        'file' => $e->getFile(),
        'line' => $e->getLine(),
        'trace' => $e->getTraceAsString(),
    ]);
}
```

## 注意事项

### 日志文件权限

设置适当的日志文件权限，保护日志文件。

**示例**：

```php
<?php
declare(strict_types=1);

$logFile = '/tmp/app.log';
file_put_contents($logFile, $message, FILE_APPEND);
chmod($logFile, 0644);  // 只允许所有者读写
```

### 日志文件大小

实现日志轮转，避免日志文件过大。

**示例**：见"示例 3：日志轮转"

### 日志性能

日志记录可能影响性能，应该合理使用。

**示例**：

```php
<?php
declare(strict_types=1);

// 避免在高频操作中记录详细日志
// for ($i = 0; $i < 1000000; $i++) {
//     $logger->debug("Processing item {$i}");  // 性能影响大
// }

// 只记录摘要
$logger->info('Processing started', ['total' => 1000000]);
// 处理...
$logger->info('Processing completed');
```

## 常见问题

### 问题 1：如何选择日志级别？

**回答**：根据信息的严重程度选择：
- DEBUG：详细的调试信息
- INFO：一般信息（应用启动、用户操作等）
- WARNING：警告信息（可能需要关注）
- ERROR：错误信息（需要处理）
- CRITICAL：严重错误（需要立即处理）

**示例**：

```php
<?php
declare(strict_types=1);

$logger->debug('变量值', ['var' => $value]);  // 调试信息
$logger->info('用户登录', ['user_id' => 123]);  // 一般信息
$logger->warning('内存使用较高', ['usage' => '80%']);  // 警告
$logger->error('数据库连接失败');  // 错误
$logger->critical('系统崩溃');  // 严重错误
```

### 问题 2：日志格式如何设计？

**回答**：日志格式应该包含：
- 时间戳
- 日志级别
- 消息内容
- 上下文信息（可选）

**示例**：见"示例 1：简单的日志系统"和"示例 2：结构化日志系统"

### 问题 3：如何处理日志轮转？

**回答**：实现日志轮转机制，当文件达到大小限制时自动轮转。

**示例**：见"示例 3：日志轮转"

### 问题 4：日志性能如何优化？

**回答**：
- 合理设置日志级别，避免记录过多日志
- 异步写入日志（如果可能）
- 定期清理旧日志

**示例**：见"日志性能"部分

## 最佳实践

### 1. 使用结构化日志

使用结构化日志格式（如 JSON），便于分析和查询。

**示例**：见"示例 2：结构化日志系统"

### 2. 合理设置日志级别

根据环境设置合适的日志级别。

**示例**：

```php
<?php
declare(strict_types=1);

// 开发环境：记录所有级别
$logger = new ConfigurableLogger('/tmp/dev.log', 'DEBUG');

// 生产环境：只记录警告及以上
$logger = new ConfigurableLogger('/tmp/prod.log', 'WARNING');
```

### 3. 实现日志轮转

实现日志轮转机制，避免日志文件过大。

**示例**：见"示例 3：日志轮转"

### 4. 保护日志文件

设置适当的文件权限，保护日志文件。

**示例**：见"日志文件权限"部分

### 5. 记录上下文信息

记录足够的上下文信息，便于问题排查。

**示例**：

```php
<?php
declare(strict_types=1);

$logger->error('操作失败', [
    'user_id' => $userId,
    'action' => 'update',
    'resource' => 'user',
    'error' => $errorMessage,
]);
```

## 对比分析

### 日志格式对比

| 格式       | 可读性 | 可分析性 | 性能 | 推荐使用 |
|:-----------|:-------|:---------|:-----|:---------|
| **文本格式** | ✅ 高  | ⚠️ 中    | ✅ 高| 简单场景 |
| **JSON 格式**| ⚠️ 中  | ✅ 高    | ✅ 高| 复杂场景 |

### 日志级别对比

| 级别     | 严重程度 | 使用场景           |
|:---------|:---------|:-------------------|
| **DEBUG**| 低       | 调试信息           |
| **INFO** | 低       | 一般信息           |
| **WARNING**| 中     | 警告信息           |
| **ERROR**| 高       | 错误信息           |
| **CRITICAL**| 很高   | 严重错误           |

## 练习任务

1. **日志系统实现**：实现一个完整的日志系统，支持多个日志级别和日志轮转。

2. **结构化日志工具**：创建一个工具，生成和分析结构化日志。

3. **日志分析工具**：实现一个工具，分析日志文件，统计错误、警告等。

4. **日志配置工具**：编写一个工具，管理日志配置。

5. **日志最佳实践指南**：创建一个指南，总结日志系统的最佳实践。

## 相关章节

- **[4.7.1 错误处理机制](../chapter-07-errors/section-01-error-handling.md)**：了解错误处理的相关内容
- **[4.9.2 日志记录实践](section-02-logging-practices.md)**：了解日志记录实践的相关内容
