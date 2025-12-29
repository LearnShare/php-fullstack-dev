# 5.1.1 HTTP 请求响应流程

## 概述

HTTP（HyperText Transfer Protocol，超文本传输协议）是 Web 应用的基础协议，定义了客户端和服务器之间如何交换信息。理解 HTTP 请求响应流程是进行 Web 开发的基础，本节详细介绍 HTTP 协议的基本概念、请求和响应的结构、完整的请求生命周期，帮助零基础学员理解 Web 交互的基本原理。

HTTP 协议采用请求-响应模型：客户端（通常是浏览器）发送 HTTP 请求到服务器，服务器处理请求后返回 HTTP 响应。每个请求都是独立的，服务器不会记住之前的请求（无状态协议）。这种设计使得 HTTP 协议简单、高效，适合分布式系统。

在 PHP Web 开发中，理解 HTTP 请求响应流程对于正确处理用户请求、生成响应、调试问题至关重要。本节将从 HTTP 协议基础开始，逐步介绍请求结构、响应结构、HTTP 方法、状态码，以及完整的请求生命周期。

**主要内容**：
- HTTP 协议概述和版本演进
- HTTP 请求结构（请求行、请求头、请求体）
- HTTP 响应结构（状态行、响应头、响应体）
- HTTP 方法（GET、POST、PUT、DELETE 等）及其使用场景
- HTTP 状态码的分类和含义
- 完整的请求生命周期（从客户端到服务器再到客户端）
- 在 PHP 中查看和处理 HTTP 请求响应
- 实际应用示例和最佳实践

## 特性

- **请求-响应模型**：客户端发送请求，服务器返回响应
- **无状态协议**：每个请求都是独立的，服务器不保存客户端状态
- **文本协议**：HTTP 消息是文本格式，便于阅读和调试
- **可扩展性**：通过请求头和响应头扩展功能
- **跨平台**：支持不同操作系统和编程语言
- **分层设计**：基于 TCP/IP 协议栈，可以运行在任何网络之上

## HTTP 协议概述

### HTTP 协议的作用

HTTP 协议定义了客户端和服务器之间通信的规则。它规定了：
- 如何格式化请求和响应消息
- 如何传输数据
- 如何处理错误
- 如何管理连接

HTTP 协议是应用层协议，运行在传输层协议（通常是 TCP）之上。客户端通过 TCP 连接到服务器，然后发送 HTTP 请求，服务器处理请求后返回 HTTP 响应。

### HTTP 版本演进

#### HTTP/1.0

- **发布时间**：1996 年
- **特点**：每个请求需要建立新的 TCP 连接
- **问题**：连接开销大，性能较差

#### HTTP/1.1

- **发布时间**：1997 年（当前主流版本）
- **特点**：
  - 支持持久连接（Keep-Alive）
  - 支持管道化（Pipelining）
  - 支持分块传输编码（Chunked Transfer Encoding）
  - 支持虚拟主机（Host 头）
- **优势**：减少连接开销，提高性能

#### HTTP/2

- **发布时间**：2015 年
- **特点**：
  - 多路复用（Multiplexing）
  - 服务器推送（Server Push）
  - 头部压缩（Header Compression）
  - 二进制分帧（Binary Framing）
- **优势**：显著提升性能，减少延迟

#### HTTP/3

- **发布时间**：2022 年
- **特点**：
  - 基于 QUIC 协议（UDP）
  - 改进的连接建立
  - 更好的移动网络支持
- **优势**：进一步减少延迟，提升移动网络性能

### 客户端-服务器模型

HTTP 协议采用客户端-服务器模型：

1. **客户端**（Client）：
   - 通常是 Web 浏览器
   - 也可以是移动应用、API 客户端等
   - 发起 HTTP 请求

2. **服务器**（Server）：
   - 接收和处理 HTTP 请求
   - 返回 HTTP 响应
   - 可以是 Apache、Nginx、PHP-FPM 等

3. **通信流程**：
   ```
   客户端 → 发送请求 → 服务器
   客户端 ← 返回响应 ← 服务器
   ```

## HTTP 请求结构

HTTP 请求由三部分组成：请求行（Request Line）、请求头（Request Headers）、请求体（Request Body）。

### 请求行

请求行包含三个部分：HTTP 方法、请求 URI、协议版本。

**格式**：`METHOD /path/to/resource HTTP/1.1`

**示例**：
```http
GET /api/users HTTP/1.1
POST /api/users HTTP/1.1
PUT /api/users/123 HTTP/1.1
DELETE /api/users/123 HTTP/1.1
```

**组成部分**：
- **HTTP 方法**：表示请求的操作类型（GET、POST、PUT、DELETE 等）
- **请求 URI**：资源的路径，可以是绝对路径或相对路径
- **协议版本**：使用的 HTTP 版本（如 HTTP/1.1）

### 请求头

请求头包含关于请求的元数据信息，每行一个键值对。

**格式**：`Header-Name: Header-Value`

**常见请求头**：

| 请求头 | 说明 | 示例 |
|:-------|:-----|:-----|
| `Host` | 服务器的主机名和端口号 | `Host: example.com` |
| `User-Agent` | 客户端信息（浏览器、操作系统等） | `User-Agent: Mozilla/5.0` |
| `Accept` | 客户端接受的内容类型 | `Accept: application/json` |
| `Content-Type` | 请求体的内容类型 | `Content-Type: application/json` |
| `Content-Length` | 请求体的长度（字节） | `Content-Length: 123` |
| `Authorization` | 认证信息 | `Authorization: Bearer token123` |
| `Cookie` | 客户端发送的 Cookie | `Cookie: session_id=abc123` |

**完整请求头示例**：
```http
Host: example.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)
Accept: application/json, text/html
Accept-Language: zh-CN,zh;q=0.9
Content-Type: application/json
Content-Length: 45
```

### 请求体

请求体包含要发送给服务器的数据，通常用于 POST、PUT、PATCH 等请求。

**常见内容类型**：
- `application/json`：JSON 格式数据
- `application/x-www-form-urlencoded`：表单数据
- `multipart/form-data`：文件上传
- `text/plain`：纯文本

**示例**（JSON 格式）：
```json
{
  "name": "John Doe",
  "email": "john@example.com"
}
```

### 完整 HTTP 请求示例

**GET 请求**（无请求体）：
```http
GET /api/users?page=1&limit=10 HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0
Accept: application/json
```

**POST 请求**（有请求体）：
```http
POST /api/users HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0
Content-Type: application/json
Content-Length: 45

{"name": "John Doe", "email": "john@example.com"}
```

## HTTP 响应结构

HTTP 响应由三部分组成：状态行（Status Line）、响应头（Response Headers）、响应体（Response Body）。

### 状态行

状态行包含三个部分：协议版本、状态码、状态文本。

**格式**：`HTTP/1.1 200 OK`

**组成部分**：
- **协议版本**：使用的 HTTP 版本
- **状态码**：三位数字，表示请求的处理结果
- **状态文本**：状态码的文本描述

**示例**：
```http
HTTP/1.1 200 OK
HTTP/1.1 404 Not Found
HTTP/1.1 500 Internal Server Error
```

### 响应头

响应头包含关于响应的元数据信息，格式与请求头相同。

**常见响应头**：

| 响应头 | 说明 | 示例 |
|:-------|:-----|:-----|
| `Content-Type` | 响应体的内容类型 | `Content-Type: application/json` |
| `Content-Length` | 响应体的长度（字节） | `Content-Length: 123` |
| `Set-Cookie` | 服务器设置的 Cookie | `Set-Cookie: session_id=abc123` |
| `Location` | 重定向的目标 URL | `Location: /login` |
| `Cache-Control` | 缓存控制指令 | `Cache-Control: no-cache` |
| `Server` | 服务器信息 | `Server: nginx/1.18.0` |

**完整响应头示例**：
```http
Content-Type: application/json
Content-Length: 123
Server: nginx/1.18.0
Date: Mon, 29 Dec 2025 08:00:00 GMT
```

### 响应体

响应体包含服务器返回的实际数据，可以是 HTML、JSON、XML、图片等任何格式。

**示例**（JSON 格式）：
```json
{
  "status": "success",
  "data": {
    "id": 1,
    "name": "John Doe",
    "email": "john@example.com"
  }
}
```

### 完整 HTTP 响应示例

**成功响应**：
```http
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 123
Server: nginx/1.18.0
Date: Mon, 29 Dec 2025 08:00:00 GMT

{"status": "success", "data": [...]}
```

**错误响应**：
```http
HTTP/1.1 404 Not Found
Content-Type: application/json
Content-Length: 45
Server: nginx/1.18.0
Date: Mon, 29 Dec 2025 08:00:00 GMT

{"status": "error", "message": "Resource not found"}
```

## HTTP 方法

HTTP 方法（也称为 HTTP 动词）表示要对资源执行的操作。不同的方法有不同的语义和用途。

### GET

**作用**：获取资源

**特点**：
- 幂等性：多次请求返回相同结果
- 安全性：不修改服务器状态
- 可缓存：响应可以被缓存

**使用场景**：
- 获取用户列表
- 获取单个用户信息
- 搜索资源

**示例**：
```http
GET /api/users HTTP/1.1
Host: example.com
```

### POST

**作用**：创建资源

**特点**：
- 非幂等性：多次请求可能产生不同结果
- 非安全性：会修改服务器状态
- 不可缓存：响应通常不被缓存

**使用场景**：
- 创建新用户
- 提交表单
- 上传文件

**示例**：
```http
POST /api/users HTTP/1.1
Host: example.com
Content-Type: application/json

{"name": "John Doe", "email": "john@example.com"}
```

### PUT

**作用**：更新资源（完整更新）

**特点**：
- 幂等性：多次请求产生相同结果
- 非安全性：会修改服务器状态
- 不可缓存：响应通常不被缓存

**使用场景**：
- 完整更新用户信息
- 替换资源

**示例**：
```http
PUT /api/users/123 HTTP/1.1
Host: example.com
Content-Type: application/json

{"name": "Jane Doe", "email": "jane@example.com"}
```

### PATCH

**作用**：更新资源（部分更新）

**特点**：
- 非幂等性：多次请求可能产生不同结果
- 非安全性：会修改服务器状态
- 不可缓存：响应通常不被缓存

**使用场景**：
- 部分更新用户信息
- 修改资源的特定字段

**示例**：
```http
PATCH /api/users/123 HTTP/1.1
Host: example.com
Content-Type: application/json

{"email": "newemail@example.com"}
```

### DELETE

**作用**：删除资源

**特点**：
- 幂等性：多次请求产生相同结果
- 非安全性：会修改服务器状态
- 不可缓存：响应通常不被缓存

**使用场景**：
- 删除用户
- 删除资源

**示例**：
```http
DELETE /api/users/123 HTTP/1.1
Host: example.com
```

### 其他 HTTP 方法

| 方法 | 作用 | 使用场景 |
|:-----|:-----|:---------|
| `HEAD` | 获取资源的元数据（不返回响应体） | 检查资源是否存在 |
| `OPTIONS` | 获取服务器支持的 HTTP 方法 | CORS 预检请求 |
| `TRACE` | 回显服务器收到的请求 | 调试和诊断 |
| `CONNECT` | 建立隧道连接 | HTTPS 代理 |

## HTTP 状态码

HTTP 状态码是三位数字，表示请求的处理结果。状态码分为五类：

### 1xx 信息性状态码

表示请求已接收，继续处理。

| 状态码 | 说明 | 使用场景 |
|:-------|:-----|:---------|
| 100 | Continue | 客户端应继续发送请求体 |
| 101 | Switching Protocols | 服务器同意切换协议 |

### 2xx 成功状态码

表示请求已成功处理。

| 状态码 | 说明 | 使用场景 |
|:-------|:-----|:---------|
| 200 | OK | 请求成功，返回数据 |
| 201 | Created | 资源创建成功 |
| 202 | Accepted | 请求已接受，但尚未处理完成 |
| 204 | No Content | 请求成功，但无返回内容 |
| 206 | Partial Content | 部分内容（用于范围请求） |

### 3xx 重定向状态码

表示需要进一步操作才能完成请求。

| 状态码 | 说明 | 使用场景 |
|:-------|:-----|:---------|
| 301 | Moved Permanently | 资源永久移动 |
| 302 | Found | 资源临时移动 |
| 304 | Not Modified | 资源未修改（使用缓存） |
| 307 | Temporary Redirect | 临时重定向 |
| 308 | Permanent Redirect | 永久重定向 |

### 4xx 客户端错误状态码

表示客户端请求有误。

| 状态码 | 说明 | 使用场景 |
|:-------|:-----|:---------|
| 400 | Bad Request | 请求格式错误 |
| 401 | Unauthorized | 未认证 |
| 403 | Forbidden | 无权限 |
| 404 | Not Found | 资源不存在 |
| 405 | Method Not Allowed | HTTP 方法不允许 |
| 409 | Conflict | 资源冲突 |
| 422 | Unprocessable Entity | 请求格式正确但语义错误 |
| 429 | Too Many Requests | 请求过于频繁 |

### 5xx 服务器错误状态码

表示服务器处理请求时出错。

| 状态码 | 说明 | 使用场景 |
|:-------|:-----|:---------|
| 500 | Internal Server Error | 服务器内部错误 |
| 502 | Bad Gateway | 网关错误 |
| 503 | Service Unavailable | 服务不可用 |
| 504 | Gateway Timeout | 网关超时 |

## 请求生命周期

HTTP 请求的完整生命周期包括以下步骤：

### 1. 客户端发起请求

客户端（浏览器、API 客户端等）构造 HTTP 请求：
- 确定请求方法（GET、POST 等）
- 确定请求 URI
- 添加请求头
- 添加请求体（如果需要）

### 2. 建立 TCP 连接

客户端通过 TCP 连接到服务器：
- 解析服务器域名（DNS 查询）
- 建立 TCP 连接（三次握手）
- 如果是 HTTPS，进行 TLS 握手

### 3. 发送 HTTP 请求

客户端通过 TCP 连接发送 HTTP 请求：
- 发送请求行
- 发送请求头
- 发送请求体（如果有）

### 4. 服务器接收请求

服务器接收并解析 HTTP 请求：
- Web 服务器（如 Nginx）接收请求
- 根据配置转发给 PHP-FPM
- PHP-FPM 处理请求

### 5. 服务器处理请求

PHP 应用处理请求：
- 解析请求参数
- 执行业务逻辑
- 访问数据库（如果需要）
- 生成响应数据

### 6. 服务器返回响应

服务器构造并发送 HTTP 响应：
- 设置状态码
- 设置响应头
- 设置响应体
- 通过 TCP 连接发送响应

### 7. 客户端接收响应

客户端接收并处理 HTTP 响应：
- 接收响应行
- 接收响应头
- 接收响应体
- 根据内容类型处理响应

### 8. 关闭连接

根据连接类型决定是否关闭：
- HTTP/1.0：立即关闭连接
- HTTP/1.1：根据 `Connection` 头决定（Keep-Alive 保持连接）

## 在 PHP 中查看 HTTP 请求

PHP 提供了多种方式查看和处理 HTTP 请求信息。

### 查看请求方法

```php
<?php
declare(strict_types=1);

// 获取请求方法
$method = $_SERVER['REQUEST_METHOD'] ?? 'GET';
echo "Request Method: {$method}\n";
```

**输出**：
```
Request Method: GET
```

### 查看请求 URI

```php
<?php
declare(strict_types=1);

// 获取请求 URI
$uri = $_SERVER['REQUEST_URI'] ?? '/';
echo "Request URI: {$uri}\n";

// 获取查询字符串
$queryString = $_SERVER['QUERY_STRING'] ?? '';
echo "Query String: {$queryString}\n";
```

**输出**（访问 `/api/users?page=1`）：
```
Request URI: /api/users
Query String: page=1
```

### 查看请求头

```php
<?php
declare(strict_types=1);

// 获取所有请求头
$headers = getallheaders();
foreach ($headers as $name => $value) {
    echo "{$name}: {$value}\n";
}
```

**输出**：
```
Host: example.com
User-Agent: Mozilla/5.0
Accept: application/json
```

### 查看请求体

```php
<?php
declare(strict_types=1);

// 获取原始请求体
$rawBody = file_get_contents('php://input');
echo "Raw Body: {$rawBody}\n";

// 解析 JSON 请求体
if (!empty($rawBody)) {
    $data = json_decode($rawBody, true);
    if (json_last_error() === JSON_ERROR_NONE) {
        print_r($data);
    }
}
```

**输出**（POST 请求，JSON 格式）：
```
Raw Body: {"name":"John Doe","email":"john@example.com"}
Array
(
    [name] => John Doe
    [email] => john@example.com
)
```

## 在 PHP 中发送 HTTP 响应

PHP 提供了多种方式发送 HTTP 响应。

### 设置状态码

```php
<?php
declare(strict_types=1);

// 设置 HTTP 状态码
http_response_code(200);
// 或
http_response_code(404);
```

### 设置响应头

```php
<?php
declare(strict_types=1);

// 设置响应头
header('Content-Type: application/json');
header('Cache-Control: no-cache');
header('X-Custom-Header: custom-value');
```

### 发送响应体

```php
<?php
declare(strict_types=1);

// 发送 JSON 响应
$data = [
    'status' => 'success',
    'data' => [
        'id' => 1,
        'name' => 'John Doe'
    ]
];

header('Content-Type: application/json');
http_response_code(200);
echo json_encode($data, JSON_UNESCAPED_UNICODE);
```

**输出**（浏览器显示）：
```json
{"status":"success","data":{"id":1,"name":"John Doe"}}
```

### 完整响应示例

```php
<?php
declare(strict_types=1);

// 设置响应头
header('Content-Type: application/json');
header('X-Request-ID: ' . uniqid());

// 设置状态码
http_response_code(200);

// 发送响应体
$response = [
    'status' => 'success',
    'message' => 'Request processed successfully',
    'data' => [
        'timestamp' => time()
    ]
];

echo json_encode($response, JSON_UNESCAPED_UNICODE | JSON_PRETTY_PRINT);
```

**输出**（浏览器显示）：
```json
{
    "status": "success",
    "message": "Request processed successfully",
    "data": {
        "timestamp": 1703846400
    }
}
```

## 使用场景

### Web 应用开发

- 处理用户请求
- 生成 HTML 响应
- 处理表单提交
- 实现页面跳转

### API 开发

- 设计 RESTful API
- 处理 JSON 请求和响应
- 实现 API 认证和授权
- 处理 API 错误

### 调试和问题排查

- 查看请求和响应内容
- 分析网络问题
- 优化性能
- 排查错误

## 注意事项

### HTTP 是无状态协议

- 每个请求都是独立的
- 服务器不保存客户端状态
- 需要使用 Cookie、Session 等技术维护状态

### 请求和响应的格式要求

- 请求行和状态行必须以 `\r\n` 结尾
- 请求头和响应头必须以 `\r\n` 结尾
- 请求头和响应体之间必须有空行
- 请求体和响应体可以是任意内容

### 状态码的选择

- 正确使用状态码表示请求结果
- 2xx 表示成功
- 4xx 表示客户端错误
- 5xx 表示服务器错误

### 协议版本的选择

- HTTP/1.1 是当前主流版本
- HTTP/2 和 HTTP/3 需要服务器和客户端支持
- 根据实际需求选择合适的版本

### 安全性考虑

- 使用 HTTPS 保护数据传输
- 验证和过滤用户输入
- 防止 CSRF 攻击
- 正确处理敏感信息

## 常见问题

### HTTP 请求的组成部分？

HTTP 请求由三部分组成：
1. **请求行**：包含方法、URI、协议版本
2. **请求头**：包含请求的元数据
3. **请求体**：包含要发送的数据（可选）

### HTTP 状态码的含义？

HTTP 状态码分为五类：
- **1xx**：信息性状态码
- **2xx**：成功状态码
- **3xx**：重定向状态码
- **4xx**：客户端错误状态码
- **5xx**：服务器错误状态码

### GET 和 POST 的区别？

| 特性 | GET | POST |
|:-----|:----|:-----|
| 用途 | 获取资源 | 创建资源 |
| 数据位置 | URL 查询字符串 | 请求体 |
| 数据大小限制 | 受 URL 长度限制 | 无限制（受服务器配置限制） |
| 安全性 | 数据在 URL 中可见 | 数据在请求体中 |
| 缓存 | 可缓存 | 不可缓存 |
| 幂等性 | 幂等 | 非幂等 |

### 如何查看 HTTP 请求响应？

**浏览器开发者工具**：
1. 打开浏览器开发者工具（F12）
2. 切换到 "Network" 标签
3. 刷新页面或发起请求
4. 点击请求查看详细信息

**命令行工具**：
```bash
# 使用 curl
curl -v http://example.com/api/users

# 使用 httpie
http GET http://example.com/api/users
```

**PHP 代码**：
```php
<?php
declare(strict_types=1);

// 查看所有请求信息
print_r($_SERVER);
print_r($_GET);
print_r($_POST);
```

## 最佳实践

### 理解 HTTP 协议基础

- 掌握 HTTP 请求和响应的结构
- 理解不同 HTTP 方法的使用场景
- 正确使用 HTTP 状态码

### 使用合适的 HTTP 方法

- GET 用于获取资源
- POST 用于创建资源
- PUT 用于完整更新资源
- PATCH 用于部分更新资源
- DELETE 用于删除资源

### 正确设置响应头

- 设置 `Content-Type` 指定响应格式
- 设置 `Cache-Control` 控制缓存
- 设置安全相关的响应头（如 `X-Content-Type-Options`）

### 遵循 HTTP 规范

- 使用标准的状态码
- 遵循 RESTful 设计原则
- 正确处理错误情况

### 安全性考虑

- 使用 HTTPS 保护数据传输
- 验证和过滤所有用户输入
- 防止常见安全漏洞（XSS、CSRF 等）

## 相关章节

- **[5.1.2 Web Server 与 PHP-FPM 协作](section-02-web-server-fpm.md)**：了解 Web 服务器与 PHP-FPM 的协作机制
- **[5.3 超全局变量：Web 输入的核心](../chapter-03-superglobals/readme.md)**：了解如何在 PHP 中访问 HTTP 请求数据
- **[5.8 响应处理与跨域（CORS）](../chapter-08-response-cors/readme.md)**：了解如何处理 HTTP 响应和跨域问题
- **[2.1.4 文件执行方式](../../stage-02-language/chapter-01-syntax/section-04-execution.md)**：了解 PHP 的不同执行模式

## 练习任务

1. **查看 HTTP 请求信息**
   - 创建一个 PHP 脚本，显示所有 HTTP 请求信息（方法、URI、请求头等）
   - 使用浏览器访问脚本，观察输出

2. **发送不同的 HTTP 响应**
   - 创建一个 PHP 脚本，根据不同的条件返回不同的状态码（200、404、500）
   - 使用浏览器和命令行工具测试

3. **处理不同的 HTTP 方法**
   - 创建一个 PHP 脚本，根据请求方法（GET、POST、PUT、DELETE）执行不同的逻辑
   - 使用不同的工具测试各种方法

4. **实现简单的 API**
   - 创建一个简单的 RESTful API，支持 GET、POST、PUT、DELETE 方法
   - 返回 JSON 格式的响应
   - 使用浏览器开发者工具查看请求和响应

5. **分析实际网站的 HTTP 请求**
   - 打开浏览器开发者工具
   - 访问一个网站，查看网络请求
   - 分析请求方法、请求头、响应状态码、响应头等信息
