# 4.9.2 结构化日志

## 概述

结构化日志是使用固定格式（通常是 JSON）记录的日志，包含结构化的字段和数据。与传统的文本日志不同，结构化日志便于程序解析、分析和查询，是现代日志系统推荐使用的格式。

在实际应用中，我们使用结构化日志记录程序运行过程中的信息，包括时间戳、日志级别、消息内容、上下文信息等。理解结构化日志的概念、格式设计、字段选择等，对于构建有效的日志系统至关重要。

**主要内容**：
- 结构化日志概述（什么是结构化日志、为什么使用结构化日志）
- JSON 格式日志
- 日志字段设计
- 上下文信息记录
- 日志库使用（Monolog 等）
- 实际应用场景和最佳实践

## 特性

- **结构化格式**：使用固定格式（JSON）记录日志
- **易于解析**：便于程序解析和分析
- **上下文丰富**：可以包含丰富的上下文信息
- **可查询**：便于日志搜索和查询

## 语法/定义

### 结构化日志格式

结构化日志通常使用 JSON 格式，包含以下字段：
- `timestamp`：时间戳
- `level`：日志级别
- `message`：消息内容
- `context`：上下文信息（可选）

## 基本用法

### 示例 1：JSON 格式日志

```php
<?php
declare(strict_types=1);

class StructuredLogger
{
    private string $logFile;
    
    public function __construct(string $logFile)
    {
        $this->logFile = $logFile;
    }
    
    public function log(string $level, string $message, array $context = []): void
    {
        $logEntry = [
            'timestamp' => date('c'),  // ISO 8601 格式
            'level' => $level,
            'message' => $message,
            'context' => $context,
        ];
        
        $logLine = json_encode($logEntry, JSON_UNESCAPED_UNICODE) . "\n";
        file_put_contents($this->logFile, $logLine, FILE_APPEND | LOCK_EX);
    }
    
    public function info(string $message, array $context = []): void
    {
        $this->log('INFO', $message, $context);
    }
    
    public function error(string $message, array $context = []): void
    {
        $this->log('ERROR', $message, $context);
    }
}

// 使用
$logger = new StructuredLogger('/tmp/app.log');

$logger->info('用户登录', [
    'user_id' => 123,
    'username' => 'alice',
    'ip' => '192.168.1.1',
    'user_agent' => 'Mozilla/5.0...',
]);

$logger->error('数据库连接失败', [
    'host' => 'localhost',
    'port' => 3306,
    'database' => 'myapp',
    'error' => 'Connection timeout',
]);
```

**说明**：
- 使用 JSON 格式记录日志
- 包含时间戳、级别、消息、上下文等信息

### 示例 2：增强的结构化日志

```php
<?php
declare(strict_types=1);

class EnhancedStructuredLogger
{
    private string $logFile;
    private string $serviceName;
    private string $environment;
    
    public function __construct(string $logFile, string $serviceName = 'app', string $environment = 'production')
    {
        $this->logFile = $logFile;
        $this->serviceName = $serviceName;
        $this->environment = $environment;
    }
    
    public function log(string $level, string $message, array $context = []): void
    {
        $logEntry = [
            'timestamp' => date('c'),
            'level' => $level,
            'service' => $this->serviceName,
            'environment' => $this->environment,
            'message' => $message,
            'context' => $context,
            'memory' => [
                'current' => memory_get_usage(),
                'peak' => memory_get_peak_usage(),
            ],
        ];
        
        // 添加请求信息（如果可用）
        if (isset($_SERVER['REQUEST_ID'])) {
            $logEntry['request_id'] = $_SERVER['REQUEST_ID'];
        }
        
        $logLine = json_encode($logEntry, JSON_UNESCAPED_UNICODE | JSON_PARTIAL_OUTPUT_ON_ERROR) . "\n";
        file_put_contents($this->logFile, $logLine, FILE_APPEND | LOCK_EX);
    }
    
    public function info(string $message, array $context = []): void
    {
        $this->log('INFO', $message, $context);
    }
    
    public function error(string $message, array $context = []): void
    {
        $this->log('ERROR', $message, $context);
    }
}

// 使用
$logger = new EnhancedStructuredLogger('/tmp/app.log', 'user-service', 'production');

$logger->info('用户创建', [
    'user_id' => 456,
    'username' => 'bob',
    'email' => 'bob@example.com',
]);
```

**说明**：
- 增强的日志格式，包含更多信息
- 包含服务名称、环境、内存使用等

### 示例 3：使用 Monolog（PSR-3）

```php
<?php
declare(strict_types=1);

// 需要安装 Monolog: composer require monolog/monolog
// 这里仅作为示例，展示结构化日志的概念

class MonologStructuredLogger
{
    /**
     * 配置 Monolog 记录结构化日志（JSON 格式）
     * 
     * 实际使用时需要安装 Monolog 库
     */
    public static function createLogger(string $logFile): object
    {
        // 实际代码示例（需要 Monolog 库）
        // use Monolog\Logger;
        // use Monolog\Handler\StreamHandler;
        // use Monolog\Formatter\JsonFormatter;
        // 
        // $logger = new Logger('app');
        // $handler = new StreamHandler($logFile, Logger::DEBUG);
        // $handler->setFormatter(new JsonFormatter());
        // $logger->pushHandler($handler);
        // 
        // return $logger;
        
        // 这里返回一个简单的实现示例
        return new class($logFile) {
            private string $logFile;
            
            public function __construct(string $logFile)
            {
                $this->logFile = $logFile;
            }
            
            public function info(string $message, array $context = []): void
            {
                $this->log('INFO', $message, $context);
            }
            
            public function error(string $message, array $context = []): void
            {
                $this->log('ERROR', $message, $context);
            }
            
            private function log(string $level, string $message, array $context): void
            {
                $logEntry = [
                    'timestamp' => date('c'),
                    'level' => $level,
                    'message' => $message,
                    'context' => $context,
                ];
                
                $logLine = json_encode($logEntry, JSON_UNESCAPED_UNICODE) . "\n";
                file_put_contents($this->logFile, $logLine, FILE_APPEND | LOCK_EX);
            }
        };
    }
}

// 使用
$logger = MonologStructuredLogger::createLogger('/tmp/app.log');
$logger->info('操作成功', ['action' => 'create', 'resource' => 'user']);
```

**说明**：
- Monolog 是 PHP 中常用的日志库，支持 PSR-3
- 支持 JSON 格式化输出
- 这里仅作为概念示例

### 示例 4：日志字段设计

```php
<?php
declare(strict_types=1);

class WellDesignedLogger
{
    private string $logFile;
    
    public function __construct(string $logFile)
    {
        $this->logFile = $logFile;
    }
    
    /**
     * 记录操作日志
     */
    public function logOperation(string $level, string $operation, array $details = []): void
    {
        $logEntry = [
            // 必需字段
            'timestamp' => date('c'),
            'level' => $level,
            'operation' => $operation,
            
            // 上下文信息
            'details' => $details,
            
            // 系统信息（可选）
            'system' => [
                'memory_usage' => memory_get_usage(),
                'memory_peak' => memory_get_peak_usage(),
            ],
        ];
        
        // 添加用户信息（如果可用）
        if (isset($details['user_id'])) {
            $logEntry['user'] = [
                'id' => $details['user_id'],
            ];
        }
        
        // 添加请求信息（如果可用）
        if (isset($_SERVER['REQUEST_ID'])) {
            $logEntry['request_id'] = $_SERVER['REQUEST_ID'];
        }
        
        $logLine = json_encode($logEntry, JSON_UNESCAPED_UNICODE) . "\n";
        file_put_contents($this->logFile, $logLine, FILE_APPEND | LOCK_EX);
    }
    
    public function logUserAction(string $action, int $userId, array $details = []): void
    {
        $this->logOperation('INFO', $action, array_merge($details, ['user_id' => $userId]));
    }
    
    public function logError(string $message, array $context = []): void
    {
        $this->logOperation('ERROR', 'error_occurred', array_merge(['message' => $message], $context));
    }
}

// 使用
$logger = new WellDesignedLogger('/tmp/app.log');

$logger->logUserAction('user_login', 123, [
    'ip' => '192.168.1.1',
    'user_agent' => 'Mozilla/5.0...',
]);

$logger->logError('Database connection failed', [
    'host' => 'localhost',
    'error_code' => 'CONN_TIMEOUT',
]);
```

**说明**：
- 设计良好的日志字段
- 包含必需字段和可选字段
- 便于后续分析和查询

## 使用场景

### 场景 1：用户操作日志

记录用户的操作，便于审计和分析。

**示例**：

```php
<?php
declare(strict_types=1);

$logger = new StructuredLogger('/tmp/user-actions.log');

$logger->info('用户登录', [
    'user_id' => 123,
    'ip' => '192.168.1.1',
    'timestamp' => time(),
]);

$logger->info('用户更新资料', [
    'user_id' => 123,
    'changes' => ['email' => 'new@example.com'],
]);
```

### 场景 2：错误日志

记录错误信息，便于问题排查。

**示例**：

```php
<?php
declare(strict_types=1);

$logger = new StructuredLogger('/tmp/errors.log');

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

### 字段一致性

保持日志字段的一致性，便于分析和查询。

**示例**：

```php
<?php
declare(strict_types=1);

// ✅ 一致：总是使用相同的字段名
$logger->info('操作', ['user_id' => 123, 'action' => 'login']);
$logger->info('操作', ['user_id' => 456, 'action' => 'logout']);

// ❌ 不一致：使用不同的字段名
// $logger->info('操作', ['userId' => 123]);  // 有时用 userId
// $logger->info('操作', ['user_id' => 456]);  // 有时用 user_id
```

### 敏感信息

不要在日志中记录敏感信息（如密码、token 等）。

**示例**：

```php
<?php
declare(strict_types=1);

// ❌ 错误：记录敏感信息
// $logger->info('用户登录', ['password' => $password]);

// ✅ 正确：不记录敏感信息
$logger->info('用户登录', [
    'user_id' => $userId,
    'username' => $username,
    // 不包含密码
]);
```

### JSON 编码错误

处理 JSON 编码可能的错误。

**示例**：

```php
<?php
declare(strict_types=1);

$logEntry = ['message' => $message, 'data' => $data];
$json = json_encode($logEntry, JSON_UNESCAPED_UNICODE | JSON_PARTIAL_OUTPUT_ON_ERROR);

if ($json === false) {
    // 处理编码错误
    $json = json_encode(['message' => 'Log encoding failed'], JSON_UNESCAPED_UNICODE);
}
```

## 常见问题

### 问题 1：为什么使用结构化日志？

**回答**：结构化日志便于程序解析、分析和查询，比文本日志更适合现代应用。

**示例**：

```php
<?php
declare(strict_types=1);

// 结构化日志（JSON）便于解析
// {"timestamp":"2024-01-01T12:00:00+00:00","level":"INFO","message":"User login","user_id":123}

// 文本日志难以解析
// [2024-01-01 12:00:00] INFO: User login (user_id: 123)
```

### 问题 2：如何设计日志字段？

**回答**：包含必需字段（时间戳、级别、消息）和有用的上下文信息，保持字段一致性。

**示例**：见"示例 4：日志字段设计"

### 问题 3：如何处理敏感信息？

**回答**：不要在日志中记录敏感信息，如密码、token、信用卡号等。

**示例**：见"敏感信息"部分

## 最佳实践

### 1. 使用 JSON 格式

使用 JSON 格式记录结构化日志，便于解析。

**示例**：见"示例 1：JSON 格式日志"

### 2. 设计一致的字段

保持日志字段的一致性，便于分析和查询。

**示例**：见"字段一致性"部分

### 3. 包含上下文信息

记录足够的上下文信息，便于问题排查。

**示例**：

```php
<?php
declare(strict_types=1);

$logger->error('操作失败', [
    'operation' => 'create_user',
    'user_id' => $userId,
    'error_code' => 'VALIDATION_FAILED',
    'validation_errors' => $errors,
]);
```

### 4. 保护敏感信息

不要在日志中记录敏感信息。

**示例**：见"敏感信息"部分

## 对比分析

### 结构化日志 vs 文本日志

| 特性         | 结构化日志（JSON）          | 文本日志                    |
|:-------------|:----------------------------|:----------------------------|
| **可解析性** | ✅ 易于程序解析             | ⚠️ 需要正则表达式解析       |
| **可查询性** | ✅ 便于查询和分析           | ⚠️ 查询困难                 |
| **可读性**   | ⚠️ 需要格式化查看           | ✅ 人类易读                 |
| **灵活性**   | ✅ 易于扩展字段             | ⚠️ 扩展困难                 |
| **推荐使用** | ✅ 现代应用推荐             | ⚠️ 简单场景                 |

## 练习任务

1. **结构化日志实现**：实现一个完整的结构化日志系统。

2. **日志字段设计**：设计一套日志字段规范。

3. **日志解析工具**：创建一个工具，解析和分析结构化日志。

4. **日志查询工具**：编写一个工具，查询结构化日志。

5. **结构化日志最佳实践指南**：创建一个指南，总结结构化日志的最佳实践。

## 相关章节

- **[4.9.1 日志体系设计](section-01-logging-system.md)**：了解日志体系设计的相关内容
- **[4.9.3 链路追踪](section-03-tracing.md)**：了解链路追踪的相关内容
