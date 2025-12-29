# 5.8.2 CORS 跨域处理

## 概述

CORS（Cross-Origin Resource Sharing，跨域资源共享）是处理跨域请求的机制。由于浏览器的同源策略，默认情况下，JavaScript 只能访问同源的资源。CORS 允许服务器明确指定哪些源可以访问其资源，从而安全地实现跨域请求。理解 CORS 的概念、掌握 CORS 的配置方法、正确处理预检请求，对于构建前后端分离的应用和提供跨域 API 服务至关重要。本节详细介绍 CORS 的概念、同源策略、CORS 请求流程、响应头设置、预检请求处理等内容，帮助零基础学员理解并实现 CORS 跨域处理。

CORS 是现代 Web 开发中的重要技术，特别是在前后端分离架构中。理解 CORS 的工作原理，正确配置 CORS 响应头，对于构建可用的跨域应用至关重要。

**主要内容**：
- CORS 概念（什么是 CORS、为什么需要 CORS、同源策略限制）
- 同源策略（同源的定义、跨域的场景、安全考虑）
- CORS 请求流程（简单请求、预检请求、实际请求）
- CORS 响应头（Access-Control-Allow-Origin、Access-Control-Allow-Methods、Access-Control-Allow-Headers、Access-Control-Allow-Credentials）
- 预检请求处理（OPTIONS 方法、预检请求响应、预检请求缓存）
- CORS 配置（允许的源、允许的方法、允许的头部、凭据传递）
- 实际应用示例和最佳实践

## 特性

- **安全控制**：允许服务器控制跨域访问
- **灵活配置**：支持细粒度的访问控制
- **标准协议**：遵循 W3C CORS 标准
- **浏览器支持**：现代浏览器都支持 CORS
- **预检机制**：通过预检请求确保安全

## CORS 概念

### 什么是 CORS

CORS（Cross-Origin Resource Sharing）是一种机制，允许 Web 服务器明确指定哪些源（协议、域名、端口）可以访问其资源。

### 为什么需要 CORS

**问题**：浏览器的同源策略阻止跨域请求。

**解决方案**：CORS 允许服务器明确指定允许的源，从而安全地实现跨域访问。

### 同源策略限制

**同源策略**：浏览器只允许 JavaScript 访问同源的资源。

**同源定义**：协议、域名、端口都相同。

**示例**：
- `https://example.com` 和 `https://example.com` → 同源 ✅
- `https://example.com` 和 `http://example.com` → 不同源 ❌（协议不同）
- `https://example.com` 和 `https://api.example.com` → 不同源 ❌（域名不同）
- `https://example.com:80` 和 `https://example.com:443` → 不同源 ❌（端口不同）

## 同源策略

### 同源的定义

**同源**：协议、域名、端口完全相同。

**示例**：
```
https://example.com:443/page1.html
https://example.com:443/page2.html
→ 同源 ✅

https://example.com/page1.html
http://example.com/page2.html
→ 不同源 ❌（协议不同：https vs http）

https://example.com/page1.html
https://api.example.com/page2.html
→ 不同源 ❌（域名不同：example.com vs api.example.com）

https://example.com:443/page1.html
https://example.com:8080/page2.html
→ 不同源 ❌（端口不同：443 vs 8080）
```

### 跨域的场景

**常见跨域场景**：
1. **前后端分离**：前端 `http://localhost:3000`，后端 `http://localhost:8000`
2. **子域名访问**：`https://www.example.com` 访问 `https://api.example.com`
3. **不同协议**：`http://example.com` 访问 `https://example.com`
4. **不同端口**：`http://example.com:80` 访问 `http://example.com:8080`

### 安全考虑

**同源策略的作用**：
- 防止恶意网站访问用户数据
- 防止 CSRF 攻击
- 保护用户隐私

**CORS 的安全机制**：
- 服务器明确指定允许的源
- 预检请求验证复杂请求
- 凭据传递需要明确允许

## CORS 请求流程

### 简单请求

**简单请求的条件**：
1. 方法为：GET、POST、HEAD
2. 请求头只包含：Accept、Accept-Language、Content-Language、Content-Type
3. Content-Type 为：`text/plain`、`multipart/form-data`、`application/x-www-form-urlencoded`

**流程**：
1. 浏览器直接发送请求
2. 服务器返回响应（包含 CORS 响应头）
3. 浏览器检查 CORS 响应头
4. 如果允许，返回响应给 JavaScript；否则，阻止访问

**示例**：
```javascript
// 前端代码
fetch('https://api.example.com/users', {
    method: 'GET',
    headers: {
        'Content-Type': 'application/json'
    }
})
.then(response => response.json())
.then(data => console.log(data));
```

### 预检请求

**预检请求的条件**（不满足简单请求的条件）：
1. 方法为：PUT、DELETE、PATCH
2. 包含自定义请求头
3. Content-Type 为：`application/json`

**流程**：
1. 浏览器先发送 OPTIONS 预检请求
2. 服务器返回预检响应（包含 CORS 响应头）
3. 浏览器检查预检响应
4. 如果允许，发送实际请求；否则，阻止请求

**示例**：
```javascript
// 前端代码（会触发预检请求）
fetch('https://api.example.com/users', {
    method: 'PUT',
    headers: {
        'Content-Type': 'application/json',
        'X-Custom-Header': 'value'
    },
    body: JSON.stringify({name: 'John'})
})
.then(response => response.json())
.then(data => console.log(data));
```

### 实际请求

预检请求通过后，浏览器发送实际请求。

## CORS 响应头

### Access-Control-Allow-Origin

**作用**：指定允许访问资源的源。

**语法**：`Access-Control-Allow-Origin: <origin> | *`

**示例**：
```php
<?php
declare(strict_types=1);

// 允许所有源（不推荐，仅用于公开 API）
header('Access-Control-Allow-Origin: *');

// 允许特定源（推荐）
header('Access-Control-Allow-Origin: https://example.com');

// 动态允许多个源
$allowedOrigins = [
    'https://example.com',
    'https://www.example.com',
    'http://localhost:3000',
];

$origin = $_SERVER['HTTP_ORIGIN'] ?? '';
if (in_array($origin, $allowedOrigins, true)) {
    header("Access-Control-Allow-Origin: {$origin}");
}
```

### Access-Control-Allow-Methods

**作用**：指定允许的 HTTP 方法。

**语法**：`Access-Control-Allow-Methods: <method>[, <method>]*`

**示例**：
```php
<?php
declare(strict_types=1);

// 允许的方法
header('Access-Control-Allow-Methods: GET, POST, PUT, DELETE, PATCH');
```

### Access-Control-Allow-Headers

**作用**：指定允许的请求头。

**语法**：`Access-Control-Allow-Headers: <header>[, <header>]*`

**示例**：
```php
<?php
declare(strict_types=1);

// 允许的请求头
header('Access-Control-Allow-Headers: Content-Type, Authorization, X-Custom-Header');
```

### Access-Control-Allow-Credentials

**作用**：指定是否允许发送凭据（Cookie、Authorization 头等）。

**语法**：`Access-Control-Allow-Credentials: true`

**注意**：如果设置为 `true`，`Access-Control-Allow-Origin` 不能为 `*`，必须指定具体源。

**示例**：
```php
<?php
declare(strict_types=1);

$origin = $_SERVER['HTTP_ORIGIN'] ?? '';
$allowedOrigins = ['https://example.com'];

if (in_array($origin, $allowedOrigins, true)) {
    header("Access-Control-Allow-Origin: {$origin}");
    header('Access-Control-Allow-Credentials: true');
    header('Access-Control-Allow-Methods: GET, POST, PUT, DELETE');
    header('Access-Control-Allow-Headers: Content-Type, Authorization');
}
```

### Access-Control-Max-Age

**作用**：指定预检请求的缓存时间（秒）。

**语法**：`Access-Control-Max-Age: <seconds>`

**示例**：
```php
<?php
declare(strict_types=1);

// 预检请求缓存 1 小时
header('Access-Control-Max-Age: 3600');
```

## 预检请求处理

### OPTIONS 方法处理

**示例**：
```php
<?php
declare(strict_types=1);

// 处理预检请求
if ($_SERVER['REQUEST_METHOD'] === 'OPTIONS') {
    $origin = $_SERVER['HTTP_ORIGIN'] ?? '';
    $allowedOrigins = ['https://example.com', 'http://localhost:3000'];
    
    if (in_array($origin, $allowedOrigins, true)) {
        header("Access-Control-Allow-Origin: {$origin}");
        header('Access-Control-Allow-Methods: GET, POST, PUT, DELETE, PATCH');
        header('Access-Control-Allow-Headers: Content-Type, Authorization, X-Custom-Header');
        header('Access-Control-Allow-Credentials: true');
        header('Access-Control-Max-Age: 3600');
    }
    
    http_response_code(204);
    exit;
}

// 处理实际请求
$origin = $_SERVER['HTTP_ORIGIN'] ?? '';
$allowedOrigins = ['https://example.com', 'http://localhost:3000'];

if (in_array($origin, $allowedOrigins, true)) {
    header("Access-Control-Allow-Origin: {$origin}");
    header('Access-Control-Allow-Credentials: true');
}

// 业务逻辑
// ...
```

### 预检请求响应

**预检请求响应示例**：
```http
HTTP/1.1 204 No Content
Access-Control-Allow-Origin: https://example.com
Access-Control-Allow-Methods: GET, POST, PUT, DELETE, PATCH
Access-Control-Allow-Headers: Content-Type, Authorization
Access-Control-Allow-Credentials: true
Access-Control-Max-Age: 3600
```

### 预检请求缓存

**缓存机制**：
- 浏览器会缓存预检请求的响应
- 在 `Access-Control-Max-Age` 指定的时间内，不会重复发送预检请求
- 提高性能，减少请求次数

## CORS 配置

### 允许的源

**配置方法**：
```php
<?php
declare(strict_types=1);

class CorsHandler
{
    private array $allowedOrigins;

    public function __construct(array $allowedOrigins)
    {
        $this->allowedOrigins = $allowedOrigins;
    }

    public function handle(): void
    {
        $origin = $_SERVER['HTTP_ORIGIN'] ?? '';
        
        if (in_array($origin, $this->allowedOrigins, true)) {
            header("Access-Control-Allow-Origin: {$origin}");
        } elseif (in_array('*', $this->allowedOrigins, true)) {
            header('Access-Control-Allow-Origin: *');
        }
    }
}

// 使用
$cors = new CorsHandler([
    'https://example.com',
    'https://www.example.com',
    'http://localhost:3000',
]);
$cors->handle();
```

### 允许的方法

**配置方法**：
```php
<?php
declare(strict_types=1);

$allowedMethods = ['GET', 'POST', 'PUT', 'DELETE', 'PATCH'];
header('Access-Control-Allow-Methods: ' . implode(', ', $allowedMethods));
```

### 允许的头部

**配置方法**：
```php
<?php
declare(strict_types=1);

$allowedHeaders = [
    'Content-Type',
    'Authorization',
    'X-Custom-Header',
];
header('Access-Control-Allow-Headers: ' . implode(', ', $allowedHeaders));
```

### 凭据传递

**配置方法**：
```php
<?php
declare(strict_types=1);

$origin = $_SERVER['HTTP_ORIGIN'] ?? '';
$allowedOrigins = ['https://example.com'];

if (in_array($origin, $allowedOrigins, true)) {
    header("Access-Control-Allow-Origin: {$origin}");
    header('Access-Control-Allow-Credentials: true');
    header('Access-Control-Allow-Methods: GET, POST, PUT, DELETE');
    header('Access-Control-Allow-Headers: Content-Type, Authorization');
}
```

**前端代码**：
```javascript
fetch('https://api.example.com/users', {
    credentials: 'include',  // 发送 Cookie
    headers: {
        'Authorization': 'Bearer token'
    }
});
```

## 使用场景

### 前后端分离

- 前端应用访问后端 API
- 不同端口的开发环境
- 生产环境的不同域名

### API 跨域访问

- 第三方应用访问 API
- 移动应用访问 API
- 浏览器扩展访问 API

### 移动应用后端

- 移动应用访问 Web API
- 跨域资源共享
- 第三方集成

## 注意事项

### 安全性考虑

- **明确指定源**：不要使用 `*`，除非是公开 API
- **验证源**：验证请求来源是否在允许列表中
- **限制方法**：只允许必要的方法
- **限制头部**：只允许必要的请求头

### 通配符的使用

- **不推荐使用 `*`**：除非是公开 API
- **凭据传递**：使用 `*` 时不能传递凭据
- **安全性**：`*` 允许所有源访问，存在安全风险

### 凭据传递

- **需要明确允许**：设置 `Access-Control-Allow-Credentials: true`
- **不能使用 `*`**：使用凭据时，`Access-Control-Allow-Origin` 不能为 `*`
- **前端配置**：前端需要设置 `credentials: 'include'`

### 预检请求处理

- **必须处理 OPTIONS**：正确处理预检请求
- **返回 204**：预检请求通常返回 204 No Content
- **设置缓存**：使用 `Access-Control-Max-Age` 缓存预检请求

## 常见问题

### 什么是 CORS？

CORS（Cross-Origin Resource Sharing）是一种机制，允许 Web 服务器明确指定哪些源可以访问其资源，从而安全地实现跨域请求。

### 如何配置 CORS？

1. 设置 `Access-Control-Allow-Origin` 指定允许的源
2. 设置 `Access-Control-Allow-Methods` 指定允许的方法
3. 设置 `Access-Control-Allow-Headers` 指定允许的请求头
4. 处理 OPTIONS 预检请求

### 预检请求是什么？

预检请求是浏览器在发送复杂请求前发送的 OPTIONS 请求，用于验证服务器是否允许该跨域请求。

### 如何安全地配置 CORS？

1. **明确指定源**：不要使用 `*`，明确指定允许的源
2. **限制方法**：只允许必要的方法
3. **限制头部**：只允许必要的请求头
4. **验证来源**：验证请求来源是否在允许列表中

## 最佳实践

### 明确指定允许的源

- 不要使用 `*`（除非是公开 API）
- 明确列出允许的源
- 验证请求来源

### 限制允许的方法和头部

- 只允许必要的方法
- 只允许必要的请求头
- 最小权限原则

### 正确处理预检请求

- 处理 OPTIONS 方法
- 返回适当的响应头
- 设置预检请求缓存

### 考虑安全性

- 验证请求来源
- 限制访问范围
- 使用 HTTPS
- 考虑使用 Token 认证

## 相关章节

- **[5.8.1 响应处理](section-01-response-handling.md)**：了解响应处理的详细内容
- **[5.7 RESTful API 设计](../chapter-07-restful-api/readme.md)**：了解 API 设计的详细内容
- **[5.1.1 HTTP 请求响应流程](../chapter-01-request-response/section-01-http-flow.md)**：了解 HTTP 请求响应的基础内容

## 练习任务

1. **实现 CORS 处理类**
   - 创建 CORS 处理类
   - 配置允许的源、方法、头部
   - 处理预检请求

2. **实现动态源验证**
   - 验证请求来源
   - 动态设置允许的源
   - 处理多个允许的源

3. **实现预检请求处理**
   - 处理 OPTIONS 请求
   - 返回预检响应
   - 设置预检缓存

4. **实现凭据传递支持**
   - 配置凭据传递
   - 验证源和凭据
   - 测试凭据传递

5. **实现完整的 CORS 系统**
   - CORS 中间件
   - 配置管理
   - 安全验证
