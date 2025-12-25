# 3.6.2 自定义异常

## 概述

自定义异常允许创建语义化的异常类，提供更好的错误信息和上下文。理解如何设计和实现自定义异常对于构建可维护的应用至关重要。

## 基础自定义异常

### 简单自定义异常

```php
class UserNotFoundException extends Exception
{
    public function __construct(int $userId, ?Throwable $previous = null)
    {
        $message = "User with ID {$userId} not found";
        parent::__construct($message, 404, $previous);
    }
}

class InvalidEmailException extends DomainException
{
    public function __construct(string $email, ?Throwable $previous = null)
    {
        $message = "Invalid email format: {$email}";
        parent::__construct($message, 0, $previous);
    }
}
```

### 使用示例

```php
class UserService
{
    public function findUser(int $id): User
    {
        $user = $this->repository->find($id);
        if ($user === null) {
            throw new UserNotFoundException($id);
        }
        return $user;
    }

    public function createUser(string $email): User
    {
        if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
            throw new InvalidEmailException($email);
        }
        // 创建用户逻辑
        return new User($email);
    }
}
```

## 异常属性扩展

### 添加额外属性

- 可以在异常中添加额外属性，提供更多上下文信息。

```php
class ValidationException extends DomainException
{
    public function __construct(
        string $message,
        public readonly array $errors,
        ?Throwable $previous = null
    ) {
        parent::__construct($message, 0, $previous);
    }

    public function getErrors(): array
    {
        return $this->errors;
    }

    public function hasErrors(): bool
    {
        return !empty($this->errors);
    }
}

// 使用
try {
    // 验证逻辑
    throw new ValidationException('Validation failed', [
        'email' => 'Invalid email format',
        'age' => 'Age must be at least 18',
    ]);
} catch (ValidationException $e) {
    foreach ($e->getErrors() as $field => $error) {
        echo "{$field}: {$error}\n";
    }
}
```

### 上下文信息

```php
class PaymentException extends DomainException
{
    public function __construct(
        string $message,
        public readonly string $transactionId,
        public readonly float $amount,
        ?Throwable $previous = null
    ) {
        parent::__construct($message, 0, $previous);
    }

    public function toArray(): array
    {
        return [
            'message' => $this->getMessage(),
            'transaction_id' => $this->transactionId,
            'amount' => $this->amount,
        ];
    }
}
```

## 异常链（Exception Chaining）

### 链接异常

- 使用 `$previous` 参数链接异常，保留完整的错误上下文。

```php
class DatabaseException extends RuntimeException
{
    public function __construct(string $message, ?Throwable $previous = null)
    {
        parent::__construct($message, 0, $previous);
    }
}

class UserService
{
    public function findUser(int $id): User
    {
        try {
            $data = $this->repository->find($id);
            if ($data === null) {
                throw new UserNotFoundException($id);
            }
            return new User($data);
        } catch (PDOException $e) {
            throw new DatabaseException('Failed to fetch user', $e);
        }
    }
}
```

### 异常链遍历

```php
function printExceptionChain(Throwable $e): void
{
    $current = $e;
    $level = 0;

    while ($current !== null) {
        echo str_repeat('  ', $level) . get_class($current) . ": {$current->getMessage()}\n";
        $current = $current->getPrevious();
        $level++;
    }
}

try {
    // 可能抛出异常链的代码
} catch (Exception $e) {
    printExceptionChain($e);
}
```

## 异常分类策略

### 按层次分类

```php
// 领域层异常
namespace App\Domain\Exceptions;

class DomainException extends \DomainException {}

class UserNotFoundException extends DomainException {}
class InvalidUserStateException extends DomainException {}

// 应用层异常
namespace App\Application\Exceptions;

class ApplicationException extends \RuntimeException {}

class ServiceUnavailableException extends ApplicationException {}
class BusinessRuleViolationException extends ApplicationException {}

// 基础设施层异常
namespace App\Infrastructure\Exceptions;

class InfrastructureException extends \RuntimeException {}

class DatabaseConnectionException extends InfrastructureException {}
class CacheException extends InfrastructureException {}
```

### 按功能分类

```php
// 验证异常
class ValidationException extends DomainException
{
    public function __construct(
        string $message,
        public readonly array $errors
    ) {
        parent::__construct($message);
    }
}

// 授权异常
class UnauthorizedException extends RuntimeException
{
    public function __construct(string $message = 'Unauthorized')
    {
        parent::__construct($message, 401);
    }
}

// 资源不存在异常
class NotFoundException extends RuntimeException
{
    public function __construct(string $resource, int|string $id)
    {
        parent::__construct("{$resource} with ID {$id} not found", 404);
    }
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

namespace App\Domain\Exceptions;

class UserNotFoundException extends DomainException
{
    public function __construct(
        int $userId,
        ?Throwable $previous = null
    ) {
        parent::__construct(
            "User with ID {$userId} not found",
            0,
            $previous
        );
    }
}

class InvalidUserStateException extends DomainException
{
    public function __construct(
        string $currentState,
        string $requiredState,
        ?Throwable $previous = null
    ) {
        parent::__construct(
            "User is in state '{$currentState}', but '{$requiredState}' is required",
            0,
            $previous
        );
    }
}

namespace App\Application\Exceptions;

class BusinessRuleViolationException extends ApplicationException
{
    public function __construct(
        string $rule,
        array $context = [],
        ?Throwable $previous = null
    ) {
        parent::__construct(
            "Business rule violated: {$rule}",
            0,
            $previous
        );
    }
}
```

## 注意事项

1. **语义化命名**：异常类名应该清晰表达异常的含义。

2. **上下文信息**：在异常中包含足够的上下文信息，便于调试。

3. **异常链**：使用 `$previous` 参数保留完整的错误上下文。

4. **分类策略**：根据项目需求选择合适的异常分类策略。

5. **继承关系**：选择合适的基类（`Exception`、`RuntimeException`、`DomainException` 等）。

## 练习

1. 创建一个异常类层次结构，包含 `AppException` 基类，以及多个子类。

2. 实现一个 `ValidationException` 类，包含错误字段映射。

3. 创建一个 `PaymentException` 类，包含交易 ID 和金额等上下文信息。

4. 实现异常链，演示如何链接多个异常。

5. 设计一个异常分类系统，按层次和功能分类异常。
