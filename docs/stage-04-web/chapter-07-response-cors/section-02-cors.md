# 4.7.2 CORS 跨域处理

## 概述

CORS（Cross-Origin Resource Sharing）是处理跨域请求的标准机制。本节详细介绍 CORS 机制、预检请求（Preflight）处理、CORS 配置，以及安全最佳实践。

## CORS 机制

### 什么是跨域

- **同源策略**：浏览器限制不同源之间的资源访问
- **同源定义**：协议、域名、端口完全相同
- **跨域场景**：前端 `http://localhost:3000` 访问后端 `http://api.example.com`

### CORS 工作原理

```
浏览器发送请求
  ↓
服务器返回 CORS Headers
  ↓
浏览器检查 Headers
  ↓
允许/拒绝请求
```

## CORS 响应头

### 必需响应头

```php
<?php
// 允许的源
header('Access-Control-Allow-Origin: http://localhost:3000');

// 允许的方法
header('Access-Control-Allow-Methods: GET, POST, PUT, DELETE, OPTIONS');

// 允许的 Headers
header('Access-Control-Allow-Headers: Content-Type, Authorization');

// 允许携带凭证
header('Access-Control-Allow-Credentials: true');

// 预检请求缓存时间
header('Access-Control-Max-Age: 3600');
```

### 完整 CORS 配置

```php
<?php
declare(strict_types=1);

class CorsHandler
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

        // 检查源是否允许
        if ($this->isOriginAllowed($origin)) {
            header("Access-Control-Allow-Origin: {$origin}");
        } elseif (in_array('*', $this->allowedOrigins, true)) {
            header('Access-Control-Allow-Origin: *');
        }

        header('Access-Control-Allow-Methods: ' . implode(', ', $this->allowedMethods));
        header('Access-Control-Allow-Headers: ' . implode(', ', $this->allowedHeaders));
        header('Access-Control-Max-Age: ' . $this->maxAge);

        if ($this->allowCredentials) {
            header('Access-Control-Allow-Credentials: true');
        }

        // 处理预检请求
        if ($_SERVER['REQUEST_METHOD'] === 'OPTIONS') {
            http_response_code(204);
            exit;
        }
    }

    private function isOriginAllowed(string $origin): bool
    {
        return in_array($origin, $this->allowedOrigins, true);
    }
}
```

## 预检请求（Preflight）

### 什么是预检请求

- **触发条件**：复杂请求（非简单请求）
- **请求方法**：OPTIONS
- **目的**：询问服务器是否允许跨域请求

### 简单请求 vs 复杂请求

**简单请求**：
- 方法：GET、POST、HEAD
- Headers：仅允许简单 Headers
- 无需预检

**复杂请求**：
- 方法：PUT、DELETE、PATCH
- Headers：自定义 Headers
- 需要预检

### 预检请求处理

```php
<?php
declare(strict_types=1);

// 处理预检请求
if ($_SERVER['REQUEST_METHOD'] === 'OPTIONS') {
    header('Access-Control-Allow-Origin: http://localhost:3000');
    header('Access-Control-Allow-Methods: GET, POST, PUT, DELETE');
    header('Access-Control-Allow-Headers: Content-Type, Authorization');
    header('Access-Control-Max-Age: 3600');
    http_response_code(204);
    exit;
}

// 处理实际请求
header('Access-Control-Allow-Origin: http://localhost:3000');
// ... 业务逻辑
```

## 安全最佳实践

### 1. 限制允许的源

```php
<?php
// [不推荐] 允许所有源
header('Access-Control-Allow-Origin: *');

// [推荐] 明确指定允许的源
$allowedOrigins = [
    'https://app.example.com',
    'https://admin.example.com',
];

$origin = $_SERVER['HTTP_ORIGIN'] ?? '';
if (in_array($origin, $allowedOrigins, true)) {
    header("Access-Control-Allow-Origin: {$origin}");
}
```

### 2. 限制允许的方法

```php
<?php
// 只允许必要的方法
header('Access-Control-Allow-Methods: GET, POST');
```

### 3. 限制允许的 Headers

```php
<?php
// 只允许必要的 Headers
header('Access-Control-Allow-Headers: Content-Type, Authorization');
```

## 完整示例

```php
<?php
declare(strict_types=1);

class CorsMiddleware
{
    public function handle(): void
    {
        $origin = $_SERVER['HTTP_ORIGIN'] ?? '';
        
        // 检查并设置 CORS Headers
        if ($this->isAllowedOrigin($origin)) {
            header("Access-Control-Allow-Origin: {$origin}");
            header('Access-Control-Allow-Credentials: true');
        }
        
        header('Access-Control-Allow-Methods: GET, POST, PUT, DELETE, OPTIONS');
        header('Access-Control-Allow-Headers: Content-Type, Authorization, X-Requested-With');
        header('Access-Control-Max-Age: 3600');
        
        // 处理预检请求
        if ($_SERVER['REQUEST_METHOD'] === 'OPTIONS') {
            http_response_code(204);
            exit;
        }
    }

    private function isAllowedOrigin(string $origin): bool
    {
        $allowed = [
            'https://app.example.com',
            'https://admin.example.com',
        ];
        return in_array($origin, $allowed, true);
    }
}

// 使用
$cors = new CorsMiddleware();
$cors->handle();
```

## 注意事项

1. **安全性**：不要使用 `*` 允许所有源，明确指定允许的源
2. **预检请求**：正确处理 OPTIONS 请求
3. **凭证**：使用 `Access-Control-Allow-Credentials` 时不能使用 `*`
4. **缓存**：合理设置 `Access-Control-Max-Age` 减少预检请求

## 练习

1. 实现一个 CORS 中间件，支持配置允许的源、方法、Headers。

2. 正确处理预检请求，返回适当的响应。

3. 创建一个 CORS 配置类，支持不同环境的配置。

4. 实现动态源验证，根据请求来源动态设置 CORS Headers。
