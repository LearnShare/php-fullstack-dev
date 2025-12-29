# 5.16.4 错误响应格式

## 概述

错误响应格式的设计直接影响用户体验和 API 的易用性。理解错误响应设计原则、掌握统一格式实现、定义错误码规范、提供用户友好信息，对于构建优秀的 Web 应用和 API 至关重要。本节详细介绍错误响应设计、统一格式、错误码规范、用户友好信息等内容，帮助零基础学员设计完善的错误响应格式。

良好的错误响应格式可以提高用户体验，便于错误追踪和问题定位。理解错误响应格式的设计原则对于构建用户友好的应用至关重要。

**主要内容**：
- 错误响应设计（响应设计原则、响应结构、响应字段、响应示例）
- 统一格式（格式规范、格式实现、格式验证、格式文档）
- 错误码规范（错误码定义、错误码分类、错误码映射、错误码文档）
- 用户友好信息（友好消息设计、错误提示、错误恢复建议、多语言支持）
- 实际应用示例和最佳实践

## 特性

- **统一格式**：统一的错误响应格式
- **用户友好**：用户友好的错误信息
- **易于追踪**：支持错误追踪
- **标准化**：遵循标准规范
- **可扩展**：易于扩展新字段

## 错误响应设计

### 响应设计原则

1. **一致性**：所有错误使用统一格式
2. **清晰性**：错误信息清晰明确
3. **可操作性**：提供可操作的建议
4. **可追踪性**：支持错误追踪

### 响应结构

**示例**：
```php
<?php
declare(strict_types=1);

function buildErrorResponse(\Exception $e): array
{
    return [
        'success' => false,
        'error' => [
            'message' => $this->getMessage($e),
            'code' => $this->getCode($e),
            'status' => $this->getStatusCode($e),
            'error_id' => uniqid(),
            'timestamp' => date('c'),
        ],
        'data' => null,
    ];
}
```

### 响应字段

**标准字段**：
- `success`: 操作是否成功（boolean）
- `error`: 错误信息对象
  - `message`: 错误消息（string）
  - `code`: 错误码（string）
  - `status`: HTTP 状态码（int）
  - `error_id`: 错误 ID（string）
  - `timestamp`: 时间戳（string）
- `data`: 数据（null 或空数组）

### 响应示例

**示例**：
```php
<?php
declare(strict_types=1);

// 验证错误响应
{
    "success": false,
    "error": {
        "message": "数据验证失败",
        "code": "VALIDATION_ERROR",
        "status": 422,
        "error_id": "err_1234567890",
        "timestamp": "2024-01-01T12:00:00+00:00",
        "details": {
            "validation_errors": {
                "email": ["邮箱格式不正确"],
                "password": ["密码至少 8 个字符"]
            }
        }
    },
    "data": null
}

// 认证错误响应
{
    "success": false,
    "error": {
        "message": "请先登录",
        "code": "AUTHENTICATION_ERROR",
        "status": 401,
        "error_id": "err_1234567891",
        "timestamp": "2024-01-01T12:00:00+00:00"
    },
    "data": null
}
```

## 统一格式

### 格式规范

**格式规范**：
```php
<?php
declare(strict_types=1);

class ErrorResponseFormat
{
    public static function format(\Exception $e): array
    {
        return [
            'success' => false,
            'error' => [
                'message' => self::getMessage($e),
                'code' => self::getCode($e),
                'status' => self::getStatusCode($e),
                'error_id' => self::generateErrorId(),
                'timestamp' => date('c'),
            ],
            'data' => null,
        ];
    }

    private static function getMessage(\Exception $e): string
    {
        // 获取用户友好的错误消息
        return '操作失败';
    }

    private static function getCode(\Exception $e): string
    {
        if ($e instanceof AppException) {
            return $e->getErrorCode();
        }
        return 'INTERNAL_ERROR';
    }

    private static function getStatusCode(\Exception $e): int
    {
        if ($e instanceof AppException) {
            return $e->getStatusCode();
        }
        return 500;
    }

    private static function generateErrorId(): string
    {
        return 'err_' . uniqid();
    }
}
```

### 格式实现

**示例**：
```php
<?php
declare(strict_types=1);

class UnifiedErrorResponse
{
    public function build(\Exception $e, bool $isDevelopment = false): ResponseInterface
    {
        $data = [
            'success' => false,
            'error' => $this->buildErrorObject($e, $isDevelopment),
            'data' => null,
        ];
        
        $statusCode = $this->getStatusCode($e);
        
        return new Response(
            json_encode($data, JSON_UNESCAPED_UNICODE | JSON_PRETTY_PRINT),
            $statusCode,
            ['Content-Type' => 'application/json']
        );
    }

    private function buildErrorObject(\Exception $e, bool $isDevelopment): array
    {
        $error = [
            'message' => $this->getMessage($e),
            'code' => $this->getCode($e),
            'status' => $this->getStatusCode($e),
            'error_id' => $this->generateErrorId(),
            'timestamp' => date('c'),
        ];
        
        if ($e instanceof AppException) {
            $details = $e->getDetails();
            if (!empty($details)) {
                $error['details'] = $details;
            }
        }
        
        if ($isDevelopment) {
            $error['debug'] = [
                'message' => $e->getMessage(),
                'file' => $e->getFile(),
                'line' => $e->getLine(),
            ];
        }
        
        return $error;
    }
}
```

### 格式验证

**示例**：
```php
<?php
declare(strict_types=1);

class ErrorResponseValidator
{
    public function validate(array $response): bool
    {
        // 验证必需字段
        if (!isset($response['success']) || $response['success'] !== false) {
            return false;
        }
        
        if (!isset($response['error']) || !is_array($response['error'])) {
            return false;
        }
        
        $error = $response['error'];
        $required = ['message', 'code', 'status', 'error_id', 'timestamp'];
        
        foreach ($required as $field) {
            if (!isset($error[$field])) {
                return false;
            }
        }
        
        return true;
    }
}
```

### 格式文档

**示例**：
```php
<?php
declare(strict_types=1);

/**
 * 错误响应格式文档
 * 
 * 标准错误响应格式：
 * {
 *   "success": false,
 *   "error": {
 *     "message": "错误消息",
 *     "code": "错误码",
 *     "status": 400,
 *     "error_id": "错误ID",
 *     "timestamp": "时间戳"
 *   },
 *   "data": null
 * }
 */
class ErrorResponseDocumentation
{
    // 文档内容
}
```

## 错误码规范

### 错误码定义

**示例**：
```php
<?php
declare(strict_types=1);

class ErrorCodes
{
    // 验证错误 (1xxx)
    public const VALIDATION_REQUIRED = 'VALIDATION_REQUIRED';
    public const VALIDATION_FORMAT = 'VALIDATION_FORMAT';
    public const VALIDATION_RANGE = 'VALIDATION_RANGE';

    // 认证错误 (2xxx)
    public const AUTH_INVALID_CREDENTIALS = 'AUTH_INVALID_CREDENTIALS';
    public const AUTH_TOKEN_EXPIRED = 'AUTH_TOKEN_EXPIRED';
    public const AUTH_TOKEN_INVALID = 'AUTH_TOKEN_INVALID';

    // 授权错误 (3xxx)
    public const AUTHZ_INSUFFICIENT_PERMISSIONS = 'AUTHZ_INSUFFICIENT_PERMISSIONS';
    public const AUTHZ_ROLE_REQUIRED = 'AUTHZ_ROLE_REQUIRED';

    // 资源错误 (4xxx)
    public const RESOURCE_NOT_FOUND = 'RESOURCE_NOT_FOUND';
    public const RESOURCE_CONFLICT = 'RESOURCE_CONFLICT';

    // 业务错误 (5xxx)
    public const BUSINESS_INVALID_STATE = 'BUSINESS_INVALID_STATE';
    public const BUSINESS_OPERATION_FAILED = 'BUSINESS_OPERATION_FAILED';

    // 系统错误 (9xxx)
    public const SYSTEM_INTERNAL_ERROR = 'SYSTEM_INTERNAL_ERROR';
    public const SYSTEM_DATABASE_ERROR = 'SYSTEM_DATABASE_ERROR';
}
```

### 错误码分类

**示例**：
```php
<?php
declare(strict_types=1);

class ErrorCodeCategories
{
    public const VALIDATION = 'validation';
    public const AUTHENTICATION = 'authentication';
    public const AUTHORIZATION = 'authorization';
    public const RESOURCE = 'resource';
    public const BUSINESS = 'business';
    public const SYSTEM = 'system';

    public static function getCategory(string $code): string
    {
        if (str_starts_with($code, 'VALIDATION_')) {
            return self::VALIDATION;
        } elseif (str_starts_with($code, 'AUTH_')) {
            return self::AUTHENTICATION;
        } elseif (str_starts_with($code, 'AUTHZ_')) {
            return self::AUTHORIZATION;
        } elseif (str_starts_with($code, 'RESOURCE_')) {
            return self::RESOURCE;
        } elseif (str_starts_with($code, 'BUSINESS_')) {
            return self::BUSINESS;
        } elseif (str_starts_with($code, 'SYSTEM_')) {
            return self::SYSTEM;
        }
        
        return self::SYSTEM;
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
    private array $codeToMessage = [
        ErrorCodes::VALIDATION_REQUIRED => '字段是必填项',
        ErrorCodes::VALIDATION_FORMAT => '字段格式不正确',
        ErrorCodes::AUTH_INVALID_CREDENTIALS => '用户名或密码错误',
        ErrorCodes::AUTH_TOKEN_EXPIRED => 'Token 已过期',
        ErrorCodes::AUTHZ_INSUFFICIENT_PERMISSIONS => '权限不足',
        ErrorCodes::RESOURCE_NOT_FOUND => '资源不存在',
    ];

    public function getMessage(string $code): string
    {
        return $this->codeToMessage[$code] ?? '未知错误';
    }

    public function getStatusCode(string $code): int
    {
        return match (true) {
            str_starts_with($code, 'VALIDATION_') => 422,
            str_starts_with($code, 'AUTH_') => 401,
            str_starts_with($code, 'AUTHZ_') => 403,
            str_starts_with($code, 'RESOURCE_') => 404,
            str_starts_with($code, 'BUSINESS_') => 400,
            default => 500,
        };
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
 * 错误码格式：CATEGORY_TYPE
 * 
 * 类别：
 * - VALIDATION_: 验证错误
 * - AUTH_: 认证错误
 * - AUTHZ_: 授权错误
 * - RESOURCE_: 资源错误
 * - BUSINESS_: 业务错误
 * - SYSTEM_: 系统错误
 */
class ErrorCodeDocumentation
{
    public static function getAllCodes(): array
    {
        return [
            'validation' => [
                'VALIDATION_REQUIRED' => [
                    'message' => '字段是必填项',
                    'status' => 422,
                ],
                'VALIDATION_FORMAT' => [
                    'message' => '字段格式不正确',
                    'status' => 422,
                ],
            ],
            'authentication' => [
                'AUTH_INVALID_CREDENTIALS' => [
                    'message' => '用户名或密码错误',
                    'status' => 401,
                ],
            ],
        ];
    }
}
```

## 用户友好信息

### 友好消息设计

**示例**：
```php
<?php
declare(strict_types=1);

class UserFriendlyMessages
{
    private array $messages = [
        ValidationException::class => '数据验证失败，请检查输入',
        AuthenticationException::class => '请先登录后再试',
        AuthorizationException::class => '您没有权限执行此操作',
        NotFoundException::class => '请求的资源不存在',
        RateLimitException::class => '请求过于频繁，请稍后再试',
    ];

    public function getMessage(\Exception $e): string
    {
        $class = get_class($e);
        
        if (isset($this->messages[$class])) {
            return $this->messages[$class];
        }
        
        if ($e instanceof AppException) {
            return $e->getMessage();
        }
        
        return '操作失败，请稍后重试';
    }
}
```

### 错误提示

**示例**：
```php
<?php
declare(strict_types=1);

class ErrorHintProvider
{
    public function getHint(\Exception $e): ?string
    {
        return match (get_class($e)) {
            ValidationException::class => '请检查所有必填字段是否已填写，格式是否正确',
            AuthenticationException::class => '请检查用户名和密码是否正确，或重新登录',
            AuthorizationException::class => '请联系管理员获取相应权限',
            NotFoundException::class => '请检查 URL 是否正确，或资源可能已被删除',
            RateLimitException::class => '请等待一段时间后再试',
            default => null,
        };
    }
}
```

### 错误恢复建议

**示例**：
```php
<?php
declare(strict_types=1);

class ErrorRecoverySuggestion
{
    public function getSuggestion(\Exception $e): ?array
    {
        return match (get_class($e)) {
            ValidationException::class => [
                'action' => '检查输入',
                'steps' => [
                    '检查所有必填字段',
                    '验证字段格式',
                    '检查字段值范围',
                ],
            ],
            AuthenticationException::class => [
                'action' => '重新登录',
                'steps' => [
                    '检查用户名和密码',
                    '点击登录按钮',
                    '如果忘记密码，使用找回密码功能',
                ],
            ],
            default => null,
        };
    }
}
```

### 多语言支持

**示例**：
```php
<?php
declare(strict_types=1);

class LocalizedErrorMessages
{
    private array $messages = [
        'zh_CN' => [
            'VALIDATION_REQUIRED' => '字段是必填项',
            'AUTH_INVALID_CREDENTIALS' => '用户名或密码错误',
        ],
        'en_US' => [
            'VALIDATION_REQUIRED' => 'Field is required',
            'AUTH_INVALID_CREDENTIALS' => 'Invalid credentials',
        ],
    ];

    public function getMessage(string $code, string $locale = 'zh_CN'): string
    {
        return $this->messages[$locale][$code] ?? $code;
    }
}
```

## 使用场景

### 所有 Web 应用

- 错误响应格式化
- 错误消息本地化
- 错误追踪

### API 错误响应

- API 错误格式
- 错误码规范
- 错误文档

### 用户错误提示

- 用户友好错误
- 错误恢复建议
- 错误帮助

### 错误追踪

- 错误 ID 追踪
- 错误分析
- 错误报告

## 注意事项

### 响应格式统一

- **统一格式**：所有错误使用统一格式
- **字段一致**：保持字段一致性
- **格式验证**：验证响应格式

### 错误码规范

- **统一规范**：统一的错误码规范
- **易于理解**：错误码易于理解
- **文档完善**：完善的错误码文档

### 用户友好信息

- **友好语言**：使用用户友好的语言
- **清晰明确**：错误信息清晰明确
- **可操作**：提供可操作的建议

### 多语言支持

- **本地化**：支持多语言
- **语言检测**：自动检测语言
- **语言切换**：支持语言切换

## 常见问题

### 如何设计错误响应格式？

使用统一的响应结构，包含 success、error、data 字段。

### 如何定义错误码？

使用统一的错误码规范，按类别组织错误码。

### 如何提供用户友好信息？

使用用户友好的语言，提供清晰的错误消息和恢复建议。

### 如何支持多语言？

实现多语言消息映射，根据用户语言返回相应消息。

## 最佳实践

### 统一响应格式

- 所有错误使用统一格式
- 保持字段一致性
- 验证响应格式

### 清晰的错误码规范

- 统一的错误码格式
- 按类别组织错误码
- 完善的错误码文档

### 用户友好的错误信息

- 使用用户友好的语言
- 提供清晰的错误消息
- 提供错误恢复建议

### 支持错误追踪

- 使用错误 ID
- 记录错误上下文
- 支持错误分析

## 相关章节

- **[5.16.1 错误处理概述](section-01-overview.md)**：了解错误处理概述的详细内容
- **[5.16.2 错误分类与自定义错误](section-02-error-types.md)**：了解错误分类的详细内容
- **[5.16.3 错误中间件](section-03-error-middleware.md)**：了解错误中间件的详细内容
- **[5.8.1 响应处理](../chapter-08-response-cors/section-01-response-handling.md)**：了解响应处理的详细内容

## 练习任务

1. **设计错误响应格式**
   - 响应结构设计
   - 响应字段定义
   - 响应示例

2. **实现统一格式**
   - 格式实现
   - 格式验证
   - 格式文档

3. **实现错误码系统**
   - 错误码定义
   - 错误码映射
   - 错误码文档

4. **实现用户友好信息**
   - 友好消息设计
   - 错误提示
   - 多语言支持

5. **实现完整的错误响应系统**
   - 统一响应格式
   - 错误码和文档
   - 用户友好信息
