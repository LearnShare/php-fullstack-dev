# 4.7.2 异常处理机制

## 概述

异常处理是现代 PHP 编程中处理错误情况的标准方法。与错误处理不同，异常是程序逻辑中的异常情况，可以通过 try-catch 块捕获和处理。异常处理提供了更灵活和结构化的错误处理方式，是现代 PHP 应用推荐使用的错误处理机制。

理解异常的概念、try-catch-finally 语法、自定义异常类、异常传播等，对于编写健壮的 PHP 应用至关重要。

**主要内容**：
- 异常概述（什么是异常、异常与错误的区别）
- try-catch-finally 语法
- Exception 类和 Throwable 接口
- 自定义异常类
- 异常传播和未捕获异常
- `set_exception_handler()` 函数
- 实际应用场景和最佳实践

## 特性

- **结构化处理**：提供结构化的错误处理方式
- **类型安全**：支持异常类型检查
- **灵活控制**：可以精确控制异常处理
- **信息丰富**：异常包含详细的错误信息

## 语法/定义

### try-catch-finally 语法

**语法**：

```php
try {
    // 可能抛出异常的代码
} catch (ExceptionType1 $e) {
    // 处理 ExceptionType1 异常
} catch (ExceptionType2 $e) {
    // 处理 ExceptionType2 异常
} finally {
    // 无论是否异常都执行的代码
}
```

### throw 语句

**语法**：`throw new Exception("错误消息");`

**参数**：
- `new Exception(...)`：抛出异常对象

**说明**：抛出异常，中断当前代码执行。

## 基本用法

### 示例 1：基本的 try-catch

```php
<?php
declare(strict_types=1);

try {
    // 可能抛出异常的代码
    $result = 10 / 0;  // 这会触发警告，但不会抛出异常
    echo "结果: {$result}\n";
} catch (DivisionByZeroError $e) {
    echo "捕获到除零错误: " . $e->getMessage() . "\n";
} catch (Exception $e) {
    echo "捕获到异常: " . $e->getMessage() . "\n";
}

// 抛出异常
try {
    throw new Exception('这是一个异常');
} catch (Exception $e) {
    echo "捕获到异常: " . $e->getMessage() . "\n";
    echo "文件: " . $e->getFile() . "\n";
    echo "行号: " . $e->getLine() . "\n";
}
```

**说明**：
- `try` 块包含可能抛出异常的代码
- `catch` 块捕获和处理异常
- 可以捕获特定类型的异常

### 示例 2：多个 catch 块

```php
<?php
declare(strict_types=1);

class ValidationException extends Exception {}
class DatabaseException extends Exception {}

try {
    // 可能抛出不同异常的代码
    validateData($data);
    saveToDatabase($data);
} catch (ValidationException $e) {
    echo "验证错误: " . $e->getMessage() . "\n";
} catch (DatabaseException $e) {
    echo "数据库错误: " . $e->getMessage() . "\n";
} catch (Exception $e) {
    echo "其他错误: " . $e->getMessage() . "\n";
}
```

**说明**：
- 可以定义多个 catch 块，按顺序匹配异常类型
- 更具体的异常类型应该放在前面
- 通用异常类型（如 `Exception`）应该放在最后

### 示例 3：finally 块

```php
<?php
declare(strict_types=1);

function processFile(string $filename): void
{
    $handle = fopen($filename, 'r');
    if ($handle === false) {
        throw new RuntimeException("Cannot open file: {$filename}");
    }
    
    try {
        // 处理文件
        while (($line = fgets($handle)) !== false) {
            processLine($line);
        }
    } finally {
        // 无论是否异常，都会执行
        fclose($handle);
        echo "文件已关闭\n";
    }
}

// 使用
try {
    processFile('data.txt');
} catch (Exception $e) {
    echo "错误: " . $e->getMessage() . "\n";
}
// 文件已关闭（即使在 catch 之前也会执行）
```

**说明**：
- `finally` 块无论是否发生异常都会执行
- 常用于资源清理（关闭文件、释放资源等）

### 示例 4：自定义异常类

```php
<?php
declare(strict_types=1);

// 自定义异常类
class ValidationException extends Exception
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

class DatabaseException extends Exception
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

// 使用自定义异常
function validateUser(array $data): void
{
    $errors = [];
    
    if (empty($data['name'])) {
        $errors['name'] = '姓名不能为空';
    }
    
    if (empty($data['email'])) {
        $errors['email'] = '邮箱不能为空';
    }
    
    if (!empty($errors)) {
        throw new ValidationException('验证失败', $errors);
    }
}

// 捕获自定义异常
try {
    validateUser(['name' => '']);
} catch (ValidationException $e) {
    echo "验证错误: " . $e->getMessage() . "\n";
    echo "详细错误: " . print_r($e->getErrors(), true) . "\n";
}
```

**说明**：
- 自定义异常类可以扩展 `Exception` 类
- 可以添加自定义属性和方法
- 提供更详细的错误信息

### 示例 5：异常链

```php
<?php
declare(strict_types=1);

try {
    try {
        // 尝试操作
        throw new RuntimeException('原始错误');
    } catch (RuntimeException $e) {
        // 捕获并重新抛出，添加更多信息
        throw new Exception('操作失败', 0, $e);
    }
} catch (Exception $e) {
    echo "当前异常: " . $e->getMessage() . "\n";
    
    // 获取原始异常
    $previous = $e->getPrevious();
    if ($previous !== null) {
        echo "原始异常: " . $previous->getMessage() . "\n";
    }
    
    // 打印完整的堆栈跟踪
    echo "堆栈跟踪:\n" . $e->getTraceAsString() . "\n";
}
```

**说明**：
- 异常可以链接（通过 `$previous` 参数）
- 可以保留原始异常信息

### 示例 6：未捕获异常处理

```php
<?php
declare(strict_types=1);

// 设置未捕获异常处理函数
set_exception_handler(function (Throwable $e): void {
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
        echo "错误: " . $e->getMessage() . "\n";
    } else {
        http_response_code(500);
        echo json_encode(['error' => 'Internal server error']);
    }
});

// 未捕获的异常会被上面的处理函数捕获
throw new Exception('这是一个未捕获的异常');
```

**说明**：
- `set_exception_handler()` 设置未捕获异常的处理函数
- 用于全局异常处理

## 使用场景

### 场景 1：数据验证

在数据验证失败时抛出异常。

**示例**：见"示例 4：自定义异常类"

### 场景 2：资源管理

使用 finally 块确保资源被正确释放。

**示例**：见"示例 3：finally 块"

### 场景 3：API 错误处理

在 API 中统一处理异常。

**示例**：

```php
<?php
declare(strict_types=1);

try {
    $result = apiOperation();
    echo json_encode(['success' => true, 'data' => $result]);
} catch (ValidationException $e) {
    http_response_code(400);
    echo json_encode(['error' => $e->getMessage(), 'errors' => $e->getErrors()]);
} catch (Exception $e) {
    http_response_code(500);
    error_log($e->getMessage());
    echo json_encode(['error' => 'Internal server error']);
}
```

## 注意事项

### 异常的性能开销

异常处理有性能开销，不应该用于正常的程序流程控制。

**示例**：

```php
<?php
declare(strict_types=1);

// ❌ 不推荐：使用异常控制流程
// try {
//     if (!isset($array[$key])) {
//         throw new Exception('Key not found');
//     }
// } catch (Exception $e) {
//     // ...
// }

// ✅ 推荐：使用正常的条件判断
if (isset($array[$key])) {
    // 处理
} else {
    // 处理不存在的情况
}
```

### 异常类型选择

根据具体情况选择合适的异常类型，或创建自定义异常类。

**示例**：

```php
<?php
declare(strict_types=1);

// 使用内置异常类型
throw new InvalidArgumentException('Invalid argument');
throw new RuntimeException('Runtime error');
throw new LogicException('Logic error');

// 或使用自定义异常类型
throw new ValidationException('Validation failed');
```

## 常见问题

### 问题 1：什么时候使用异常？

**回答**：异常用于处理程序逻辑中的异常情况，如验证失败、资源不可用等。不应该用于正常的程序流程控制。

**示例**：

```php
<?php
declare(strict_types=1);

// ✅ 适合使用异常：验证失败
if (!validate($data)) {
    throw new ValidationException('Validation failed');
}

// ❌ 不适合使用异常：正常流程控制
// if (!condition) {
//     throw new Exception('Condition not met');  // 不推荐
// }
```

### 问题 2：try-catch-finally 的执行顺序？

**回答**：
1. 执行 try 块
2. 如果发生异常，执行匹配的 catch 块
3. 无论是否异常，都执行 finally 块

**示例**：见"示例 3：finally 块"

### 问题 3：如何创建自定义异常？

**回答**：继承 `Exception` 类，可以添加自定义属性和方法。

**示例**：见"示例 4：自定义异常类"

### 问题 4：异常如何传播？

**回答**：异常会向上传播，直到被捕获。如果未被捕获，会触发未捕获异常处理函数。

**示例**：见"示例 6：未捕获异常处理"

## 最佳实践

### 1. 使用异常处理业务逻辑错误

使用异常处理业务逻辑中的异常情况。

**示例**：

```php
<?php
declare(strict_types=1);

function processOrder(array $order): void
{
    if (empty($order['items'])) {
        throw new InvalidArgumentException('Order must have items');
    }
    
    // 处理订单
}
```

### 2. 创建有意义的异常类型

根据具体情况创建自定义异常类，提供更详细的错误信息。

**示例**：见"示例 4：自定义异常类"

### 3. 提供详细的异常信息

在异常消息中包含有用的信息，便于调试。

**示例**：

```php
<?php
declare(strict_types=1);

throw new ValidationException("用户 ID {$userId} 验证失败: " . implode(', ', $errors));
```

### 4. 合理使用 finally 块

使用 finally 块确保资源被正确释放。

**示例**：见"示例 3：finally 块"

### 5. 全局异常处理

设置全局异常处理函数，处理未捕获的异常。

**示例**：见"示例 6：未捕获异常处理"

## 对比分析

### 异常 vs 错误

| 特性         | 异常（Exception）              | 错误（Error）                  |
|:-------------|:-------------------------------|:-------------------------------|
| **类型**     | 程序逻辑异常                   | 系统级错误                     |
| **可捕获**   | ✅ 可以捕获                    | ⚠️ 部分可捕获                  |
| **处理方式** | try-catch                     | set_error_handler()           |
| **使用场景** | 业务逻辑异常                   | 系统错误                       |

### try-catch vs set_error_handler()

| 特性         | try-catch                      | set_error_handler()            |
|:-------------|:-------------------------------|:-------------------------------|
| **处理类型** | 异常                           | 错误                           |
| **结构化**   | ✅ 结构化（代码块）            | ⚠️ 函数式                      |
| **类型安全** | ✅ 支持类型检查                | ⚠️ 基于错误号                  |
| **推荐使用** | ✅ 现代 PHP 推荐               | ⚠️ 用于错误处理                |

## 练习任务

1. **异常处理工具类**：创建一个工具类，封装异常处理功能。

2. **自定义异常类库**：实现一组自定义异常类，用于不同的场景。

3. **异常处理框架**：创建一个异常处理框架，统一处理应用中的异常。

4. **资源管理工具**：编写一个工具，使用 finally 块管理资源。

5. **异常处理最佳实践指南**：创建一个指南，总结异常处理的最佳实践。

## 相关章节

- **[4.7.1 错误处理机制](section-01-error-handling.md)**：了解错误处理的相关内容
- **[4.7.3 错误和异常的最佳实践](section-03-best-practices.md)**：了解错误和异常的最佳实践
