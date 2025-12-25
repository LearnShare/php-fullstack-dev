# 5.9.3 链路追踪

## 概述

链路追踪（Tracing）可以跟踪请求在系统中的完整路径。本节详细介绍 Trace ID、Span ID、请求追踪、分布式追踪，以及链路追踪的实现。

## Trace ID 生成

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

## 在日志中使用 Trace ID

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
```

## 请求追踪

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

## 分布式追踪

### Span ID

```php
<?php
declare(strict_types=1);

class Span
{
    private string $spanId;
    private string $traceId;
    private float $startTime;
    
    public function __construct(string $traceId)
    {
        $this->traceId = $traceId;
        $this->spanId = bin2hex(random_bytes(8));
        $this->startTime = microtime(true);
    }
    
    public function finish(): array
    {
        return [
            'trace_id' => $this->traceId,
            'span_id' => $this->spanId,
            'duration' => microtime(true) - $this->startTime,
        ];
    }
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

class TracingService
{
    private Logger $logger;
    private string $traceId;

    public function __construct(Logger $logger)
    {
        $this->logger = $logger;
        $this->traceId = TraceIdGenerator::generate();
        $this->logger->pushProcessor(new TraceIdProcessor($this->traceId));
    }

    public function getTraceId(): string
    {
        return $this->traceId;
    }

    public function logSpan(string $operation, callable $callback): mixed
    {
        $span = new Span($this->traceId);
        $this->logger->info("Span started: {$operation}");
        
        try {
            $result = $callback();
            $spanData = $span->finish();
            $this->logger->info("Span finished: {$operation}", $spanData);
            return $result;
        } catch (Exception $e) {
            $spanData = $span->finish();
            $this->logger->error("Span failed: {$operation}", array_merge($spanData, [
                'error' => $e->getMessage(),
            ]));
            throw $e;
        }
    }
}
```

## 注意事项

1. **Trace ID 传递**：在请求间传递 Trace ID
2. **Span 管理**：正确管理 Span 的生命周期
3. **性能考虑**：追踪可能影响性能
4. **采样**：生产环境考虑采样

## 练习

1. 实现完整的链路追踪系统。

2. 创建一个 Trace ID 中间件。

3. 实现分布式追踪，跨服务追踪。

4. 编写追踪数据分析工具。
