# 5.16.3 错误中间件

## 概述

错误中间件是统一处理应用错误的重要机制。理解错误中间件的概念、掌握异常捕获和错误格式化、实现错误响应，对于构建健壮的错误处理系统至关重要。本节详细介绍错误中间件概念、异常捕获、错误格式化、错误响应等内容，帮助零基础学员掌握错误中间件的实现。

错误中间件可以统一捕获和处理应用中的错误，提供一致的错误响应格式。理解错误中间件的实现原理对于构建完善的错误处理系统至关重要。

**主要内容**：
- 错误中间件概念（什么是错误中间件、错误中间件的作用、错误中间件的优势）
- 异常捕获（全局异常捕获、异常类型判断、异常处理、异常传播）
- 错误格式化（错误格式化方法、错误格式统一、错误消息处理、错误详情处理）
- 错误响应（错误响应格式、HTTP 状态码、错误响应头、错误响应体）
- 实际应用示例和最佳实践

## 特性

- **统一处理**：统一处理所有错误
- **灵活配置**：支持灵活的配置
- **易于扩展**：易于扩展新功能
- **可测试**：易于单元测试
- **标准化**：遵循标准接口

## 错误中间件概念

### 什么是错误中间件

错误中间件是捕获和处理应用错误的中间件，在请求处理过程中捕获异常并返回适当的错误响应。

### 错误中间件的作用

1. **统一捕获**：统一捕获所有异常
2. **统一处理**：统一处理错误
3. **统一格式**：统一错误响应格式
4. **错误日志**：记录错误日志

### 错误中间件的优势

1. **代码复用**：避免重复的错误处理代码
2. **统一格式**：统一的错误响应格式
3. **易于维护**：易于维护和扩展
4. **清晰结构**：清晰的错误处理结构

## 异常捕获

### 全局异常捕获

**示例**：
```php
<?php
declare(strict_types=1);

use Psr\Http\Server\MiddlewareInterface;
use Psr\Http\Server\RequestHandlerInterface;
use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;

class ErrorHandlingMiddleware implements MiddlewareInterface
{
    public function process(
        ServerRequestInterface $request,
        RequestHandlerInterface $handler
    ): ResponseInterface {
        try {
            return $handler->handle($request);
        } catch (\Exception $e) {
            return $this->handleException($e, $request);
        }
    }

    private function handleException(
        \Exception $e,
        ServerRequestInterface $request
    ): ResponseInterface {
        // 记录错误日志
        $this->logError($e, $request);
        
        // 格式化错误响应
        return $this->formatErrorResponse($e, $request);
    }
}
```

### 异常类型判断

**示例**：
```php
<?php
declare(strict_types=1);

private function handleException(
    \Exception $e,
    ServerRequestInterface $request
): ResponseInterface {
    if ($e instanceof ValidationException) {
        return $this->handleValidationError($e, $request);
    } elseif ($e instanceof AuthenticationException) {
        return $this->handleAuthenticationError($e, $request);
    } elseif ($e instanceof AuthorizationException) {
        return $this->handleAuthorizationError($e, $request);
    } elseif ($e instanceof NotFoundException) {
        return $this->handleNotFoundError($e, $request);
    } else {
        return $this->handleGenericError($e, $request);
    }
}
```

### 异常处理

**示例**：
```php
<?php
declare(strict_types=1);

private function handleValidationError(
    ValidationException $e,
    ServerRequestInterface $request
): ResponseInterface {
    $statusCode = $e->getStatusCode();
    $errors = $e->getDetails()['validation_errors'] ?? [];
    
    return new Response(
        json_encode([
            'success' => false,
            'error' => $e->getMessage(),
            'errors' => $errors,
            'status' => $statusCode,
        ]),
        $statusCode,
        ['Content-Type' => 'application/json']
    );
}
```

### 异常传播

**示例**：
```php
<?php
declare(strict_types=1);

class ErrorHandlingMiddleware implements MiddlewareInterface
{
    private array $ignoredExceptions = [];

    public function ignoreException(string $exceptionClass): self
    {
        $this->ignoredExceptions[] = $exceptionClass;
        return $this;
    }

    public function process(
        ServerRequestInterface $request,
        RequestHandlerInterface $handler
    ): ResponseInterface {
        try {
            return $handler->handle($request);
        } catch (\Exception $e) {
            // 检查是否应该忽略此异常
            if ($this->shouldIgnore($e)) {
                throw $e;  // 重新抛出，让其他中间件处理
            }
            
            return $this->handleException($e, $request);
        }
    }

    private function shouldIgnore(\Exception $e): bool
    {
        foreach ($this->ignoredExceptions as $class) {
            if ($e instanceof $class) {
                return true;
            }
        }
        return false;
    }
}
```

## 错误格式化

### 错误格式化方法

**示例**：
```php
<?php
declare(strict_types=1);

class ErrorFormatter
{
    public function format(\Exception $e, bool $isDevelopment = false): array
    {
        $base = [
            'success' => false,
            'error' => $this->getUserMessage($e),
            'status' => $this->getStatusCode($e),
            'error_id' => uniqid(),
        ];

        if ($isDevelopment) {
            $base['debug'] = [
                'message' => $e->getMessage(),
                'file' => $e->getFile(),
                'line' => $e->getLine(),
                'trace' => $e->getTraceAsString(),
            ];
        }

        if ($e instanceof AppException) {
            $base['code'] = $e->getErrorCode();
            $base['details'] = $e->getDetails();
        }

        return $base;
    }

    private function getUserMessage(\Exception $e): string
    {
        return match (get_class($e)) {
            ValidationException::class => '数据验证失败',
            AuthenticationException::class => '请先登录',
            AuthorizationException::class => '您没有权限',
            NotFoundException::class => '资源不存在',
            default => '操作失败，请稍后重试',
        };
    }

    private function getStatusCode(\Exception $e): int
    {
        if ($e instanceof AppException) {
            return $e->getStatusCode();
        }
        return 500;
    }
}
```

### 错误格式统一

**示例**：
```php
<?php
declare(strict_types=1);

class UnifiedErrorFormatter
{
    public function format(\Exception $e): array
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

    private function getMessage(\Exception $e): string
    {
        if ($e instanceof AppException) {
            return $e->getMessage();
        }
        return 'Internal Server Error';
    }

    private function getCode(\Exception $e): string
    {
        if ($e instanceof AppException) {
            return $e->getErrorCode();
        }
        return 'INTERNAL_ERROR';
    }

    private function getStatusCode(\Exception $e): int
    {
        if ($e instanceof AppException) {
            return $e->getStatusCode();
        }
        return 500;
    }
}
```

### 错误消息处理

**示例**：
```php
<?php
declare(strict_types=1);

class ErrorMessageHandler
{
    private array $messageMap = [
        ValidationException::class => '数据验证失败，请检查输入',
        AuthenticationException::class => '请先登录后再试',
        AuthorizationException::class => '您没有权限执行此操作',
        NotFoundException::class => '请求的资源不存在',
    ];

    public function getMessage(\Exception $e, bool $isDevelopment = false): string
    {
        $class = get_class($e);
        
        if (isset($this->messageMap[$class])) {
            return $this->messageMap[$class];
        }
        
        if ($isDevelopment) {
            return $e->getMessage();
        }
        
        return '操作失败，请稍后重试';
    }
}
```

### 错误详情处理

**示例**：
```php
<?php
declare(strict_types=1);

class ErrorDetailsHandler
{
    public function getDetails(\Exception $e, bool $isDevelopment = false): array
    {
        $details = [];
        
        if ($e instanceof AppException) {
            $details = $e->getDetails();
        }
        
        if ($isDevelopment) {
            $details['debug'] = [
                'file' => $e->getFile(),
                'line' => $e->getLine(),
                'trace' => $e->getTraceAsString(),
            ];
        }
        
        return $details;
    }
}
```

## 错误响应

### 错误响应格式

**示例**：
```php
<?php
declare(strict_types=1);

class ErrorResponseBuilder
{
    public function build(\Exception $e, ServerRequestInterface $request): ResponseInterface
    {
        $formatter = new ErrorFormatter();
        $data = $formatter->format($e, $this->isDevelopment());
        
        $statusCode = $this->getStatusCode($e);
        $headers = $this->getHeaders($e);
        
        return new Response(
            json_encode($data),
            $statusCode,
            array_merge([
                'Content-Type' => 'application/json',
            ], $headers)
        );
    }

    private function getHeaders(\Exception $e): array
    {
        $headers = [];
        
        if ($e instanceof RateLimitException) {
            $headers['Retry-After'] = (string) $e->getRetryAfter();
        }
        
        return $headers;
    }
}
```

### HTTP 状态码

**示例**：
```php
<?php
declare(strict_types=1);

function getStatusCode(\Exception $e): int
{
    if ($e instanceof AppException) {
        return $e->getStatusCode();
    }
    
    return match (get_class($e)) {
        ValidationException::class => 422,
        AuthenticationException::class => 401,
        AuthorizationException::class => 403,
        NotFoundException::class => 404,
        RateLimitException::class => 429,
        default => 500,
    };
}
```

### 错误响应头

**示例**：
```php
<?php
declare(strict_types=1);

function buildErrorResponse(\Exception $e): ResponseInterface
{
    $statusCode = getStatusCode($e);
    $headers = ['Content-Type' => 'application/json'];
    
    // 添加特定错误头
    if ($e instanceof RateLimitException) {
        $headers['Retry-After'] = (string) $e->getRetryAfter();
    }
    
    $body = json_encode([
        'error' => getErrorMessage($e),
        'status' => $statusCode,
    ]);
    
    return new Response($body, $statusCode, $headers);
}
```

### 错误响应体

**示例**：
```php
<?php
declare(strict_types=1);

function buildErrorBody(\Exception $e, bool $isDevelopment = false): array
{
    $body = [
        'success' => false,
        'error' => getErrorMessage($e),
        'status' => getStatusCode($e),
        'error_id' => uniqid(),
    ];
    
    if ($e instanceof AppException) {
        $body['code'] = $e->getErrorCode();
        $body['details'] = $e->getDetails();
    }
    
    if ($isDevelopment) {
        $body['debug'] = [
            'message' => $e->getMessage(),
            'file' => $e->getFile(),
            'line' => $e->getLine(),
        ];
    }
    
    return $body;
}
```

## 使用场景

### 所有 Web 应用

- 统一错误处理
- 错误响应格式化
- 错误日志记录

### API 错误处理

- API 错误响应
- 统一错误格式
- 错误状态码

### 错误日志记录

- 错误日志记录
- 错误上下文
- 错误分析

### 错误追踪

- 错误 ID 生成
- 错误追踪
- 错误报告

## 注意事项

### 异常捕获

- **全面捕获**：捕获所有异常
- **类型判断**：正确判断异常类型
- **异常传播**：合理处理异常传播

### 错误格式化

- **统一格式**：统一的错误格式
- **用户友好**：用户友好的错误消息
- **环境区分**：区分开发和生产环境

### 错误响应

- **状态码**：使用正确的 HTTP 状态码
- **响应头**：设置适当的响应头
- **响应体**：格式化的响应体

### 性能考虑

- **性能影响**：考虑错误处理的性能影响
- **日志性能**：优化日志记录性能
- **响应性能**：优化错误响应性能

## 常见问题

### 如何实现错误中间件？

实现 MiddlewareInterface 接口，在 process 方法中捕获异常并返回错误响应。

### 如何捕获所有异常？

使用 try-catch 在中间件中捕获所有异常。

### 如何格式化错误响应？

使用错误格式化器格式化错误响应，统一错误格式。

### 如何处理不同类型的错误？

根据异常类型使用不同的处理方法，返回适当的错误响应。

## 最佳实践

### 统一异常捕获

- 在错误中间件中统一捕获异常
- 避免在业务代码中重复捕获
- 保持错误处理的一致性

### 统一错误格式

- 使用统一的错误格式
- 提供清晰的错误信息
- 支持错误追踪

### 区分开发和生产环境

- 开发环境显示详细错误
- 生产环境显示友好错误
- 记录详细日志

### 记录错误日志

- 记录完整的错误信息
- 包含错误上下文
- 支持错误分析

## 相关章节

- **[5.16.1 错误处理概述](section-01-overview.md)**：了解错误处理概述的详细内容
- **[5.16.2 错误分类与自定义错误](section-02-error-types.md)**：了解错误分类的详细内容
- **[5.16.4 错误响应格式](section-04-response-format.md)**：了解错误响应格式的详细内容
- **[5.13.3 编写自定义中间件](../chapter-13-middleware/section-03-custom.md)**：了解中间件编写的详细内容

## 练习任务

1. **实现基本错误中间件**
   - 异常捕获
   - 错误格式化
   - 错误响应

2. **实现错误类型判断**
   - 异常类型判断
   - 不同错误处理
   - 错误响应生成

3. **实现错误格式化器**
   - 错误格式化
   - 错误消息处理
   - 错误详情处理

4. **实现错误响应构建**
   - 错误响应格式
   - HTTP 状态码
   - 错误响应头

5. **实现完整的错误中间件系统**
   - 统一错误处理
   - 错误日志记录
   - 错误追踪和分析
