# 5.8.1 响应处理

## 概述

HTTP 响应处理是 Web 应用返回数据给客户端的过程。理解 HTTP 响应的结构、掌握响应头的设置方法、正确使用状态码、统一响应格式，对于构建规范的 Web 应用和 API 至关重要。本节详细介绍 HTTP 响应的处理方法，包括响应头设置、状态码设置、响应体格式、JSON 响应等内容，帮助零基础学员掌握响应处理技术。

HTTP 响应由状态行、响应头和响应体组成。响应头包含响应的元数据信息（如 Content-Type、Content-Length 等），状态码表示响应的状态，响应体包含实际的数据内容。正确处理这些组成部分，对于提供规范的 API 响应至关重要。

**主要内容**：
- HTTP 响应概述（响应结构、响应头作用、状态码作用、响应体内容）
- `header()` 函数的语法、参数和使用方法
- `http_response_code()` 函数的语法和使用方法
- 响应头设置（Content-Type、Content-Length、Cache-Control 等）
- JSON 响应（JSON 格式、编码选项、响应结构）
- 响应格式统一（统一响应类、错误响应、成功响应）
- 实际应用示例和最佳实践

## 特性

- **灵活控制**：可以灵活设置响应头和状态码
- **格式统一**：支持统一的响应格式
- **类型明确**：通过 Content-Type 明确响应类型
- **状态清晰**：通过状态码明确响应状态
- **易于扩展**：支持自定义响应头

## HTTP 响应概述

### 响应结构

HTTP 响应由以下部分组成：

```
HTTP/1.1 200 OK
Content-Type: application/json; charset=UTF-8
Content-Length: 123
Cache-Control: no-cache

{"status":"success","data":[]}
```

**组成部分**：
1. **状态行**：`HTTP/1.1 200 OK`（协议版本、状态码、状态文本）
2. **响应头**：`Content-Type`、`Content-Length` 等
3. **空行**：分隔响应头和响应体
4. **响应体**：实际的数据内容

### 响应头作用

响应头提供响应的元数据信息：

| 响应头 | 作用 | 示例 |
|:-------|:-----|:-----|
| `Content-Type` | 指定响应体的 MIME 类型 | `application/json` |
| `Content-Length` | 指定响应体的长度（字节） | `1234` |
| `Cache-Control` | 控制缓存行为 | `no-cache` |
| `Location` | 重定向目标 URL | `https://example.com/new-page` |
| `Set-Cookie` | 设置 Cookie | `session_id=abc123; Path=/` |

### 状态码作用

状态码表示响应的状态：

| 状态码范围 | 含义 | 示例 |
|:-----------|:-----|:-----|
| 1xx | 信息性响应 | `100 Continue` |
| 2xx | 成功 | `200 OK`、`201 Created` |
| 3xx | 重定向 | `301 Moved Permanently`、`302 Found` |
| 4xx | 客户端错误 | `400 Bad Request`、`404 Not Found` |
| 5xx | 服务器错误 | `500 Internal Server Error` |

### 响应体内容

响应体包含实际的数据内容，格式取决于 `Content-Type`：
- **JSON**：`{"key": "value"}`
- **HTML**：`<html>...</html>`
- **XML**：`<root>...</root>`
- **纯文本**：`Plain text content`

## header() 函数

### 语法

**语法**：`header(string $header, bool $replace = true, int $response_code = 0): void`

### 参数

- `$header`：响应头字符串（格式：`Header-Name: value`）
- `$replace`：可选，是否替换同名的响应头（默认 `true`）
- `$response_code`：可选，强制设置 HTTP 状态码

### 返回值

无返回值（`void`）。

### 基本用法

**示例 1：设置 Content-Type**
```php
<?php
declare(strict_types=1);

header('Content-Type: application/json; charset=UTF-8');
echo json_encode(['message' => 'Hello']);
```

**示例 2：设置多个响应头**
```php
<?php
declare(strict_types=1);

header('Content-Type: application/json; charset=UTF-8');
header('Cache-Control: no-cache, no-store, must-revalidate');
header('X-Custom-Header: custom-value');
```

**示例 3：设置状态码**
```php
<?php
declare(strict_types=1);

header('HTTP/1.1 404 Not Found');
// 或使用 http_response_code()
http_response_code(404);
```

### 设置时机

**重要**：`header()` 必须在任何输出之前调用。

**正确**：
```php
<?php
declare(strict_types=1);

header('Content-Type: application/json');
echo json_encode(['data' => []]);
```

**错误**：
```php
<?php
declare(strict_types=1);

echo "Some output";  // 错误：在 header() 之前有输出
header('Content-Type: application/json');  // 会失败
```

**解决方案**：使用输出缓冲
```php
<?php
declare(strict_types=1);

ob_start();
echo "Some output";
header('Content-Type: application/json');  // 可以成功
ob_end_flush();
```

### 多次设置

**默认行为**（`$replace = true`）：
```php
<?php
declare(strict_types=1);

header('X-Custom-Header: value1');
header('X-Custom-Header: value2');  // 替换为 value2
// 最终响应头：X-Custom-Header: value2
```

**允许多个同名响应头**（`$replace = false`）：
```php
<?php
declare(strict_types=1);

header('X-Custom-Header: value1', false);
header('X-Custom-Header: value2', false);
// 最终响应头：X-Custom-Header: value1, value2
```

## http_response_code() 函数

### 语法

**语法**：`http_response_code(?int $response_code = null): int|bool`

### 参数

- `$response_code`：可选，要设置的 HTTP 状态码。如果为 `null`，返回当前状态码。

### 返回值

- **设置状态码**：成功返回 `true`，失败返回 `false`
- **获取状态码**：返回当前状态码（整数）

### 基本用法

**示例 1：设置状态码**
```php
<?php
declare(strict_types=1);

http_response_code(200);
echo "Success";

http_response_code(404);
echo "Not Found";

http_response_code(500);
echo "Internal Server Error";
```

**示例 2：获取状态码**
```php
<?php
declare(strict_types=1);

// 设置状态码
http_response_code(201);

// 获取状态码
$code = http_response_code();
echo "Current status code: {$code}";  // 201
```

**示例 3：与 header() 的区别**
```php
<?php
declare(strict_types=1);

// 方法 1：使用 http_response_code()（推荐）
http_response_code(404);

// 方法 2：使用 header()
header('HTTP/1.1 404 Not Found');

// 两种方法效果相同，但 http_response_code() 更简洁
```

## 响应头设置

### Content-Type

**作用**：指定响应体的 MIME 类型和字符编码。

**常用类型**：
```php
<?php
declare(strict_types=1);

// JSON
header('Content-Type: application/json; charset=UTF-8');

// HTML
header('Content-Type: text/html; charset=UTF-8');

// XML
header('Content-Type: application/xml; charset=UTF-8');

// 纯文本
header('Content-Type: text/plain; charset=UTF-8');

// PDF
header('Content-Type: application/pdf');

// 图片
header('Content-Type: image/jpeg');
```

### Content-Length

**作用**：指定响应体的长度（字节）。

**示例**：
```php
<?php
declare(strict_types=1);

$content = json_encode(['data' => []]);
header('Content-Length: ' . strlen($content));
echo $content;
```

### Cache-Control

**作用**：控制缓存行为。

**示例**：
```php
<?php
declare(strict_types=1);

// 禁止缓存
header('Cache-Control: no-cache, no-store, must-revalidate');
header('Pragma: no-cache');
header('Expires: 0');

// 允许缓存（1 小时）
header('Cache-Control: public, max-age=3600');

// 私有缓存（30 分钟）
header('Cache-Control: private, max-age=1800');
```

### Location（重定向）

**作用**：指定重定向目标 URL。

**示例**：
```php
<?php
declare(strict_types=1);

// 临时重定向（302）
http_response_code(302);
header('Location: https://example.com/new-page');
exit;

// 永久重定向（301）
http_response_code(301);
header('Location: https://example.com/new-page');
exit;
```

### Set-Cookie

**作用**：设置 Cookie。

**示例**：
```php
<?php
declare(strict_types=1);

header('Set-Cookie: session_id=abc123; Path=/; HttpOnly; Secure');
```

**注意**：通常使用 `setcookie()` 函数设置 Cookie，而不是直接使用 `header()`。

## JSON 响应

### 基本 JSON 响应

**示例**：
```php
<?php
declare(strict_types=1);

header('Content-Type: application/json; charset=UTF-8');
http_response_code(200);

$data = [
    'success' => true,
    'message' => 'Operation successful',
    'data' => ['id' => 123, 'name' => 'John'],
];

echo json_encode($data);
```

### JSON 编码选项

**常用选项**：
```php
<?php
declare(strict_types=1);

$data = [
    'name' => 'John',
    'message' => 'Hello "World"',
    'url' => 'https://example.com/path',
];

// 基本编码
$json = json_encode($data);
// {"name":"John","message":"Hello \"World\"","url":"https:\/\/example.com\/path"}

// 不转义斜杠
$json = json_encode($data, JSON_UNESCAPED_SLASHES);
// {"name":"John","message":"Hello \"World\"","url":"https://example.com/path"}

// 不转义 Unicode
$json = json_encode($data, JSON_UNESCAPED_UNICODE);
// {"name":"John","message":"Hello \"World\"","url":"https:\/\/example.com\/path"}

// 组合选项
$json = json_encode($data, JSON_UNESCAPED_SLASHES | JSON_UNESCAPED_UNICODE);
```

### 统一响应结构

**示例**：
```php
<?php
declare(strict_types=1);

class ApiResponse
{
    public static function success(mixed $data = null, string $message = 'Success', int $statusCode = 200): void
    {
        http_response_code($statusCode);
        header('Content-Type: application/json; charset=UTF-8');
        
        $response = [
            'success' => true,
            'message' => $message,
        ];
        
        if ($data !== null) {
            $response['data'] = $data;
        }
        
        echo json_encode($response, JSON_UNESCAPED_UNICODE | JSON_UNESCAPED_SLASHES);
        exit;
    }

    public static function error(string $message, int $statusCode = 400, array $errors = []): void
    {
        http_response_code($statusCode);
        header('Content-Type: application/json; charset=UTF-8');
        
        $response = [
            'success' => false,
            'message' => $message,
        ];
        
        if (!empty($errors)) {
            $response['errors'] = $errors;
        }
        
        echo json_encode($response, JSON_UNESCAPED_UNICODE | JSON_UNESCAPED_SLASHES);
        exit;
    }
}

// 使用
ApiResponse::success(['id' => 123], 'User created', 201);
ApiResponse::error('Validation failed', 422, ['email' => 'Invalid format']);
```

## 响应格式统一

### 成功响应格式

**标准格式**：
```json
{
  "success": true,
  "message": "Operation successful",
  "data": {
    "id": 123,
    "name": "John"
  }
}
```

**示例**：
```php
<?php
declare(strict_types=1);

function sendSuccessResponse(mixed $data = null, string $message = 'Success', int $statusCode = 200): void
{
    http_response_code($statusCode);
    header('Content-Type: application/json; charset=UTF-8');
    
    $response = [
        'success' => true,
        'message' => $message,
    ];
    
    if ($data !== null) {
        $response['data'] = $data;
    }
    
    echo json_encode($response, JSON_UNESCAPED_UNICODE | JSON_UNESCAPED_SLASHES);
    exit;
}
```

### 错误响应格式

**标准格式**：
```json
{
  "success": false,
  "message": "Error message",
  "errors": {
    "field": ["Error detail"]
  }
}
```

**示例**：
```php
<?php
declare(strict_types=1);

function sendErrorResponse(string $message, int $statusCode = 400, array $errors = []): void
{
    http_response_code($statusCode);
    header('Content-Type: application/json; charset=UTF-8');
    
    $response = [
        'success' => false,
        'message' => $message,
    ];
    
    if (!empty($errors)) {
        $response['errors'] = $errors;
    }
    
    echo json_encode($response, JSON_UNESCAPED_UNICODE | JSON_UNESCAPED_SLASHES);
    exit;
}
```

### 分页响应格式

**标准格式**：
```json
{
  "success": true,
  "data": [...],
  "pagination": {
    "page": 1,
    "limit": 10,
    "total": 100,
    "pages": 10
  }
}
```

**示例**：
```php
<?php
declare(strict_types=1);

function sendPaginatedResponse(array $data, int $page, int $limit, int $total): void
{
    http_response_code(200);
    header('Content-Type: application/json; charset=UTF-8');
    
    $response = [
        'success' => true,
        'data' => $data,
        'pagination' => [
            'page' => $page,
            'limit' => $limit,
            'total' => $total,
            'pages' => (int) ceil($total / $limit),
        ],
    ];
    
    echo json_encode($response, JSON_UNESCAPED_UNICODE | JSON_UNESCAPED_SLASHES);
    exit;
}
```

## 使用场景

### API 响应

- RESTful API 响应
- JSON API 响应
- 统一响应格式

### 错误响应

- 错误信息返回
- 验证错误返回
- 异常错误返回

### 重定向响应

- 页面重定向
- 登录重定向
- 权限重定向

### 文件下载

- 文件下载响应
- 文件流响应
- 大文件下载

## 注意事项

### header() 必须在输出前调用

- **问题**：在输出后调用 `header()` 会失败
- **解决**：使用输出缓冲或确保在输出前调用
- **检查**：使用 `headers_sent()` 检查是否已发送

**示例**：
```php
<?php
declare(strict_types=1);

if (headers_sent($file, $line)) {
    die("Headers already sent in {$file} on line {$line}");
}

header('Content-Type: application/json');
```

### 状态码的正确使用

- **2xx**：成功响应
- **4xx**：客户端错误
- **5xx**：服务器错误
- **使用标准状态码**：遵循 HTTP 标准

### 响应格式一致性

- **统一格式**：所有响应使用统一格式
- **统一结构**：成功和错误响应结构一致
- **统一编码**：使用相同的 JSON 编码选项

### 字符编码设置

- **UTF-8**：使用 UTF-8 编码
- **明确指定**：在 Content-Type 中明确指定字符集
- **一致性**：确保编码一致性

## 常见问题

### 如何设置响应头？

使用 `header()` 函数：

```php
<?php
declare(strict_types=1);

header('Content-Type: application/json; charset=UTF-8');
header('Cache-Control: no-cache');
```

### 如何设置状态码？

使用 `http_response_code()` 函数：

```php
<?php
declare(strict_types=1);

http_response_code(200);  // 成功
http_response_code(404);  // 未找到
http_response_code(500);  // 服务器错误
```

### header() 调用失败的原因？

常见原因：
1. **已有输出**：在输出后调用 `header()`
2. **BOM 字符**：文件包含 BOM 字符
3. **空白字符**：文件开头或结尾有空白字符

**解决方案**：
```php
<?php
declare(strict_types=1);

// 检查是否已发送
if (headers_sent()) {
    die('Headers already sent');
}

// 使用输出缓冲
ob_start();
// ... 代码 ...
header('Content-Type: application/json');
ob_end_flush();
```

### 如何统一响应格式？

创建统一的响应类：

```php
<?php
declare(strict_types=1);

class Response
{
    public static function json(mixed $data, int $statusCode = 200): void
    {
        http_response_code($statusCode);
        header('Content-Type: application/json; charset=UTF-8');
        echo json_encode($data, JSON_UNESCAPED_UNICODE | JSON_UNESCAPED_SLASHES);
        exit;
    }
}
```

## 最佳实践

### 在输出前设置响应头

- 确保在输出前调用 `header()`
- 使用 `headers_sent()` 检查
- 使用输出缓冲避免问题

### 使用标准状态码

- 遵循 HTTP 标准
- 使用正确的状态码
- 保持状态码一致性

### 统一响应格式

- 使用统一的响应结构
- 成功和错误格式一致
- 使用统一的编码选项

### 设置正确的 Content-Type

- 明确指定 MIME 类型
- 指定字符编码（UTF-8）
- 根据内容类型设置

## 相关章节

- **[5.8.2 CORS 跨域处理](section-02-cors.md)**：了解 CORS 跨域处理的详细内容
- **[5.7 RESTful API 设计](../chapter-07-restful-api/readme.md)**：了解 API 响应格式的设计
- **[5.1.1 HTTP 请求响应流程](../chapter-01-request-response/section-01-http-flow.md)**：了解 HTTP 响应的基础内容

## 练习任务

1. **实现响应处理函数**
   - 创建响应处理函数
   - 设置响应头和状态码
   - 统一响应格式

2. **实现 JSON 响应类**
   - 创建 JSON 响应类
   - 支持成功和错误响应
   - 支持分页响应

3. **实现响应头管理**
   - 统一设置响应头
   - 管理缓存策略
   - 处理重定向

4. **实现错误响应处理**
   - 统一错误响应格式
   - 根据错误类型设置状态码
   - 提供详细的错误信息

5. **实现完整的响应处理系统**
   - 响应类设计
   - 响应格式统一
   - 响应头管理
