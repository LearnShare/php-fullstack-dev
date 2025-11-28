# 3.6 异常体系与框架级设计

## 目标

- 深入理解 PHP 异常体系（`Throwable`、`Exception`、`Error`）的层次结构。
- 掌握自定义异常的创建与使用，设计语义化的异常类层次。
- 理解框架级异常设计原则，能够构建可维护的异常处理体系。
- 熟悉异常传播、捕获策略与日志记录的最佳实践。

## 异常体系层次

### Throwable 接口

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

### Exception vs Error

| 类型        | 用途                           | 示例                     |
| :---------- | :----------------------------- | :----------------------- |
| `Exception` | 程序逻辑错误，可预期           | 验证失败、业务规则违反   |
| `Error`     | 系统级错误，通常不可恢复       | 内存不足、类型错误       |

```php
// Exception：业务逻辑异常
class InvalidOrderException extends Exception {}

// Error：系统级错误（通常不需要手动创建）
// PHP 会在类型错误、内存不足等情况下自动抛出 Error
```

### 异常类层次

```
Throwable
├── Error
│   ├── TypeError
│   ├── ParseError
│   ├── ArithmeticError
│   └── ...
└── Exception
    ├── RuntimeException
    ├── LogicException
    │   ├── InvalidArgumentException
    │   ├── DomainException
    │   └── ...
    └── 自定义异常
```

## 自定义异常

### 基础自定义异常

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

### 异常属性扩展

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

### 异常链（Exception Chaining）

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

## 异常处理策略

### 捕获与重新抛出

```php
class OrderService
{
    public function __construct(
        private OrderRepository $repository,
        private PaymentGateway $paymentGateway
    ) {
    }

    public function processOrder(Order $order): void
    {
        try {
            $this->paymentGateway->charge($order->getTotal());
            $this->repository->save($order);
        } catch (PaymentException $e) {
            // 记录支付异常，但重新抛出为业务异常
            logger()->error('Payment failed', [
                'order_id' => $order->getId(),
                'error' => $e->getMessage(),
            ]);
            throw new OrderProcessingException('Failed to process payment', $e);
        } catch (DatabaseException $e) {
            // 数据库异常需要立即处理
            throw new OrderProcessingException('Failed to save order', $e);
        }
    }
}
```

### 异常转换

- 将底层异常转换为领域异常，保持领域层独立性。

```php
class UserRepository
{
    public function find(int $id): ?User
    {
        try {
            $data = $this->db->query("SELECT * FROM users WHERE id = ?", [$id]);
            return $data ? new User($data) : null;
        } catch (PDOException $e) {
            // 将基础设施异常转换为领域异常
            throw new UserRepositoryException("Failed to find user {$id}", $e);
        }
    }
}
```

### 异常处理中间件

```php
class ExceptionHandler
{
    public function handle(Throwable $exception): Response
    {
        return match (true) {
            $exception instanceof ValidationException => $this->handleValidation($exception),
            $exception instanceof NotFoundException => $this->handleNotFound($exception),
            $exception instanceof UnauthorizedException => $this->handleUnauthorized($exception),
            default => $this->handleGeneric($exception),
        };
    }

    private function handleValidation(ValidationException $e): Response
    {
        return new JsonResponse([
            'error' => 'Validation failed',
            'errors' => $e->getErrors(),
        ], 422);
    }

    private function handleNotFound(NotFoundException $e): Response
    {
        return new JsonResponse([
            'error' => $e->getMessage(),
        ], 404);
    }

    private function handleUnauthorized(UnauthorizedException $e): Response
    {
        return new JsonResponse([
            'error' => $e->getMessage(),
        ], 401);
    }

    private function handleGeneric(Throwable $e): Response
    {
        logger()->error('Unhandled exception', [
            'message' => $e->getMessage(),
            'trace' => $e->getTraceAsString(),
        ]);

        return new JsonResponse([
            'error' => 'Internal server error',
        ], 500);
    }
}
```

## 框架级异常设计

### 异常基类

```php
namespace App\Exceptions;

abstract class AppException extends Exception
{
    protected array $context = [];

    public function __construct(
        string $message = '',
        int $code = 0,
        ?Throwable $previous = null,
        array $context = []
    ) {
        parent::__construct($message, $code, $previous);
        $this->context = $context;
    }

    public function getContext(): array
    {
        return $this->context;
    }

    public function toArray(): array
    {
        return [
            'type' => static::class,
            'message' => $this->getMessage(),
            'code' => $this->getCode(),
            'context' => $this->context,
        ];
    }
}
```

### 领域异常

```php
namespace App\Domain\Exceptions;

class DomainException extends \App\Exceptions\AppException
{
    public function __construct(
        string $message,
        array $context = [],
        ?Throwable $previous = null
    ) {
        parent::__construct($message, 0, $previous, $context);
    }
}

class UserNotFoundException extends DomainException
{
    public function __construct(int $userId, ?Throwable $previous = null)
    {
        parent::__construct(
            "User with ID {$userId} not found",
            ['user_id' => $userId],
            $previous
        );
    }
}
```

### 应用异常

```php
namespace App\Application\Exceptions;

class ApplicationException extends \App\Exceptions\AppException
{
    public function __construct(
        string $message,
        array $context = [],
        ?Throwable $previous = null
    ) {
        parent::__construct($message, 0, $previous, $context);
    }
}

class BusinessRuleViolationException extends ApplicationException
{
    public function __construct(
        string $rule,
        array $context = [],
        ?Throwable $previous = null
    ) {
        parent::__construct(
            "Business rule violated: {$rule}",
            array_merge(['rule' => $rule], $context),
            $previous
        );
    }
}
```

## 异常与日志

### 结构化日志记录

```php
class ExceptionLogger
{
    public function log(Throwable $exception): void
    {
        $context = [
            'exception' => get_class($exception),
            'message' => $exception->getMessage(),
            'file' => $exception->getFile(),
            'line' => $exception->getLine(),
            'trace' => $exception->getTraceAsString(),
        ];

        if ($exception instanceof AppException) {
            $context = array_merge($context, $exception->getContext());
        }

        if ($exception->getPrevious() !== null) {
            $context['previous'] = [
                'exception' => get_class($exception->getPrevious()),
                'message' => $exception->getPrevious()->getMessage(),
            ];
        }

        logger()->error('Exception occurred', $context);
    }
}
```

### 异常上下文

```php
class OrderService
{
    public function cancelOrder(int $orderId): void
    {
        try {
            $order = $this->repository->find($orderId);
            if ($order === null) {
                throw new OrderNotFoundException($orderId);
            }

            if (!$order->canCancel()) {
                throw new OrderCannotBeCancelledException($orderId, [
                    'status' => $order->getStatus(),
                    'reason' => 'Order is already shipped',
                ]);
            }

            $order->cancel();
            $this->repository->save($order);
        } catch (OrderException $e) {
            logger()->warning('Order cancellation failed', [
                'order_id' => $orderId,
                'exception' => get_class($e),
                'context' => $e->getContext(),
            ]);
            throw $e;
        }
    }
}
```

## 异常处理最佳实践

### 1. 使用语义化异常

```php
// 不推荐：使用通用异常
throw new Exception('User not found');

// 推荐：使用语义化异常
throw new UserNotFoundException($userId);
```

### 2. 提供足够的上下文

```php
class PaymentException extends DomainException
{
    public function __construct(
        string $message,
        public readonly string $transactionId,
        public readonly float $amount,
        ?Throwable $previous = null
    ) {
        parent::__construct($message, [
            'transaction_id' => $transactionId,
            'amount' => $amount,
        ], $previous);
    }
}
```

### 3. 避免捕获过于宽泛的异常

```php
// 不推荐：捕获所有异常
try {
    // ...
} catch (Exception $e) {
    // 处理所有异常
}

// 推荐：捕获特定异常
try {
    // ...
} catch (UserNotFoundException $e) {
    // 处理用户不存在
} catch (ValidationException $e) {
    // 处理验证错误
} catch (Throwable $e) {
    // 处理其他所有异常
}
```

### 4. 在适当的层次处理异常

```php
// 领域层：抛出领域异常
class User
{
    public function changeEmail(string $email): void
    {
        if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
            throw new InvalidEmailException($email);
        }
        $this->email = $email;
    }
}

// 应用层：转换异常
class UserService
{
    public function updateEmail(int $userId, string $email): void
    {
        try {
            $user = $this->repository->find($userId);
            if ($user === null) {
                throw new UserNotFoundException($userId);
            }
            $user->changeEmail($email);
            $this->repository->save($user);
        } catch (InvalidEmailException $e) {
            throw new ValidationException('Invalid email format', ['email' => $e->getMessage()]);
        }
    }
}

// 表现层：处理异常并返回响应
class UserController
{
    public function updateEmail(Request $request): Response
    {
        try {
            $this->userService->updateEmail(
                $request->get('user_id'),
                $request->get('email')
            );
            return new JsonResponse(['success' => true]);
        } catch (ValidationException $e) {
            return new JsonResponse(['errors' => $e->getErrors()], 422);
        } catch (UserNotFoundException $e) {
            return new JsonResponse(['error' => $e->getMessage()], 404);
        }
    }
}
```

## 练习

1. 创建一个异常类层次结构，包含 `AppException` 基类，以及 `DomainException`、`ApplicationException`、`InfrastructureException` 子类。

2. 实现一个 `ValidationException` 类，包含错误字段映射，并创建一个验证器类在验证失败时抛出该异常。

3. 设计一个异常处理中间件，能够根据异常类型返回不同的 HTTP 状态码和响应格式。

4. 创建一个 `UserService`，在查找用户、更新用户等操作中抛出语义化的异常，并实现异常链（chaining）。

5. 实现一个异常日志记录器，能够记录异常的完整信息（包括上下文、堆栈跟踪、前一个异常等）。

6. 设计一个异常转换器，将底层异常（如 `PDOException`）转换为领域异常，保持领域层的独立性。
