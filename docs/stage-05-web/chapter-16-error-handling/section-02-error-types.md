# 5.16.2 错误分类与自定义错误

## 概述

错误分类是构建完善错误处理系统的基础。理解错误分类方法、掌握自定义错误类的设计、实现错误层次和错误码定义，对于构建清晰、可维护的错误处理系统至关重要。本节详细介绍错误分类、自定义错误类、错误层次、错误码定义等内容，帮助零基础学员掌握错误分类和自定义错误的设计。

合理的错误分类可以提高错误处理的效率和准确性。理解错误分类方法对于设计完善的错误处理系统至关重要。

**主要内容**：
- 错误分类（错误类型划分、错误级别、错误来源、错误影响）
- 自定义错误类（错误类设计、错误类继承、错误类属性、错误类方法）
- 错误层次（错误层次结构、错误继承关系、错误组织）
- 错误码定义（错误码规范、错误码分类、错误码映射、错误码文档）
- 实际应用示例和最佳实践

## 特性

- **清晰分类**：清晰的错误分类体系
- **易于扩展**：易于扩展新错误类型
- **标准化**：遵循标准错误码
- **可维护**：易于维护和管理
- **可追踪**：支持错误追踪

## 错误分类

### 错误类型划分

**主要类型**：
```php
<?php
declare(strict_types=1);

enum ErrorType: string
{
    case VALIDATION = 'validation';      // 验证错误
    case AUTHENTICATION = 'authentication';  // 认证错误
    case AUTHORIZATION = 'authorization';    // 授权错误
    case NOT_FOUND = 'not_found';        // 资源不存在
    case BUSINESS = 'business';          // 业务错误
    case SYSTEM = 'system';              // 系统错误
    case NETWORK = 'network';            // 网络错误
    case DATABASE = 'database';          // 数据库错误
}
```

### 错误级别

**示例**：
```php
<?php
declare(strict_types=1);

enum ErrorLevel: int
{
    case DEBUG = 100;
    case INFO = 200;
    case NOTICE = 250;
    case WARNING = 300;
    case ERROR = 400;
    case CRITICAL = 500;
    case ALERT = 550;
    case EMERGENCY = 600;
}
```

### 错误来源

**示例**：
```php
<?php
declare(strict_types=1);

enum ErrorSource: string
{
    case CLIENT = 'client';      // 客户端错误
    case SERVER = 'server';      // 服务器错误
    case DATABASE = 'database';  // 数据库错误
    case EXTERNAL = 'external';  // 外部服务错误
    case INTERNAL = 'internal'; // 内部错误
}
```

### 错误影响

**示例**：
```php
<?php
declare(strict_types=1);

enum ErrorImpact: string
{
    case LOW = 'low';        // 低影响
    case MEDIUM = 'medium';  // 中等影响
    case HIGH = 'high';      // 高影响
    case CRITICAL = 'critical';  // 严重影响
}
```

## 自定义错误类

### 错误类设计

**示例**：
```php
<?php
declare(strict_types=1);

abstract class AppException extends \Exception
{
    protected int $statusCode;
    protected string $errorCode;
    protected array $details;

    public function __construct(
        string $message,
        int $statusCode = 500,
        string $errorCode = 'UNKNOWN_ERROR',
        array $details = [],
        \Throwable $previous = null
    ) {
        parent::__construct($message, 0, $previous);
        $this->statusCode = $statusCode;
        $this->errorCode = $errorCode;
        $this->details = $details;
    }

    public function getStatusCode(): int
    {
        return $this->statusCode;
    }

    public function getErrorCode(): string
    {
        return $this->errorCode;
    }

    public function getDetails(): array
    {
        return $this->details;
    }

    public function toArray(): array
    {
        return [
            'error' => $this->getMessage(),
            'code' => $this->errorCode,
            'status' => $this->statusCode,
            'details' => $this->details,
        ];
    }
}
```

### 错误类继承

**示例**：
```php
<?php
declare(strict_types=1);

class ValidationException extends AppException
{
    public function __construct(
        string $message = '数据验证失败',
        array $errors = [],
        \Throwable $previous = null
    ) {
        parent::__construct(
            $message,
            422,
            'VALIDATION_ERROR',
            ['validation_errors' => $errors],
            $previous
        );
    }
}

class AuthenticationException extends AppException
{
    public function __construct(
        string $message = '未认证',
        \Throwable $previous = null
    ) {
        parent::__construct(
            $message,
            401,
            'AUTHENTICATION_ERROR',
            [],
            $previous
        );
    }
}

class AuthorizationException extends AppException
{
    public function __construct(
        string $message = '无权限',
        \Throwable $previous = null
    ) {
        parent::__construct(
            $message,
            403,
            'AUTHORIZATION_ERROR',
            [],
            $previous
        );
    }
}

class NotFoundException extends AppException
{
    public function __construct(
        string $message = '资源不存在',
        \Throwable $previous = null
    ) {
        parent::__construct(
            $message,
            404,
            'NOT_FOUND',
            [],
            $previous
        );
    }
}
```

### 错误类属性

**示例**：
```php
<?php
declare(strict_types=1);

class BusinessException extends AppException
{
    public function __construct(
        string $message,
        private string $businessCode,
        private array $context = [],
        \Throwable $previous = null
    ) {
        parent::__construct(
            $message,
            400,
            'BUSINESS_ERROR',
            ['business_code' => $businessCode, 'context' => $context],
            $previous
        );
    }

    public function getBusinessCode(): string
    {
        return $this->businessCode;
    }

    public function getContext(): array
    {
        return $this->context;
    }
}
```

### 错误类方法

**示例**：
```php
<?php
declare(strict_types=1);

class RateLimitException extends AppException
{
    public function __construct(
        string $message = '请求过于频繁',
        private int $retryAfter = 60,
        \Throwable $previous = null
    ) {
        parent::__construct(
            $message,
            429,
            'RATE_LIMIT_EXCEEDED',
            ['retry_after' => $retryAfter],
            $previous
        );
    }

    public function getRetryAfter(): int
    {
        return $this->retryAfter;
    }

    public function getRetryAfterHeader(): string
    {
        return (string) $this->retryAfter;
    }
}
```

## 错误层次

### 错误层次结构

**示例**：
```php
<?php
declare(strict_types=1);

// 基础异常类
abstract class AppException extends \Exception {}

// 客户端错误
abstract class ClientException extends AppException
{
    protected int $statusCode = 400;
}

class ValidationException extends ClientException {}
class AuthenticationException extends ClientException
{
    protected int $statusCode = 401;
}
class AuthorizationException extends ClientException
{
    protected int $statusCode = 403;
}
class NotFoundException extends ClientException
{
    protected int $statusCode = 404;
}

// 服务器错误
abstract class ServerException extends AppException
{
    protected int $statusCode = 500;
}

class DatabaseException extends ServerException {}
class ExternalServiceException extends ServerException {}
class SystemException extends ServerException {}
```

### 错误继承关系

**示例**：
```php
<?php
declare(strict_types=1);

// 错误继承层次
Exception
  └── AppException
       ├── ClientException
       │    ├── ValidationException
       │    ├── AuthenticationException
       │    ├── AuthorizationException
       │    └── NotFoundException
       └── ServerException
            ├── DatabaseException
            ├── ExternalServiceException
            └── SystemException
```

### 错误组织

**示例**：
```php
<?php
declare(strict_types=1);

namespace App\Exceptions;

// 基础异常
abstract class AppException extends \Exception {}

// 客户端异常
namespace App\Exceptions\Client;

class ValidationException extends \App\Exceptions\AppException {}
class AuthenticationException extends \App\Exceptions\AppException {}
class AuthorizationException extends \App\Exceptions\AppException {}

// 服务器异常
namespace App\Exceptions\Server;

class DatabaseException extends \App\Exceptions\AppException {}
class SystemException extends \App\Exceptions\AppException {}
```

## 错误码定义

### 错误码规范

**示例**：
```php
<?php
declare(strict_types=1);

class ErrorCodes
{
    // 验证错误 (1xxx)
    public const VALIDATION_REQUIRED = '1001';
    public const VALIDATION_FORMAT = '1002';
    public const VALIDATION_RANGE = '1003';

    // 认证错误 (2xxx)
    public const AUTH_INVALID_CREDENTIALS = '2001';
    public const AUTH_TOKEN_EXPIRED = '2002';
    public const AUTH_TOKEN_INVALID = '2003';

    // 授权错误 (3xxx)
    public const AUTHZ_INSUFFICIENT_PERMISSIONS = '3001';
    public const AUTHZ_ROLE_REQUIRED = '3002';

    // 资源错误 (4xxx)
    public const RESOURCE_NOT_FOUND = '4001';
    public const RESOURCE_CONFLICT = '4002';

    // 业务错误 (5xxx)
    public const BUSINESS_INVALID_STATE = '5001';
    public const BUSINESS_OPERATION_FAILED = '5002';

    // 系统错误 (9xxx)
    public const SYSTEM_INTERNAL_ERROR = '9001';
    public const SYSTEM_DATABASE_ERROR = '9002';
    public const SYSTEM_EXTERNAL_SERVICE_ERROR = '9003';
}
```

### 错误码分类

**示例**：
```php
<?php
declare(strict_types=1);

class ErrorCodeManager
{
    private array $errorCodes = [
        'validation' => [
            'required' => '1001',
            'format' => '1002',
            'range' => '1003',
        ],
        'authentication' => [
            'invalid_credentials' => '2001',
            'token_expired' => '2002',
            'token_invalid' => '2003',
        ],
        'authorization' => [
            'insufficient_permissions' => '3001',
            'role_required' => '3002',
        ],
    ];

    public function getErrorCode(string $category, string $type): ?string
    {
        return $this->errorCodes[$category][$type] ?? null;
    }

    public function getErrorMessage(string $code): string
    {
        $messages = [
            '1001' => '字段是必填项',
            '1002' => '字段格式不正确',
            '2001' => '用户名或密码错误',
            '2002' => 'Token 已过期',
        ];
        
        return $messages[$code] ?? '未知错误';
    }
}
```

### 错误码映射

**示例**：
```php
<?php
declare(strict_types=1);

class ErrorCodeMapper
{
    public function mapExceptionToCode(\Exception $e): string
    {
        return match (get_class($e)) {
            ValidationException::class => ErrorCodes::VALIDATION_REQUIRED,
            AuthenticationException::class => ErrorCodes::AUTH_INVALID_CREDENTIALS,
            AuthorizationException::class => ErrorCodes::AUTHZ_INSUFFICIENT_PERMISSIONS,
            NotFoundException::class => ErrorCodes::RESOURCE_NOT_FOUND,
            default => ErrorCodes::SYSTEM_INTERNAL_ERROR,
        };
    }

    public function mapCodeToMessage(string $code): string
    {
        $messages = [
            ErrorCodes::VALIDATION_REQUIRED => '字段是必填项',
            ErrorCodes::AUTH_INVALID_CREDENTIALS => '认证失败',
            ErrorCodes::AUTHZ_INSUFFICIENT_PERMISSIONS => '权限不足',
            ErrorCodes::RESOURCE_NOT_FOUND => '资源不存在',
        ];
        
        return $messages[$code] ?? '未知错误';
    }
}
```

### 错误码文档

**示例**：
```php
<?php
declare(strict_types=1);

/**
 * 错误码文档
 * 
 * 错误码格式：XXXX
 * - 1xxx: 验证错误
 * - 2xxx: 认证错误
 * - 3xxx: 授权错误
 * - 4xxx: 资源错误
 * - 5xxx: 业务错误
 * - 9xxx: 系统错误
 */
class ErrorCodeDocumentation
{
    public static function getAllCodes(): array
    {
        return [
            'validation' => [
                '1001' => '字段是必填项',
                '1002' => '字段格式不正确',
                '1003' => '字段值超出范围',
            ],
            'authentication' => [
                '2001' => '用户名或密码错误',
                '2002' => 'Token 已过期',
                '2003' => 'Token 无效',
            ],
        ];
    }
}
```

## 使用场景

### 所有 Web 应用

- 错误分类和处理
- 错误码定义
- 错误追踪

### API 错误处理

- API 错误码
- 错误响应格式
- 错误文档

### 业务错误处理

- 业务错误分类
- 业务错误码
- 业务错误处理

### 系统错误处理

- 系统错误分类
- 系统错误码
- 系统错误追踪

## 注意事项

### 错误分类设计

- **清晰分类**：错误分类要清晰
- **合理层次**：合理的错误层次
- **易于扩展**：易于扩展新错误类型

### 错误类设计

- **单一职责**：每个错误类只代表一种错误
- **清晰属性**：清晰的错误属性
- **易于使用**：易于创建和使用

### 错误码规范

- **统一规范**：统一的错误码规范
- **易于理解**：错误码易于理解
- **文档完善**：完善的错误码文档

### 错误追踪

- **错误 ID**：使用错误 ID 追踪
- **错误上下文**：记录错误上下文
- **错误分析**：定期分析错误

## 常见问题

### 如何分类错误？

按错误类型、错误级别、错误来源、错误影响等维度分类。

### 如何设计自定义错误类？

继承基础异常类，定义错误属性，实现错误方法。

### 如何定义错误码？

使用统一的错误码规范，按类别组织错误码。

### 如何组织错误层次？

使用继承关系组织错误层次，保持层次清晰。

## 最佳实践

### 清晰的错误分类

- 按错误类型分类
- 使用错误层次
- 保持分类清晰

### 合理的错误类设计

- 单一职责原则
- 清晰的错误属性
- 易于使用和维护

### 统一的错误码规范

- 统一的错误码格式
- 按类别组织错误码
- 完善的错误码文档

### 完善的错误追踪

- 使用错误 ID
- 记录错误上下文
- 支持错误分析

## 相关章节

- **[5.16.1 错误处理概述](section-01-overview.md)**：了解错误处理概述的详细内容
- **[5.16.3 错误中间件](section-03-error-middleware.md)**：了解错误中间件的详细内容
- **[5.16.4 错误响应格式](section-04-response-format.md)**：了解错误响应格式的详细内容

## 练习任务

1. **实现错误分类系统**
   - 错误类型定义
   - 错误级别定义
   - 错误分类管理

2. **实现自定义错误类**
   - 基础错误类
   - 具体错误类
   - 错误类继承

3. **实现错误码系统**
   - 错误码定义
   - 错误码映射
   - 错误码文档

4. **实现错误层次结构**
   - 错误继承关系
   - 错误组织
   - 错误管理

5. **实现完整的错误分类系统**
   - 错误分类和处理
   - 错误码和文档
   - 错误追踪和分析
