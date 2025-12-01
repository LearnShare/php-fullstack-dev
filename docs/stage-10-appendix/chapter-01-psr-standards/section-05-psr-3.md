# 10.1.5 PSR-3 日志接口

## 概述

PSR-3（PHP Standards Recommendation 3）定义了日志接口标准，实现了框架和库之间的日志互操作。通过使用 PSR-3 接口，可以在不同的框架和库中使用统一的日志记录方式。

## 官方文档

- **标准名称**：Logger Interface
- **状态**：已接受（Accepted）
- **版本**：1.0.0
- **官方链接**：https://www.php-fig.org/psr/psr-3/

## LoggerInterface 接口

### 接口定义

```php
<?php

namespace Psr\Log;

interface LoggerInterface
{
    /**
     * 系统不可用
     */
    public function emergency(string|\Stringable $message, array $context = []): void;

    /**
     * 必须立即采取行动
     */
    public function alert(string|\Stringable $message, array $context = []): void;

    /**
     * 严重情况
     */
    public function critical(string|\Stringable $message, array $context = []): void;

    /**
     * 运行时错误，不需要立即处理但需要记录和监控
     */
    public function error(string|\Stringable $message, array $context = []): void;

    /**
     * 例外情况，但不是错误
     */
    public function warning(string|\Stringable $message, array $context = []): void;

    /**
     * 正常但重要的事件
     */
    public function notice(string|\Stringable $message, array $context = []): void;

    /**
     * 有趣的事件
     */
    public function info(string|\Stringable $message, array $context = []): void;

    /**
     * 详细的调试信息
     */
    public function debug(string|\Stringable $message, array $context = []): void;

    /**
     * 使用任意日志级别记录日志
     */
    public function log($level, string|\Stringable $message, array $context = []): void;
}
```

## 日志级别

### 级别定义

PSR-3 定义了 8 个日志级别，按严重程度从高到低：

| 级别 | 值 | 说明 | 使用场景 |
| :--- | :--- | :--- | :--- |
| `emergency` | 0 | 系统不可用 | 系统崩溃、数据库连接失败 |
| `alert` | 1 | 必须立即采取行动 | 需要立即处理的严重问题 |
| `critical` | 2 | 严重情况 | 关键业务逻辑失败 |
| `error` | 3 | 运行时错误 | 异常、错误但不影响系统运行 |
| `warning` | 4 | 警告 | 潜在问题、不推荐的操作 |
| `notice` | 5 | 正常但重要的事件 | 重要业务事件 |
| `info` | 6 | 信息 | 一般信息、业务流程 |
| `debug` | 7 | 调试信息 | 详细的调试信息 |

### 级别使用示例

```php
<?php
declare(strict_types=1);

use Psr\Log\LoggerInterface;

class PaymentService
{
    public function __construct(
        private LoggerInterface $logger
    ) {}
    
    public function processPayment(float $amount): void
    {
        // info: 记录业务流程
        $this->logger->info('Processing payment', [
            'amount' => $amount,
            'timestamp' => time(),
        ]);
        
        try {
            // 处理支付逻辑
            $this->charge($amount);
            
            $this->logger->info('Payment processed successfully', [
                'amount' => $amount,
            ]);
        } catch (PaymentException $e) {
            // error: 记录错误
            $this->logger->error('Payment processing failed', [
                'amount' => $amount,
                'error' => $e->getMessage(),
                'exception' => $e,
            ]);
            
            throw $e;
        }
    }
    
    public function checkSystemHealth(): void
    {
        if (!$this->isDatabaseConnected()) {
            // emergency: 系统不可用
            $this->logger->emergency('Database connection failed', [
                'host' => $this->dbHost,
            ]);
        }
    }
}
```

## 消息格式

### 占位符

消息可以包含占位符，占位符格式：`{key}`

**示例**：

```php
<?php
use Psr\Log\LoggerInterface;

$logger->info('User {username} logged in', [
    'username' => 'john',
    'ip' => '192.168.1.1',
]);
```

实现类应该将占位符替换为上下文数组中的对应值。

### 上下文数组

上下文数组可以包含任意数据，用于提供额外的日志信息。

**示例**：

```php
<?php
$logger->error('Order processing failed', [
    'order_id' => 12345,
    'user_id' => 67890,
    'amount' => 99.99,
    'exception' => $e,
    'trace' => $e->getTraceAsString(),
]);
```

## Monolog 实现

### 安装

```bash
composer require monolog/monolog
```

### 基础使用

```php
<?php
declare(strict_types=1);

require 'vendor/autoload.php';

use Monolog\Logger;
use Monolog\Handler\StreamHandler;

// 创建日志实例
$logger = new Logger('app');

// 添加处理器（输出到文件）
$logger->pushHandler(new StreamHandler('app.log', Logger::DEBUG));

// 记录日志
$logger->info('User logged in', ['user_id' => 123]);
$logger->error('Payment failed', ['order_id' => 456]);
```

### 多个处理器

```php
<?php
use Monolog\Logger;
use Monolog\Handler\StreamHandler;
use Monolog\Handler\RotatingFileHandler;

$logger = new Logger('app');

// 所有日志写入文件
$logger->pushHandler(new StreamHandler('app.log', Logger::DEBUG));

// 错误日志单独文件（按日期轮转）
$logger->pushHandler(
    new RotatingFileHandler('error.log', 30, Logger::ERROR)
);
```

### 格式化器

```php
<?php
use Monolog\Logger;
use Monolog\Handler\StreamHandler;
use Monolog\Formatter\JsonFormatter;

$logger = new Logger('app');
$handler = new StreamHandler('app.log', Logger::DEBUG);
$handler->setFormatter(new JsonFormatter());
$logger->pushHandler($handler);

$logger->info('User action', ['action' => 'login', 'user_id' => 123]);
// 输出 JSON 格式
```

## 实际应用

### 在框架中使用

**Laravel**：

```php
<?php
use Illuminate\Support\Facades\Log;

Log::info('User logged in', ['user_id' => $userId]);
Log::error('Payment failed', ['order_id' => $orderId]);
```

**Symfony**：

```php
<?php
use Psr\Log\LoggerInterface;

class UserService
{
    public function __construct(
        private LoggerInterface $logger
    ) {}
    
    public function createUser(array $data): void
    {
        $this->logger->info('Creating user', $data);
        // ...
    }
}
```

### 在库中使用

```php
<?php
declare(strict_types=1);

namespace MyLibrary;

use Psr\Log\LoggerInterface;
use Psr\Log\NullLogger;

class MyService
{
    private LoggerInterface $logger;
    
    public function __construct(?LoggerInterface $logger = null)
    {
        // 如果没有提供 logger，使用 NullLogger（不记录任何日志）
        $this->logger = $logger ?? new NullLogger();
    }
    
    public function doSomething(): void
    {
        $this->logger->info('Doing something');
        // ...
    }
}
```

## 最佳实践

### 1. 使用适当的日志级别

- `emergency`：系统崩溃
- `alert`：需要立即处理
- `critical`：关键业务失败
- `error`：错误但不影响运行
- `warning`：潜在问题
- `notice`：重要事件
- `info`：一般信息
- `debug`：调试信息

### 2. 提供有意义的上下文

**推荐**：

```php
$logger->error('Payment failed', [
    'order_id' => $orderId,
    'user_id' => $userId,
    'amount' => $amount,
    'payment_method' => $method,
    'error_code' => $errorCode,
]);
```

**不推荐**：

```php
$logger->error('Payment failed');
```

### 3. 不要在日志中记录敏感信息

**错误**：

```php
$logger->info('User login', [
    'password' => $password,  // 危险！
    'credit_card' => $cardNumber,  // 危险！
]);
```

**正确**：

```php
$logger->info('User login', [
    'user_id' => $userId,
    'ip' => $ip,
]);
```

### 4. 使用结构化日志

**推荐**：JSON 格式

```php
$handler->setFormatter(new JsonFormatter());
```

**输出**：

```json
{
    "message": "User logged in",
    "context": {
        "user_id": 123,
        "ip": "192.168.1.1"
    },
    "level": 200,
    "level_name": "INFO",
    "channel": "app",
    "datetime": "2025-01-15T10:30:00+00:00"
}
```

## 练习

1. 创建一个实现 PSR-3 接口的简单日志类。

2. 使用 Monolog 配置多处理器日志系统。

3. 在实际项目中使用 PSR-3 接口记录日志。

4. 配置日志轮转和格式化。

## 总结

PSR-3 日志接口提供了：

- 统一的日志记录接口
- 8 个标准日志级别
- 上下文信息支持
- 占位符插值
- 框架和库之间的互操作性

遵循 PSR-3 标准，可以：

- 在不同框架和库中使用统一的日志方式
- 提高代码的可移植性
- 简化日志记录的实现
- 使日志更容易被分析和监控
