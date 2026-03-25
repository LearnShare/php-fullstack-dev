# 7.4.1 异常体系层次

## 概述

异常体系层次是构建健壮应用程序的基础。一个设计良好的异常体系能够帮助开发者快速定位问题、准确处理错误、优雅地处理边界情况。在 PHP 应用开发中，合理地组织和使用异常可以显著提升代码的可维护性和可读性。

本章将详细介绍如何设计一个清晰、合理的异常体系，包括异常层次的划分、基础异常类的设计、自定义异常的使用，以及异常处理最佳实践。

**主要内容**：
- 异常体系的概念和重要性
- PHP 内置异常层次
- 自定义异常层次设计
- 基础异常类设计
- 异常分类方法

## PHP 内置异常层次

### Exception 类层次

PHP 提供了一个基础的异常类层次：

```
Throwable
├── Error
│   ├── TypeError
│   ├── ArgumentCountError
│   ├── ArithmeticError
│   │   ├── DivisionByZeroError
│   │   └── FloatNaNError
│   └── ParseError
│   └── ValueError
└── Exception
    ├── RuntimeException
    │   ├── OutOfBoundsException
    │   ├── LengthException
    │   ├── InvalidArgumentException
    │   └── RangeException
    ├── LogicException
    │   ├── BadMethodCallException
    │   ├── BadFunctionCallException
    │   ├── DomainException
    │   ├── LengthException
    │   └── OutOfRangeException
    └── ...
```

### 核心异常类说明

```php
<?php
declare(strict_types=1);

// Exception - 所有异常的基类
// RuntimeException - 运行时异常（通常由外部因素导致）
// LogicException - 逻辑异常（代码错误）

// 常见使用场景：

// InvalidArgumentException - 无效参数
function divide(int $a, int $b): int 
{
    if ($b === 0) {
        throw new InvalidArgumentException('Divisor cannot be zero');
    }
    return $a / $b;
}

// OutOfBoundsException - 越界访问
$array = [1, 2, 3];
if (!isset($array[10])) {
    throw new OutOfBoundsException('Index out of bounds');
}

// DomainException - 领域错误
function processOrder(Order $order): void
{
    if ($order->getStatus() !== 'pending') {
        throw new DomainException('Only pending orders can be processed');
    }
}

// LengthException - 长度错误
function validateLength(string $input, int $max): void
{
    if (strlen($input) > $max) {
        throw new LengthException("Input exceeds maximum length of {$max}");
    }
}
```

## 自定义异常层次设计

### 设计原则

设计异常层次时应遵循以下原则：

1. **层次清晰**：异常应该按照业务领域组织
2. **粒度适中**：不要过度细分，也不要过于笼统
3. **继承合理**：子类应该能够替换父类
4. **命名规范**：异常名称应该清晰表达含义

### 基础异常类设计

```php
<?php
declare(strict_types=1);

// 应用基础异常
abstract class AppException extends Exception
{
    protected string $errorCode;
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
    
    public function getErrorCode(): string
    {
        return $this->errorCode;
    }
    
    public function getContext(): array
    {
        return $this->context;
    }
}

// 业务异常基类
abstract class BusinessException extends AppException
{
    // 业务异常用于表达业务逻辑错误
}

// 系统异常基类
abstract class SystemException extends AppException
{
    // 系统异常用于表达底层系统错误
}

// 验证异常
class ValidationException extends BusinessException
{
    protected string $errorCode = 'VALIDATION_ERROR';
    
    private array $errors = [];
    
    public function __construct(
        string $message = 'Validation failed',
        array $errors = [],
        int $code = 0,
        ?Throwable $previous = null
    ) {
        parent::__construct($message, $code, $previous, ['errors' => $errors]);
        $this->errors = $errors;
    }
    
    public function getErrors(): array
    {
        return $this->errors;
    }
}

// 资源不存在异常
class ResourceNotFoundException extends BusinessException
{
    protected string $errorCode = 'RESOURCE_NOT_FOUND';
    
    private string $resourceType;
    private mixed $resourceId;
    
    public function __construct(
        string $resourceType,
        mixed $resourceId,
        string $message = '',
        int $code = 0,
        ?Throwable $previous = null
    ) {
        $message = $message ?: "{$resourceType} with ID {$resourceId} not found";
        parent::__construct(
            $message, 
            $code, 
            $previous,
            ['resource_type' => $resourceType, 'resource_id' => $resourceId]
        );
        $this->resourceType = $resourceType;
        $this->resourceId = $resourceId;
    }
    
    public function getResourceType(): string
    {
        return $this->resourceType;
    }
    
    public function getResourceId(): mixed
    {
        return $this->resourceId;
    }
}

// 权限异常
class AuthorizationException extends BusinessException
{
    protected string $errorCode = 'AUTHORIZATION_ERROR';
    
    private ?string $requiredPermission;
    private ?string $currentPermission;
    
    public function __construct(
        string $message = 'Access denied',
        ?string $requiredPermission = null,
        ?string $currentPermission = null,
        int $code = 403,
        ?Throwable $previous = null
    ) {
        parent::__construct(
            $message,
            $code,
            $previous,
            [
                'required_permission' => $requiredPermission,
                'current_permission' => $currentPermission
            ]
        );
        $this->requiredPermission = $requiredPermission;
        $this->currentPermission = $currentPermission;
    }
}

// 数据库异常
class DatabaseException extends SystemException
{
    protected string $errorCode = 'DATABASE_ERROR';
    
    private ?string $query;
    
    public function __construct(
        string $message = 'Database operation failed',
        ?string $query = null,
        int $code = 0,
        ?Throwable $previous = null
    ) {
        parent::__construct($message, $code, $previous, ['query' => $query]);
        $this->query = $query;
    }
}

// 网络异常
class NetworkException extends SystemException
{
    protected string $errorCode = 'NETWORK_ERROR';
    
    private ?string $host;
    private ?int $port;
    
    public function __construct(
        string $message = 'Network operation failed',
        ?string $host = null,
        ?int $port = null,
        int $code = 0,
        ?Throwable $previous = null
    ) {
        parent::__construct($message, $code, $previous, [
            'host' => $host,
            'port' => $port
        ]);
        $this->host = $host;
        $this->port = $port;
    }
}
```

### 完整异常层次示例

```php
<?php
declare(strict_types=1);

// 完整的异常体系结构

// 1. 应用基础异常
abstract class AppException extends Exception
{
    protected string $errorCode = 'APP_ERROR';
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
    
    public function getErrorCode(): string { return $this->errorCode; }
    public function getContext(): array { return $this->context; }
}

// 2. 业务异常
abstract class BusinessException extends AppException
{
    protected string $errorCode = 'BUSINESS_ERROR';
}

// 3. 验证异常
class ValidationException extends BusinessException
{
    protected string $errorCode = 'VALIDATION_ERROR';
    private array $errors = [];
    
    public function __construct(string $message = 'Validation failed', array $errors = [])
    {
        parent::__construct($message, 422, null, ['errors' => $errors]);
        $this->errors = $errors;
    }
    
    public function getErrors(): array { return $this->errors; }
}

// 4. 资源异常
class ResourceNotFoundException extends BusinessException
{
    protected string $errorCode = 'NOT_FOUND';
    private string $resource;
    
    public function __construct(string $resource, $id)
    {
        parent::__construct("{$resource} #{$id} not found", 404, null, [
            'resource' => $resource,
            'id' => $id
        ]);
        $this->resource = $resource;
    }
}

class DuplicateResourceException extends BusinessException
{
    protected string $errorCode = 'DUPLICATE_RESOURCE';
    private string $resource;
    
    public function __construct(string $resource, string $field, $value)
    {
        parent::__construct("{$resource} with {$field}={$value} already exists", 409);
        $this->resource = $resource;
    }
}

// 5. 权限异常
class AuthorizationException extends BusinessException
{
    protected string $errorCode = 'FORBIDDEN';
}

class AuthenticationException extends BusinessException
{
    protected string $errorCode = 'UNAUTHORIZED';
}

// 6. 系统异常
abstract class SystemException extends AppException
{
    protected string $errorCode = 'SYSTEM_ERROR';
}

class DatabaseException extends SystemException
{
    protected string $errorCode = 'DATABASE_ERROR';
}

class ExternalServiceException extends SystemException
{
    protected string $errorCode = 'EXTERNAL_SERVICE_ERROR';
    private string $serviceName;
    
    public function __construct(string $serviceName, string $message)
    {
        parent::__construct("{$serviceName}: {$message}", 503);
        $this->serviceName = $serviceName;
    }
}
```

## 异常分类方法

### 按严重程度分类

```php
<?php
declare(strict_types=1);

// 致命异常 - 无法恢复
class FatalException extends AppException
{
    // 需要立即终止程序
}

// 可恢复异常 - 可以尝试恢复
class RecoverableException extends AppException
{
    // 可以尝试处理或回退
}

// 警告性异常 - 不影响程序运行
class WarningException extends AppException
{
    // 仅作为警告，不影响主流程
}
```

### 按业务领域分类

```php
<?php
declare(strict_types=1);

// 用户模块异常
namespace App\Modules\User\Exceptions;

class UserException extends BusinessException {}
class UserNotFoundException extends UserException {}
class UserAlreadyExistsException extends UserException {}
class InvalidUserDataException extends UserException {}

// 订单模块异常
namespace App\Modules\Order\Exceptions;

class OrderException extends BusinessException {}
class OrderNotFoundException extends OrderException {}
class OrderProcessingException extends OrderException {}
class InsufficientStockException extends OrderException {}

// 支付模块异常
namespace App\Modules\Payment\Exceptions;

class PaymentException extends BusinessException {}
class PaymentFailedException extends PaymentException {}
class InvalidPaymentMethodException extends PaymentException {}
```

## 使用示例

```php
<?php
declare(strict_types=1);

// 使用自定义异常体系

class UserService
{
    private UserRepository $repository;
    
    public function __construct(UserRepository $repository)
    {
        $this->repository = $repository;
    }
    
    public function getUser(int $id): User
    {
        $user = $this->repository->findById($id);
        
        if ($user === null) {
            throw new ResourceNotFoundException('User', $id);
        }
        
        return $user;
    }
    
    public function createUser(array $data): User
    {
        // 验证数据
        $errors = $this->validate($data);
        if (!empty($errors)) {
            throw new ValidationException('Invalid user data', $errors);
        }
        
        // 检查重复
        if ($this->repository->findByEmail($data['email']) !== null) {
            throw new DuplicateResourceException('User', 'email', $data['email']);
        }
        
        // 创建用户
        $user = new User($data['name'], $data['email']);
        $this->repository->save($user);
        
        return $user;
    }
    
    private function validate(array $data): array
    {
        $errors = [];
        
        if (empty($data['name'])) {
            $errors['name'] = 'Name is required';
        }
        
        if (empty($data['email'])) {
            $errors['email'] = 'Email is required';
        } elseif (!filter_var($data['email'], FILTER_VALIDATE_EMAIL)) {
            $errors['email'] = 'Invalid email format';
        }
        
        return $errors;
    }
}
```

## 练习任务

1. **设计异常体系**：为一个电商系统设计完整的异常体系

2. **重构异常处理**：将使用错误码的代码重构为使用异常

3. **创建自定义异常**：为你的项目创建自定义异常类

4. **实现异常工厂**：设计一个异常工厂来统一创建异常
