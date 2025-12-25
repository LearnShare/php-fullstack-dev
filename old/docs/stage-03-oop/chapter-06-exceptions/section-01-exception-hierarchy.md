# 3.6.1 异常体系层次

## 概述

PHP 的异常体系以 `Throwable` 接口为根，分为 `Exception` 和 `Error` 两大类。理解异常体系层次对于正确处理不同类型的错误至关重要。

## Throwable 接口

### 基本定义

- PHP 7.0+ 所有可抛出对象都实现 `Throwable` 接口。
- `Exception` 和 `Error` 都继承自 `Throwable`。

```php
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

### Throwable 方法

| 方法 | 说明 | 返回值 |
| :--- | :--- | :----- |
| `getMessage()` | 获取异常消息 | `string` |
| `getCode()` | 获取异常代码 | `int` |
| `getFile()` | 获取抛出异常的文件 | `string` |
| `getLine()` | 获取抛出异常的行号 | `int` |
| `getTrace()` | 获取堆栈跟踪 | `array` |
| `getTraceAsString()` | 获取堆栈跟踪字符串 | `string` |
| `getPrevious()` | 获取前一个异常 | `?Throwable` |
| `__toString()` | 转换为字符串 | `string` |

## Exception vs Error

### 区别对比

| 类型        | 用途                           | 示例                     |
| :---------- | :----------------------------- | :----------------------- |
| `Exception` | 程序逻辑错误，可预期           | 验证失败、业务规则违反   |
| `Error`     | 系统级错误，通常不可恢复       | 内存不足、类型错误       |

### Exception 示例

```php
// Exception：业务逻辑异常
class InvalidOrderException extends Exception {}

class UserService
{
    public function findUser(int $id): User
    {
        if ($id <= 0) {
            throw new InvalidArgumentException('User ID must be positive');
        }
        // ...
    }
}
```

### Error 示例

```php
// Error：系统级错误（通常不需要手动创建）
// PHP 会在类型错误、内存不足等情况下自动抛出 Error

function divide(int $a, int $b): int
{
    return $a / $b;
}

// 类型错误会抛出 TypeError
// divide('10', '2'); // TypeError

// 内存不足会抛出 Error
// 通常不需要手动处理
```

## 异常类层次

### 完整层次结构

```
Throwable
├── Error
│   ├── TypeError
│   ├── ParseError
│   ├── ArithmeticError
│   │   ├── DivisionByZeroError
│   │   └── ...
│   └── ...
└── Exception
    ├── RuntimeException
    ├── LogicException
    │   ├── InvalidArgumentException
    │   ├── DomainException
    │   └── ...
    └── 自定义异常
```

### 内置异常类型

```php
// LogicException 系列
throw new InvalidArgumentException('Invalid argument');
throw new DomainException('Domain rule violated');
throw new LengthException('Length mismatch');
throw new OutOfRangeException('Value out of range');

// RuntimeException 系列
throw new RuntimeException('Runtime error');
throw new OutOfBoundsException('Index out of bounds');
throw new OverflowException('Overflow occurred');
throw new UnderflowException('Underflow occurred');
```

## 异常捕获

### 捕获 Throwable

```php
try {
    // 可能抛出异常或错误的代码
    riskyOperation();
} catch (Throwable $e) {
    // 捕获所有可抛出对象
    echo "Error: {$e->getMessage()}\n";
}
```

### 分别捕获 Exception 和 Error

```php
try {
    riskyOperation();
} catch (Exception $e) {
    // 处理业务异常
    echo "Exception: {$e->getMessage()}\n";
} catch (Error $e) {
    // 处理系统错误
    echo "Error: {$e->getMessage()}\n";
    // 通常需要记录日志并终止执行
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

class ExceptionDemo
{
    public static function demonstrate(): void
    {
        // Exception：业务逻辑异常
        try {
            self::validateUser(-1);
        } catch (InvalidArgumentException $e) {
            echo "Validation failed: {$e->getMessage()}\n";
        }

        // Error：系统级错误
        try {
            self::typeError();
        } catch (TypeError $e) {
            echo "Type error: {$e->getMessage()}\n";
        }

        // 捕获所有 Throwable
        try {
            self::riskyOperation();
        } catch (Throwable $e) {
            echo "Caught: " . get_class($e) . "\n";
            echo "Message: {$e->getMessage()}\n";
        }
    }

    private static function validateUser(int $id): void
    {
        if ($id <= 0) {
            throw new InvalidArgumentException('User ID must be positive');
        }
    }

    private static function typeError(): void
    {
        // 这会抛出 TypeError
        $result = array_sum(['a', 'b', 'c']);
    }

    private static function riskyOperation(): void
    {
        // 可能抛出各种异常
        throw new RuntimeException('Something went wrong');
    }
}
```

## 注意事项

1. **Exception vs Error**：Exception 用于业务逻辑错误，Error 用于系统级错误。

2. **捕获顺序**：先捕获具体异常，再捕获通用异常。

3. **Error 处理**：Error 通常表示严重问题，可能需要终止执行。

4. **类型提示**：使用 `Throwable` 类型提示可以捕获所有异常和错误。

5. **错误报告**：生产环境应该记录所有 Throwable，但只向用户显示友好的错误信息。

## 练习

1. 创建一个示例，演示 Exception 和 Error 的区别。

2. 实现一个函数，捕获不同类型的异常并处理。

3. 创建一个异常层次结构，包含多个自定义异常类。

4. 演示如何使用 `getPrevious()` 方法获取异常链。

5. 实现一个异常处理器，能够区分 Exception 和 Error 并采取不同的处理策略。
