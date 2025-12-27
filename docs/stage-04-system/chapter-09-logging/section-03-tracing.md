# 4.9.3 链路追踪

## 概述

链路追踪是跟踪请求在系统中完整路径的技术，通过为每个请求分配唯一的追踪 ID，可以追踪请求在不同服务、组件之间的流转。在实际应用中，链路追踪用于理解系统行为、排查问题、性能分析等。

理解链路追踪的概念、追踪 ID 的生成和传递、请求追踪的实现等，对于构建可观测的分布式系统至关重要。

**主要内容**：
- 链路追踪概述（什么是链路追踪、为什么需要链路追踪）
- 追踪 ID 生成
- 追踪 ID 传递
- 请求追踪实现
- 分布式追踪
- 实际应用场景和最佳实践

## 特性

- **唯一标识**：每个请求有唯一的追踪 ID
- **完整路径**：可以追踪请求的完整路径
- **性能分析**：可以分析请求在各个组件的耗时
- **问题定位**：便于定位问题所在的组件

## 语法/定义

### 链路追踪要点

链路追踪没有单一的语法定义，而是通过追踪 ID 和相关机制实现。

## 基本用法

### 示例 1：生成追踪 ID

```php
<?php
declare(strict_types=1);

class TraceIdGenerator
{
    /**
     * 生成追踪 ID
     */
    public static function generate(): string
    {
        // 方法1：使用 uniqid()
        return uniqid('', true);
        
        // 方法2：使用 UUID v4
        // return sprintf(
        //     '%04x%04x-%04x-%04x-%04x-%04x%04x%04x',
        //     mt_rand(0, 0xffff), mt_rand(0, 0xffff),
        //     mt_rand(0, 0xffff),
        //     mt_rand(0, 0x0fff) | 0x4000,
        //     mt_rand(0, 0x3fff) | 0x8000,
        //     mt_rand(0, 0xffff), mt_rand(0, 0xffff), mt_rand(0, 0xffff)
        // );
        
        // 方法3：使用随机字符串
        // return bin2hex(random_bytes(16));
    }
    
    /**
     * 从请求中获取追踪 ID，如果不存在则生成新的
     */
    public static function getOrGenerate(): string
    {
        // 尝试从请求头获取
        if (isset($_SERVER['HTTP_X_TRACE_ID'])) {
            return $_SERVER['HTTP_X_TRACE_ID'];
        }
        
        // 尝试从环境变量获取
        if (getenv('TRACE_ID') !== false) {
            return getenv('TRACE_ID');
        }
        
        // 生成新的追踪 ID
        return self::generate();
    }
}

// 使用
$traceId = TraceIdGenerator::getOrGenerate();
echo "追踪 ID: {$traceId}\n";
```

**说明**：
- 生成唯一的追踪 ID
- 可以从请求中获取已有的追踪 ID，或生成新的

### 示例 2：请求追踪实现

```php
<?php
declare(strict_types=1);

class RequestTracer
{
    private string $traceId;
    private array $spans = [];
    
    public function __construct(?string $traceId = null)
    {
        $this->traceId = $traceId ?? TraceIdGenerator::generate();
    }
    
    public function getTraceId(): string
    {
        return $this->traceId;
    }
    
    /**
     * 开始一个操作（span）
     */
    public function startSpan(string $operation, array $tags = []): string
    {
        $spanId = TraceIdGenerator::generate();
        
        $this->spans[$spanId] = [
            'operation' => $operation,
            'start_time' => microtime(true),
            'tags' => $tags,
        ];
        
        return $spanId;
    }
    
    /**
     * 结束一个操作（span）
     */
    public function endSpan(string $spanId, array $tags = []): void
    {
        if (!isset($this->spans[$spanId])) {
            return;
        }
        
        $span = &$this->spans[$spanId];
        $span['end_time'] = microtime(true);
        $span['duration'] = $span['end_time'] - $span['start_time'];
        $span['tags'] = array_merge($span['tags'], $tags);
    }
    
    /**
     * 记录日志，包含追踪 ID
     */
    public function log(string $level, string $message, array $context = []): void
    {
        $logEntry = [
            'timestamp' => date('c'),
            'trace_id' => $this->traceId,
            'level' => $level,
            'message' => $message,
            'context' => $context,
        ];
        
        $logLine = json_encode($logEntry, JSON_UNESCAPED_UNICODE) . "\n";
        file_put_contents('/tmp/trace.log', $logLine, FILE_APPEND | LOCK_EX);
    }
    
    /**
     * 获取追踪信息
     */
    public function getTraceInfo(): array
    {
        return [
            'trace_id' => $this->traceId,
            'spans' => $this->spans,
        ];
    }
}

// 使用
$tracer = new RequestTracer();

// 开始追踪
$tracer->log('INFO', '请求开始');

// 操作1
$span1 = $tracer->startSpan('database_query', ['table' => 'users']);
// 执行数据库查询
usleep(50000);  // 模拟查询
$tracer->endSpan($span1, ['rows' => 10]);

// 操作2
$span2 = $tracer->startSpan('cache_set', ['key' => 'user:123']);
// 设置缓存
usleep(10000);  // 模拟操作
$tracer->endSpan($span2);

$tracer->log('INFO', '请求完成');

// 获取追踪信息
$traceInfo = $tracer->getTraceInfo();
print_r($traceInfo);
```

**说明**：
- 实现请求追踪功能
- 记录操作（span）的开始和结束时间
- 日志中包含追踪 ID

### 示例 3：追踪 ID 传递

```php
<?php
declare(strict_types=1);

class TraceContext
{
    private static ?string $traceId = null;
    
    /**
     * 设置追踪 ID
     */
    public static function setTraceId(string $traceId): void
    {
        self::$traceId = $traceId;
    }
    
    /**
     * 获取追踪 ID
     */
    public static function getTraceId(): string
    {
        if (self::$traceId === null) {
            self::$traceId = TraceIdGenerator::getOrGenerate();
        }
        return self::$traceId;
    }
    
    /**
     * 将追踪 ID 添加到 HTTP 头
     */
    public static function addToHeaders(array &$headers): void
    {
        $headers['X-Trace-Id'] = self::getTraceId();
    }
    
    /**
     * 从 HTTP 请求中获取追踪 ID
     */
    public static function fromRequest(): ?string
    {
        // 从请求头获取
        if (isset($_SERVER['HTTP_X_TRACE_ID'])) {
            return $_SERVER['HTTP_X_TRACE_ID'];
        }
        
        // 从 GET 参数获取
        if (isset($_GET['trace_id'])) {
            return $_GET['trace_id'];
        }
        
        return null;
    }
    
    /**
     * 初始化追踪上下文
     */
    public static function init(): void
    {
        $traceId = self::fromRequest();
        if ($traceId === null) {
            $traceId = TraceIdGenerator::generate();
        }
        self::setTraceId($traceId);
    }
}

// 使用
// 在请求开始时初始化
TraceContext::init();

// 在代码中获取追踪 ID
$traceId = TraceContext::getTraceId();

// 在 HTTP 请求中传递追踪 ID
$headers = [];
TraceContext::addToHeaders($headers);
// 使用 $headers 发送 HTTP 请求
```

**说明**：
- 实现追踪上下文的传递
- 可以从请求中获取，也可以在代码中设置
- 可以在 HTTP 请求中传递追踪 ID

### 示例 4：完整的追踪系统

```php
<?php
declare(strict_types=1);

class TracingSystem
{
    private string $traceId;
    private array $spans = [];
    private array $logs = [];
    
    public function __construct()
    {
        $this->traceId = TraceIdGenerator::getOrGenerate();
    }
    
    public function getTraceId(): string
    {
        return $this->traceId;
    }
    
    public function trace(string $operation, callable $callback): mixed
    {
        $spanId = TraceIdGenerator::generate();
        $startTime = microtime(true);
        
        $this->spans[$spanId] = [
            'operation' => $operation,
            'start_time' => $startTime,
        ];
        
        $this->log('INFO', "开始操作: {$operation}", ['span_id' => $spanId]);
        
        try {
            $result = $callback();
            
            $endTime = microtime(true);
            $this->spans[$spanId]['end_time'] = $endTime;
            $this->spans[$spanId]['duration'] = $endTime - $startTime;
            $this->spans[$spanId]['status'] = 'success';
            
            $this->log('INFO', "完成操作: {$operation}", [
                'span_id' => $spanId,
                'duration' => $this->spans[$spanId]['duration'],
            ]);
            
            return $result;
        } catch (Exception $e) {
            $endTime = microtime(true);
            $this->spans[$spanId]['end_time'] = $endTime;
            $this->spans[$spanId]['duration'] = $endTime - $startTime;
            $this->spans[$spanId]['status'] = 'error';
            $this->spans[$spanId]['error'] = $e->getMessage();
            
            $this->log('ERROR', "操作失败: {$operation}", [
                'span_id' => $spanId,
                'error' => $e->getMessage(),
            ]);
            
            throw $e;
        }
    }
    
    public function log(string $level, string $message, array $context = []): void
    {
        $logEntry = [
            'timestamp' => microtime(true),
            'trace_id' => $this->traceId,
            'level' => $level,
            'message' => $message,
            'context' => $context,
        ];
        
        $this->logs[] = $logEntry;
        
        // 也可以写入日志文件
        $logLine = json_encode($logEntry, JSON_UNESCAPED_UNICODE) . "\n";
        file_put_contents('/tmp/trace.log', $logLine, FILE_APPEND | LOCK_EX);
    }
    
    public function getTraceReport(): array
    {
        return [
            'trace_id' => $this->traceId,
            'spans' => $this->spans,
            'logs' => $this->logs,
            'total_duration' => $this->calculateTotalDuration(),
        ];
    }
    
    private function calculateTotalDuration(): float
    {
        if (empty($this->spans)) {
            return 0;
        }
        
        $startTime = min(array_column($this->spans, 'start_time'));
        $endTime = max(array_column(array_filter($this->spans, fn($s) => isset($s['end_time'])), 'end_time'));
        
        return $endTime - $startTime;
    }
}

// 使用
$tracing = new TracingSystem();

$result = $tracing->trace('process_user', function () use ($tracing) {
    $tracing->log('INFO', '开始处理用户');
    
    // 操作1
    $tracing->trace('validate_user', function () {
        usleep(10000);
        return true;
    });
    
    // 操作2
    $tracing->trace('save_user', function () {
        usleep(50000);
        return true;
    });
    
    $tracing->log('INFO', '用户处理完成');
    return true;
});

// 获取追踪报告
$report = $tracing->getTraceReport();
print_r($report);
```

**说明**：
- 完整的追踪系统实现
- 支持嵌套操作追踪
- 生成追踪报告

## 使用场景

### 场景 1：请求追踪

追踪 HTTP 请求的处理过程。

**示例**：

```php
<?php
declare(strict_types=1);

// 在请求开始时初始化
TraceContext::init();
$traceId = TraceContext::getTraceId();

// 在日志中包含追踪 ID
$logger->info('请求开始', ['trace_id' => $traceId]);
// 处理请求...
$logger->info('请求完成', ['trace_id' => $traceId]);
```

### 场景 2：性能分析

通过追踪分析请求在各个组件的耗时。

**示例**：见"示例 4：完整的追踪系统"

## 注意事项

### 追踪 ID 的唯一性

确保追踪 ID 在系统中是唯一的。

**示例**：

```php
<?php
declare(strict_types=1);

// ✅ 使用足够长的随机字符串
$traceId = bin2hex(random_bytes(16));  // 32 字符

// ⚠️ 避免使用简单的 ID
// $traceId = uniqid();  // 可能不够唯一
```

### 追踪 ID 的传递

在分布式系统中，需要在服务间传递追踪 ID。

**示例**：见"示例 3：追踪 ID 传递"

### 性能影响

追踪操作可能影响性能，应该合理使用。

**示例**：

```php
<?php
declare(strict_types=1);

// 只在需要时启用追踪
if (getenv('ENABLE_TRACING') === '1') {
    $tracing->trace('operation', $callback);
} else {
    $callback();
}
```

## 常见问题

### 问题 1：如何生成追踪 ID？

**回答**：使用随机字符串、UUID、uniqid() 等方法生成唯一的 ID。

**示例**：见"示例 1：生成追踪 ID"

### 问题 2：如何在服务间传递追踪 ID？

**回答**：通过 HTTP 头、环境变量、上下文等方式传递。

**示例**：见"示例 3：追踪 ID 传递"

### 问题 3：链路追踪的性能影响？

**回答**：追踪操作有性能开销，应该合理使用，或使用异步记录。

**示例**：见"性能影响"部分

## 最佳实践

### 1. 使用唯一的追踪 ID

确保追踪 ID 在系统中是唯一的。

**示例**：

```php
<?php
declare(strict_types=1);

$traceId = bin2hex(random_bytes(16));  // 32 字符，足够唯一
```

### 2. 在日志中包含追踪 ID

在日志中包含追踪 ID，便于关联日志。

**示例**：

```php
<?php
declare(strict_types=1);

$logger->info('操作', ['trace_id' => $traceId, 'operation' => 'create']);
```

### 3. 传递追踪 ID

在服务调用时传递追踪 ID。

**示例**：见"示例 3：追踪 ID 传递"

### 4. 记录操作时间

记录操作的开始和结束时间，便于性能分析。

**示例**：见"示例 4：完整的追踪系统"

## 对比分析

### 链路追踪 vs 日志

| 特性         | 链路追踪                   | 日志                       |
|:-------------|:---------------------------|:---------------------------|
| **关注点**   | 请求的完整路径             | 单个事件                   |
| **追踪 ID**  | ✅ 使用追踪 ID 关联        | ⚠️ 可能有追踪 ID           |
| **时间信息** | ✅ 记录操作时间            | ⚠️ 只有时间戳              |
| **性能分析** | ✅ 便于性能分析            | ⚠️ 需要额外处理            |

## 练习任务

1. **追踪系统实现**：实现一个完整的链路追踪系统。

2. **追踪 ID 生成工具**：创建一个工具，生成和管理追踪 ID。

3. **追踪上下文管理工具**：实现一个工具，管理追踪上下文。

4. **追踪报告生成工具**：编写一个工具，生成追踪报告。

5. **链路追踪最佳实践指南**：创建一个指南，总结链路追踪的最佳实践。

## 相关章节

- **[4.9.1 日志体系设计](section-01-logging-system.md)**：了解日志体系设计的相关内容
- **[4.9.2 结构化日志](section-02-structured-logs.md)**：了解结构化日志的相关内容
- **[4.9.4 日志分析与监控](section-04-log-analysis.md)**：了解日志分析的相关内容
