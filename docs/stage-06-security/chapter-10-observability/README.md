# 6.9 可观测性体系

## 目标

- 理解可观测性的三大支柱：日志、指标、链路追踪。
- 了解 OpenTelemetry 的使用。
- 熟悉 Prometheus 指标暴露和 Grafana 可视化。
- 掌握 Sentry 错误监控的配置。

## 三大支柱

### 日志（Logs）

- 记录事件和错误信息。
- 使用结构化日志（JSON 格式）。

### 指标（Metrics）

- 量化系统性能（请求数、响应时间、错误率）。
- 使用 Prometheus 收集和存储。

### 链路追踪（Tracing）

- 跟踪请求在系统中的完整路径。
- 使用 OpenTelemetry 实现。

## OpenTelemetry

### 基础配置

```php
<?php
require __DIR__ . '/vendor/autoload.php';

use OpenTelemetry\SDK\Trace\TracerProvider;
use OpenTelemetry\SDK\Trace\SpanExporter\ConsoleSpanExporter;
use OpenTelemetry\SDK\Trace\SpanProcessor\BatchSpanProcessor;

$exporter = new ConsoleSpanExporter();
$processor = new BatchSpanProcessor($exporter);
$tracerProvider = new TracerProvider($processor);
$tracer = $tracerProvider->getTracer('my-app');
```

### 创建 Span

```php
<?php
$span = $tracer->spanBuilder('process-order')
    ->startSpan();

$scope = $span->activate();
try {
    // 业务逻辑
    $span->setAttribute('order.id', $orderId);
    $span->setAttribute('order.amount', $amount);
    
    processOrder($orderId);
} finally {
    $span->end();
    $scope->detach();
}
```

## Prometheus

### 暴露指标

```php
<?php
declare(strict_types=1);

class Metrics
{
    private array $counters = [];
    private array $gauges = [];
    
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
        return "{$name}{{$labelStr}}";
    }
}

// 使用
$metrics = new Metrics();
$metrics->incrementCounter('http_requests_total', ['method' => 'GET', 'status' => '200']);
$metrics->setGauge('memory_usage_bytes', memory_get_usage());

// 暴露指标端点
if ($_SERVER['REQUEST_URI'] === '/metrics') {
    header('Content-Type: text/plain');
    echo $metrics->export();
}
```

## Sentry

### 配置

```php
<?php
\Sentry\init([
    'dsn' => 'https://your-dsn@sentry.io/project-id',
    'environment' => 'production',
    'traces_sample_rate' => 1.0,
]);
```

### 使用

```php
<?php
try {
    // 业务代码
} catch (Exception $e) {
    \Sentry\captureException($e);
    throw $e;
}

// 添加上下文
\Sentry\configureScope(function (\Sentry\State\Scope $scope): void {
    $scope->setUser(['id' => $userId, 'email' => $email]);
    $scope->setTag('environment', 'production');
    $scope->setContext('order', ['id' => $orderId, 'amount' => $amount]);
});
```

## 实际应用示例

### 完整的可观测性集成

```php
<?php
declare(strict_types=1);

namespace App\Services;

use Monolog\Logger;
use OpenTelemetry\API\Trace\TracerInterface;
use Prometheus\CollectorRegistry;

/**
 * 可观测性服务
 * 
 * 整合日志、指标、链路追踪
 */
class ObservabilityService
{
    private Logger $logger;
    private TracerInterface $tracer;
    private CollectorRegistry $metrics;
    
    public function __construct(
        Logger $logger,
        TracerInterface $tracer,
        CollectorRegistry $metrics
    ) {
        $this->logger = $logger;
        $this->tracer = $tracer;
        $this->metrics = $metrics;
    }
    
    /**
     * 记录请求处理
     * 
     * 同时记录日志、指标和链路追踪
     */
    public function recordRequest(string $method, string $path, callable $handler): mixed
    {
        $startTime = microtime(true);
        $traceId = bin2hex(random_bytes(16));
        
        // 创建 Span（链路追踪）
        $span = $this->tracer->spanBuilder("{$method} {$path}")
            ->setAttribute('http.method', $method)
            ->setAttribute('http.path', $path)
            ->startSpan();
        
        $scope = $span->activate();
        
        try {
            // 记录请求开始日志
            $this->logger->info('Request started', [
                'trace_id' => $traceId,
                'method' => $method,
                'path' => $path,
            ]);
            
            // 增加请求计数（指标）
            $this->metrics->getOrRegisterCounter(
                'app',
                'http_requests_total',
                'Total HTTP requests',
                ['method', 'path']
            )->inc([$method, $path]);
            
            // 执行处理函数
            $result = $handler();
            
            // 记录成功日志
            $this->logger->info('Request completed', [
                'trace_id' => $traceId,
                'method' => $method,
                'path' => $path,
                'status' => 200,
            ]);
            
            // 设置 Span 属性
            $span->setAttribute('http.status_code', 200);
            $span->setStatus(\OpenTelemetry\API\Trace\StatusCode::STATUS_OK);
            
            return $result;
            
        } catch (\Exception $e) {
            // 记录错误日志
            $this->logger->error('Request failed', [
                'trace_id' => $traceId,
                'method' => $method,
                'path' => $path,
                'error' => $e->getMessage(),
                'trace' => $e->getTraceAsString(),
            ]);
            
            // 增加错误计数（指标）
            $this->metrics->getOrRegisterCounter(
                'app',
                'http_errors_total',
                'Total HTTP errors',
                ['method', 'path', 'status']
            )->inc([$method, $path, '500']);
            
            // 设置 Span 错误状态
            $span->setStatus(\OpenTelemetry\API\Trace\StatusCode::STATUS_ERROR, $e->getMessage());
            $span->recordException($e);
            
            throw $e;
            
        } finally {
            // 记录响应时间（指标）
            $duration = microtime(true) - $startTime;
            $this->metrics->getOrRegisterHistogram(
                'app',
                'http_request_duration_seconds',
                'HTTP request duration',
                ['method', 'path'],
                [0.1, 0.5, 1.0, 2.0, 5.0]  // 直方图桶
            )->observe($duration, [$method, $path]);
            
            // 结束 Span
            $span->end();
            $scope->detach();
        }
    }
}
```

### 中间件集成

```php
<?php
declare(strict_types=1);

namespace App\Middleware;

use App\Services\ObservabilityService;

/**
 * 可观测性中间件
 * 
 * 自动记录所有请求的日志、指标和追踪
 */
class ObservabilityMiddleware
{
    private ObservabilityService $observability;
    
    public function __construct(ObservabilityService $observability)
    {
        $this->observability = $observability;
    }
    
    public function handle(array $request, callable $next): void
    {
        $method = $request['REQUEST_METHOD'] ?? 'GET';
        $path = parse_url($request['REQUEST_URI'] ?? '/', PHP_URL_PATH);
        
        $this->observability->recordRequest($method, $path, function () use ($next, $request) {
            return $next($request);
        });
    }
}
```

### 业务指标监控

```php
<?php
declare(strict_types=1);

namespace App\Services;

use Prometheus\CollectorRegistry;

/**
 * 业务指标监控
 * 
 * 监控业务关键指标（订单数、支付金额等）
 */
class BusinessMetrics
{
    private CollectorRegistry $registry;
    
    public function __construct(CollectorRegistry $registry)
    {
        $this->registry = $registry;
    }
    
    /**
     * 记录订单创建
     */
    public function recordOrderCreated(float $amount, string $paymentMethod): void
    {
        // 增加订单计数
        $this->registry->getOrRegisterCounter(
            'business',
            'orders_created_total',
            'Total orders created',
            ['payment_method']
        )->inc([$paymentMethod]);
        
        // 记录订单金额
        $this->registry->getOrRegisterCounter(
            'business',
            'orders_amount_total',
            'Total order amount',
            ['payment_method']
        )->incBy($amount, [$paymentMethod]);
    }
    
    /**
     * 记录用户注册
     */
    public function recordUserRegistered(string $source): void
    {
        $this->registry->getOrRegisterCounter(
            'business',
            'users_registered_total',
            'Total users registered',
            ['source']
        )->inc([$source]);
    }
    
    /**
     * 记录支付成功
     */
    public function recordPaymentSuccess(float $amount, string $method): void
    {
        $this->registry->getOrRegisterCounter(
            'business',
            'payments_success_total',
            'Total successful payments',
            ['method']
        )->inc([$method]);
        
        $this->registry->getOrRegisterCounter(
            'business',
            'payments_amount_total',
            'Total payment amount',
            ['method']
        )->incBy($amount, [$method]);
    }
}
```

## 练习

1. 配置 OpenTelemetry，实现请求链路追踪。

2. 创建 Prometheus 指标收集系统，暴露应用指标。

3. 集成 Sentry，实现错误监控和告警。

4. 设计一个可观测性仪表板，整合日志、指标和追踪。

5. 实现自定义指标收集，监控业务关键指标。

6. 创建一个性能分析工具，结合日志、指标和追踪数据。

7. **实现完整的可观测性集成**：整合日志、指标、链路追踪到统一服务。

8. **实现业务指标监控**：监控订单、支付、用户注册等业务关键指标。

9. **创建可观测性中间件**：自动记录所有请求的可观测性数据。
