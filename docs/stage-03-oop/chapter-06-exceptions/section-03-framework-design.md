# 3.6.3 框架级异常设计

## 概述

框架级异常设计涉及异常的分类、处理策略、日志记录和最佳实践。理解这些原则对于构建可维护的异常处理体系至关重要。

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

## 异常处理中间件

### 统一异常处理

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

## 完整示例

```php
<?php
declare(strict_types=1);

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
}

namespace App\Application;

use App\Exceptions\AppException;

class ExceptionHandler
{
    public function handle(Throwable $exception): array
    {
        $this->log($exception);

        return match (true) {
            $exception instanceof ValidationException => [
                'error' => 'Validation failed',
                'errors' => $exception->getErrors(),
                'status' => 422
            ],
            $exception instanceof NotFoundException => [
                'error' => $exception->getMessage(),
                'status' => 404
            ],
            $exception instanceof AppException => [
                'error' => $exception->getMessage(),
                'context' => $exception->getContext(),
                'status' => 500
            ],
            default => [
                'error' => 'Internal server error',
                'status' => 500
            ]
        };
    }

    private function log(Throwable $exception): void
    {
        $context = [
            'exception' => get_class($exception),
            'message' => $exception->getMessage(),
        ];

        if ($exception instanceof AppException) {
            $context = array_merge($context, $exception->getContext());
        }

        error_log(json_encode($context));
    }
}
```

## 注意事项

1. **异常层次**：设计清晰的异常层次结构，便于分类和处理。

2. **上下文信息**：在异常中包含足够的上下文信息，便于调试和日志记录。

3. **异常转换**：在适当的层次转换异常，保持领域层独立性。

4. **统一处理**：使用异常处理中间件统一处理异常，返回一致的响应格式。

5. **日志记录**：记录所有异常的详细信息，包括上下文和堆栈跟踪。

## 练习

1. 设计一个异常处理中间件，能够根据异常类型返回不同的 HTTP 状态码和响应格式。

2. 创建一个 `UserService`，在查找用户、更新用户等操作中抛出语义化的异常，并实现异常链。

3. 实现一个异常日志记录器，能够记录异常的完整信息（包括上下文、堆栈跟踪、前一个异常等）。

4. 设计一个异常转换器，将底层异常（如 `PDOException`）转换为领域异常。

5. 创建一个完整的异常处理体系，包含异常基类、领域异常、应用异常和异常处理器。
