# 5.9.2 结构化日志

## 概述

结构化日志使用机器可读的格式（如 JSON），便于日志分析和处理。本节详细介绍结构化日志格式、JSON 日志、日志字段设计，以及日志聚合。

## 结构化日志格式

### JSON 格式

```php
<?php
declare(strict_types=1);

use Monolog\Logger;
use Monolog\Handler\StreamHandler;
use Monolog\Formatter\JsonFormatter;

$logger = new Logger('app');

$handler = new StreamHandler(__DIR__ . '/logs/app.json', Logger::DEBUG);
$handler->setFormatter(new JsonFormatter());
$logger->pushHandler($handler);

// 结构化日志
$logger->info('Order created', [
    'order_id' => 123,
    'user_id' => 1,
    'amount' => 99.99,
    'items' => [
        ['product_id' => 1, 'quantity' => 2],
        ['product_id' => 2, 'quantity' => 1],
    ],
]);
```

### 自定义格式化器

```php
<?php
declare(strict_types=1);

use Monolog\Formatter\FormatterInterface;

class CustomJsonFormatter implements FormatterInterface
{
    public function format(array $record): string
    {
        $output = [
            'timestamp' => $record['datetime']->format('Y-m-d\TH:i:s.v\Z'),
            'level' => $record['level_name'],
            'channel' => $record['channel'],
            'message' => $record['message'],
            'context' => $record['context'],
            'extra' => $record['extra'],
        ];
        
        return json_encode($output, JSON_UNESCAPED_UNICODE | JSON_UNESCAPED_SLASHES) . "\n";
    }
    
    public function formatBatch(array $records): string
    {
        $output = [];
        foreach ($records as $record) {
            $output[] = $this->format($record);
        }
        return implode('', $output);
    }
}
```

## 日志字段

### 标准字段

```php
<?php
declare(strict_types=1);

$logger->info('User action', [
    'user_id' => 1,
    'action' => 'login',
    'ip' => '192.168.1.1',
    'user_agent' => $_SERVER['HTTP_USER_AGENT'] ?? '',
    'timestamp' => time(),
]);
```

### 上下文信息

```php
<?php
declare(strict_types=1);

use Monolog\Processor\ProcessorInterface;

class ContextProcessor implements ProcessorInterface
{
    public function __invoke(array $record): array
    {
        $record['context']['request_id'] = $_SERVER['REQUEST_ID'] ?? uniqid();
        $record['context']['ip'] = $_SERVER['REMOTE_ADDR'] ?? 'unknown';
        $record['context']['user_agent'] = $_SERVER['HTTP_USER_AGENT'] ?? '';
        return $record;
    }
}

$logger->pushProcessor(new ContextProcessor());
```

## 日志聚合

### 集中式日志

```php
<?php
declare(strict_types=1);

use Monolog\Handler\SocketHandler;

// 发送到日志服务器
$handler = new SocketHandler('tcp://log-server:514', Logger::DEBUG);
$logger->pushHandler($handler);
```

## 完整示例

```php
<?php
declare(strict_types=1);

class StructuredLogger
{
    private Logger $logger;

    public function __construct()
    {
        $this->logger = new Logger('app');
        
        $handler = new StreamHandler(__DIR__ . '/logs/app.json', Logger::DEBUG);
        $handler->setFormatter(new JsonFormatter());
        $this->logger->pushHandler($handler);
        
        $this->logger->pushProcessor(new ContextProcessor());
    }

    public function log(string $level, string $message, array $context = []): void
    {
        $this->logger->log($level, $message, $context);
    }
}
```

## 注意事项

1. **格式统一**：保持日志格式的一致性
2. **字段设计**：合理设计日志字段
3. **性能考虑**：JSON 编码可能影响性能
4. **日志聚合**：使用集中式日志系统

## 练习

1. 实现结构化日志系统，使用 JSON 格式。

2. 创建自定义日志格式化器。

3. 实现日志聚合功能。

4. 编写日志分析工具。
