# 7.4.2 自定义异常

## 概述

自定义异常是表达特定业务逻辑错误的关键工具。一个设计良好的自定义异常应该清晰地表达错误的性质、包含必要的上下文信息、并且便于调试和处理。本节将详细介绍如何设计有意义的自定义异常，包括异常属性、异常消息、异常上下文等方面。

**主要内容**：
- 自定义异常的设计原则
- 异常属性的设计
- 异常消息的设计
- 异常上下文信息的包含

## 自定义异常设计原则

### 1. 异常应该是具体的

```php
<?php
declare(strict_types=1);

// 不好的设计 - 太笼统
class ServiceException extends Exception {}

// 好的设计 - 具体明确
class UserNotFoundException extends Exception {}
class InvalidCredentialsException extends Exception {}
class UserAlreadyExistsException extends Exception {}
class InsufficientPermissionsException extends Exception {}
```

### 2. 异常应该包含上下文

```php
<?php
declare(strict_types=1);

// 包含上下文的异常设计
class ResourceNotFoundException extends Exception
{
    private string $resourceType;
    private mixed $resourceId;
    
    public function __construct(
        string $resourceType,
        mixed $resourceId,
        string $message = '',
        ?Throwable $previous = null
    ) {
        $this->resourceType = $resourceType;
        $this->resourceId = $resourceId;
        $message = $message ?: "{$resourceType} #{$resourceId} not found";
        parent::__construct($message, 404, $previous);
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

// 使用
throw new ResourceNotFoundException('User', 123);
// 异常消息: "User #123 not found"
// 可以通过 getResourceType() 和 getResourceId() 获取详细信息
```

### 3. 异常应该可序列化

```php
<?php
declare(strict_types=1);

// 可序列化的异常
class DetailedException extends Exception
{
    private array $context;
    private string $errorCode;
    
    public function __construct(
        string $message,
        string $errorCode,
        array $context = [],
        int $code = 0,
        ?Throwable $previous = null
    ) {
        parent::__construct($message, $code, $previous);
        $this->errorCode = $errorCode;
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
    
    public function __serialize(): array
    {
        return [
            'message' => $this->message,
            'code' => $this->code,
            'errorCode' => $this->errorCode,
            'context' => $this->context,
            'file' => $this->file,
            'line' => $this->line,
        ];
    }
    
    public function __unserialize(array $data): void
    {
        $this->message = $data['message'];
        $this->code = $data['code'];
        $this->errorCode = $data['errorCode'];
        $this->context = $data['context'];
        $this->file = $data['file'];
        $this->line = $data['line'];
    }
}
```

## 异常属性设计

### 错误码属性

```php
<?php
declare(strict_types=1);

// 错误码枚举
enum ErrorCode: string
{
    case VALIDATION_ERROR = 'VALIDATION_ERROR';
    case NOT_FOUND = 'NOT_FOUND';
    case DUPLICATE = 'DUPLICATE_ERROR';
    case AUTHENTICATION_ERROR = 'AUTH_ERROR';
    case AUTHORIZATION_ERROR = 'AUTHZ_ERROR';
    case DATABASE_ERROR = 'DB_ERROR';
    case EXTERNAL_SERVICE_ERROR = 'EXTERNAL_ERROR';
}

// 带错误码的异常
class CodeException extends Exception
{
    private ErrorCode $errorCode;
    
    public function __construct(
        ErrorCode $errorCode,
        string $message,
        ?Throwable $previous = null
    ) {
        $this->errorCode = $errorCode;
        parent::__construct($message, 0, $previous);
    }
    
    public function getErrorCode(): ErrorCode
    {
        return $this->errorCode;
    }
}

// 使用
throw new CodeException(
    ErrorCode::VALIDATION_ERROR,
    'Invalid email format'
);
```

### 验证错误属性

```php
<?php
declare(strict_types=1);

// 验证异常 - 包含详细的验证错误列表
class ValidationException extends Exception
{
    private array $errors;
    private array $validatedData;
    
    public function __construct(
        string $message = 'Validation failed',
        array $errors = [],
        array $validatedData = []
    ) {
        parent::__construct($message, 422);
        $this->errors = $errors;
        $this->validatedData = $validatedData;
    }
    
    public function getErrors(): array
    {
        return $this->errors;
    }
    
    public function getValidatedData(): array
    {
        return $this->validatedData;
    }
    
    public function getFirstError(): ?string
    {
        return $this->errors[0] ?? null;
    }
    
    public function hasError(string $field): bool
    {
        return isset($this->errors[$field]);
    }
    
    public function getFieldError(string $field): ?string
    {
        return $this->errors[$field] ?? null;
    }
}

// 使用
$errors = [
    'email' => 'Invalid email format',
    'name' => 'Name is required',
    'age' => 'Age must be greater than 18'
];

throw new ValidationException('Please fix the validation errors', $errors, [
    'email' => 'invalid-email',
    'name' => '',
    'age' => 15
]);

// 捕获后可以获取详细信息
try {
    // ... validation code
} catch (ValidationException $e) {
    echo $e->getMessage(); // "Please fix the validation errors"
    echo $e->hasError('email'); // true
    echo $e->getFieldError('email'); // "Invalid email format"
}
```

### HTTP 状态码属性

```php
<?php
declare(strict_types=1);

// HTTP 异常 - 包含 HTTP 状态码
class HttpException extends Exception
{
    private int $statusCode;
    private array $headers;
    
    public function __construct(
        int $statusCode,
        string $message = '',
        array $headers = [],
        ?Throwable $previous = null
    ) {
        $this->statusCode = $statusCode;
        $this->headers = $headers;
        parent::__construct($message ?: self::getDefaultMessage($statusCode), $statusCode, $previous);
    }
    
    public function getStatusCode(): int
    {
        return $this->statusCode;
    }
    
    public function getHeaders(): array
    {
        return $this->headers;
    }
    
    private static function getDefaultMessage(int $statusCode): string
    {
        return match($statusCode) {
            400 => 'Bad Request',
            401 => 'Unauthorized',
            403 => 'Forbidden',
            404 => 'Not Found',
            422 => 'Unprocessable Entity',
            500 => 'Internal Server Error',
            default => 'Error'
        };
    }
}

// 便捷方法
class NotFoundException extends HttpException
{
    public function __construct(string $message = 'Resource not found')
    {
        parent::__construct(404, $message);
    }
}

class ForbiddenException extends HttpException
{
    public function __construct(string $message = 'Access forbidden')
    {
        parent::__construct(403, $message);
    }
}

class UnauthorizedException extends HttpException
{
    public function __construct(string $message = 'Authentication required')
    {
        parent::__construct(401, $message);
    }
}
```

## 异常消息设计

### 消息模板

```php
<?php
declare(strict_types=1);

// 可配置的异常消息
class TemplatedException extends Exception
{
    private string $template;
    private array $params;
    
    public function __construct(
        string $template,
        array $params = [],
        ?Throwable $previous = null
    ) {
        $this->template = $template;
        $this->params = $params;
        $message = $this->buildMessage();
        parent::__construct($message, 0, $previous);
    }
    
    private function buildMessage(): string
    {
        return strtr($this->template, array_combine(
            array_map(fn($k) => '{{' . $k . '}}', array_keys($this->params)),
            array_values($this->params)
        ));
    }
}

// 使用
throw new TemplatedException(
    'User {{user_id}} not found in system {{system_id}}',
    ['user_id' => 123, 'system_id' => 'PROD']
);
// 消息: "User 123 not found in system PROD"
```

### 多语言支持

```php
<?php
declare(strict_types=1);

// 可翻译的异常
class TranslatableException extends Exception
{
    private string $messageKey;
    private array $params;
    private string $locale;
    
    private static array $messages = [
        'en' => [
            'validation_failed' => 'Validation failed: {{errors}}',
            'not_found' => '{{resource}} #{{id}} not found',
            'duplicate' => '{{resource}} with {{field}}={{value}} already exists'
        ],
        'zh' => [
            'validation_failed' => '验证失败: {{errors}}',
            'not_found' => '{{resource}} #{{id}} 不存在',
            'duplicate' => '{{resource}} 中 {{field}}={{value}} 已存在'
        ]
    ];
    
    public function __construct(
        string $messageKey,
        array $params = [],
        string $locale = 'en'
    ) {
        $this->messageKey = $messageKey;
        $this->params = $params;
        $this->locale = $locale;
        
        $message = self::$messages[$locale][$messageKey] ?? $messageKey;
        $message = strtr($message, array_combine(
            array_map(fn($k) => '{{' . $k . '}}', array_keys($params)),
            array_values($params)
        ));
        
        parent::__construct($message);
    }
}

// 使用
throw new TranslatableException('not_found', [
    'resource' => 'User',
    'id' => 123
], 'zh');
// 消息: "User #123 不存在"
```

## 完整示例

```php
<?php
declare(strict_types=1);

// 完整的自定义异常示例

// 基础异常类
abstract class AppException extends Exception
{
    protected string $errorCode;
    protected array $context = [];
    
    public function __construct(
        string $message,
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

// 业务异常
abstract class BusinessException extends AppException
{
    protected string $errorCode = 'BUSINESS_ERROR';
}

// API 异常
class ApiException extends BusinessException
{
    private int $statusCode;
    
    public function __construct(
        int $statusCode,
        string $errorCode,
        string $message,
        array $context = [],
        ?Throwable $previous = null
    ) {
        $this->statusCode = $statusCode;
        parent::__construct($message, $statusCode, $previous, $context);
    }
    
    public function getStatusCode(): int
    {
        return $this->statusCode;
    }
}

// 具体异常实现
class UserNotFoundException extends ApiException
{
    public function __construct(int $userId)
    {
        parent::__construct(
            404,
            'USER_NOT_FOUND',
            "User #{$userId} not found",
            ['user_id' => $userId]
        );
    }
}

class InsufficientBalanceException extends ApiException
{
    public function __construct(float $available, float $required)
    {
        parent::__construct(
            400,
            'INSUFFICIENT_BALANCE',
            "Insufficient balance. Available: {$available}, Required: {$required}",
            [
                'available_balance' => $available,
                'required_balance' => $required
            ]
        );
    }
}

// 使用示例
try {
    $user = $userService->getUser(123);
} catch (UserNotFoundException $e) {
    // 处理用户不存在
    http_response_code($e->getStatusCode());
    echo json_encode([
        'error' => [
            'code' => $e->getErrorCode(),
            'message' => $e->getMessage()
        ]
    ]);
}
```

## 练习任务

1. **设计自定义异常**：为支付系统设计完整的自定义异常体系

2. **添加异常属性**：为一个现有的异常类添加错误码和上下文属性

3. **实现消息模板**：设计一个支持消息模板的异常基类

4. **异常国际化**：实现一个支持多语言的异常系统
