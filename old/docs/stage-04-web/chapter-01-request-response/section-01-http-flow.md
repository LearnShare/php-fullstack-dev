# 4.1.1 HTTP 请求响应流程

## 概述

理解 HTTP 请求响应的完整流程是 Web 开发的基础。本节详细介绍 HTTP 协议、请求响应格式、从浏览器到服务器的完整流程，以及请求生命周期的完整示例。

## HTTP 协议基础

### HTTP 请求方法

| 方法 | 说明 | 幂等性 | 安全性 |
| :--- | :--- | :--- | :--- |
| GET | 获取资源 | 是 | 是 |
| POST | 创建资源 | 否 | 否 |
| PUT | 更新资源（完整） | 是 | 否 |
| PATCH | 更新资源（部分） | 否 | 否 |
| DELETE | 删除资源 | 是 | 否 |
| HEAD | 获取响应头 | 是 | 是 |
| OPTIONS | 获取支持的方法 | 是 | 是 |

### HTTP 状态码

| 状态码 | 类别 | 说明 |
| :--- | :--- | :--- |
| 1xx | 信息性 | 请求已接收，继续处理 |
| 2xx | 成功 | 请求成功处理 |
| 3xx | 重定向 | 需要进一步操作 |
| 4xx | 客户端错误 | 请求有误 |
| 5xx | 服务器错误 | 服务器处理失败 |

**常用状态码**：
- `200 OK`：请求成功
- `201 Created`：资源创建成功
- `400 Bad Request`：请求格式错误
- `401 Unauthorized`：未认证
- `403 Forbidden`：无权限
- `404 Not Found`：资源不存在
- `500 Internal Server Error`：服务器内部错误

## HTTP 请求格式

### 请求结构

```
请求行
Headers（请求头）
空行
Body（请求体，可选）
```

### 请求示例

```
GET /users/1 HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)
Accept: application/json
Accept-Language: zh-CN,zh;q=0.9
Connection: keep-alive
```

### POST 请求示例

```
POST /users HTTP/1.1
Host: example.com
Content-Type: application/json
Content-Length: 45

{"name":"Alice","email":"alice@example.com"}
```

## HTTP 响应格式

### 响应结构

```
状态行
Headers（响应头）
空行
Body（响应体）
```

### 响应示例

```
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 65
Date: Mon, 15 Jan 2024 10:30:00 GMT
Server: nginx/1.24.0

{"id":1,"name":"Alice","email":"alice@example.com"}
```

## 完整请求响应流程

### 流程概览

```
浏览器
  ↓ (1. 发送 HTTP 请求)
Web Server (Nginx/Apache)
  ↓ (2. 转发请求)
PHP-FPM / 运行时
  ↓ (3. 执行 PHP 代码)
PHP 脚本处理
  ↓ (4. 生成响应)
Web Server
  ↓ (5. 返回 HTTP 响应)
浏览器
```

### 详细流程说明

#### 1. 浏览器发送请求

- 用户在浏览器输入 URL 或点击链接
- 浏览器构造 HTTP 请求（包含方法、URL、Headers、Body）
- 请求通过 TCP/IP 协议发送到服务器

**示例请求：**

```http
GET /users/1 HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0
Accept: application/json
```

#### 2. Web Server 接收请求

- **Nginx** 或 **Apache** 接收 HTTP 请求
- 根据配置决定如何处理请求：
  - 静态文件：直接返回文件内容
  - PHP 文件：转发给 PHP-FPM 处理

#### 3. PHP-FPM 处理请求

- **FastCGI Process Manager (FPM)** 接收请求
- 创建或复用 PHP 进程执行脚本
- 将请求信息传递给 PHP 脚本（通过超全局变量）

#### 4. PHP 脚本执行

- PHP 脚本接收请求数据（`$_GET`、`$_POST`、`$_SERVER` 等）
- 执行业务逻辑（数据库查询、计算等）
- 生成响应内容（HTML、JSON、文件等）

#### 5. 响应返回浏览器

- PHP 脚本输出内容
- PHP-FPM 将响应返回给 Web Server
- Web Server 添加 HTTP Headers，返回给浏览器
- 浏览器解析响应并渲染页面

## PHP 中的请求处理

### 获取请求信息

```php
<?php
declare(strict_types=1);

// 请求方法
$method = $_SERVER['REQUEST_METHOD'] ?? 'GET';

// 请求 URI
$uri = $_SERVER['REQUEST_URI'] ?? '/';

// 请求头
$headers = getallheaders();

// 查询参数
$queryParams = $_GET;

// POST 数据
$postData = $_POST;

// JSON 请求体
$jsonData = json_decode(file_get_contents('php://input'), true);

// 客户端 IP
$clientIp = $_SERVER['REMOTE_ADDR'] ?? '';

// User-Agent
$userAgent = $_SERVER['HTTP_USER_AGENT'] ?? '';
```

### 生成响应

```php
<?php
declare(strict_types=1);

// 设置状态码
http_response_code(200);

// 设置响应头
header('Content-Type: application/json');
header('X-Custom-Header: value');

// 输出响应体
echo json_encode(['message' => 'Success'], JSON_UNESCAPED_UNICODE);

// 或使用 exit 结束
exit;
```

## 完整示例

```php
<?php
declare(strict_types=1);

// 1. 请求开始
$startTime = microtime(true);

// 2. 获取请求信息
$method = $_SERVER['REQUEST_METHOD'] ?? 'GET';
$uri = $_SERVER['REQUEST_URI'] ?? '/';
$path = parse_url($uri, PHP_URL_PATH);

// 3. 路由定义
$routes = [
    'GET /users' => 'listUsers',
    'GET /users/{id}' => 'getUser',
    'POST /users' => 'createUser',
];

// 4. 业务逻辑函数
function listUsers(): array
{
    return [
        ['id' => 1, 'name' => 'Alice'],
        ['id' => 2, 'name' => 'Bob'],
    ];
}

function getUser(int $id): ?array
{
    if ($id === 1) {
        return ['id' => 1, 'name' => 'Alice', 'email' => 'alice@example.com'];
    }
    return null;
}

function createUser(array $data): array
{
    if (empty($data['name']) || empty($data['email'])) {
        http_response_code(400);
        return ['error' => 'Name and email are required'];
    }

    return [
        'id' => 3,
        'name' => $data['name'],
        'email' => $data['email'],
    ];
}

// 5. 路由匹配
$routeKey = "{$method} {$path}";
$route = $routes[$routeKey] ?? null;

if ($route === null) {
    http_response_code(404);
    header('Content-Type: application/json');
    echo json_encode(['error' => 'Not found']);
    exit;
}

// 6. 执行处理函数
try {
    $result = match ($route) {
        'listUsers' => listUsers(),
        'getUser' => getUser((int) ($_GET['id'] ?? 0)),
        'createUser' => createUser(json_decode(file_get_contents('php://input'), true) ?? []),
    };

    // 7. 生成响应
    header('Content-Type: application/json');
    header('X-Response-Time: ' . (microtime(true) - $startTime));
    echo json_encode($result, JSON_UNESCAPED_UNICODE);
} catch (Exception $e) {
    http_response_code(500);
    header('Content-Type: application/json');
    echo json_encode(['error' => $e->getMessage()]);
}

// 8. 记录日志
$duration = microtime(true) - $startTime;
error_log(sprintf(
    '[%s] %s %s - %s - %.3fs',
    date('Y-m-d H:i:s'),
    $method,
    $path,
    $_SERVER['REMOTE_ADDR'] ?? 'unknown',
    $duration
));
```

## 注意事项

1. **请求验证**：始终验证和清理用户输入，防止安全漏洞
2. **错误处理**：使用适当的 HTTP 状态码，提供清晰的错误信息
3. **性能监控**：记录请求处理时间，识别性能瓶颈
4. **日志记录**：记录关键请求信息，便于调试和审计

## 练习

1. 创建一个 PHP 脚本，输出所有 `$_SERVER` 变量的值，理解请求信息的结构。

2. 实现一个请求日志记录器，记录每个请求的方法、URI、IP 地址和处理时间。

3. 编写一个脚本，模拟不同的 HTTP 方法（GET、POST、PUT、DELETE），并返回相应的响应。

4. 创建一个简单的路由系统，根据请求方法和路径调用不同的处理函数。

5. 实现一个请求性能监控器，记录每个请求的执行时间，并在响应头中返回 `X-Response-Time`。
