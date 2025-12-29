# 5.16.1 错误处理概述

## 概述

Web 层面的错误处理需要特殊的策略和机制。理解 Web 错误处理的特点、掌握错误分类和处理策略、实现错误日志和用户友好错误，对于构建稳定、用户友好的 Web 应用至关重要。本节详细介绍 Web 错误处理概述、错误类型（4xx、5xx）、错误处理策略、错误日志记录、用户友好错误等内容，帮助零基础学员理解 Web 错误处理的特点。

Web 错误处理与 CLI 错误处理不同，需要考虑用户体验、错误信息安全性、错误响应格式等因素。理解 Web 错误处理的特点对于正确实现错误处理系统至关重要。

**主要内容**：
- Web 错误处理概述（Web 错误的特点、错误处理的目标、错误处理策略）
- 错误类型（客户端错误 4xx、服务器错误 5xx、业务逻辑错误、系统错误）
- 错误处理策略（开发环境策略、生产环境策略、错误显示、错误日志）
- 错误日志记录（日志级别、日志格式、日志存储、日志分析）
- 用户友好错误（错误信息设计、错误提示、错误恢复建议、错误页面）
- 实际应用示例和最佳实践

## 特性

- **用户友好**：提供用户友好的错误信息
- **安全可靠**：不泄露敏感信息
- **完整日志**：完整记录错误信息
- **灵活策略**：支持不同环境的策略
- **易于维护**：易于维护和扩展

## Web 错误处理概述

### Web 错误的特点

1. **用户可见**：错误会直接展示给用户
2. **HTTP 状态码**：使用 HTTP 状态码表示错误
3. **错误响应**：需要返回格式化的错误响应
4. **用户体验**：影响用户体验

### 错误处理的目标

1. **用户友好**：提供用户友好的错误信息
2. **安全可靠**：不泄露敏感信息
3. **完整记录**：完整记录错误信息
4. **快速恢复**：帮助用户快速恢复

### 错误处理策略

**主要策略**：
- **开发环境**：显示详细错误信息
- **生产环境**：显示友好错误信息，记录详细日志
- **错误分类**：区分不同类型的错误
- **错误恢复**：提供错误恢复建议

## 错误类型

### 客户端错误（4xx）

**常见 4xx 错误**：
- **400 Bad Request**：请求格式错误
- **401 Unauthorized**：未认证
- **403 Forbidden**：无权限
- **404 Not Found**：资源不存在
- **422 Unprocessable Entity**：数据验证失败

**示例**：
```php
<?php
declare(strict_types=1);

function handleClientError(string $message, int $statusCode = 400): void
{
    http_response_code($statusCode);
    header('Content-Type: application/json');
    echo json_encode([
        'error' => $message,
        'status' => $statusCode,
    ]);
    exit;
}

// 使用
if (empty($_POST['email'])) {
    handleClientError('邮箱是必填项', 400);
}
```

### 服务器错误（5xx）

**常见 5xx 错误**：
- **500 Internal Server Error**：服务器内部错误
- **502 Bad Gateway**：网关错误
- **503 Service Unavailable**：服务不可用
- **504 Gateway Timeout**：网关超时

**示例**：
```php
<?php
declare(strict_types=1);

function handleServerError(\Exception $e, bool $isProduction = false): void
{
    // 记录详细错误日志
    error_log(sprintf(
        'Error: %s in %s:%d',
        $e->getMessage(),
        $e->getFile(),
        $e->getLine()
    ));
    
    http_response_code(500);
    header('Content-Type: application/json');
    
    if ($isProduction) {
        echo json_encode([
            'error' => 'Internal Server Error',
            'status' => 500,
        ]);
    } else {
        echo json_encode([
            'error' => $e->getMessage(),
            'file' => $e->getFile(),
            'line' => $e->getLine(),
            'status' => 500,
        ]);
    }
    exit;
}
```

### 业务逻辑错误

**示例**：
```php
<?php
declare(strict_types=1);

class BusinessException extends \Exception
{
    public function __construct(
        string $message,
        private int $statusCode = 400,
        private array $details = []
    ) {
        parent::__construct($message);
    }

    public function getStatusCode(): int
    {
        return $this->statusCode;
    }

    public function getDetails(): array
    {
        return $this->details;
    }
}

// 使用
if ($user->isLocked()) {
    throw new BusinessException('用户已被锁定', 403, ['user_id' => $user->getId()]);
}
```

### 系统错误

**示例**：
```php
<?php
declare(strict_types=1);

class SystemException extends \Exception
{
    // 系统级错误，需要记录详细日志
}

function handleSystemError(\Exception $e): void
{
    // 记录系统错误
    error_log(sprintf(
        'System Error: %s in %s:%d\nStack trace:\n%s',
        $e->getMessage(),
        $e->getFile(),
        $e->getLine(),
        $e->getTraceAsString()
    ));
    
    // 发送告警（可选）
    sendAlert($e);
    
    // 返回错误响应
    http_response_code(500);
    echo json_encode(['error' => '系统错误，请稍后重试']);
}
```

## 错误处理策略

### 开发环境策略

**示例**：
```php
<?php
declare(strict_types=1);

function handleError(\Exception $e, bool $isDevelopment = false): void
{
    if ($isDevelopment) {
        // 开发环境：显示详细错误
        http_response_code(500);
        header('Content-Type: application/json');
        echo json_encode([
            'error' => $e->getMessage(),
            'file' => $e->getFile(),
            'line' => $e->getLine(),
            'trace' => $e->getTrace(),
        ]);
    } else {
        // 生产环境：显示友好错误
        error_log($e->getMessage());
        http_response_code(500);
        echo json_encode(['error' => 'Internal Server Error']);
    }
    exit;
}
```

### 生产环境策略

**示例**：
```php
<?php
declare(strict_types=1);

function handleErrorProduction(\Exception $e): void
{
    // 记录详细错误日志
    error_log(sprintf(
        '[%s] %s in %s:%d',
        date('Y-m-d H:i:s'),
        $e->getMessage(),
        $e->getFile(),
        $e->getLine()
    ));
    
    // 返回友好错误信息
    http_response_code(500);
    header('Content-Type: application/json');
    echo json_encode([
        'error' => '服务器错误，请稍后重试',
        'error_id' => uniqid(),  // 错误 ID，用于追踪
    ]);
    exit;
}
```

### 错误显示

**示例**：
```php
<?php
declare(strict_types=1);

function displayError(\Exception $e, bool $isDevelopment = false): void
{
    if ($isDevelopment) {
        // 开发环境：显示详细错误
        echo "<pre>";
        echo "Error: " . $e->getMessage() . "\n";
        echo "File: " . $e->getFile() . "\n";
        echo "Line: " . $e->getLine() . "\n";
        echo "Trace:\n" . $e->getTraceAsString();
        echo "</pre>";
    } else {
        // 生产环境：显示友好错误页面
        include 'error_page.php';
    }
}
```

### 错误日志

**示例**：
```php
<?php
declare(strict_types=1);

function logError(\Exception $e, array $context = []): void
{
    $log = [
        'timestamp' => date('Y-m-d H:i:s'),
        'message' => $e->getMessage(),
        'file' => $e->getFile(),
        'line' => $e->getLine(),
        'code' => $e->getCode(),
        'trace' => $e->getTraceAsString(),
        'context' => $context,
    ];
    
    error_log(json_encode($log, JSON_UNESCAPED_UNICODE));
}
```

## 错误日志记录

### 日志级别

**示例**：
```php
<?php
declare(strict_types=1);

enum LogLevel: string
{
    case DEBUG = 'debug';
    case INFO = 'info';
    case WARNING = 'warning';
    case ERROR = 'error';
    case CRITICAL = 'critical';
}

function logError(\Exception $e, LogLevel $level = LogLevel::ERROR): void
{
    $message = sprintf(
        '[%s] %s: %s in %s:%d',
        $level->value,
        get_class($e),
        $e->getMessage(),
        $e->getFile(),
        $e->getLine()
    );
    
    error_log($message);
}
```

### 日志格式

**示例**：
```php
<?php
declare(strict_types=1);

function logErrorFormatted(\Exception $e, array $context = []): void
{
    $log = [
        'timestamp' => date('c'),
        'level' => 'error',
        'message' => $e->getMessage(),
        'exception' => get_class($e),
        'file' => $e->getFile(),
        'line' => $e->getLine(),
        'trace' => $e->getTraceAsString(),
        'context' => $context,
        'request' => [
            'method' => $_SERVER['REQUEST_METHOD'] ?? 'CLI',
            'uri' => $_SERVER['REQUEST_URI'] ?? '',
            'ip' => $_SERVER['REMOTE_ADDR'] ?? '',
        ],
    ];
    
    error_log(json_encode($log, JSON_UNESCAPED_UNICODE | JSON_PRETTY_PRINT));
}
```

### 日志存储

**示例**：
```php
<?php
declare(strict_types=1);

class ErrorLogger
{
    private string $logFile;

    public function __construct(string $logFile = 'error.log')
    {
        $this->logFile = $logFile;
    }

    public function log(\Exception $e, array $context = []): void
    {
        $log = [
            'timestamp' => date('Y-m-d H:i:s'),
            'message' => $e->getMessage(),
            'file' => $e->getFile(),
            'line' => $e->getLine(),
            'context' => $context,
        ];
        
        file_put_contents(
            $this->logFile,
            json_encode($log) . "\n",
            FILE_APPEND
        );
    }
}
```

### 日志分析

**示例**：
```php
<?php
declare(strict_types=1);

function analyzeErrorLog(string $logFile): array
{
    $errors = [];
    $lines = file($logFile, FILE_IGNORE_NEW_LINES);
    
    foreach ($lines as $line) {
        $log = json_decode($line, true);
        if ($log) {
            $errors[] = $log;
        }
    }
    
    // 统计错误类型
    $stats = [];
    foreach ($errors as $error) {
        $type = $error['exception'] ?? 'Unknown';
        $stats[$type] = ($stats[$type] ?? 0) + 1;
    }
    
    return [
        'total' => count($errors),
        'stats' => $stats,
        'recent' => array_slice($errors, -10),
    ];
}
```

## 用户友好错误

### 错误信息设计

**示例**：
```php
<?php
declare(strict_types=1);

function formatUserError(\Exception $e): array
{
    return [
        'error' => $this->getUserFriendlyMessage($e),
        'suggestion' => $this->getSuggestion($e),
        'error_id' => uniqid(),
    ];
}

private function getUserFriendlyMessage(\Exception $e): string
{
    return match (get_class($e)) {
        ValidationException::class => '数据验证失败，请检查输入',
        AuthenticationException::class => '请先登录',
        AuthorizationException::class => '您没有权限执行此操作',
        NotFoundException::class => '请求的资源不存在',
        default => '操作失败，请稍后重试',
    };
}
```

### 错误提示

**示例**：
```php
<?php
declare(strict_types=1);

function showErrorToUser(\Exception $e): void
{
    $message = match (get_class($e)) {
        ValidationException::class => '请检查输入数据',
        AuthenticationException::class => '请先登录',
        default => '操作失败，请稍后重试',
    };
    
    http_response_code($this->getStatusCode($e));
    header('Content-Type: application/json');
    echo json_encode([
        'success' => false,
        'message' => $message,
        'error_id' => uniqid(),
    ]);
}
```

### 错误恢复建议

**示例**：
```php
<?php
declare(strict_types=1);

function getErrorSuggestion(\Exception $e): ?string
{
    return match (get_class($e)) {
        ValidationException::class => '请检查所有必填字段是否已填写',
        AuthenticationException::class => '请先登录后再试',
        RateLimitException::class => '请求过于频繁，请稍后再试',
        NotFoundException::class => '请检查 URL 是否正确',
        default => null,
    };
}
```

### 错误页面

**示例**：
```php
<?php
declare(strict_types=1);

function showErrorPage(int $statusCode, string $message): void
{
    http_response_code($statusCode);
    
    if ($_SERVER['HTTP_ACCEPT'] === 'application/json') {
        header('Content-Type: application/json');
        echo json_encode([
            'error' => $message,
            'status' => $statusCode,
        ]);
    } else {
        include "error_pages/{$statusCode}.php";
    }
    exit;
}
```

## 使用场景

### 所有 Web 应用

- 错误处理和显示
- 错误日志记录
- 用户友好错误

### API 错误处理

- API 错误响应
- 错误状态码
- 错误格式统一

### 用户错误提示

- 表单验证错误
- 操作失败提示
- 错误恢复建议

### 系统错误处理

- 系统错误记录
- 错误告警
- 错误分析

## 注意事项

### 错误信息的安全性

- **不泄露敏感信息**：不泄露系统内部信息
- **开发环境区分**：开发和生产环境区分
- **错误 ID**：使用错误 ID 追踪错误

### 开发和生产环境差异

- **开发环境**：显示详细错误
- **生产环境**：显示友好错误，记录详细日志
- **环境检测**：自动检测环境

### 错误日志记录

- **完整记录**：记录完整的错误信息
- **结构化日志**：使用结构化日志格式
- **日志分析**：定期分析错误日志

### 用户体验

- **友好错误**：提供用户友好的错误信息
- **错误恢复**：提供错误恢复建议
- **错误追踪**：提供错误 ID 用于追踪

## 常见问题

### Web 错误处理的特点？

- 用户可见
- 使用 HTTP 状态码
- 需要格式化响应
- 影响用户体验

### 如何区分错误类型？

- 4xx：客户端错误
- 5xx：服务器错误
- 业务错误：自定义异常
- 系统错误：系统异常

### 如何提供用户友好错误？

- 使用友好的错误消息
- 提供错误恢复建议
- 使用错误 ID 追踪
- 区分开发和生产环境

### 如何记录错误日志？

- 记录完整错误信息
- 使用结构化日志
- 包含上下文信息
- 定期分析日志

## 最佳实践

### 区分错误类型

- 区分客户端和服务器错误
- 区分业务和系统错误
- 使用适当的 HTTP 状态码

### 开发环境显示详细错误

- 开发环境显示详细错误
- 便于调试和开发
- 提高开发效率

### 生产环境记录错误到日志

- 生产环境记录详细日志
- 显示友好错误信息
- 不泄露敏感信息

### 提供用户友好的错误信息

- 使用用户友好的语言
- 提供错误恢复建议
- 使用错误 ID 追踪

## 相关章节

- **[5.16.2 错误分类与自定义错误](section-02-error-types.md)**：了解错误分类的详细内容
- **[5.16.3 错误中间件](section-03-error-middleware.md)**：了解错误中间件的详细内容
- **[5.16.4 错误响应格式](section-04-response-format.md)**：了解错误响应格式的详细内容
- **[4.7 错误与异常处理](../stage-04-system/chapter-07-error-handling/readme.md)**：了解错误处理的基础知识

## 练习任务

1. **实现基本错误处理**
   - 错误分类
   - 错误响应
   - 错误日志

2. **实现环境区分**
   - 开发环境错误显示
   - 生产环境错误处理
   - 环境检测

3. **实现错误日志系统**
   - 日志记录
   - 日志格式
   - 日志分析

4. **实现用户友好错误**
   - 错误消息设计
   - 错误恢复建议
   - 错误页面

5. **实现完整的错误处理系统**
   - 错误分类和处理
   - 错误日志和告警
   - 错误分析和优化
