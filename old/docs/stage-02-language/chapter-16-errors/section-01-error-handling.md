# 2.16.1 错误处理

## 概述

PHP 的错误处理机制包括错误级别、错误报告、错误处理器等。理解错误处理对于编写健壮的代码至关重要。

## 错误级别

### 错误类型详解

PHP 中的错误分为多个级别，每个级别代表不同的严重程度和处理方式：

| 常量                  | 值   | 说明                     | 是否可捕获 | 脚本是否继续 |
| :-------------------- | :--- | :----------------------- | :--------- | :----------- |
| `E_ERROR`             | 1    | 致命错误，脚本终止       | ❌ 否      | ❌ 终止      |
| `E_WARNING`           | 2    | 警告，脚本继续执行       | ✅ 是      | ✅ 继续      |
| `E_PARSE`             | 4    | 解析错误                 | ❌ 否      | ❌ 终止      |
| `E_NOTICE`            | 8    | 通知，轻微问题           | ✅ 是      | ✅ 继续      |
| `E_CORE_ERROR`        | 16   | PHP 核心致命错误          | ❌ 否      | ❌ 终止      |
| `E_CORE_WARNING`       | 32   | PHP 核心警告             | ✅ 是      | ✅ 继续      |
| `E_COMPILE_ERROR`     | 64   | 编译时致命错误           | ❌ 否      | ❌ 终止      |
| `E_COMPILE_WARNING`    | 128  | 编译时警告               | ✅ 是      | ✅ 继续      |
| `E_USER_ERROR`        | 256  | 用户触发的致命错误        | ✅ 是      | ❌ 终止      |
| `E_USER_WARNING`       | 512  | 用户触发的警告           | ✅ 是      | ✅ 继续      |
| `E_USER_NOTICE`        | 1024 | 用户触发的通知           | ✅ 是      | ✅ 继续      |
| `E_RECOVERABLE_ERROR`  | 4096 | 可捕获的错误            | ✅ 是      | ✅ 继续      |
| `E_DEPRECATED`         | 8192 | 已弃用的功能警告         | ✅ 是      | ✅ 继续      |
| `E_USER_DEPRECATED`    | 16384 | 用户触发的弃用警告       | ✅ 是      | ✅ 继续      |
| `E_ALL`                | 32767 | 所有错误（PHP 7.4+）    | -          | -            |

### 错误级别分类

**致命错误（Fatal Errors）**：
- `E_ERROR`：运行时致命错误，如调用未定义的函数、内存不足等
- `E_PARSE`：语法解析错误，如缺少分号、括号不匹配等
- `E_CORE_ERROR`：PHP 核心致命错误
- `E_COMPILE_ERROR`：编译时致命错误
- `E_USER_ERROR`：用户通过 `trigger_error()` 触发的致命错误

**可恢复错误（Recoverable Errors）**：
- `E_WARNING`：运行时警告，如文件不存在、函数参数错误等
- `E_NOTICE`：运行时通知，如使用未定义变量、数组键不存在等
- `E_RECOVERABLE_ERROR`：可捕获的错误，如类型错误（PHP 7+）
- `E_DEPRECATED`：已弃用的功能警告
- `E_USER_WARNING`、`E_USER_NOTICE`、`E_USER_DEPRECATED`：用户触发的警告和通知

### 错误级别的使用场景

**开发环境**：
- 报告所有错误：`error_reporting(E_ALL)`
- 显示所有错误：`ini_set('display_errors', '1')`
- 记录所有错误：`ini_set('log_errors', '1')`

**生产环境**：
- 报告致命错误和警告：`error_reporting(E_ERROR | E_WARNING | E_USER_ERROR | E_USER_WARNING)`
- 不显示错误：`ini_set('display_errors', '0')`
- 记录所有错误：`ini_set('log_errors', '1')`

### 设置错误报告

**语法**：`error_reporting(?int $level = null): int`

**参数**：
- `$level`：可选，错误报告级别（位掩码），如果为 `null`，返回当前错误报告级别

**返回值**：返回之前的错误报告级别。

**常用错误报告级别组合**：

```php
<?php
declare(strict_types=1);

// 报告所有错误（开发环境）
error_reporting(E_ALL);

// 报告所有错误，包括弃用警告（PHP 7.4+）
error_reporting(E_ALL | E_STRICT);

// 报告致命错误和警告（生产环境）
error_reporting(E_ERROR | E_WARNING | E_USER_ERROR | E_USER_WARNING);

// 禁用错误报告（不推荐）
error_reporting(0);

// 获取当前错误报告级别
$currentLevel = error_reporting();
```

### 错误显示配置

**开发环境配置**：

```php
<?php
declare(strict_types=1);

// 显示所有错误
ini_set('display_errors', '1');
ini_set('display_startup_errors', '1');

// 错误格式（HTML 或 plain text）
ini_set('html_errors', '1');  // HTML 格式（默认）
ini_set('html_errors', '0');  // 纯文本格式

// 错误报告级别
error_reporting(E_ALL);
```

**生产环境配置**：

```php
<?php
declare(strict_types=1);

// 不显示错误给用户
ini_set('display_errors', '0');
ini_set('display_startup_errors', '0');

// 记录错误到日志
ini_set('log_errors', '1');
ini_set('error_log', __DIR__ . '/logs/php-errors.log');

// 只报告严重错误
error_reporting(E_ERROR | E_WARNING | E_USER_ERROR | E_USER_WARNING);
```

### 错误日志配置

**错误日志位置**：

```php
<?php
declare(strict_types=1);

// 系统日志（Linux）
ini_set('error_log', 'syslog');

// 文件日志
ini_set('error_log', __DIR__ . '/logs/php-errors.log');

// 获取当前错误日志位置
$logFile = ini_get('error_log');
```

**错误日志格式**：

错误日志通常包含以下信息：
- 时间戳
- 错误级别
- 错误消息
- 文件路径
- 行号
- 堆栈跟踪（如果可用）

**示例日志条目**：

```
[2024-01-15 10:30:45] PHP Warning:  Undefined variable $name in /path/to/file.php on line 42
[2024-01-15 10:30:46] PHP Fatal error:  Uncaught Error: Call to undefined function undefinedFunction() in /path/to/file.php:50
Stack trace:
#0 {main}
  thrown in /path/to/file.php on line 50
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

将 PHP 错误转换为异常，可以使用统一的异常处理机制：

```php
<?php
declare(strict_types=1);

set_error_handler(function (
    int $severity,
    string $message,
    string $file,
    int $line
): void {
    // 只有可恢复的错误才转换为异常
    if (!(error_reporting() & $severity)) {
        return;  // 错误被抑制，不处理
    }
    
    throw new ErrorException($message, 0, $severity, $file, $line);
});

try {
    $result = 10 / 0;  // 会抛出异常
} catch (ErrorException $e) {
    echo "Caught error: " . $e->getMessage() . "\n";
    echo "File: " . $e->getFile() . "\n";
    echo "Line: " . $e->getLine() . "\n";
    echo "Severity: " . $e->getSeverity() . "\n";
}
```

**注意事项**：
- 致命错误（`E_ERROR`、`E_PARSE` 等）无法被错误处理器捕获
- 使用 `@` 操作符抑制的错误不会被错误处理器处理
- 错误处理器返回 `true` 表示已处理，`false` 继续默认处理

### 致命错误处理

致命错误无法被 `set_error_handler()` 捕获，需要使用 `register_shutdown_function()`：

```php
<?php
declare(strict_types=1);

register_shutdown_function(function (): void {
    $error = error_get_last();
    
    if ($error !== null && in_array($error['type'], [E_ERROR, E_PARSE, E_CORE_ERROR, E_COMPILE_ERROR])) {
        // 记录致命错误
        error_log(sprintf(
            "Fatal error: %s in %s on line %d",
            $error['message'],
            $error['file'],
            $error['line']
        ));
        
        // 发送错误响应（如果是 Web 请求）
        if (php_sapi_name() !== 'cli') {
            http_response_code(500);
            echo "Internal Server Error";
        }
    }
});

// 触发致命错误
undefinedFunction();  // 会触发 E_ERROR
```

### 错误恢复策略

**错误恢复**是指在发生错误后，尝试恢复程序执行或提供降级方案：

```php
<?php
declare(strict_types=1);

function safeFileRead(string $file): ?string
{
    $content = @file_get_contents($file);
    
    if ($content === false) {
        // 错误恢复：使用默认值
        error_log("Failed to read file: {$file}, using default");
        return null;
    }
    
    return $content;
}

function safeDatabaseQuery(PDO $pdo, string $query): array
{
    try {
        $stmt = $pdo->query($query);
        return $stmt->fetchAll(PDO::FETCH_ASSOC);
    } catch (PDOException $e) {
        // 错误恢复：记录错误并返回空数组
        error_log("Database query failed: " . $e->getMessage());
        return [];
    }
}
```

## 完整示例

### 错误处理类

```php
<?php
declare(strict_types=1);

class ErrorHandler
{
    private string $logFile;
    private bool $isDevelopment;
    
    public function __construct(string $logFile, bool $isDevelopment = false)
    {
        $this->logFile = $logFile;
        $this->isDevelopment = $isDevelopment;
    }
    
    /**
     * 初始化错误处理
     */
    public function initialize(): void
    {
        // 设置错误报告级别
        if ($this->isDevelopment) {
            error_reporting(E_ALL);
            ini_set('display_errors', '1');
            ini_set('display_startup_errors', '1');
        } else {
            error_reporting(E_ERROR | E_WARNING | E_USER_ERROR | E_USER_WARNING);
            ini_set('display_errors', '0');
            ini_set('display_startup_errors', '0');
        }
        
        // 配置错误日志
        ini_set('log_errors', '1');
        ini_set('error_log', $this->logFile);
        
        // 设置自定义错误处理器
        set_error_handler([$this, 'handleError']);
        
        // 设置致命错误处理器
        register_shutdown_function([$this, 'handleShutdown']);
    }
    
    /**
     * 错误处理器
     */
    public function handleError(
        int $severity,
        string $message,
        string $file,
        int $line
    ): bool {
        // 检查错误是否应该被报告
        if (!(error_reporting() & $severity)) {
            return false;
        }
        
        // 格式化错误信息
        $errorInfo = [
            'severity' => $this->getSeverityName($severity),
            'message' => $message,
            'file' => $file,
            'line' => $line,
            'time' => date('Y-m-d H:i:s'),
        ];
        
        // 记录错误
        $this->logError($errorInfo);
        
        // 开发环境显示错误
        if ($this->isDevelopment) {
            $this->displayError($errorInfo);
        }
        
        // 致命错误需要停止执行
        if (in_array($severity, [E_USER_ERROR, E_RECOVERABLE_ERROR])) {
            return false;  // 继续默认处理
        }
        
        return true;  // 已处理
    }
    
    /**
     * 致命错误处理器
     */
    public function handleShutdown(): void
    {
        $error = error_get_last();
        
        if ($error !== null && in_array($error['type'], [E_ERROR, E_PARSE, E_CORE_ERROR, E_COMPILE_ERROR])) {
            $errorInfo = [
                'severity' => 'FATAL',
                'message' => $error['message'],
                'file' => $error['file'],
                'line' => $error['line'],
                'time' => date('Y-m-d H:i:s'),
            ];
            
            $this->logError($errorInfo);
            
            if (php_sapi_name() !== 'cli') {
                http_response_code(500);
                if ($this->isDevelopment) {
                    $this->displayError($errorInfo);
                } else {
                    echo "Internal Server Error";
                }
            }
        }
    }
    
    /**
     * 记录错误到日志
     */
    private function logError(array $errorInfo): void
    {
        $logMessage = sprintf(
            "[%s] %s: %s in %s on line %d\n",
            $errorInfo['time'],
            $errorInfo['severity'],
            $errorInfo['message'],
            $errorInfo['file'],
            $errorInfo['line']
        );
        
        file_put_contents($this->logFile, $logMessage, FILE_APPEND | LOCK_EX);
    }
    
    /**
     * 显示错误（开发环境）
     */
    private function displayError(array $errorInfo): void
    {
        if (php_sapi_name() === 'cli') {
            echo sprintf(
                "\n[%s] %s: %s in %s on line %d\n",
                $errorInfo['severity'],
                $errorInfo['message'],
                $errorInfo['file'],
                $errorInfo['line']
            );
        } else {
            echo sprintf(
                "<div style='background: #fee; border: 1px solid #f00; padding: 10px; margin: 10px;'>" .
                "<strong>%s</strong>: %s<br>" .
                "<small>File: %s, Line: %d</small>" .
                "</div>",
                $errorInfo['severity'],
                htmlspecialchars($errorInfo['message']),
                htmlspecialchars($errorInfo['file']),
                $errorInfo['line']
            );
        }
    }
    
    /**
     * 获取错误级别名称
     */
    private function getSeverityName(int $severity): string
    {
        $map = [
            E_ERROR => 'ERROR',
            E_WARNING => 'WARNING',
            E_PARSE => 'PARSE',
            E_NOTICE => 'NOTICE',
            E_CORE_ERROR => 'CORE_ERROR',
            E_CORE_WARNING => 'CORE_WARNING',
            E_COMPILE_ERROR => 'COMPILE_ERROR',
            E_COMPILE_WARNING => 'COMPILE_WARNING',
            E_USER_ERROR => 'USER_ERROR',
            E_USER_WARNING => 'USER_WARNING',
            E_USER_NOTICE => 'USER_NOTICE',
            E_RECOVERABLE_ERROR => 'RECOVERABLE_ERROR',
            E_DEPRECATED => 'DEPRECATED',
            E_USER_DEPRECATED => 'USER_DEPRECATED',
        ];
        
        return $map[$severity] ?? 'UNKNOWN';
    }
}

// 使用示例
$isDevelopment = getenv('APP_ENV') === 'development';
$errorHandler = new ErrorHandler(__DIR__ . '/logs/php-errors.log', $isDevelopment);
$errorHandler->initialize();
```

## 最佳实践

### 1. 环境配置

**开发环境**：
- 报告所有错误：`error_reporting(E_ALL)`
- 显示错误：`ini_set('display_errors', '1')`
- 记录错误：`ini_set('log_errors', '1')`

**生产环境**：
- 只报告严重错误：`error_reporting(E_ERROR | E_WARNING | E_USER_ERROR | E_USER_WARNING)`
- 不显示错误：`ini_set('display_errors', '0')`
- 记录所有错误：`ini_set('log_errors', '1')`
- 使用自定义错误处理器记录详细错误信息

### 2. 错误日志管理

- **日志轮转**：定期轮转日志文件，避免日志文件过大
- **日志级别**：根据错误级别记录到不同的日志文件
- **日志格式**：使用统一的日志格式，便于分析和监控
- **日志安全**：确保日志文件权限正确，避免敏感信息泄露

### 3. 错误处理策略

- **可恢复错误**：尝试恢复或提供降级方案
- **致命错误**：记录详细错误信息，优雅地终止程序
- **用户错误**：向用户显示友好的错误信息，不泄露系统细节

### 4. 错误监控

- **实时监控**：使用监控工具实时监控错误日志
- **告警机制**：设置错误阈值，超过阈值时发送告警
- **错误分析**：定期分析错误日志，找出常见错误模式

## 注意事项

1. **错误级别**：根据环境设置合适的错误报告级别。

2. **错误日志**：生产环境必须记录错误日志，并定期轮转。

3. **错误显示**：生产环境不要显示错误给用户，避免泄露系统信息。

4. **错误处理器**：自定义错误处理器应返回 `true` 表示已处理，`false` 继续默认处理。

5. **致命错误**：致命错误无法被错误处理器捕获，使用 `register_shutdown_function()` 处理。

6. **错误抑制**：避免使用 `@` 操作符抑制错误，应该正确处理错误。

7. **错误恢复**：对于可恢复的错误，提供降级方案或默认值。

## 练习

1. 创建一个完整的错误处理类，支持开发和生产环境配置。

2. 编写一个函数，将 PHP 错误转换为异常，并记录详细错误信息。

3. 实现一个错误日志管理器，支持日志轮转和错误分析。

4. 创建一个函数，处理致命错误，并发送友好的错误响应。

5. 编写一个错误监控系统，实时监控错误日志并发送告警。
