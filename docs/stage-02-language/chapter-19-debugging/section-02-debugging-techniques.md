# 2.19.2 调试技巧与工具链

## 概述

调试效率取决于是否具备“可观测性”。本节展示五类常用手段：错误报告、异常/错误处理、日志、性能分析、数据库调试，并给出常用工具链（Xdebug、Symfony VarDumper、Whoops）。

## 错误报告配置

```php
<?php
// 开发环境：全部输出	error_reporting(E_ALL);
ini_set('display_errors', '1');
ini_set('display_startup_errors', '1');

// 生产环境：隐藏并记录
error_reporting(E_ALL & ~E_DEPRECATED & ~E_STRICT);
ini_set('display_errors', '0');
ini_set('log_errors', '1');
ini_set('error_log', '/var/log/php/errors.log');
```

- 在 `bootstrap/app.php` 等入口统一设置，避免环境不一致。
- 给 CLI/Worker 单独配置日志文件，避免与 Web 日志混淆。

## 异常 & 错误处理

```php
<?php
declare(strict_types=1);

set_exception_handler(function (Throwable $e): void {
    error_log(sprintf(
        "Uncaught exception: %s in %s:%d\nStack trace:\n%s",
        $e->getMessage(),
        $e->getFile(),
        $e->getLine(),
        $e->getTraceAsString()
    ));

    if (getenv('APP_ENV') === 'development') {
        echo "<pre>{$e}</pre>";
    } else {
        http_response_code(500);
        echo json_encode(['error' => 'Internal server error']);
    }
});

set_error_handler(function (int $severity, string $message, string $file, int $line): bool {
    if (!(error_reporting() & $severity)) {
        return false; // 忽略被屏蔽的错误
    }

    throw new ErrorException($message, 0, $severity, $file, $line);
});
```

- 统一入口捕获所有错误，规避“白屏”问题。
- 记录 trace，方便后续复盘。

## 日志调试

```php
<?php
class DebugLogger
{
    public function __construct(private string $logFile = '/tmp/debug.log') {}

    public function log(string $message, array $context = []): void
    {
        $timestamp = date('Y-m-d H:i:s');
        $contextStr = $context === [] ? '' : ' ' . json_encode($context, JSON_UNESCAPED_UNICODE);
        file_put_contents($this->logFile, "[{$timestamp}] {$message}{$contextStr}\n", FILE_APPEND);
    }

    public function dump(mixed $variable, string $label = 'DEBUG'): void
    {
        $this->log("{$label}: " . print_r($variable, true));
    }
}

$logger = new DebugLogger();
$logger->log('User logged in', ['user_id' => 1, 'ip' => '192.168.1.1']);
$logger->dump($complexObject, 'User');
```

- 日志中追加 `request_id/trace_id`，便于跨服务串联。
- 使用 Monolog/Psr\Log 封装，方便切换输出通道。

## 性能 / 内存分析

```php
<?php
class PerformanceProfiler
{
    private array $timers = [];

    public function start(string $label): void
    {
        $this->timers[$label] = [
            'start' => microtime(true),
            'memory' => memory_get_usage(),
        ];
    }

    public function end(string $label): array
    {
        $timer = $this->timers[$label] ?? throw new InvalidArgumentException("Timer {$label} not started");

        return [
            'time' => microtime(true) - $timer['start'],
            'memory' => memory_get_usage() - $timer['memory'],
            'peak_memory' => memory_get_peak_usage(),
        ];
    }
}

$profiler = new PerformanceProfiler();
$profiler->start('database_query');
// ... 执行查询
$result = $profiler->end('database_query');
echo "Query took: {$result['time']} seconds\n";
```

- 配合 `hrtime()`、`memory_get_usage()` 观察性能瓶颈。
- 生产环境使用 Tideways/XHProf/Xdebug Profiler 获取火焰图。

## 数据库查询调试

```php
<?php
class QueryLogger
{
    private array $queries = [];

    public function log(string $sql, array $params = [], float $time = 0): void
    {
        $this->queries[] = [
            'sql' => $sql,
            'params' => $params,
            'time' => $time,
            'trace' => debug_backtrace(DEBUG_BACKTRACE_IGNORE_ARGS, 5),
        ];
    }

    public function report(): void
    {
        echo "Total queries: " . count($this->queries) . "\n";
        echo "Total time: " . array_sum(array_column($this->queries, 'time')) . " seconds\n";
    }
}
```

- 框架（Laravel/Symfony）通常提供查询监听器，可直接启用。
- 重点关注 N+1 查询、未命中索引、长事务等问题。

## 常用调试工具

### Xdebug

- `xdebug.break`：强制断点
- `xdebug_get_function_stack()`：获取堆栈
- 浏览器 IDE 配置好 `XDEBUG_SESSION`，即可单步调试。

### Symfony VarDumper

```php
<?php
use Symfony\Component\VarDumper\VarDumper;
VarDumper::dump($variable);
// 或使用全局函数
dump($variable);
```

- 格式化输出复杂结构，CLI/浏览器均可读。

### Whoops（优雅错误页）

```php
<?php
use Whoops\Run;
use Whoops\Handler\PrettyPageHandler;

$whoops = new Run();
$whoops->pushHandler(new PrettyPageHandler());
$whoops->register();
```

- 适合独立脚本或轻量服务，在开发环境显示高亮异常页面。

## 小结

- 把错误、日志、性能、数据库四大能力视为“调试基建”。
- 在脚手架或框架扩展包中预置上述配置，避免重复造轮子。
- 一旦线上出现问题，可快速基于这些“探针”复现与定位。