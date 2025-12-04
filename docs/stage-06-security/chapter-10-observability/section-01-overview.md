# 6.10.1 可观测性概述

## 概述

可观测性是现代应用运维的关键。本节介绍可观测性概念、三大支柱（日志、指标、追踪）、可观测性 vs 监控等内容。

## 可观测性概念

### 什么是可观测性

可观测性是指通过外部输出（日志、指标、追踪）理解系统内部状态的能力。

### 三大支柱

1. **日志（Logs）**：记录事件和状态
2. **指标（Metrics）**：数值化的系统状态
3. **追踪（Traces）**：请求的完整路径

## 日志

### 结构化日志

```php
<?php
declare(strict_types=1);

use Monolog\Logger;
use Monolog\Handler\StreamHandler;
use Monolog\Formatter\JsonFormatter;

$logger = new Logger('app');
$handler = new StreamHandler('php://stdout', Logger::DEBUG);
$handler->setFormatter(new JsonFormatter());
$logger->pushHandler($handler);

$logger->info('User logged in', [
    'user_id' => 123,
    'ip' => '192.168.1.1',
    'timestamp' => time(),
]);
```

## 指标

### 性能指标

```php
<?php
declare(strict_types=1);

class MetricsCollector
{
    private array $metrics = [];
    
    public function increment(string $name, array $labels = []): void
    {
        $key = $this->getKey($name, $labels);
        $this->metrics[$key] = ($this->metrics[$key] ?? 0) + 1;
    }
    
    public function gauge(string $name, float $value, array $labels = []): void
    {
        $key = $this->getKey($name, $labels);
        $this->metrics[$key] = $value;
    }
    
    private function getKey(string $name, array $labels): string
    {
        $labelStr = json_encode($labels);
        return "{$name}{$labelStr}";
    }
    
    public function getMetrics(): array
    {
        return $this->metrics;
    }
}
```

## 追踪

### 分布式追踪

```php
<?php
declare(strict_types=1);

class Tracer
{
    private string $traceId;
    private array $spans = [];
    
    public function __construct()
    {
        $this->traceId = bin2hex(random_bytes(16));
    }
    
    public function startSpan(string $name): Span
    {
        $span = new Span($name, $this->traceId);
        $this->spans[] = $span;
        return $span;
    }
    
    public function getTraceId(): string
    {
        return $this->traceId;
    }
}
```

## 可观测性 vs 监控

### 监控

- 预设指标
- 告警驱动
- 被动响应

### 可观测性

- 探索性分析
- 主动发现
- 全面理解

## 完整示例

```php
<?php
declare(strict_types=1);

class ObservabilityService
{
    private Logger $logger;
    private MetricsCollector $metrics;
    private Tracer $tracer;
    
    public function __construct(Logger $logger, MetricsCollector $metrics, Tracer $tracer)
    {
        $this->logger = $logger;
        $this->metrics = $metrics;
        $this->tracer = $tracer;
    }
    
    public function handleRequest(Request $request): Response
    {
        $span = $this->tracer->startSpan('handle_request');
        
        try {
            $this->metrics->increment('requests_total', ['method' => $request->getMethod()]);
            
            $response = $this->processRequest($request);
            
            $this->logger->info('Request processed', [
                'trace_id' => $this->tracer->getTraceId(),
                'method' => $request->getMethod(),
                'status' => $response->getStatusCode(),
            ]);
            
            return $response;
        } finally {
            $span->finish();
        }
    }
}
```

## 最佳实践

1. **结构化日志**：使用 JSON 格式
2. **关键指标**：收集关键业务指标
3. **分布式追踪**：追踪请求完整路径
4. **统一格式**：使用统一的数据格式

## 注意事项

1. 日志不要记录敏感信息
2. 指标应该有意义
3. 追踪不应该影响性能
4. 定期审查可观测性数据

## 练习

1. 实现结构化日志记录，使用 JSON 格式。

2. 创建一个指标收集器，收集应用指标。

3. 实现分布式追踪，追踪请求路径。

4. 集成可观测性工具，建立完整的可观测性体系。
