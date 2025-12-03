# 2.17.2 异常处理

## 概述

异常（Exception）是 PHP 中处理错误和异常情况的现代方式。PHP 7.0+ 引入了 `Throwable` 接口，统一了异常和错误的处理。

## 异常体系

### Throwable 接口

所有可捕获的异常和错误都实现 `Throwable` 接口：

```php
<?php
declare(strict_types=1);

try {
    // 可能抛出异常或错误的代码
} catch (Throwable $e) {
    echo "Caught: " . $e->getMessage() . "\n";
}
```

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

执行时错误，如类型错误、调用未定义函数等：

```php
<?php
declare(strict_types=1);

try {
    $result = 10 / 0;  // 会抛出 DivisionByZeroError
} catch (Error $e) {
    echo "Error: " . $e->getMessage() . "\n";
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

```php
<?php
declare(strict_types=1);

try {
    $result = externalService->call();
} catch (Exception $e) {
    // 包装为更通用的异常
    throw new RuntimeException(
        "Service call failed: " . $e->getMessage(),
        0,
        $e
    );
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

## 注意事项

1. **异常类型**：捕获异常时，先捕获具体类型，再捕获通用类型。

2. **finally 块**：用于清理资源，无论是否发生异常都会执行。

3. **异常信息**：提供有意义的异常消息，便于调试。

4. **异常传播**：不要捕获异常后忽略，要么处理，要么重新抛出。

5. **性能考虑**：异常处理有性能开销，不要用于控制流程。

## 练习

1. 创建一个异常处理类，统一管理异常。

2. 编写一个函数，将不同类型的错误转换为相应的异常。

3. 实现一个验证器，使用自定义异常报告验证错误。

4. 创建一个函数，使用 `finally` 块确保资源释放。

5. 编写一个函数，实现异常的链式传播。
