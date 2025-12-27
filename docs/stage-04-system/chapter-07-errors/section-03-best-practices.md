# 4.7.3 错误和异常的最佳实践

## 概述

错误和异常处理需要遵循最佳实践，以确保代码的健壮性、可维护性和用户体验。在实际应用中，我们需要根据环境配置错误处理、设计合理的异常类型、提供用户友好的错误信息等。理解并应用这些最佳实践，对于开发高质量的 PHP 应用至关重要。

本节总结错误和异常处理的最佳实践，包括处理策略、日志记录、用户友好的错误信息等，帮助零基础学员编写高质量的错误处理代码。

**主要内容**：
- 错误处理策略（开发环境 vs 生产环境）
- 异常处理策略（异常类型设计、异常信息设计）
- 错误和异常的区分（何时使用错误、何时使用异常）
- 错误日志记录（日志级别、日志格式）
- 用户友好的错误信息
- 错误恢复机制
- 实际应用场景和完整示例

## 特性

- **环境适配**：根据不同环境采用不同的处理策略
- **用户友好**：提供用户友好的错误信息
- **日志完善**：完善的错误日志记录
- **健壮性**：提高程序的健壮性

## 语法/定义

### 错误和异常处理最佳实践要点

最佳实践没有单一的语法定义，而是综合性的策略和实践方法。

## 基本用法

### 示例 1：环境配置错误处理

```php
<?php
declare(strict_types=1);

class ErrorHandlingConfig
{
    public static function configure(string $environment): void
    {
        match ($environment) {
            'development' => self::configureDevelopment(),
            'production' => self::configureProduction(),
            'testing' => self::configureTesting(),
            default => self::configureDefault(),
        };
    }
    
    private static function configureDevelopment(): void
    {
        // 开发环境：显示所有错误
        error_reporting(E_ALL);
        ini_set('display_errors', '1');
        ini_set('display_startup_errors', '1');
        ini_set('log_errors', '1');
    }
    
    private static function configureProduction(): void
    {
        // 生产环境：隐藏错误，记录到日志
        error_reporting(E_ALL & ~E_DEPRECATED & ~E_STRICT);
        ini_set('display_errors', '0');
        ini_set('display_startup_errors', '0');
        ini_set('log_errors', '1');
        ini_set('error_log', '/var/log/php_errors.log');
        
        // 设置异常处理
        set_exception_handler([self::class, 'handleException']);
        set_error_handler([self::class, 'handleError']);
    }
    
    private static function configureTesting(): void
    {
        // 测试环境：报告所有错误，不显示
        error_reporting(E_ALL);
        ini_set('display_errors', '0');
        ini_set('log_errors', '1');
    }
    
    private static function configureDefault(): void
    {
        error_reporting(E_ALL);
        ini_set('display_errors', '0');
        ini_set('log_errors', '1');
    }
    
    public static function handleException(Throwable $e): void
    {
        // 记录异常
        error_log(sprintf(
            "Uncaught exception: %s in %s:%d\nStack trace:\n%s",
            $e->getMessage(),
            $e->getFile(),
            $e->getLine(),
            $e->getTraceAsString()
        ));
        
        // 显示用户友好的错误信息
        if (php_sapi_name() === 'cli') {
            echo "错误: 发生了一个错误，请查看日志\n";
        } else {
            http_response_code(500);
            echo json_encode(['error' => 'Internal server error']);
        }
    }
    
    public static function handleError(int $errno, string $errstr, string $errfile, int $errline): bool
    {
        error_log(sprintf(
            "Error [%d]: %s in %s on line %d",
            $errno,
            $errstr,
            $errfile,
            $errline
        ));
        
        return true;  // 已处理
    }
}

// 使用
$env = getenv('APP_ENV') ?: 'production';
ErrorHandlingConfig::configure($env);
```

**说明**：
- 根据环境配置不同的错误处理策略
- 开发环境显示错误，生产环境记录到日志

### 示例 2：异常类型层次设计

```php
<?php
declare(strict_types=1);

// 基础异常类
class AppException extends Exception {}

// 验证异常
class ValidationException extends AppException
{
    private array $errors = [];
    
    public function __construct(string $message, array $errors = [], int $code = 0, ?Throwable $previous = null)
    {
        parent::__construct($message, $code, $previous);
        $this->errors = $errors;
    }
    
    public function getErrors(): array
    {
        return $this->errors;
    }
}

// 数据库异常
class DatabaseException extends AppException
{
    private ?string $sql = null;
    
    public function __construct(string $message, ?string $sql = null, int $code = 0, ?Throwable $previous = null)
    {
        parent::__construct($message, $code, $previous);
        $this->sql = $sql;
    }
    
    public function getSql(): ?string
    {
        return $this->sql;
    }
}

// 业务逻辑异常
class BusinessException extends AppException {}

// 使用异常层次
function processData(array $data): void
{
    try {
        validateData($data);
        saveToDatabase($data);
    } catch (ValidationException $e) {
        // 处理验证错误
        throw $e;
    } catch (DatabaseException $e) {
        // 处理数据库错误
        throw new BusinessException('数据处理失败', 0, $e);
    }
}
```

**说明**：
- 设计合理的异常类型层次
- 提供详细的错误信息

### 示例 3：用户友好的错误信息

```php
<?php
declare(strict_types=1);

class UserFriendlyErrorHandler
{
    public static function handleException(Throwable $e, bool $showDetails = false): void
    {
        // 记录详细错误（用于调试）
        error_log(sprintf(
            "Exception: %s in %s:%d\nStack trace:\n%s",
            $e->getMessage(),
            $e->getFile(),
            $e->getLine(),
            $e->getTraceAsString()
        ));
        
        // 显示用户友好的错误信息
        if ($showDetails) {
            // 开发环境：显示详细信息
            echo "错误: " . $e->getMessage() . "\n";
            echo "文件: " . $e->getFile() . "\n";
            echo "行号: " . $e->getLine() . "\n";
        } else {
            // 生产环境：显示通用错误信息
            if (php_sapi_name() === 'cli') {
                echo "发生了一个错误，请稍后重试\n";
            } else {
                http_response_code(500);
                echo json_encode([
                    'error' => 'Internal server error',
                    'message' => 'An error occurred. Please try again later.'
                ]);
            }
        }
    }
    
    public static function handleValidationException(ValidationException $e): void
    {
        // 验证错误：显示具体错误信息（对用户有用）
        http_response_code(400);
        echo json_encode([
            'error' => 'Validation failed',
            'errors' => $e->getErrors()
        ]);
    }
}

// 使用
try {
    // 业务逻辑
} catch (ValidationException $e) {
    UserFriendlyErrorHandler::handleValidationException($e);
} catch (Exception $e) {
    $showDetails = getenv('APP_ENV') === 'development';
    UserFriendlyErrorHandler::handleException($e, $showDetails);
}
```

**说明**：
- 区分技术错误和用户错误
- 用户错误显示具体信息，技术错误显示通用信息

### 示例 4：错误日志记录

```php
<?php
declare(strict_types=1);

class ErrorLogger
{
    private string $logFile;
    
    public function __construct(string $logFile = '/var/log/php_errors.log')
    {
        $this->logFile = $logFile;
    }
    
    public function logError(int $errno, string $errstr, string $errfile, int $errline): void
    {
        $message = sprintf(
            "[%s] [ERROR] %s in %s on line %d\n",
            date('Y-m-d H:i:s'),
            $errstr,
            $errfile,
            $errline
        );
        
        $this->writeLog($message);
    }
    
    public function logException(Throwable $e, string $level = 'ERROR'): void
    {
        $message = sprintf(
            "[%s] [%s] %s in %s:%d\nStack trace:\n%s\n",
            date('Y-m-d H:i:s'),
            $level,
            $e->getMessage(),
            $e->getFile(),
            $e->getLine(),
            $e->getTraceAsString()
        );
        
        $this->writeLog($message);
    }
    
    private function writeLog(string $message): void
    {
        error_log($message, 3, $this->logFile);
    }
}

// 使用
$logger = new ErrorLogger();

try {
    // 业务逻辑
} catch (Exception $e) {
    $logger->logException($e);
    throw $e;
}
```

**说明**：
- 记录详细的错误信息到日志
- 包含时间戳、错误消息、文件位置、堆栈跟踪等信息

### 示例 5：错误恢复机制

```php
<?php
declare(strict_types=1);

class RetryableOperation
{
    private int $maxRetries;
    private int $delaySeconds;
    
    public function __construct(int $maxRetries = 3, int $delaySeconds = 1)
    {
        $this->maxRetries = $maxRetries;
        $this->delaySeconds = $delaySeconds;
    }
    
    public function execute(callable $operation): mixed
    {
        $attempt = 0;
        $lastException = null;
        
        while ($attempt < $this->maxRetries) {
            try {
                return $operation();
            } catch (Exception $e) {
                $lastException = $e;
                $attempt++;
                
                if ($attempt < $this->maxRetries) {
                    // 等待后重试
                    sleep($this->delaySeconds);
                    continue;
                }
            }
        }
        
        // 所有重试都失败，抛出最后一个异常
        throw $lastException;
    }
}

// 使用
$retryable = new RetryableOperation(3, 1);

try {
    $result = $retryable->execute(function () {
        // 可能失败的操作
        return riskyOperation();
    });
} catch (Exception $e) {
    echo "操作失败，已重试 3 次: " . $e->getMessage() . "\n";
}
```

**说明**：
- 实现错误恢复机制（重试）
- 适合临时性错误（如网络错误）

## 使用场景

### 场景 1：API 错误处理

在 API 中统一处理错误和异常。

**示例**：

```php
<?php
declare(strict_types=1);

try {
    $result = apiOperation();
    echo json_encode(['success' => true, 'data' => $result]);
} catch (ValidationException $e) {
    http_response_code(400);
    echo json_encode(['error' => 'Validation failed', 'errors' => $e->getErrors()]);
} catch (NotFoundException $e) {
    http_response_code(404);
    echo json_encode(['error' => 'Resource not found']);
} catch (Exception $e) {
    http_response_code(500);
    error_log($e->getMessage());
    echo json_encode(['error' => 'Internal server error']);
}
```

### 场景 2：CLI 错误处理

在 CLI 程序中使用适当的错误处理。

**示例**：

```php
<?php
declare(strict_types=1);

try {
    // CLI 操作
    processCommand();
} catch (Exception $e) {
    fwrite(STDERR, "错误: " . $e->getMessage() . "\n");
    exit(1);  // 返回错误退出码
}
```

## 注意事项

### 错误信息的敏感性

在生产环境，不要向用户显示敏感的错误信息。

**示例**：

```php
<?php
declare(strict_types=1);

// ❌ 错误：显示敏感信息
// echo "数据库连接失败: " . $e->getMessage();

// ✅ 正确：显示通用错误信息
echo "系统错误，请稍后重试";
error_log("Database connection failed: " . $e->getMessage());  // 记录详细错误
```

### 错误处理和性能

错误处理不应该影响正常流程的性能。

**示例**：

```php
<?php
declare(strict_types=1);

// ❌ 不推荐：使用异常控制流程
// try {
//     if (condition) {
//         throw new Exception();
//     }
// } catch (Exception $e) {
//     // ...
// }

// ✅ 推荐：使用正常流程控制
if (condition) {
    // 处理
} else {
    // 处理
}
```

## 常见问题

### 问题 1：如何区分错误和异常？

**回答**：
- **错误**：系统级问题（如内存不足、文件不存在），使用错误处理机制
- **异常**：程序逻辑异常（如验证失败、业务规则违反），使用异常处理机制

**示例**：

```php
<?php
declare(strict_types=1);

// 错误：系统级问题（通常使用错误处理）
if (!file_exists($file)) {
    trigger_error("File not found: {$file}", E_USER_WARNING);
}

// 异常：业务逻辑异常（使用异常处理）
if (!validate($data)) {
    throw new ValidationException('Validation failed');
}
```

### 问题 2：如何设计异常类型？

**回答**：根据业务场景设计异常类型层次，提供详细的错误信息。

**示例**：见"示例 2：异常类型层次设计"

### 问题 3：如何提供用户友好的错误信息？

**回答**：区分技术错误和用户错误，用户错误显示具体信息，技术错误显示通用信息。

**示例**：见"示例 3：用户友好的错误信息"

### 问题 4：如何平衡错误处理和性能？

**回答**：不要使用异常控制正常流程，只在真正的异常情况使用异常。

**示例**：见"错误处理和性能"部分

## 最佳实践

### 1. 根据环境配置错误处理

开发环境显示错误，生产环境记录到日志。

**示例**：见"示例 1：环境配置错误处理"

### 2. 使用异常处理业务逻辑错误

业务逻辑中的异常情况使用异常处理。

**示例**：

```php
<?php
declare(strict_types=1);

if (!validate($data)) {
    throw new ValidationException('Validation failed');
}
```

### 3. 提供用户友好的错误信息

区分技术错误和用户错误，提供适当的错误信息。

**示例**：见"示例 3：用户友好的错误信息"

### 4. 记录详细的错误日志

记录详细的错误信息到日志，便于调试和问题排查。

**示例**：见"示例 4：错误日志记录"

### 5. 实现错误恢复机制

对于临时性错误，实现重试等恢复机制。

**示例**：见"示例 5：错误恢复机制"

## 对比分析

### 开发环境 vs 生产环境

| 配置项             | 开发环境 | 生产环境   |
|:-------------------|:---------|:-----------|
| **display_errors** | On       | Off        |
| **log_errors**     | On       | On         |
| **错误信息详细度** | 详细     | 通用       |
| **堆栈跟踪**       | 显示     | 仅日志     |

### 错误 vs 异常使用场景

| 场景           | 推荐使用 | 说明                        |
|:---------------|:---------|:----------------------------|
| **系统错误**   | 错误处理 | 内存不足、文件不存在等      |
| **业务异常**   | 异常处理 | 验证失败、业务规则违反等    |
| **配置错误**   | 错误处理 | 配置无效等                  |
| **数据验证**   | 异常处理 | 数据格式错误、验证失败等    |

## 练习任务

1. **错误处理框架**：创建一个完整的错误处理框架，支持环境配置、日志记录等。

2. **异常类型库**：实现一组异常类型，用于不同的业务场景。

3. **用户友好错误处理工具**：创建一个工具，将技术错误转换为用户友好的错误信息。

4. **错误恢复框架**：实现一个框架，支持错误重试、降级等恢复机制。

5. **错误处理最佳实践指南**：创建一个完整的指南，总结错误和异常处理的最佳实践。

## 相关章节

- **[4.7.1 错误处理机制](section-01-error-handling.md)**：了解错误处理的相关内容
- **[4.7.2 异常处理机制](section-02-exceptions.md)**：了解异常处理的相关内容
- **[4.9.1 日志体系设计](../chapter-09-logging/section-01-logging-system.md)**：了解日志系统的相关内容
