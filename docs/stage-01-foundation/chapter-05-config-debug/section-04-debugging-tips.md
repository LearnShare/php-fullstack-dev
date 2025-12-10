# 1.4.4 调试技巧与实践

## 概述

除了 Xdebug，PHP 还提供了许多内置的调试函数和技巧。本章介绍常用的调试方法和最佳实践。

## 基础调试函数

### var_dump() - 详细输出

**语法**：`var_dump(mixed ...$values): void`

**特点**：
- 显示变量的类型和值
- 递归显示数组和对象
- 输出格式较详细

**示例**：

```php
<?php
declare(strict_types=1);

$data = [
    'name' => 'Alice',
    'age' => 25,
    'hobbies' => ['reading', 'coding']
];

var_dump($data);
```

**输出**：
```
array(3) {
  ["name"]=>
  string(5) "Alice"
  ["age"]=>
  int(25)
  ["hobbies"]=>
  array(2) {
    [0]=>
    string(7) "reading"
    [1]=>
    string(6) "coding"
  }
}
```

### print_r() - 可读输出

**语法**：`print_r(mixed $value, bool $return = false): string|bool`

**特点**：
- 输出格式更易读
- 可以返回字符串而不是直接输出

**示例**：

```php
<?php
declare(strict_types=1);

$data = ['name' => 'Alice', 'age' => 25];

// 直接输出
print_r($data);

// 返回字符串
$output = print_r($data, true);
echo $output;

// 格式化输出
echo '<pre>';
print_r($data);
echo '</pre>';
```

### var_export() - 可执行输出

**语法**：`var_export(mixed $value, bool $return = false): ?string`

**特点**：
- 输出格式是有效的 PHP 代码
- 可以用于生成配置文件

**示例**：

```php
<?php
declare(strict_types=1);

$data = ['name' => 'Alice', 'age' => 25];

$code = var_export($data, true);
echo $code;
// 输出：array ( 'name' => 'Alice', 'age' => 25, )
```

## 错误日志

### error_log() - 写入日志

**语法**：`error_log(string $message, int $message_type = 0, ?string $destination = null, ?string $additional_headers = null): bool`

**参数**：
- `$message`：日志消息
- `$message_type`：日志类型（0=系统日志，1=邮件，3=文件）
- `$destination`：目标文件（类型为 3 时）

**示例**：

```php
<?php
declare(strict_types=1);

// 写入系统日志
error_log('Debug: User logged in');

// 写入文件
error_log('Debug: ' . print_r($data, true), 3, '/var/log/php/debug.log');

// 带上下文
error_log(sprintf(
    'User %d accessed %s at %s',
    $userId,
    $url,
    date('Y-m-d H:i:s')
), 3, '/var/log/php/access.log');
```

### 日志级别

```php
<?php
declare(strict_types=1);

// 不同级别的日志
error_log('DEBUG: ' . $message, 3, '/var/log/php/debug.log');
error_log('INFO: ' . $message, 3, '/var/log/php/info.log');
error_log('WARNING: ' . $message, 3, '/var/log/php/warning.log');
error_log('ERROR: ' . $message, 3, '/var/log/php/error.log');
```

## 堆栈跟踪

### debug_backtrace() - 获取调用栈

**语法**：`debug_backtrace(int $options = DEBUG_BACKTRACE_PROVIDE_OBJECT, int $limit = 0): array`

**参数**：
- `$options`：选项（`DEBUG_BACKTRACE_IGNORE_ARGS` 忽略参数）
- `$limit`：限制返回的帧数

**示例**：

```php
<?php
declare(strict_types=1);

function debugTrace(): void
{
    $trace = debug_backtrace(DEBUG_BACKTRACE_IGNORE_ARGS, 5);
    
    foreach ($trace as $frame) {
        echo sprintf(
            "%s:%d in %s()\n",
            $frame['file'] ?? 'unknown',
            $frame['line'] ?? 0,
            $frame['function'] ?? 'unknown'
        );
    }
}

function a(): void
{
    b();
}

function b(): void
{
    debugTrace();
}

a();
```

### debug_print_backtrace() - 打印调用栈

**语法**：`debug_print_backtrace(int $options = 0, int $limit = 0): void`

**示例**：

```php
<?php
declare(strict_types=1);

function a(): void
{
    b();
}

function b(): void
{
    debug_print_backtrace();
}

a();
```

## Xdebug 函数

### xdebug_break() - 强制断点

**语法**：`xdebug_break(): void`

**说明**：在代码中强制设置断点，即使 IDE 未设置断点。

**示例**：

```php
<?php
declare(strict_types=1);

function processData(array $data): void
{
    // 强制断点
    if (count($data) > 100) {
        xdebug_break(); // 会在这里停止
    }
    
    // 处理数据
}
```

### xdebug_info() - 诊断信息

**语法**：`xdebug_info(?string $category = null): array|string`

**示例**：

```php
<?php
declare(strict_types=1);

// 查看 Xdebug 信息
var_dump(xdebug_info());

// 查看特定类别
var_dump(xdebug_info('mode'));
```

### xdebug_get_function_stack() - 获取调用栈

**语法**：`xdebug_get_function_stack(): array`

**示例**：

```php
<?php
declare(strict_types=1);

function debugStack(): void
{
    $stack = xdebug_get_function_stack();
    print_r($stack);
}
```

## 调试工具类

### 创建调试工具类

```php
<?php
declare(strict_types=1);

class DebugHelper
{
    public static function dump(mixed $value, bool $die = false): void
    {
        echo '<pre>';
        var_dump($value);
        echo '</pre>';
        
        if ($die) {
            die();
        }
    }
    
    public static function log(mixed $value, string $label = ''): void
    {
        $message = $label ? "[{$label}] " : '';
        $message .= print_r($value, true);
        error_log($message, 3, __DIR__ . '/debug.log');
    }
    
    public static function trace(): void
    {
        $trace = debug_backtrace(DEBUG_BACKTRACE_IGNORE_ARGS);
        echo '<pre>';
        foreach ($trace as $frame) {
            echo sprintf(
                "%s:%d in %s()\n",
                $frame['file'] ?? 'unknown',
                $frame['line'] ?? 0,
                $frame['function'] ?? 'unknown'
            );
        }
        echo '</pre>';
    }
    
    public static function time(string $label = ''): void
    {
        static $times = [];
        
        $now = microtime(true);
        
        if (isset($times[$label])) {
            $elapsed = $now - $times[$label];
            echo "{$label}: {$elapsed} seconds\n";
            unset($times[$label]);
        } else {
            $times[$label] = $now;
        }
    }
}

// 使用
DebugHelper::dump($data);
DebugHelper::log($data, 'User Data');
DebugHelper::trace();
DebugHelper::time('process');
// ... 代码 ...
DebugHelper::time('process'); // 输出耗时
```

## 性能调试

### 测量执行时间

```php
<?php
declare(strict_types=1);

$start = microtime(true);

// 执行代码
for ($i = 0; $i < 1000000; $i++) {
    // ...
}

$end = microtime(true);
$elapsed = $end - $start;

echo "Execution time: {$elapsed} seconds\n";
```

### 测量内存使用

```php
<?php
declare(strict_types=1);

$start = memory_get_usage();

// 执行代码
$data = range(1, 1000000);

$end = memory_get_usage();
$peak = memory_get_peak_usage();

echo "Memory used: " . ($end - $start) . " bytes\n";
echo "Peak memory: {$peak} bytes\n";
```

### 高精度时间

```php
<?php
declare(strict_types=1);

// 纳秒级精度
$start = hrtime(true);

// 执行代码
usleep(1000);

$end = hrtime(true);
$elapsed = ($end - $start) / 1e9; // 转换为秒

echo "Execution time: {$elapsed} seconds\n";
```

## 配置差异追踪

### 对比配置

```php
<?php
declare(strict_types=1);

function compareConfigs(): void
{
    // CLI 配置
    $cliConfig = [
        'memory_limit' => ini_get('memory_limit'),
        'max_execution_time' => ini_get('max_execution_time'),
    ];
    
    // 从 phpinfo() 获取 FPM 配置（需要实际请求）
    // 或通过其他方式获取
    
    echo "CLI Config:\n";
    print_r($cliConfig);
}
```

### 检查配置来源

```bash
# 查看配置来源
php -i | grep "Loaded Configuration File"

# 查看扫描的目录
php -i | grep "Scan for additional .ini files"
```

## 完整示例

```php
<?php
declare(strict_types=1);

class DebuggingTools
{
    public static function demonstrate(): void
    {
        $data = ['name' => 'Alice', 'age' => 25];
        
        echo "=== var_dump ===\n";
        var_dump($data);
        
        echo "\n=== print_r ===\n";
        print_r($data);
        
        echo "\n=== error_log ===\n";
        error_log('Debug: ' . print_r($data, true), 3, '/tmp/debug.log');
        
        echo "\n=== debug_backtrace ===\n";
        self::showTrace();
        
        echo "\n=== Performance ===\n";
        self::measurePerformance();
    }
    
    private static function showTrace(): void
    {
        $trace = debug_backtrace(DEBUG_BACKTRACE_IGNORE_ARGS, 3);
        foreach ($trace as $frame) {
            echo sprintf(
                "%s:%d in %s()\n",
                $frame['file'] ?? 'unknown',
                $frame['line'] ?? 0,
                $frame['function'] ?? 'unknown'
            );
        }
    }
    
    private static function measurePerformance(): void
    {
        $start = microtime(true);
        $memStart = memory_get_usage();
        
        // 模拟操作
        $array = range(1, 10000);
        
        $end = microtime(true);
        $memEnd = memory_get_usage();
        
        echo "Time: " . ($end - $start) . " seconds\n";
        echo "Memory: " . ($memEnd - $memStart) . " bytes\n";
    }
}

DebuggingTools::demonstrate();
```

## 注意事项

1. **生产环境**：生产环境不要使用 `var_dump()`、`print_r()` 等调试函数。

2. **日志安全**：确保日志文件权限正确，避免泄露敏感信息。

3. **性能影响**：调试代码会影响性能，记得在生产环境移除。

4. **错误处理**：使用 `try-catch` 捕获异常，记录详细的错误信息。

5. **调试工具**：使用专业的调试工具（Xdebug）而不是 `var_dump()`。

## 练习

1. 创建一个调试工具类，封装常用的调试函数。

2. 使用 `error_log()` 记录应用程序的关键操作。

3. 使用 `debug_backtrace()` 实现错误追踪功能。

4. 测量代码的执行时间和内存使用。

5. 实现一个性能分析工具，找出代码瓶颈。
