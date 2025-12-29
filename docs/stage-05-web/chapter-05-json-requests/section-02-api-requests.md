# 5.5.2 API 请求处理

## 概述

API（Application Programming Interface，应用程序编程接口）请求处理是构建现代 Web 应用的核心技能。理解 API 的概念、掌握 API 请求的处理方法、实现统一的响应格式、正确处理错误，对于构建健壮的 API 接口至关重要。本节详细介绍 API 请求的概念、Content-Type 处理、请求方法检查、请求验证、响应格式、错误处理等内容，帮助零基础学员掌握 API 开发技术。

在现代 Web 开发中，前后端分离架构越来越普遍。前端通过 API 请求与后端交互，后端需要正确处理这些请求，包括验证请求方法、检查 Content-Type、解析请求数据、验证数据格式、返回统一的响应格式等。

**主要内容**：
- API 请求概述（什么是 API、API 请求的特点、RESTful API 概念）
- Content-Type 处理（检查 Content-Type、支持多种格式、处理不同编码）
- 请求方法检查（GET、POST、PUT、DELETE、PATCH 等）
- 请求验证（数据格式验证、必填字段检查、数据类型验证）
- 响应格式（JSON 响应、状态码设置、统一响应结构）
- 错误处理（错误分类、错误响应格式、HTTP 状态码）
- 实际应用示例和最佳实践

## 特性

- **统一格式**：API 响应使用统一的 JSON 格式
- **RESTful**：遵循 REST 架构风格
- **类型安全**：验证请求数据类型
- **错误处理**：提供清晰的错误信息
- **状态码**：使用标准 HTTP 状态码

## API 请求概述

### 什么是 API

API（Application Programming Interface）是应用程序之间的接口，定义了如何请求数据、如何传递参数、如何返回结果。

### API 请求的特点

1. **标准化**：使用标准的 HTTP 方法和状态码
2. **无状态**：每个请求都是独立的
3. **数据格式**：通常使用 JSON 格式
4. **RESTful**：遵循 REST 架构原则

### RESTful API 概念

REST（Representational State Transfer）是一种架构风格，RESTful API 遵循以下原则：

- **资源**：使用名词表示资源（如 `/users`、`/posts`）
- **HTTP 方法**：使用 HTTP 方法表示操作（GET、POST、PUT、DELETE）
- **状态码**：使用 HTTP 状态码表示结果
- **无状态**：每个请求包含所有必要信息

## Content-Type 处理

### Content-Type 检查

API 请求通常使用 `application/json` Content-Type。

**示例**：
```php
<?php
declare(strict_types=1);

function checkContentType(): void
{
    $contentType = $_SERVER['CONTENT_TYPE'] ?? '';
    
    if (!str_contains($contentType, 'application/json')) {
        http_response_code(400);
        echo json_encode([
            'error' => 'Content-Type must be application/json'
        ]);
        exit;
    }
}

checkContentType();
```

### 支持多种 Content-Type

**示例**：
```php
<?php
declare(strict_types=1);

function getRequestData(): array
{
    $contentType = $_SERVER['CONTENT_TYPE'] ?? '';
    
    if (str_contains($contentType, 'application/json')) {
        // JSON 请求
        $json = file_get_contents('php://input');
        $data = json_decode($json, true);
        
        if ($data === null && json_last_error() !== JSON_ERROR_NONE) {
            throw new InvalidArgumentException('Invalid JSON');
        }
        
        return $data;
    } elseif (str_contains($contentType, 'application/x-www-form-urlencoded')) {
        // 表单请求
        return $_POST;
    } elseif (str_contains($contentType, 'multipart/form-data')) {
        // 文件上传
        return $_POST;
    } else {
        throw new RuntimeException('Unsupported Content-Type');
    }
}
```

## 请求方法检查

### HTTP 方法常量

| 方法 | 说明 | 用途 |
|:-----|:-----|:-----|
| `GET` | 获取资源 | 查询数据 |
| `POST` | 创建资源 | 创建新数据 |
| `PUT` | 更新资源 | 完整更新 |
| `PATCH` | 部分更新 | 部分更新 |
| `DELETE` | 删除资源 | 删除数据 |

### 请求方法检查

**示例**：
```php
<?php
declare(strict_types=1);

function checkRequestMethod(string $allowedMethod): void
{
    $method = $_SERVER['REQUEST_METHOD'] ?? 'GET';
    
    if ($method !== $allowedMethod) {
        http_response_code(405);
        echo json_encode([
            'error' => "Method {$method} not allowed. Use {$allowedMethod}."
        ]);
        exit;
    }
}

// 使用
checkRequestMethod('POST');
```

### 支持多种方法

**示例**：
```php
<?php
declare(strict_types=1);

function checkRequestMethods(array $allowedMethods): void
{
    $method = $_SERVER['REQUEST_METHOD'] ?? 'GET';
    
    if (!in_array($method, $allowedMethods, true)) {
        http_response_code(405);
        echo json_encode([
            'error' => "Method {$method} not allowed. Allowed methods: " . implode(', ', $allowedMethods)
        ]);
        exit;
    }
}

// 使用
checkRequestMethods(['GET', 'POST']);
```

### 路由方法处理

**示例**：
```php
<?php
declare(strict_types=1);

class ApiRouter
{
    private array $routes = [];

    public function add(string $method, string $path, callable $handler): void
    {
        $this->routes[] = [
            'method' => $method,
            'path' => $path,
            'handler' => $handler,
        ];
    }

    public function handle(): void
    {
        $requestMethod = $_SERVER['REQUEST_METHOD'] ?? 'GET';
        $requestPath = parse_url($_SERVER['REQUEST_URI'] ?? '/', PHP_URL_PATH);

        foreach ($this->routes as $route) {
            if ($route['method'] === $requestMethod && $route['path'] === $requestPath) {
                $route['handler']();
                return;
            }
        }

        http_response_code(404);
        echo json_encode(['error' => 'Not found']);
    }
}

// 使用
$router = new ApiRouter();
$router->add('GET', '/api/users', function () {
    echo json_encode(['users' => []]);
});
$router->add('POST', '/api/users', function () {
    echo json_encode(['message' => 'User created']);
});

$router->handle();
```

## 请求验证

### 数据格式验证

**示例**：
```php
<?php
declare(strict_types=1);

class ApiRequestValidator
{
    public static function validate(array $data, array $rules): array
    {
        $errors = [];

        foreach ($rules as $field => $rule) {
            $value = $data[$field] ?? null;

            // 检查必需字段
            if (($rule['required'] ?? false) && $value === null) {
                $errors[$field] = "Field {$field} is required";
                continue;
            }

            // 如果字段为空且不是必需的，跳过验证
            if ($value === null) {
                continue;
            }

            // 类型验证
            if (isset($rule['type'])) {
                $expectedType = $rule['type'];
                $actualType = gettype($value);
                
                if (!$this->validateType($value, $expectedType)) {
                    $errors[$field] = "Field {$field} must be {$expectedType}";
                }
            }

            // 字符串长度验证
            if (isset($rule['min_length']) && is_string($value) && strlen($value) < $rule['min_length']) {
                $errors[$field] = "Field {$field} must be at least {$rule['min_length']} characters";
            }

            if (isset($rule['max_length']) && is_string($value) && strlen($value) > $rule['max_length']) {
                $errors[$field] = "Field {$field} must be at most {$rule['max_length']} characters";
            }

            // 数值范围验证
            if (isset($rule['min']) && is_numeric($value) && $value < $rule['min']) {
                $errors[$field] = "Field {$field} must be at least {$rule['min']}";
            }

            if (isset($rule['max']) && is_numeric($value) && $value > $rule['max']) {
                $errors[$field] = "Field {$field} must be at most {$rule['max']}";
            }

            // 正则表达式验证
            if (isset($rule['pattern']) && is_string($value) && !preg_match($rule['pattern'], $value)) {
                $errors[$field] = "Field {$field} format is invalid";
            }
        }

        return $errors;
    }

    private static function validateType(mixed $value, string $type): bool
    {
        return match ($type) {
            'string' => is_string($value),
            'int' => is_int($value),
            'float' => is_float($value),
            'bool' => is_bool($value),
            'array' => is_array($value),
            'email' => filter_var($value, FILTER_VALIDATE_EMAIL) !== false,
            'url' => filter_var($value, FILTER_VALIDATE_URL) !== false,
            default => true,
        };
    }
}

// 使用
$data = ['name' => 'John', 'email' => 'john@example.com', 'age' => 30];
$rules = [
    'name' => ['type' => 'string', 'required' => true, 'min_length' => 2, 'max_length' => 50],
    'email' => ['type' => 'email', 'required' => true],
    'age' => ['type' => 'int', 'required' => true, 'min' => 0, 'max' => 120],
];

$errors = ApiRequestValidator::validate($data, $rules);
if (!empty($errors)) {
    http_response_code(400);
    echo json_encode(['errors' => $errors]);
    exit;
}
```

## 响应格式

### JSON 响应

**示例**：
```php
<?php
declare(strict_types=1);

function jsonResponse(array $data, int $statusCode = 200): void
{
    http_response_code($statusCode);
    header('Content-Type: application/json; charset=UTF-8');
    echo json_encode($data, JSON_UNESCAPED_UNICODE | JSON_UNESCAPED_SLASHES);
    exit;
}

// 使用
jsonResponse(['message' => 'Success', 'data' => ['id' => 123]]);
```

### 统一响应结构

**示例**：
```php
<?php
declare(strict_types=1);

class ApiResponse
{
    public static function success(mixed $data = null, string $message = 'Success'): void
    {
        $response = [
            'success' => true,
            'message' => $message,
        ];
        
        if ($data !== null) {
            $response['data'] = $data;
        }
        
        http_response_code(200);
        header('Content-Type: application/json; charset=UTF-8');
        echo json_encode($response, JSON_UNESCAPED_UNICODE | JSON_UNESCAPED_SLASHES);
        exit;
    }

    public static function error(string $message, int $statusCode = 400, array $errors = []): void
    {
        $response = [
            'success' => false,
            'message' => $message,
        ];
        
        if (!empty($errors)) {
            $response['errors'] = $errors;
        }
        
        http_response_code($statusCode);
        header('Content-Type: application/json; charset=UTF-8');
        echo json_encode($response, JSON_UNESCAPED_UNICODE | JSON_UNESCAPED_SLASHES);
        exit;
    }
}

// 使用
ApiResponse::success(['id' => 123], 'User created');
ApiResponse::error('Validation failed', 400, ['email' => 'Invalid email format']);
```

### 状态码设置

**常用 HTTP 状态码**：

| 状态码 | 说明 | 使用场景 |
|:-------|:-----|:---------|
| `200` | OK | 请求成功 |
| `201` | Created | 资源创建成功 |
| `400` | Bad Request | 请求参数错误 |
| `401` | Unauthorized | 未认证 |
| `403` | Forbidden | 无权限 |
| `404` | Not Found | 资源不存在 |
| `422` | Unprocessable Entity | 验证失败 |
| `500` | Internal Server Error | 服务器错误 |

**示例**：
```php
<?php
declare(strict_types=1);

// 成功响应
http_response_code(200);
echo json_encode(['message' => 'Success']);

// 创建成功
http_response_code(201);
echo json_encode(['message' => 'Created', 'id' => 123]);

// 验证失败
http_response_code(422);
echo json_encode(['errors' => ['email' => 'Invalid format']]);

// 未找到
http_response_code(404);
echo json_encode(['error' => 'Not found']);

// 服务器错误
http_response_code(500);
echo json_encode(['error' => 'Internal server error']);
```

## 错误处理

### 错误分类

**示例**：
```php
<?php
declare(strict_types=1);

class ApiException extends Exception
{
    public function __construct(
        string $message,
        private int $statusCode = 400,
        private array $errors = []
    ) {
        parent::__construct($message);
    }

    public function getStatusCode(): int
    {
        return $this->statusCode;
    }

    public function getErrors(): array
    {
        return $this->errors;
    }

    public function toArray(): array
    {
        $response = [
            'success' => false,
            'message' => $this->getMessage(),
        ];
        
        if (!empty($this->errors)) {
            $response['errors'] = $this->errors;
        }
        
        return $response;
    }
}

// 使用
try {
    throw new ApiException('Validation failed', 422, ['email' => 'Invalid format']);
} catch (ApiException $e) {
    http_response_code($e->getStatusCode());
    echo json_encode($e->toArray());
}
```

### 错误响应格式

**示例**：
```php
<?php
declare(strict_types=1);

function handleApiError(Throwable $e): void
{
    if ($e instanceof ApiException) {
        http_response_code($e->getStatusCode());
        echo json_encode($e->toArray());
    } else {
        // 生产环境不暴露详细错误信息
        $message = $_ENV['APP_ENV'] === 'production' 
            ? 'Internal server error' 
            : $e->getMessage();
        
        http_response_code(500);
        echo json_encode([
            'success' => false,
            'message' => $message,
        ]);
    }
    exit;
}

// 全局错误处理
set_exception_handler('handleApiError');
```

### 完整 API 处理示例

**示例**：
```php
<?php
declare(strict_types=1);

// 设置错误处理
set_exception_handler(function (Throwable $e) {
    http_response_code(500);
    header('Content-Type: application/json');
    echo json_encode([
        'success' => false,
        'message' => 'Internal server error',
    ]);
});

// 检查请求方法
$method = $_SERVER['REQUEST_METHOD'] ?? 'GET';
if ($method !== 'POST') {
    http_response_code(405);
    echo json_encode(['error' => 'Method not allowed']);
    exit;
}

// 检查 Content-Type
$contentType = $_SERVER['CONTENT_TYPE'] ?? '';
if (!str_contains($contentType, 'application/json')) {
    http_response_code(400);
    echo json_encode(['error' => 'Content-Type must be application/json']);
    exit;
}

// 读取和解析 JSON
$json = file_get_contents('php://input');
$data = json_decode($json, true);

if ($data === null && json_last_error() !== JSON_ERROR_NONE) {
    http_response_code(400);
    echo json_encode(['error' => 'Invalid JSON']);
    exit;
}

// 验证数据
$rules = [
    'name' => ['type' => 'string', 'required' => true],
    'email' => ['type' => 'email', 'required' => true],
];

$errors = ApiRequestValidator::validate($data, $rules);
if (!empty($errors)) {
    http_response_code(422);
    echo json_encode(['errors' => $errors]);
    exit;
}

// 处理请求
// ... 业务逻辑 ...

// 返回成功响应
http_response_code(201);
header('Content-Type: application/json');
echo json_encode([
    'success' => true,
    'message' => 'User created',
    'data' => ['id' => 123],
]);
```

## 使用场景

### RESTful API 开发

- 构建 RESTful API
- 处理 CRUD 操作
- 实现资源管理

### 微服务通信

- 服务间通信
- API 网关
- 服务发现

### 前后端分离

- 前端应用后端 API
- 移动应用后端
- SPA 应用支持

### 移动应用后端

- 移动应用 API
- 第三方集成
- 数据同步

## 注意事项

### 请求方法验证

- **检查方法**：验证请求方法是否符合预期
- **支持的方法**：明确支持哪些 HTTP 方法
- **错误响应**：方法不允许时返回 405 状态码

### Content-Type 检查

- **验证类型**：检查 Content-Type 是否正确
- **支持多种格式**：根据需要支持多种 Content-Type
- **错误响应**：类型不正确时返回 400 状态码

### 数据验证

- **格式验证**：验证 JSON 格式
- **类型验证**：验证数据类型
- **业务验证**：验证业务规则

### 错误处理

- **统一格式**：使用统一的错误响应格式
- **状态码**：使用正确的 HTTP 状态码
- **错误信息**：提供清晰的错误信息

## 常见问题

### 如何检查请求方法？

使用 `$_SERVER['REQUEST_METHOD']` 获取请求方法：

```php
<?php
declare(strict_types=1);

$method = $_SERVER['REQUEST_METHOD'] ?? 'GET';
if ($method !== 'POST') {
    http_response_code(405);
    exit;
}
```

### 如何验证 Content-Type？

检查 `$_SERVER['CONTENT_TYPE']`：

```php
<?php
declare(strict_types=1);

$contentType = $_SERVER['CONTENT_TYPE'] ?? '';
if (!str_contains($contentType, 'application/json')) {
    http_response_code(400);
    exit;
}
```

### 如何统一 API 响应格式？

创建统一的响应类：

```php
<?php
declare(strict_types=1);

class ApiResponse
{
    public static function success(mixed $data = null): void
    {
        http_response_code(200);
        header('Content-Type: application/json');
        echo json_encode(['success' => true, 'data' => $data]);
        exit;
    }

    public static function error(string $message, int $code = 400): void
    {
        http_response_code($code);
        header('Content-Type: application/json');
        echo json_encode(['success' => false, 'message' => $message]);
        exit;
    }
}
```

### 如何处理 API 错误？

使用异常处理和统一的错误响应：

```php
<?php
declare(strict_types=1);

try {
    // API 处理逻辑
} catch (ApiException $e) {
    http_response_code($e->getStatusCode());
    echo json_encode($e->toArray());
} catch (Exception $e) {
    http_response_code(500);
    echo json_encode(['error' => 'Internal server error']);
}
```

## 最佳实践

### 验证请求方法和 Content-Type

- 检查请求方法是否符合预期
- 验证 Content-Type 是否正确
- 返回适当的错误响应

### 统一响应格式

- 使用统一的响应结构
- 包含 success、message、data 字段
- 使用正确的 HTTP 状态码

### 提供清晰的错误信息

- 使用描述性的错误消息
- 包含验证错误详情
- 使用标准 HTTP 状态码

### 实现请求验证中间件

- 创建可复用的验证中间件
- 统一处理请求验证
- 提高代码复用性

## 相关章节

- **[5.5.1 JSON 请求解析](section-01-json-parsing.md)**：了解 JSON 解析的详细内容
- **[5.7 RESTful API 设计](../chapter-07-restful-api/readme.md)**：了解 RESTful API 的深入内容
- **[5.13 中间件深入](../chapter-13-middleware/readme.md)**：了解中间件的使用方法

## 练习任务

1. **实现 API 请求处理类**
   - 检查请求方法
   - 验证 Content-Type
   - 解析 JSON 数据

2. **实现请求验证器**
   - 验证数据格式
   - 验证数据类型
   - 验证业务规则

3. **实现统一响应格式**
   - 创建响应类
   - 统一成功响应
   - 统一错误响应

4. **实现错误处理**
   - 创建异常类
   - 统一错误处理
   - 返回适当状态码

5. **实现完整的 API 端点**
   - 创建 API 端点
   - 处理各种请求
   - 返回统一响应
