# 5.9 日志体系与监控

## 目标

- 理解 PSR-3 日志标准。
- 掌握 Monolog 的使用与配置。
- 熟悉结构化日志（JSON 日志）的格式。
- 了解请求链路追踪（Trace ID）的实现。
- 掌握错误通知机制（Email/Slack/Sentry）。

## PSR-3 日志标准

### 日志级别

| 级别      | 说明                     | 使用场景                           |
| :-------- | :----------------------- | :--------------------------------- |
| `DEBUG`   | 调试信息                 | 详细的调试信息                     |
| `INFO`    | 一般信息                 | 程序正常运行信息                   |
| `NOTICE`  | 通知信息                 | 正常但值得注意的事件               |
| `WARNING` | 警告                     | 潜在问题，但不影响运行             |
| `ERROR`   | 错误                     | 运行时错误，但程序可以继续         |
| `CRITICAL` | 严重错误                 | 严重错误，需要立即处理             |
| `ALERT`   | 警报                     | 需要立即采取行动                   |
| `EMERGENCY` | 紧急情况                 | 系统不可用                         |

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

## Monolog

### 基础配置

```php
<?php
declare(strict_types=1);

require __DIR__ . '/vendor/autoload.php';

use Monolog\Logger;
use Monolog\Handler\StreamHandler;
use Monolog\Formatter\LineFormatter;

// 创建 Logger
$logger = new Logger('app');

// 添加处理器
$handler = new StreamHandler(__DIR__ . '/logs/app.log', Logger::DEBUG);
$handler->setFormatter(new LineFormatter(
    "[%datetime%] %channel%.%level_name%: %message% %context% %extra%\n"
));
$logger->pushHandler($handler);

// 使用
$logger->info('User logged in', ['user_id' => 1, 'ip' => '192.168.1.1']);
$logger->error('Database connection failed', ['error' => $e->getMessage()]);
```

### 多种处理器

```php
<?php
declare(strict_types=1);

use Monolog\Logger;
use Monolog\Handler\StreamHandler;
use Monolog\Handler\RotatingFileHandler;
use Monolog\Handler\SyslogHandler;
use Monolog\Handler\ErrorLogHandler;

$logger = new Logger('app');

// 文件处理器（按日期轮转）
$logger->pushHandler(new RotatingFileHandler(
    __DIR__ . '/logs/app.log',
    30,  // 保留 30 天
    Logger::DEBUG
));

// 系统日志处理器
$logger->pushHandler(new SyslogHandler('myapp', LOG_USER, Logger::WARNING));

// PHP 错误日志处理器
$logger->pushHandler(new ErrorLogHandler(ErrorLogHandler::OPERATING_SYSTEM, Logger::ERROR));
```

### 频道（Channel）

```php
<?php
declare(strict_types=1);

// 不同模块使用不同频道
$dbLogger = new Logger('database');
$dbLogger->pushHandler(new StreamHandler(__DIR__ . '/logs/database.log'));

$apiLogger = new Logger('api');
$apiLogger->pushHandler(new StreamHandler(__DIR__ . '/logs/api.log'));

$authLogger = new Logger('auth');
$authLogger->pushHandler(new StreamHandler(__DIR__ . '/logs/auth.log'));

// 使用
$dbLogger->info('Query executed', ['sql' => 'SELECT * FROM users']);
$apiLogger->info('API request', ['endpoint' => '/api/users', 'method' => 'GET']);
$authLogger->warning('Failed login attempt', ['ip' => '192.168.1.1']);
```

## 结构化日志

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

// 输出 JSON：
// {"message":"Order created","context":{"order_id":123,"user_id":1,"amount":99.99},"level":200,"level_name":"INFO","channel":"app","datetime":"2025-01-01T12:00:00+00:00"}
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

## 请求链路追踪

### Trace ID 生成

```php
<?php
declare(strict_types=1);

class TraceIdGenerator
{
    public static function generate(): string
    {
        return bin2hex(random_bytes(16));
    }
}

// 在请求开始时生成
$traceId = TraceIdGenerator::generate();
```

### 在日志中使用 Trace ID

```php
<?php
declare(strict_types=1);

use Monolog\Processor\ProcessorInterface;

class TraceIdProcessor implements ProcessorInterface
{
    private string $traceId;
    
    public function __construct(?string $traceId = null)
    {
        $this->traceId = $traceId ?? TraceIdGenerator::generate();
    }
    
    public function __invoke(array $record): array
    {
        $record['extra']['trace_id'] = $this->traceId;
        return $record;
    }
}

// 使用
$logger = new Logger('app');
$logger->pushProcessor(new TraceIdProcessor($traceId));
$logger->pushHandler(new StreamHandler(__DIR__ . '/logs/app.log'));

// 所有日志都会包含 trace_id
$logger->info('Request started', ['endpoint' => '/api/users']);
$logger->info('Database query', ['sql' => 'SELECT * FROM users']);
$logger->info('Request completed', ['status' => 200]);
```

### 中间件集成

```php
<?php
declare(strict_types=1);

class LoggingMiddleware
{
    private Logger $logger;
    
    public function __construct(Logger $logger)
    {
        $this->logger = $logger;
    }
    
    public function handle(callable $next): mixed
    {
        $traceId = TraceIdGenerator::generate();
        $this->logger->pushProcessor(new TraceIdProcessor($traceId));
        
        // 设置响应头
        header("X-Trace-Id: {$traceId}");
        
        $startTime = microtime(true);
        
        try {
            $this->logger->info('Request started', [
                'method' => $_SERVER['REQUEST_METHOD'],
                'uri' => $_SERVER['REQUEST_URI'],
                'ip' => $_SERVER['REMOTE_ADDR'] ?? 'unknown',
            ]);
            
            $response = $next();
            
            $duration = microtime(true) - $startTime;
            
            $this->logger->info('Request completed', [
                'status' => http_response_code(),
                'duration' => $duration,
            ]);
            
            return $response;
            
        } catch (Exception $e) {
            $duration = microtime(true) - $startTime;
            
            $this->logger->error('Request failed', [
                'error' => $e->getMessage(),
                'trace' => $e->getTraceAsString(),
                'duration' => $duration,
            ]);
            
            throw $e;
        }
    }
}
```

## 错误通知

### Email 通知

```php
<?php
declare(strict_types=1);

use Monolog\Logger;
use Monolog\Handler\NativeMailerHandler;

$logger = new Logger('app');

// Email 处理器（仅 ERROR 及以上级别）
$mailHandler = new NativeMailerHandler(
    'admin@example.com',  // 收件人
    'Error Alert',  // 主题
    'noreply@example.com',  // 发件人
    Logger::ERROR  // 最低级别
);

$logger->pushHandler($mailHandler);

// 触发邮件通知
$logger->error('Critical error occurred', [
    'error' => $e->getMessage(),
    'file' => $e->getFile(),
    'line' => $e->getLine(),
]);
```

### Slack 通知

```php
<?php
declare(strict_types=1);

use Monolog\Logger;
use Monolog\Handler\SlackWebhookHandler;

$logger = new Logger('app');

$slackHandler = new SlackWebhookHandler(
    'https://hooks.slack.com/services/YOUR/WEBHOOK/URL',
    '#alerts',  // 频道
    'App Bot',  // 用户名
    false,  // 使用附件
    null,  // 图标
    Logger::ERROR  // 最低级别
);

$logger->pushHandler($slackHandler);

$logger->error('Error occurred', ['error' => $e->getMessage()]);
```

### Sentry 集成

```php
<?php
declare(strict_types=1);

require __DIR__ . '/vendor/autoload.php';

use Monolog\Logger;
use Sentry\Monolog\Handler as SentryHandler;
use Sentry\State\Scope;

\Sentry\init([
    'dsn' => 'https://your-sentry-dsn@sentry.io/project-id',
    'environment' => 'production',
]);

$logger = new Logger('app');

$sentryHandler = new SentryHandler(Logger::ERROR);
$logger->pushHandler($sentryHandler);

// 使用
try {
    // 业务代码
} catch (Exception $e) {
    $logger->error('Error occurred', [
        'error' => $e->getMessage(),
        'exception' => $e,
    ]);
    
    // 或直接使用 Sentry
    \Sentry\captureException($e);
}
```

## 日志配置示例

### 完整配置

```php
<?php
declare(strict_types=1);

use Monolog\Logger;
use Monolog\Handler\RotatingFileHandler;
use Monolog\Handler\StreamHandler;
use Monolog\Formatter\JsonFormatter;
use Monolog\Processor\MemoryUsageProcessor;
use Monolog\Processor\ProcessIdProcessor;
use Monolog\Processor\UidProcessor;

function createLogger(string $channel, string $traceId = null): Logger
{
    $logger = new Logger($channel);
    
    // 添加处理器 ID
    $logger->pushProcessor(new UidProcessor());
    $logger->pushProcessor(new ProcessIdProcessor());
    $logger->pushProcessor(new MemoryUsageProcessor());
    
    if ($traceId !== null) {
        $logger->pushProcessor(new TraceIdProcessor($traceId));
    }
    
    // 文件处理器（JSON 格式）
    $fileHandler = new RotatingFileHandler(
        __DIR__ . "/logs/{$channel}.json",
        30,
        Logger::DEBUG
    );
    $fileHandler->setFormatter(new JsonFormatter());
    $logger->pushHandler($fileHandler);
    
    // 控制台处理器（开发环境）
    if (getenv('APP_ENV') === 'development') {
        $consoleHandler = new StreamHandler('php://stdout', Logger::DEBUG);
        $logger->pushHandler($consoleHandler);
    }
    
    return $logger;
}

// 使用
$logger = createLogger('app', $traceId);
$logger->info('Application started');
```

## 最佳实践

### 1. 使用结构化日志

```php
// 不推荐
$logger->info("User {$userId} logged in from {$ip}");

// 推荐
$logger->info('User logged in', [
    'user_id' => $userId,
    'ip' => $ip,
]);
```

### 2. 包含足够的上下文

```php
$logger->error('Payment failed', [
    'user_id' => $userId,
    'order_id' => $orderId,
    'amount' => $amount,
    'payment_method' => $method,
    'error_code' => $errorCode,
]);
```

### 3. 不要记录敏感信息

```php
// 不推荐
$logger->info('User data', ['password' => $password, 'token' => $token]);

// 推荐
$logger->info('User data', [
    'user_id' => $userId,
    'email' => $email,
    // 不记录密码和 token
]);
```

### 4. 使用适当的日志级别

```php
$logger->debug('Detailed debug information');
$logger->info('User action completed');
$logger->warning('Potential issue');
$logger->error('Error occurred');
$logger->critical('Critical system error');
```

## 练习

1. 配置一个完整的 Monolog 日志系统，支持多频道和文件轮转。

2. 实现一个请求链路追踪系统，为每个请求生成 Trace ID 并记录在日志中。

3. 创建一个结构化日志格式化器，输出 JSON 格式的日志。

4. 集成 Sentry 错误监控，捕获和报告应用错误。

5. 设计一个日志分析系统，解析 JSON 日志并生成统计报告。

6. 实现一个日志轮转和归档系统，自动压缩和清理旧日志。
