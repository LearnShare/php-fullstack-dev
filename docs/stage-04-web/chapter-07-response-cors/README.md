# 4.7 响应处理与跨域（CORS）

## 目标

- 掌握 HTTP 状态码的设置方法。
- 熟悉响应头的设置与常用 Header。
- 深入理解 CORS（跨域资源共享）机制。
- 能够正确处理预检请求（Preflight）和跨域问题。

## HTTP 状态码设置

### http_response_code 函数

- **语法**：`http_response_code(?int $response_code = null): int|bool`
- 设置 HTTP 响应状态码。

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
echo "Current status code: {$code}\n";
```

### 常用状态码

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
sendResponse(['error' => 'Forbidden'], 403);
sendResponse(['error' => 'Validation failed'], 422);
sendResponse(['error' => 'Internal error'], 500);
```

## 响应头设置

### header 函数

- **语法**：`header(string $header, bool $replace = true, int $response_code = 0): void`
- 设置 HTTP 响应头。

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
header('Content-Type: application/xml');

// Content-Length
$content = json_encode($data);
header('Content-Length: ' . strlen($content));

// Cache-Control
header('Cache-Control: no-cache, no-store, must-revalidate');
header('Cache-Control: public, max-age=3600');

// Expires
header('Expires: ' . gmdate('D, d M Y H:i:s', time() + 3600) . ' GMT');

// Location（重定向）
header('Location: /users/1', true, 301);

// Set-Cookie
header('Set-Cookie: session_id=abc123; Path=/; HttpOnly; Secure; SameSite=Strict');

// X-* 自定义头
header('X-Request-ID: ' . uniqid());
header('X-Response-Time: ' . (microtime(true) - $startTime) . 's');
```

### 响应头工具类

```php
<?php
declare(strict_types=1);

class ResponseHeaders
{
    private array $headers = [];

    public function set(string $name, string $value, bool $replace = true): self
    {
        $this->headers[] = [
            'name' => $name,
            'value' => $value,
            'replace' => $replace,
        ];
        return $this;
    }

    public function contentType(string $type, string $charset = 'UTF-8'): self
    {
        return $this->set('Content-Type', "{$type}; charset={$charset}");
    }

    public function cacheControl(string $directive): self
    {
        return $this->set('Cache-Control', $directive);
    }

    public function location(string $url, int $code = 302): self
    {
        $this->set('Location', $url);
        http_response_code($code);
        return $this;
    }

    public function send(): void
    {
        foreach ($this->headers as $header) {
            header(
                "{$header['name']}: {$header['value']}",
                $header['replace']
            );
        }
    }
}

// 使用
$headers = new ResponseHeaders();
$headers->contentType('application/json')
        ->cacheControl('no-cache')
        ->set('X-Custom-Header', 'value')
        ->send();
```

## CORS 基础

### 什么是 CORS

- **CORS**：Cross-Origin Resource Sharing（跨域资源共享）。
- 浏览器的同源策略限制跨域请求。
- CORS 允许服务器声明哪些源可以访问资源。

### 同源策略

- **同源**：协议、域名、端口完全相同。
- **跨域**：协议、域名、端口任一不同。

```
http://example.com:80  →  http://example.com:80      // 同源
http://example.com:80  →  https://example.com:80     // 跨域（协议不同）
http://example.com:80  →  http://api.example.com:80  // 跨域（域名不同）
http://example.com:80  →  http://example.com:8080    // 跨域（端口不同）
```

## CORS 实现

### 简单请求（Simple Request）

**条件：**
- 方法：GET、POST、HEAD
- 头：仅允许简单头（Accept、Content-Type 等）
- Content-Type：`application/x-www-form-urlencoded`、`multipart/form-data`、`text/plain`

**处理：**

```php
<?php
declare(strict_types=1);

// 允许的源
$allowedOrigins = [
    'http://localhost:3000',
    'https://example.com',
];

$origin = $_SERVER['HTTP_ORIGIN'] ?? '';

if (in_array($origin, $allowedOrigins, true)) {
    header("Access-Control-Allow-Origin: {$origin}");
}

// 或允许所有源（不推荐用于生产环境）
header('Access-Control-Allow-Origin: *');
```

### 预检请求（Preflight Request）

**触发条件：**
- 方法：PUT、DELETE、PATCH 等
- 自定义头：`X-Custom-Header` 等
- Content-Type：`application/json` 等

**OPTIONS 请求处理：**

```php
<?php
declare(strict_types=1);

// 处理预检请求
if ($_SERVER['REQUEST_METHOD'] === 'OPTIONS') {
    $origin = $_SERVER['HTTP_ORIGIN'] ?? '';
    
    // 允许的源
    $allowedOrigins = ['http://localhost:3000', 'https://example.com'];
    if (in_array($origin, $allowedOrigins, true)) {
        header("Access-Control-Allow-Origin: {$origin}");
    }
    
    // 允许的方法
    header('Access-Control-Allow-Methods: GET, POST, PUT, DELETE, OPTIONS');
    
    // 允许的头
    header('Access-Control-Allow-Headers: Content-Type, Authorization, X-Requested-With');
    
    // 允许携带凭证
    header('Access-Control-Allow-Credentials: true');
    
    // 预检请求缓存时间（秒）
    header('Access-Control-Max-Age: 3600');
    
    http_response_code(204);
    exit;
}
```

### 完整 CORS 中间件

```php
<?php
declare(strict_types=1);

class CorsMiddleware
{
    private array $allowedOrigins;
    private array $allowedMethods;
    private array $allowedHeaders;
    private bool $allowCredentials;
    private int $maxAge;

    public function __construct(
        array $allowedOrigins = ['*'],
        array $allowedMethods = ['GET', 'POST', 'PUT', 'DELETE', 'OPTIONS'],
        array $allowedHeaders = ['Content-Type', 'Authorization'],
        bool $allowCredentials = false,
        int $maxAge = 3600
    ) {
        $this->allowedOrigins = $allowedOrigins;
        $this->allowedMethods = $allowedMethods;
        $this->allowedHeaders = $allowedHeaders;
        $this->allowCredentials = $allowCredentials;
        $this->maxAge = $maxAge;
    }

    public function handle(): void
    {
        $origin = $_SERVER['HTTP_ORIGIN'] ?? '';
        
        // 处理预检请求
        if ($_SERVER['REQUEST_METHOD'] === 'OPTIONS') {
            $this->handlePreflight($origin);
            return;
        }
        
        // 处理实际请求
        $this->handleRequest($origin);
    }

    private function handlePreflight(string $origin): void
    {
        if ($this->isOriginAllowed($origin)) {
            header("Access-Control-Allow-Origin: {$origin}");
        } elseif (in_array('*', $this->allowedOrigins, true)) {
            header('Access-Control-Allow-Origin: *');
        }
        
        header('Access-Control-Allow-Methods: ' . implode(', ', $this->allowedMethods));
        header('Access-Control-Allow-Headers: ' . implode(', ', $this->allowedHeaders));
        header("Access-Control-Max-Age: {$this->maxAge}");
        
        if ($this->allowCredentials) {
            header('Access-Control-Allow-Credentials: true');
        }
        
        http_response_code(204);
        exit;
    }

    private function handleRequest(string $origin): void
    {
        if ($this->isOriginAllowed($origin)) {
            header("Access-Control-Allow-Origin: {$origin}");
        } elseif (in_array('*', $this->allowedOrigins, true)) {
            header('Access-Control-Allow-Origin: *');
        }
        
        if ($this->allowCredentials) {
            header('Access-Control-Allow-Credentials: true');
        }
        
        // 暴露响应头
        header('Access-Control-Expose-Headers: X-Request-ID, X-Response-Time');
    }

    private function isOriginAllowed(string $origin): bool
    {
        if (empty($origin)) {
            return false;
        }
        
        return in_array($origin, $this->allowedOrigins, true);
    }
}

// 使用
$cors = new CorsMiddleware(
    allowedOrigins: ['http://localhost:3000', 'https://example.com'],
    allowCredentials: true
);
$cors->handle();
```

## CORS 配置示例

### 开发环境

```php
<?php
// 开发环境：允许所有源
header('Access-Control-Allow-Origin: *');
header('Access-Control-Allow-Methods: GET, POST, PUT, DELETE, OPTIONS');
header('Access-Control-Allow-Headers: Content-Type, Authorization');
```

### 生产环境

```php
<?php
declare(strict_types=1);

$allowedOrigins = [
    'https://app.example.com',
    'https://admin.example.com',
];

$origin = $_SERVER['HTTP_ORIGIN'] ?? '';

if (in_array($origin, $allowedOrigins, true)) {
    header("Access-Control-Allow-Origin: {$origin}");
    header('Access-Control-Allow-Credentials: true');
} else {
    http_response_code(403);
    exit('Forbidden');
}
```

## 带凭证的请求

### 客户端设置

```javascript
// JavaScript
fetch('https://api.example.com/users', {
    method: 'POST',
    credentials: 'include',  // 携带 Cookie
    headers: {
        'Content-Type': 'application/json',
    },
    body: JSON.stringify({ name: 'Alice' }),
});
```

### 服务器响应

```php
<?php
// 必须指定具体源，不能使用 *
header('Access-Control-Allow-Origin: https://app.example.com');
header('Access-Control-Allow-Credentials: true');
header('Access-Control-Allow-Methods: GET, POST, PUT, DELETE');
header('Access-Control-Allow-Headers: Content-Type, Authorization');
```

## 响应头暴露

### Access-Control-Expose-Headers

```php
<?php
// 默认暴露的头：Cache-Control、Content-Language、Content-Type、Expires、Last-Modified、Pragma

// 暴露自定义头
header('Access-Control-Expose-Headers: X-Request-ID, X-Response-Time, X-Total-Count');

// 客户端可以访问这些头
// response.headers.get('X-Request-ID')
```

## 实际应用示例

### API 端点配置

```php
<?php
declare(strict_types=1);

require __DIR__ . '/vendor/autoload.php';

// CORS 处理
$cors = new CorsMiddleware(
    allowedOrigins: ['http://localhost:3000', 'https://app.example.com'],
    allowedMethods: ['GET', 'POST', 'PUT', 'DELETE', 'OPTIONS'],
    allowedHeaders: ['Content-Type', 'Authorization', 'X-Requested-With'],
    allowCredentials: true
);
$cors->handle();

// API 路由
$method = $_SERVER['REQUEST_METHOD'];
$path = parse_url($_SERVER['REQUEST_URI'], PHP_URL_PATH);

if ($path === '/api/users') {
    match ($method) {
        'GET' => getUsers(),
        'POST' => createUser(),
        default => sendError('Method not allowed', 405),
    };
}

function getUsers(): void
{
    header('Content-Type: application/json; charset=UTF-8');
    header('X-Total-Count: 100');
    
    echo json_encode([
        'data' => [
            ['id' => 1, 'name' => 'Alice'],
            ['id' => 2, 'name' => 'Bob'],
        ],
    ], JSON_UNESCAPED_UNICODE);
}

function createUser(): void
{
    $data = json_decode(file_get_contents('php://input'), true);
    
    // 处理创建逻辑...
    
    http_response_code(201);
    header('Content-Type: application/json; charset=UTF-8');
    header('Location: /api/users/1');
    
    echo json_encode(['data' => ['id' => 1, 'name' => $data['name']]], JSON_UNESCAPED_UNICODE);
}
```

## 最佳实践

### 1. 明确指定允许的源

```php
// 不推荐
header('Access-Control-Allow-Origin: *');

// 推荐
$allowedOrigins = ['https://app.example.com'];
if (in_array($origin, $allowedOrigins, true)) {
    header("Access-Control-Allow-Origin: {$origin}");
}
```

### 2. 限制允许的方法和头

```php
// 只允许必要的方法
header('Access-Control-Allow-Methods: GET, POST');

// 只允许必要的头
header('Access-Control-Allow-Headers: Content-Type, Authorization');
```

### 3. 使用预检请求缓存

```php
// 减少预检请求频率
header('Access-Control-Max-Age: 3600');
```

### 4. 安全考虑

- 生产环境不要使用 `*` 作为允许的源。
- 使用 HTTPS 传输敏感数据。
- 验证 Origin 头，防止 CSRF 攻击。

## 练习

1. 创建一个 CORS 中间件类，支持配置允许的源、方法、头等。

2. 实现一个 API 端点，正确处理预检请求和实际请求。

3. 编写一个响应头管理类，统一设置 Content-Type、Cache-Control 等常用头。

4. 设计一个支持带凭证的跨域请求系统，正确处理 Cookie 和认证信息。

5. 实现一个动态 CORS 配置系统，根据请求的 Origin 动态决定是否允许访问。

6. 创建一个 API 网关，统一处理所有 API 请求的 CORS 配置。
