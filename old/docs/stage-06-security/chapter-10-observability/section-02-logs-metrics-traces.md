# 6.10.2 日志、指标、追踪

## 概述

日志、指标、追踪是可观测性的三大支柱。本节详细介绍结构化日志、Prometheus 指标、OpenTelemetry 追踪等内容。

## 结构化日志

### Monolog 配置

```php
<?php
declare(strict_types=1);

use Monolog\Logger;
use Monolog\Handler\StreamHandler;
use Monolog\Handler\RotatingFileHandler;
use Monolog\Formatter\JsonFormatter;

$logger = new Logger('app');

// 文件处理器
$fileHandler = new RotatingFileHandler(__DIR__ . '/logs/app.log', 7, Logger::DEBUG);
$fileHandler->setFormatter(new JsonFormatter());
$logger->pushHandler($fileHandler);

// 控制台处理器
$consoleHandler = new StreamHandler('php://stdout', Logger::INFO);
$consoleHandler->setFormatter(new JsonFormatter());
$logger->pushHandler($consoleHandler);
```

### 日志记录

```php
<?php
declare(strict_types=1);

$logger->debug('Debug message', ['context' => 'value']);
$logger->info('User logged in', ['user_id' => 123]);
$logger->warning('High memory usage', ['memory' => '512MB']);
$logger->error('Database connection failed', ['error' => $e->getMessage()]);
```

## Prometheus 指标

### 指标类型

```php
<?php
declare(strict_types=1);

class PrometheusMetrics
{
    private array $counters = [];
    private array $gauges = [];
    private array $histograms = [];
    
    public function incrementCounter(string $name, array $labels = []): void
    {
        $key = $this->getKey($name, $labels);
        $this->counters[$key] = ($this->counters[$key] ?? 0) + 1;
    }
    
    public function setGauge(string $name, float $value, array $labels = []): void
    {
        $key = $this->getKey($name, $labels);
        $this->gauges[$key] = $value;
    }
    
    public function observeHistogram(string $name, float $value, array $labels = []): void
    {
        $key = $this->getKey($name, $labels);
        if (!isset($this->histograms[$key])) {
            $this->histograms[$key] = [];
        }
        $this->histograms[$key][] = $value;
    }
    
    public function export(): string
    {
        $output = [];
        
        foreach ($this->counters as $key => $value) {
            $output[] = "# TYPE {$key} counter";
            $output[] = "{$key} {$value}";
        }
        
        foreach ($this->gauges as $key => $value) {
            $output[] = "# TYPE {$key} gauge";
            $output[] = "{$key} {$value}";
        }
        
        return implode("\n", $output);
    }
    
    private function getKey(string $name, array $labels): string
    {
        if (empty($labels)) {
            return $name;
        }
        $labelStr = implode(',', array_map(fn($k, $v) => "{$k}=\"{$v}\"", array_keys($labels), $labels));
        return "{$name}{{{$labelStr}}}";
    }
}
```

## OpenTelemetry 追踪

### 基础追踪

```php
<?php
declare(strict_types=1);

require __DIR__ . '/vendor/autoload.php';

use OpenTelemetry\SDK\Trace\TracerProvider;
use OpenTelemetry\SDK\Trace\SpanProcessor;
use OpenTelemetry\Exporter\Jaeger\Exporter;

$exporter = new Exporter('http://localhost:14268/api/traces');
$processor = new SpanProcessor($exporter);
$tracerProvider = new TracerProvider($processor);
$tracer = $tracerProvider->getTracer('my-app');

$span = $tracer->spanBuilder('operation')
    ->startSpan();

try {
    // 业务逻辑
    $span->setAttribute('user.id', 123);
} finally {
    $span->end();
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

class ObservabilityManager
{
    private Logger $logger;
    private PrometheusMetrics $metrics;
    private Tracer $tracer;
    
    public function __construct(Logger $logger, PrometheusMetrics $metrics, Tracer $tracer)
    {
        $this->logger = $logger;
        $this->metrics = $metrics;
        $this->tracer = $tracer;
    }
    
    public function handleRequest(Request $request): Response
    {
        $span = $this->tracer->startSpan('handle_request');
        $span->setAttribute('http.method', $request->getMethod());
        $span->setAttribute('http.path', $request->getPath());
        
        $startTime = microtime(true);
        
        try {
            $this->metrics->incrementCounter('requests_total', [
                'method' => $request->getMethod(),
            ]);
            
            $response = $this->processRequest($request);
            
            $duration = microtime(true) - $startTime;
            $this->metrics->observeHistogram('request_duration_seconds', $duration);
            
            $this->logger->info('Request processed', [
                'trace_id' => $span->getTraceId(),
                'method' => $request->getMethod(),
                'status' => $response->getStatusCode(),
                'duration' => $duration,
            ]);
            
            return $response;
        } catch (Exception $e) {
            $this->metrics->incrementCounter('requests_errors_total');
            $this->logger->error('Request failed', [
                'trace_id' => $span->getTraceId(),
                'error' => $e->getMessage(),
            ]);
            throw $e;
        } finally {
            $span->end();
        }
    }
}
```

## 最佳实践

1. **结构化日志**：使用 JSON 格式，包含上下文
2. **关键指标**：收集业务和性能指标
3. **分布式追踪**：追踪请求完整路径
4. **统一格式**：使用标准格式（Prometheus、OpenTelemetry）

## 注意事项

1. 日志不要记录敏感信息
2. 指标应该有意义且可操作
3. 追踪采样率要合理
4. 定期审查和清理数据

## 练习

1. 配置 Monolog，实现结构化日志记录。

2. 实现 Prometheus 指标收集和导出。

3. 集成 OpenTelemetry，实现分布式追踪。

4. 建立完整的可观测性体系，整合日志、指标和追踪。
