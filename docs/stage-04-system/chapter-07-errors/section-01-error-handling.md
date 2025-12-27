# 4.7.1 错误处理机制

## 概述

错误处理是 PHP 程序中的重要部分。在实际应用中，程序可能遇到各种错误，如语法错误、运行时错误、逻辑错误等。理解 PHP 的错误处理机制、如何配置错误报告、如何捕获和处理错误，对于开发健壮的 PHP 应用至关重要。

PHP 提供了多种错误处理机制，包括错误报告配置、自定义错误处理函数、异常处理等。本节介绍 PHP 的错误处理机制，帮助零基础学员掌握错误处理的方法。

**主要内容**：
- 错误处理概述（什么是错误、错误类型）
- 错误报告配置（`error_reporting()`、`ini_set()`）
- 自定义错误处理（`set_error_handler()`）
- 错误类型（E_ERROR、E_WARNING、E_NOTICE 等）
- `trigger_error()` 函数（触发用户错误）
- 实际应用场景和最佳实践

## 特性

- **错误类型多样**：支持多种错误类型
- **灵活配置**：可以配置错误报告级别
- **自定义处理**：可以自定义错误处理函数
- **开发友好**：提供详细的错误信息

## 语法/定义

### error_reporting() 函数

**语法**：`error_reporting(?int $level = null): int`

**参数**：
- `$level`：可选，错误报告级别（使用错误常量）

**返回值**：返回当前的错误报告级别。

### set_error_handler() 函数

**语法**：`set_error_handler(?callable $callback, int $error_levels = E_ALL): ?callable`

**参数**：
- `$callback`：错误处理函数
- `$error_levels`：可选，处理的错误级别

**返回值**：返回之前的错误处理函数。

### trigger_error() 函数

**语法**：`trigger_error(string $message, int $error_level = E_USER_NOTICE): bool`

**参数**：
- `$message`：错误消息
- `$error_level`：可选，错误级别（E_USER_ERROR、E_USER_WARNING、E_USER_NOTICE、E_USER_DEPRECATED）

**返回值**：成功返回 `true`，失败返回 `false`。

## 基本用法

### 示例 1：配置错误报告

```php
<?php
declare(strict_types=1);

// 报告所有错误
error_reporting(E_ALL);

// 报告所有错误，除了警告和通知
error_reporting(E_ALL & ~E_WARNING & ~E_NOTICE);

// 只在开发环境报告错误
if (getenv('APP_ENV') === 'development') {
    error_reporting(E_ALL);
    ini_set('display_errors', '1');
} else {
    error_reporting(E_ALL & ~E_DEPRECATED & ~E_STRICT);
    ini_set('display_errors', '0');
    ini_set('log_errors', '1');
    ini_set('error_log', '/var/log/php_errors.log');
}

// 获取当前错误报告级别
$currentLevel = error_reporting();
echo "当前错误报告级别: {$currentLevel}\n";
```

**说明**：
- `error_reporting()` 配置错误报告级别
- 使用错误常量组合设置报告级别
- 生产环境应该关闭错误显示，记录到日志

### 示例 2：自定义错误处理函数

```php
<?php
declare(strict_types=1);

function customErrorHandler(int $errno, string $errstr, string $errfile, int $errline): bool
{
    // 错误类型映射
    $errorTypes = [
        E_ERROR => 'Fatal Error',
        E_WARNING => 'Warning',
        E_PARSE => 'Parse Error',
        E_NOTICE => 'Notice',
        E_CORE_ERROR => 'Core Error',
        E_CORE_WARNING => 'Core Warning',
        E_COMPILE_ERROR => 'Compile Error',
        E_COMPILE_WARNING => 'Compile Warning',
        E_USER_ERROR => 'User Error',
        E_USER_WARNING => 'User Warning',
        E_USER_NOTICE => 'User Notice',
        E_STRICT => 'Strict Notice',
        E_RECOVERABLE_ERROR => 'Recoverable Error',
        E_DEPRECATED => 'Deprecated',
        E_USER_DEPRECATED => 'User Deprecated',
    ];
    
    $errorType = $errorTypes[$errno] ?? 'Unknown Error';
    
    // 记录错误
    $message = sprintf(
        "[%s] %s: %s in %s on line %d",
        date('Y-m-d H:i:s'),
        $errorType,
        $errstr,
        $errfile,
        $errline
    );
    
    // 写入日志
    error_log($message);
    
    // 如果是致命错误，停止执行
    if ($errno === E_ERROR || $errno === E_USER_ERROR) {
        die("Fatal error: {$errstr}\n");
    }
    
    // 返回 true 表示已处理，false 表示继续使用默认处理
    return true;
}

// 设置自定义错误处理函数
set_error_handler('customErrorHandler', E_ALL);

// 触发错误测试
trigger_error('这是一个用户警告', E_USER_WARNING);
trigger_error('这是一个用户错误', E_USER_ERROR);
```

**说明**：
- `set_error_handler()` 设置自定义错误处理函数
- 错误处理函数接收错误号、错误消息、文件名、行号
- 返回 `true` 表示已处理，`false` 表示使用默认处理

### 示例 3：使用 trigger_error() 触发用户错误

```php
<?php
declare(strict_types=1);

function validateAge(int $age): void
{
    if ($age < 0) {
        trigger_error('年龄不能为负数', E_USER_ERROR);
    }
    
    if ($age > 150) {
        trigger_error('年龄超过合理范围', E_USER_WARNING);
    }
    
    if ($age < 18) {
        trigger_error('用户未满 18 岁', E_USER_NOTICE);
    }
}

// 使用
validateAge(-5);   // 触发 E_USER_ERROR
validateAge(200);  // 触发 E_USER_WARNING
validateAge(16);   // 触发 E_USER_NOTICE
```

**说明**：
- `trigger_error()` 触发用户定义的错误
- 可以使用不同的错误级别（E_USER_ERROR、E_USER_WARNING、E_USER_NOTICE）

### 示例 4：错误处理工具类

```php
<?php
declare(strict_types=1);

class ErrorHandler
{
    private static bool $registered = false;
    
    public static function register(): void
    {
        if (self::$registered) {
            return;
        }
        
        set_error_handler([self::class, 'handleError'], E_ALL);
        register_shutdown_function([self::class, 'handleShutdown']);
        
        self::$registered = true;
    }
    
    public static function handleError(int $errno, string $errstr, string $errfile, int $errline): bool
    {
        // 根据错误级别处理
        switch ($errno) {
            case E_ERROR:
            case E_USER_ERROR:
                self::logError('ERROR', $errstr, $errfile, $errline);
                return true;  // 已处理
                
            case E_WARNING:
            case E_USER_WARNING:
                self::logError('WARNING', $errstr, $errfile, $errline);
                return true;
                
            case E_NOTICE:
            case E_USER_NOTICE:
                self::logError('NOTICE', $errstr, $errfile, $errline);
                return false;  // 继续使用默认处理
                
            default:
                return false;
        }
    }
    
    public static function handleShutdown(): void
    {
        $error = error_get_last();
        if ($error !== null && in_array($error['type'], [E_ERROR, E_PARSE, E_CORE_ERROR, E_COMPILE_ERROR], true)) {
            self::logError('FATAL', $error['message'], $error['file'], $error['line']);
        }
    }
    
    private static function logError(string $level, string $message, string $file, int $line): void
    {
        $logMessage = sprintf(
            "[%s] [%s] %s in %s on line %d\n",
            date('Y-m-d H:i:s'),
            $level,
            $message,
            $file,
            $line
        );
        
        error_log($logMessage);
    }
}

// 使用
ErrorHandler::register();

// 触发错误测试
trigger_error('测试错误', E_USER_WARNING);
```

**说明**：
- 封装错误处理逻辑
- 使用 `register_shutdown_function()` 处理致命错误

### 示例 5：不同环境的错误处理

```php
<?php
declare(strict_types=1);

class EnvironmentErrorHandler
{
    public static function setup(string $environment): void
    {
        match ($environment) {
            'development' => self::setupDevelopment(),
            'production' => self::setupProduction(),
            default => self::setupDefault(),
        };
    }
    
    private static function setupDevelopment(): void
    {
        // 开发环境：显示所有错误
        error_reporting(E_ALL);
        ini_set('display_errors', '1');
        ini_set('display_startup_errors', '1');
    }
    
    private static function setupProduction(): void
    {
        // 生产环境：隐藏错误，记录到日志
        error_reporting(E_ALL & ~E_DEPRECATED & ~E_STRICT);
        ini_set('display_errors', '0');
        ini_set('display_startup_errors', '0');
        ini_set('log_errors', '1');
        ini_set('error_log', '/var/log/php_errors.log');
    }
    
    private static function setupDefault(): void
    {
        // 默认配置
        error_reporting(E_ALL);
        ini_set('display_errors', '0');
        ini_set('log_errors', '1');
    }
}

// 使用
$env = getenv('APP_ENV') ?: 'production';
EnvironmentErrorHandler::setup($env);
```

**说明**：
- 根据环境配置不同的错误处理策略
- 开发环境显示错误，生产环境记录到日志

## 使用场景

### 场景 1：开发环境错误处理

在开发环境显示详细错误信息。

**示例**：见"示例 5：不同环境的错误处理"

### 场景 2：生产环境错误处理

在生产环境记录错误到日志。

**示例**：见"示例 5：不同环境的错误处理"

### 场景 3：自定义错误处理

实现自定义的错误处理逻辑。

**示例**：见"示例 2：自定义错误处理函数"和"示例 4：错误处理工具类"

## 注意事项

### 错误处理函数的返回值

错误处理函数返回 `true` 表示已处理错误，`false` 表示继续使用默认处理。

**示例**：

```php
<?php
declare(strict_types=1);

function errorHandler(int $errno, string $errstr, string $errfile, int $errline): bool
{
    // 处理错误
    error_log("Error: {$errstr}");
    
    // 返回 true 表示已处理，false 表示继续使用默认处理
    return true;
}
```

### 致命错误处理

致命错误（E_ERROR 等）不能被错误处理函数捕获，需要使用 `register_shutdown_function()`。

**示例**：见"示例 4：错误处理工具类"的 `handleShutdown()` 方法

### 错误报告级别

根据环境设置适当的错误报告级别。

**示例**：

```php
<?php
declare(strict_types=1);

// 开发环境
error_reporting(E_ALL);

// 生产环境
error_reporting(E_ALL & ~E_DEPRECATED & ~E_STRICT);
```

## 常见问题

### 问题 1：如何配置错误报告？

**回答**：使用 `error_reporting()` 函数配置错误报告级别，使用 `ini_set()` 配置错误显示和日志。

**示例**：见"示例 1：配置错误报告"

### 问题 2：如何自定义错误处理？

**回答**：使用 `set_error_handler()` 设置自定义错误处理函数。

**示例**：见"示例 2：自定义错误处理函数"

### 问题 3：如何触发用户错误？

**回答**：使用 `trigger_error()` 函数触发用户定义的错误。

**示例**：见"示例 3：使用 trigger_error() 触发用户错误"

### 问题 4：如何处理致命错误？

**回答**：致命错误不能被错误处理函数捕获，需要使用 `register_shutdown_function()` 处理。

**示例**：见"示例 4：错误处理工具类"的 `handleShutdown()` 方法

## 最佳实践

### 1. 根据环境配置错误处理

开发环境显示错误，生产环境记录到日志。

**示例**：见"示例 5：不同环境的错误处理"

### 2. 使用自定义错误处理函数

实现统一的错误处理逻辑。

**示例**：见"示例 4：错误处理工具类"

### 3. 记录错误到日志

在生产环境，记录所有错误到日志文件。

**示例**：

```php
<?php
declare(strict_types=1);

ini_set('log_errors', '1');
ini_set('error_log', '/var/log/php_errors.log');
```

### 4. 不要在生产环境显示错误

在生产环境，关闭错误显示，只记录到日志。

**示例**：

```php
<?php
declare(strict_types=1);

ini_set('display_errors', '0');
ini_set('log_errors', '1');
```

## 对比分析

### 错误类型对比

| 错误类型           | 严重程度 | 是否可捕获 | 是否停止执行 |
|:-------------------|:---------|:-----------|:-------------|
| **E_ERROR**        | 致命     | ❌ 否      | ✅ 是        |
| **E_WARNING**      | 警告     | ✅ 是      | ❌ 否        |
| **E_NOTICE**       | 通知     | ✅ 是      | ❌ 否        |
| **E_USER_ERROR**   | 致命     | ✅ 是      | ✅ 是        |
| **E_USER_WARNING** | 警告     | ✅ 是      | ❌ 否        |

### 开发环境 vs 生产环境

| 配置项             | 开发环境 | 生产环境   |
|:-------------------|:---------|:-----------|
| **display_errors** | On       | Off        |
| **log_errors**     | On       | On         |
| **error_reporting**| E_ALL    | E_ALL & ~E_DEPRECATED |

## 练习任务

1. **错误处理工具类**：创建一个完整的错误处理工具类。

2. **环境配置工具**：实现一个工具，根据环境配置错误处理。

3. **错误日志分析工具**：创建一个工具，分析错误日志。

4. **错误报告生成工具**：编写一个工具，生成错误报告。

5. **错误处理最佳实践指南**：创建一个指南，总结错误处理的最佳实践。

## 相关章节

- **[4.7.2 异常处理机制](section-02-exceptions.md)**：了解异常处理的相关内容
- **[4.7.3 错误和异常的最佳实践](section-03-best-practices.md)**：了解错误处理的最佳实践
- **[4.9.1 日志体系设计](../chapter-09-logging/section-01-logging-system.md)**：了解日志系统的相关内容
