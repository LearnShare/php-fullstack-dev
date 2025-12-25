# 2.16.2 异常处理

## 概述

异常（Exception）是 PHP 中处理错误和异常情况的现代方式。PHP 7.0+ 引入了 `Throwable` 接口，统一了异常和错误的处理。

## 异常体系

### Throwable 接口

PHP 7.0+ 引入了 `Throwable` 接口，统一了异常和错误的处理。所有可捕获的异常和错误都实现 `Throwable` 接口：

```php
<?php
declare(strict_types=1);

interface Throwable
{
    public function getMessage(): string;
    public function getCode(): int;
    public function getFile(): string;
    public function getLine(): int;
    public function getTrace(): array;
    public function getTraceAsString(): string;
    public function getPrevious(): ?Throwable;
    public function __toString(): string;
}
```

**使用 Throwable 捕获所有异常和错误**：

```php
<?php
declare(strict_types=1);

try {
    // 可能抛出异常或错误的代码
    undefinedFunction();  // 会抛出 Error
    throw new Exception('Custom exception');
} catch (Throwable $e) {
    echo "Caught: " . $e->getMessage() . "\n";
    echo "Type: " . get_class($e) . "\n";
    echo "File: " . $e->getFile() . "\n";
    echo "Line: " . $e->getLine() . "\n";
}
```

### 异常层次结构

```
Throwable
├── Error (PHP 7.0+)
│   ├── ArithmeticError
│   │   ├── DivisionByZeroError
│   │   └── TypeError
│   ├── AssertionError
│   ├── ParseError
│   └── TypeError
│       └── ArgumentCountError
└── Exception
    ├── RuntimeException
    │   ├── OutOfBoundsException
    │   ├── OverflowException
    │   ├── RangeException
    │   ├── UnderflowException
    │   └── UnexpectedValueException
    ├── LogicException
    │   ├── BadFunctionCallException
    │   │   └── BadMethodCallException
    │   ├── DomainException
    │   ├── InvalidArgumentException
    │   └── LengthException
    └── ErrorException
```

### Throwable 的方法

所有 `Throwable` 对象都提供以下方法：

- `getMessage()`：获取异常消息
- `getCode()`：获取异常代码
- `getFile()`：获取抛出异常的文件
- `getLine()`：获取抛出异常的行号
- `getTrace()`：获取堆栈跟踪数组
- `getTraceAsString()`：获取堆栈跟踪字符串
- `getPrevious()`：获取前一个异常（异常链）
- `__toString()`：转换为字符串

### Exception 类

传统异常类，用于业务逻辑错误：

```php
<?php
declare(strict_types=1);

class InvalidOrderException extends Exception {}

function placeOrder(array $payload): void
{
    if (!isset($payload['items'])) {
        throw new InvalidOrderException('Items required');
    }
}
```

### Error 类

`Error` 类（PHP 7.0+）用于表示执行时错误，如类型错误、调用未定义函数等。这些错误通常表示程序中的严重问题。

**常见的 Error 子类**：

| 类名 | 说明 | 示例 |
| :--- | :--- | :--- |
| `TypeError` | 类型错误 | 参数类型不匹配 |
| `DivisionByZeroError` | 除以零错误 | `10 / 0` |
| `ParseError` | 解析错误 | 语法错误 |
| `ArgumentCountError` | 参数数量错误 | 函数调用参数不足 |

**示例**：

```php
<?php
declare(strict_types=1);

// DivisionByZeroError
try {
    $result = 10 / 0;
} catch (DivisionByZeroError $e) {
    echo "Division by zero: " . $e->getMessage() . "\n";
}

// TypeError
try {
    function add(int $a, int $b): int {
        return $a + $b;
    }
    add("10", "20");  // 会抛出 TypeError
} catch (TypeError $e) {
    echo "Type error: " . $e->getMessage() . "\n";
}

// ArgumentCountError
try {
    function greet(string $name): void {
        echo "Hello, {$name}\n";
    }
    greet();  // 会抛出 ArgumentCountError
} catch (ArgumentCountError $e) {
    echo "Argument count error: " . $e->getMessage() . "\n";
}
```

## try/catch/finally

### 基本语法

```php
try {
    // 可能抛出异常的代码
} catch (ExceptionType $e) {
    // 处理异常
} finally {
    // 总是执行的代码
}
```

### 基本用法

```php
<?php
declare(strict_types=1);

try {
    $result = divide(10, 0);
} catch (DivisionByZeroError $e) {
    echo "Cannot divide by zero\n";
} catch (Exception $e) {
    echo "Error: " . $e->getMessage() . "\n";
} finally {
    echo "Cleanup\n";
}
```

### 捕获多个异常类型

```php
<?php
declare(strict_types=1);

try {
    processData($data);
} catch (InvalidArgumentException | TypeError $e) {
    echo "Invalid data: " . $e->getMessage() . "\n";
} catch (Exception $e) {
    echo "General error: " . $e->getMessage() . "\n";
}
```

### finally 块

`finally` 块无论是否发生异常都会执行：

```php
<?php
declare(strict_types=1);

$file = fopen('data.txt', 'r');
try {
    $content = fread($file, 1024);
} catch (Exception $e) {
    echo "Error: " . $e->getMessage() . "\n";
} finally {
    fclose($file);  // 总是执行
}
```

## 自定义异常

### 创建自定义异常

```php
<?php
declare(strict_types=1);

class ValidationException extends Exception
{
    private array $errors;
    
    public function __construct(string $message, array $errors = [])
    {
        parent::__construct($message);
        $this->errors = $errors;
    }
    
    public function getErrors(): array
    {
        return $this->errors;
    }
}

function validateUser(array $data): void
{
    $errors = [];
    
    if (empty($data['name'])) {
        $errors['name'] = 'Name is required';
    }
    
    if (empty($data['email'])) {
        $errors['email'] = 'Email is required';
    }
    
    if (!empty($errors)) {
        throw new ValidationException('Validation failed', $errors);
    }
}
```

## 异常传播

### 重新抛出异常

```php
<?php
declare(strict_types=1);

try {
    processData($data);
} catch (InvalidArgumentException $e) {
    // 记录日志
    error_log("Invalid data: " . $e->getMessage());
    // 重新抛出
    throw $e;
}
```

### 包装异常

包装异常是指将底层异常包装为更通用的异常，同时保留原始异常信息：

```php
<?php
declare(strict_types=1);

try {
    $result = $externalService->call();
} catch (PDOException $e) {
    // 包装为更通用的异常，保留原始异常
    throw new RuntimeException(
        "Service call failed: " . $e->getMessage(),
        0,
        $e  // 前一个异常
    );
}

// 获取异常链
try {
    // 可能抛出包装异常
} catch (RuntimeException $e) {
    echo "Current: " . $e->getMessage() . "\n";
    
    $previous = $e->getPrevious();
    if ($previous !== null) {
        echo "Previous: " . $previous->getMessage() . "\n";
    }
}
```

### 异常链

异常链允许在包装异常时保留原始异常信息，便于调试：

```php
<?php
declare(strict_types=1);

function processData(array $data): void
{
    try {
        validateData($data);
        saveToDatabase($data);
    } catch (ValidationException $e) {
        // 包装为业务异常
        throw new BusinessException(
            "Data processing failed",
            0,
            $e
        );
    } catch (PDOException $e) {
        // 包装为业务异常
        throw new BusinessException(
            "Database operation failed",
            0,
            $e
        );
    }
}

// 遍历异常链
function printExceptionChain(Throwable $e): void
{
    $level = 0;
    while ($e !== null) {
        echo str_repeat("  ", $level) . get_class($e) . ": " . $e->getMessage() . "\n";
        $e = $e->getPrevious();
        $level++;
    }
}
```

## 全局异常处理器

### set_exception_handler() - 设置全局异常处理器

**语法**：`set_exception_handler(?callable $callback): ?callable`

**参数**：
- `$callback`：异常处理回调函数，接收 `Throwable $exception` 参数，无返回值。如果为 `null`，则恢复默认异常处理

**返回值**：返回之前设置的异常处理函数，如果没有则返回 `null`。

**用途**：捕获所有未捕获的异常，避免脚本意外终止。

```php
<?php
declare(strict_types=1);

set_exception_handler(function (Throwable $e): void {
    // 记录异常
    error_log(sprintf(
        "Uncaught exception: %s in %s on line %d\nStack trace:\n%s",
        $e->getMessage(),
        $e->getFile(),
        $e->getLine(),
        $e->getTraceAsString()
    ));
    
    // 发送错误响应
    if (php_sapi_name() !== 'cli') {
        http_response_code(500);
        echo "Internal Server Error";
    } else {
        echo "Uncaught exception: " . $e->getMessage() . "\n";
    }
    
    // 可选：退出脚本
    exit(1);
});

// 未捕获的异常会被全局处理器捕获
throw new Exception("This will be caught by the global handler");
```

### 异常日志记录

```php
<?php
declare(strict_types=1);

class ExceptionLogger
{
    private string $logFile;
    
    public function __construct(string $logFile)
    {
        $this->logFile = $logFile;
    }
    
    public function log(Throwable $e): void
    {
        $logEntry = [
            'timestamp' => date('Y-m-d H:i:s'),
            'type' => get_class($e),
            'message' => $e->getMessage(),
            'file' => $e->getFile(),
            'line' => $e->getLine(),
            'trace' => $e->getTraceAsString(),
            'previous' => $e->getPrevious() !== null ? $this->formatException($e->getPrevious()) : null,
        ];
        
        $logMessage = json_encode($logEntry, JSON_PRETTY_PRINT | JSON_UNESCAPED_UNICODE) . "\n";
        file_put_contents($this->logFile, $logMessage, FILE_APPEND | LOCK_EX);
    }
    
    private function formatException(Throwable $e): array
    {
        return [
            'type' => get_class($e),
            'message' => $e->getMessage(),
            'file' => $e->getFile(),
            'line' => $e->getLine(),
        ];
    }
}

// 使用
$logger = new ExceptionLogger(__DIR__ . '/logs/exceptions.log');

try {
    throw new RuntimeException("Test exception");
} catch (Throwable $e) {
    $logger->log($e);
}
```

## 异常恢复策略

### 重试机制

```php
<?php
declare(strict_types=1);

function retryOperation(callable $operation, int $maxRetries = 3): mixed
{
    $attempt = 0;
    $lastException = null;
    
    while ($attempt < $maxRetries) {
        try {
            return $operation();
        } catch (Exception $e) {
            $lastException = $e;
            $attempt++;
            
            if ($attempt < $maxRetries) {
                sleep(1);  // 等待后重试
                continue;
            }
        }
    }
    
    throw $lastException;
}

// 使用
$result = retryOperation(function () {
    return riskyOperation();
});
```

### 降级方案

```php
<?php
declare(strict_types=1);

function getDataWithFallback(): array
{
    try {
        // 尝试从主数据源获取
        return getDataFromPrimarySource();
    } catch (Exception $e) {
        error_log("Primary source failed: " . $e->getMessage());
        
        try {
            // 降级到备用数据源
            return getDataFromFallbackSource();
        } catch (Exception $e) {
            error_log("Fallback source failed: " . $e->getMessage());
            
            // 返回默认值
            return [];
        }
    }
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

class ExceptionHandling
{
    public static function demonstrate(): void
    {
        echo "=== 基本异常处理 ===\n";
        try {
            $result = divide(10, 2);
            echo "Result: {$result}\n";
        } catch (DivisionByZeroError $e) {
            echo "Error: " . $e->getMessage() . "\n";
        }
        
        echo "\n=== 自定义异常 ===\n";
        try {
            validateUser(['name' => '']);
        } catch (ValidationException $e) {
            echo "Validation failed: " . $e->getMessage() . "\n";
            print_r($e->getErrors());
        }
        
        echo "\n=== finally 块 ===\n";
        $file = fopen('php://memory', 'r+');
        try {
            fwrite($file, 'test');
        } finally {
            fclose($file);
            echo "File closed\n";
        }
    }
}

function divide(int $a, int $b): float
{
    if ($b === 0) {
        throw new DivisionByZeroError('Cannot divide by zero');
    }
    return $a / $b;
}

class ValidationException extends Exception
{
    private array $errors;
    
    public function __construct(string $message, array $errors = [])
    {
        parent::__construct($message);
        $this->errors = $errors;
    }
    
    public function getErrors(): array
    {
        return $this->errors;
    }
}

function validateUser(array $data): void
{
    $errors = [];
    if (empty($data['name'])) {
        $errors['name'] = 'Name is required';
    }
    if (!empty($errors)) {
        throw new ValidationException('Validation failed', $errors);
    }
}

ExceptionHandling::demonstrate();
```

## 异常处理最佳实践

### 1. 异常层次结构设计

**设计原则**：
- 创建有意义的异常层次结构
- 使用具体的异常类型，而不是通用的 `Exception`
- 异常类名应该清楚地表达错误类型

**示例**：

```php
<?php
declare(strict_types=1);

// 基础业务异常
abstract class BusinessException extends Exception {}

// 具体业务异常
class ValidationException extends BusinessException {}
class NotFoundException extends BusinessException {}
class UnauthorizedException extends BusinessException {}
class PaymentException extends BusinessException {}
```

### 2. 异常消息规范

**好的异常消息**：
- 清晰描述问题
- 包含必要的上下文信息
- 便于调试和定位问题

```php
<?php
declare(strict_types=1);

// ❌ 不好的异常消息
throw new Exception("Error");

// ✅ 好的异常消息
throw new ValidationException("User email '{$email}' is invalid");
throw new NotFoundException("User with ID {$userId} not found");
```

### 3. 异常处理策略

**处理原则**：
- 只捕获可以处理的异常
- 不要捕获异常后忽略（空 catch 块）
- 要么处理异常，要么重新抛出
- 使用 `finally` 块清理资源

```php
<?php
declare(strict_types=1);

// ❌ 不好的做法：忽略异常
try {
    riskyOperation();
} catch (Exception $e) {
    // 什么都不做
}

// ✅ 好的做法：处理或重新抛出
try {
    riskyOperation();
} catch (ValidationException $e) {
    // 处理验证错误
    return ['error' => $e->getMessage()];
} catch (Exception $e) {
    // 记录日志并重新抛出
    error_log($e->getMessage());
    throw $e;
}
```

### 4. 性能考虑

**注意事项**：
- 异常处理有性能开销，不要用于控制流程
- 使用条件检查代替异常处理来控制流程
- 只在真正的异常情况下抛出异常

```php
<?php
declare(strict_types=1);

// ❌ 不好的做法：使用异常控制流程
function findUser(int $id): User
{
    try {
        return $database->getUser($id);
    } catch (NotFoundException $e) {
        return null;  // 使用异常控制流程
    }
}

// ✅ 好的做法：使用条件检查
function findUser(int $id): ?User
{
    $user = $database->findUser($id);
    return $user !== null ? $user : null;
}
```

### 5. 异常日志记录

**日志记录原则**：
- 记录所有未捕获的异常
- 记录异常链（previous exception）
- 包含足够的上下文信息
- 使用结构化日志格式

## 注意事项

1. **异常类型**：捕获异常时，先捕获具体类型，再捕获通用类型。

2. **finally 块**：用于清理资源，无论是否发生异常都会执行。

3. **异常信息**：提供有意义的异常消息，便于调试。

4. **异常传播**：不要捕获异常后忽略，要么处理，要么重新抛出。

5. **性能考虑**：异常处理有性能开销，不要用于控制流程。

6. **全局异常处理器**：设置全局异常处理器捕获未捕获的异常。

7. **异常恢复**：对于可恢复的异常，提供重试或降级方案。

8. **异常链**：使用异常链保留原始异常信息，便于调试。

## 练习

1. 创建一个完整的异常处理类，支持异常日志记录和全局异常处理。

2. 设计一个异常层次结构，用于业务逻辑错误处理。

3. 实现一个异常恢复机制，支持重试和降级方案。

4. 创建一个函数，遍历和打印异常链。

5. 编写一个全局异常处理器，记录异常并发送友好的错误响应。
