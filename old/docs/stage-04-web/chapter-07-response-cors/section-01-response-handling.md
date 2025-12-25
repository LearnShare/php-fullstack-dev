# 4.7.1 响应处理

## 概述

HTTP 响应处理是 Web 开发的基础。本节详细介绍 HTTP 状态码设置、响应头设置、常用响应头、响应格式，以及错误响应的标准化处理。

## HTTP 状态码设置

### http_response_code 函数

```php
<?php
declare(strict_types=1);

// 设置状态码
http_response_code(200);  // OK
http_response_code(201);  // Created
http_response_code(404);  // Not Found
http_response_code(500);  // Internal Server Error

// 获取当前状态码
$code = http_response_code();
```

### 常用状态码

| 状态码 | 说明 | 使用场景 |
| :--- | :--- | :--- |
| 200 | OK | 成功获取资源 |
| 201 | Created | 资源创建成功 |
| 204 | No Content | 删除成功，无返回内容 |
| 400 | Bad Request | 请求格式错误 |
| 401 | Unauthorized | 未认证 |
| 403 | Forbidden | 无权限 |
| 404 | Not Found | 资源不存在 |
| 422 | Unprocessable Entity | 验证失败 |
| 429 | Too Many Requests | 请求过多 |
| 500 | Internal Server Error | 服务器错误 |

### 状态码使用示例

```php
<?php
declare(strict_types=1);

function sendResponse(mixed $data, int $statusCode = 200): void
{
    http_response_code($statusCode);
    header('Content-Type: application/json; charset=UTF-8');
    echo json_encode($data, JSON_UNESCAPED_UNICODE);
}

// 成功响应
sendResponse(['message' => 'Success'], 200);

// 创建成功
sendResponse(['id' => 1, 'name' => 'Alice'], 201);

// 无内容（删除成功）
http_response_code(204);
exit;

// 错误响应
sendResponse(['error' => 'Not found'], 404);
sendResponse(['error' => 'Unauthorized'], 401);
sendResponse(['error' => 'Validation failed'], 422);
```

## 响应头设置

### header 函数

```php
<?php
// 设置单个 Header
header('Content-Type: application/json; charset=UTF-8');

// 设置多个 Header
header('Content-Type: application/json');
header('X-Custom-Header: value');

// 替换已存在的 Header
header('Content-Type: text/html', true);

// 添加新的 Header（不替换）
header('X-Additional-Header: value', false);
```

### 常用响应头

```php
<?php
declare(strict_types=1);

// Content-Type
header('Content-Type: application/json; charset=UTF-8');
header('Content-Type: text/html; charset=UTF-8');

// Content-Length
$content = json_encode($data);
header('Content-Length: ' . strlen($content));

// Cache-Control
header('Cache-Control: no-cache, no-store, must-revalidate');
header('Cache-Control: public, max-age=3600');

// Location（重定向）
header('Location: /users/1', true, 301);

// X-Response-Time（性能监控）
header('X-Response-Time: 0.123s');
```

## 响应格式

### JSON 响应

```php
<?php
declare(strict_types=1);

class JsonResponse
{
    public static function success(mixed $data = null, string $message = 'Success'): void
    {
        http_response_code(200);
        header('Content-Type: application/json; charset=UTF-8');
        echo json_encode([
            'success' => true,
            'message' => $message,
            'data' => $data,
        ], JSON_UNESCAPED_UNICODE);
        exit;
    }

    public static function error(string $message, int $code = 400, mixed $errors = null): void
    {
        http_response_code($code);
        header('Content-Type: application/json; charset=UTF-8');
        echo json_encode([
            'success' => false,
            'message' => $message,
            'errors' => $errors,
        ], JSON_UNESCAPED_UNICODE);
        exit;
    }
}
```

### XML 响应

```php
<?php
declare(strict_types=1);

header('Content-Type: application/xml; charset=UTF-8');
echo '<?xml version="1.0" encoding="UTF-8"?>';
echo '<response><status>success</status></response>';
```

## 完整示例

```php
<?php
declare(strict_types=1);

class ResponseHandler
{
    public static function json(mixed $data, int $statusCode = 200): void
    {
        http_response_code($statusCode);
        header('Content-Type: application/json; charset=UTF-8');
        echo json_encode($data, JSON_UNESCAPED_UNICODE);
    }

    public static function success(mixed $data = null): void
    {
        self::json(['success' => true, 'data' => $data], 200);
    }

    public static function created(mixed $data = null, string $location = null): void
    {
        if ($location !== null) {
            header("Location: {$location}");
        }
        self::json(['success' => true, 'data' => $data], 201);
    }

    public static function error(string $message, int $code = 400): void
    {
        self::json(['success' => false, 'error' => $message], $code);
    }
}
```

## 注意事项

1. **状态码准确性**：使用正确的状态码反映操作结果
2. **Content-Type**：正确设置响应内容类型
3. **字符编码**：统一使用 UTF-8
4. **响应格式**：保持 API 响应格式一致

## 练习

1. 创建一个统一的响应处理类，支持成功和错误响应。

2. 实现不同状态码的响应处理函数。

3. 编写一个响应格式化工具，支持 JSON、XML 等格式。

4. 实现响应头管理类，统一设置常用响应头。
