# 2.17.1 错误处理

## 概述

PHP 的错误处理机制包括错误级别、错误报告、错误处理器等。理解错误处理对于编写健壮的代码至关重要。

## 错误级别

### 错误类型

| 常量                  | 值   | 说明                     |
| :-------------------- | :--- | :----------------------- |
| `E_ERROR`             | 1    | 致命错误，脚本终止       |
| `E_WARNING`           | 2    | 警告，脚本继续执行       |
| `E_PARSE`             | 4    | 解析错误                 |
| `E_NOTICE`            | 8    | 通知，轻微问题           |
| `E_CORE_ERROR`        | 16   | PHP 核心致命错误          |
| `E_CORE_WARNING`       | 32   | PHP 核心警告             |
| `E_COMPILE_ERROR`     | 64   | 编译时致命错误           |
| `E_COMPILE_WARNING`    | 128  | 编译时警告               |
| `E_USER_ERROR`        | 256  | 用户触发的致命错误        |
| `E_USER_WARNING`       | 512  | 用户触发的警告           |
| `E_USER_NOTICE`        | 1024 | 用户触发的通知           |
| `E_RECOVERABLE_ERROR`  | 4096 | 可捕获的错误            |
| `E_ALL`                | 32767 | 所有错误                |

### 设置错误报告

```php
<?php
declare(strict_types=1);

// 报告所有错误
error_reporting(E_ALL);

// 开发环境：显示错误
ini_set('display_errors', '1');
ini_set('display_startup_errors', '1');

// 生产环境：记录错误，不显示
ini_set('display_errors', '0');
ini_set('log_errors', '1');
ini_set('error_log', __DIR__ . '/logs/php-errors.log');
```

## 错误处理器

### set_error_handler() - 自定义错误处理

**语法**：`set_error_handler(?callable $callback, int $error_levels = E_ALL): ?callable`

**参数**：
- `$callback`：错误处理回调函数，接收 `(int $severity, string $message, string $file, int $line)` 参数，返回 `bool`。如果为 `null`，则恢复默认错误处理
- `$error_levels`：可选，要处理的错误级别（位掩码），默认为 `E_ALL`

**返回值**：返回之前设置的错误处理函数，如果没有则返回 `null`。

```php
<?php
declare(strict_types=1);

set_error_handler(function (
    int $severity,
    string $message,
    string $file,
    int $line
): bool {
    // 记录错误
    error_log("Error [{$severity}]: {$message} in {$file} on line {$line}");
    
    // 返回 true 表示已处理，false 继续默认处理
    return true;
});

// 触发错误
trigger_error('Custom error', E_USER_WARNING);
```

### 将错误转换为异常

```php
<?php
declare(strict_types=1);

set_error_handler(function (
    int $severity,
    string $message,
    string $file,
    int $line
): void {
    throw new ErrorException($message, 0, $severity, $file, $line);
});

try {
    $result = 10 / 0;  // 会抛出异常
} catch (ErrorException $e) {
    echo "Caught error: " . $e->getMessage() . "\n";
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

class ErrorHandling
{
    public static function setupErrorHandling(): void
    {
        // 设置错误报告级别
        error_reporting(E_ALL);
        
        // 开发环境
        if (getenv('APP_ENV') === 'development') {
            ini_set('display_errors', '1');
        } else {
            ini_set('display_errors', '0');
            ini_set('log_errors', '1');
        }
        
        // 自定义错误处理器
        set_error_handler(function (
            int $severity,
            string $message,
            string $file,
            int $line
        ): bool {
            error_log("Error [{$severity}]: {$message} in {$file} on line {$line}");
            return true;
        });
    }
}

ErrorHandling::setupErrorHandling();
```

## 注意事项

1. **错误级别**：根据环境设置合适的错误报告级别。

2. **错误日志**：生产环境必须记录错误日志。

3. **错误显示**：生产环境不要显示错误给用户。

4. **错误处理器**：自定义错误处理器应返回 `true` 表示已处理。

5. **致命错误**：致命错误无法被错误处理器捕获，使用 `register_shutdown_function()`。

## 练习

1. 创建一个错误处理类，统一管理错误报告和日志。

2. 编写一个函数，将 PHP 错误转换为异常。

3. 实现一个函数，根据环境配置错误处理。

4. 创建一个函数，记录错误到文件。

5. 编写一个函数，处理致命错误。
