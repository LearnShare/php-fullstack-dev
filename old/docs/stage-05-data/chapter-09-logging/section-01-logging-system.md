# 5.9.1 日志体系设计

## 概述

完整的日志体系是生产系统的基础。本节详细介绍 PSR-3 标准、日志级别、Monolog 配置、日志处理器，以及日志体系的设计。

## PSR-3 日志标准

### 日志级别

| 级别 | 说明 | 使用场景 |
| :--- | :--- | :--- |
| `DEBUG` | 调试信息 | 详细的调试信息 |
| `INFO` | 一般信息 | 程序正常运行信息 |
| `NOTICE` | 通知信息 | 正常但值得注意的事件 |
| `WARNING` | 警告 | 潜在问题，但不影响运行 |
| `ERROR` | 错误 | 运行时错误，但程序可以继续 |
| `CRITICAL` | 严重错误 | 严重错误，需要立即处理 |
| `ALERT` | 警报 | 需要立即采取行动 |
| `EMERGENCY` | 紧急情况 | 系统不可用 |

### PSR-3 接口

```php
<?php
declare(strict_types=1);

namespace Psr\Log;

interface LoggerInterface
{
    public function emergency(string $message, array $context = []): void;
    public function alert(string $message, array $context = []): void;
    public function critical(string $message, array $context = []): void;
    public function error(string $message, array $context = []): void;
    public function warning(string $message, array $context = []): void;
    public function notice(string $message, array $context = []): void;
    public function info(string $message, array $context = []): void;
    public function debug(string $message, array $context = []): void;
    public function log($level, string $message, array $context = []): void;
}
```

## Monolog 配置

### 基础配置

```php
<?php
declare(strict_types=1);

require __DIR__ . '/vendor/autoload.php';

use Monolog\Logger;
use Monolog\Handler\StreamHandler;
use Monolog\Formatter\LineFormatter;

$logger = new Logger('app');

$handler = new StreamHandler(__DIR__ . '/logs/app.log', Logger::DEBUG);
$handler->setFormatter(new LineFormatter(
    "[%datetime%] %channel%.%level_name%: %message% %context% %extra%\n"
));
$logger->pushHandler($handler);

$logger->info('User logged in', ['user_id' => 1, 'ip' => '192.168.1.1']);
```

### 多种处理器

```php
<?php
declare(strict_types=1);

use Monolog\Logger;
use Monolog\Handler\RotatingFileHandler;
use Monolog\Handler\SyslogHandler;

$logger = new Logger('app');

// 文件处理器（按日期轮转）
$logger->pushHandler(new RotatingFileHandler(
    __DIR__ . '/logs/app.log',
    30,  // 保留 30 天
    Logger::DEBUG
));

// 系统日志处理器
$logger->pushHandler(new SyslogHandler('myapp', LOG_USER, Logger::WARNING));
```

## 日志处理器

### 不同级别的处理器

```php
<?php
declare(strict_types=1);

$logger = new Logger('app');

// DEBUG 及以上写入文件
$logger->pushHandler(new StreamHandler(__DIR__ . '/logs/app.log', Logger::DEBUG));

// ERROR 及以上发送邮件
$logger->pushHandler(new NativeMailerHandler(
    'admin@example.com',
    'Error Alert',
    'noreply@example.com',
    Logger::ERROR
));
```

## 完整示例

```php
<?php
declare(strict_types=1);

class LoggingService
{
    private Logger $logger;

    public function __construct()
    {
        $this->logger = new Logger('app');
        
        // 文件处理器
        $this->logger->pushHandler(
            new RotatingFileHandler(__DIR__ . '/logs/app.log', 30, Logger::DEBUG)
        );
        
        // 错误邮件通知
        $this->logger->pushHandler(
            new NativeMailerHandler('admin@example.com', 'Error', 'noreply@example.com', Logger::ERROR)
        );
    }

    public function log(string $level, string $message, array $context = []): void
    {
        $this->logger->log($level, $message, $context);
    }
}
```

## 注意事项

1. **日志级别**：合理设置日志级别
2. **日志轮转**：使用 RotatingFileHandler 防止日志文件过大
3. **性能考虑**：异步日志处理提升性能
4. **安全考虑**：避免记录敏感信息

## 练习

1. 创建一个完整的日志系统，支持多种处理器。

2. 实现日志级别配置和动态调整。

3. 编写日志轮转和清理机制。

4. 实现日志聚合和分析功能。
