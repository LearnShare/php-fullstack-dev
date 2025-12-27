# 4.3.2 HTTP 客户端编程

## 概述

HTTP 客户端编程是 Web 开发中的常见需求。在实际应用中，我们经常需要调用第三方 API、抓取网页内容、进行服务间通信等，这些都需要进行 HTTP 请求。PHP 提供了多种方式进行 HTTP 请求，包括 `file_get_contents()`、cURL 扩展、以及第三方库如 Guzzle。

理解不同 HTTP 客户端方法的特点、使用场景，以及如何进行错误处理和配置，对于构建健壮的 PHP 应用至关重要。

**主要内容**：
- HTTP 请求方法概述（GET、POST、PUT、DELETE 等）
- `file_get_contents()` 进行 HTTP 请求
- cURL 扩展的使用
- Guzzle HTTP 客户端库
- 请求头设置
- POST 请求处理
- 响应处理和错误处理
- 实际应用场景和最佳实践

## 特性

- **多种方式**：支持 `file_get_contents()`、cURL、Guzzle 等多种方式
- **功能丰富**：支持各种 HTTP 方法、请求头、超时等配置
- **错误处理**：提供完善的错误处理机制
- **SSL 支持**：支持 HTTPS 请求
- **灵活配置**：支持自定义请求选项

## 语法/定义

### curl_init() 函数

**语法**：`curl_init(?string $url = null): CurlHandle|false`

**参数**：
- `$url`：可选，请求的 URL

**返回值**：成功返回 cURL 句柄，失败返回 `false`。

### curl_setopt() 函数

**语法**：`curl_setopt(CurlHandle $handle, int $option, mixed $value): bool`

**参数**：
- `$handle`：cURL 句柄
- `$option`：cURL 选项常量
- `$value`：选项值

**返回值**：成功返回 `true`，失败返回 `false`。

### curl_exec() 函数

**语法**：`curl_exec(CurlHandle $handle): string|bool`

**参数**：
- `$handle`：cURL 句柄

**返回值**：成功返回响应内容，失败返回 `false`。

### curl_getinfo() 函数

**语法**：`curl_getinfo(CurlHandle $handle, ?int $option = null): mixed`

**参数**：
- `$handle`：cURL 句柄
- `$option`：可选，特定的信息选项

**返回值**：如果指定了 `$option`，返回对应的值；否则返回包含所有信息的数组。

## 基本用法

### 示例 1：使用 file_get_contents() 进行 GET 请求

```php
<?php
declare(strict_types=1);

// 简单 GET 请求
$content = file_get_contents('https://api.example.com/data');
if ($content === false) {
    throw new RuntimeException('Failed to fetch data');
}

echo $content;

// 带上下文的 GET 请求
$context = stream_context_create([
    'http' => [
        'method' => 'GET',
        'header' => [
            'User-Agent: MyApp/1.0',
            'Accept: application/json',
        ],
        'timeout' => 30,
    ],
]);

$content = file_get_contents('https://api.example.com/data', false, $context);
if ($content !== false) {
    echo $content;
}
```

**说明**：
- `file_get_contents()` 可以用于简单的 HTTP GET 请求
- 需要启用 `allow_url_fopen` 配置选项
- 可以使用流上下文设置请求选项

### 示例 2：使用 file_get_contents() 进行 POST 请求

```php
<?php
declare(strict_types=1);

// POST 请求
$data = json_encode(['name' => 'John', 'age' => 30]);

$context = stream_context_create([
    'http' => [
        'method' => 'POST',
        'header' => [
            'Content-Type: application/json',
            'Content-Length: ' . strlen($data),
        ],
        'content' => $data,
        'timeout' => 30,
    ],
]);

$response = file_get_contents('https://api.example.com/users', false, $context);
if ($response !== false) {
    echo $response;
}
```

**说明**：
- 使用流上下文设置 POST 方法和请求体
- 需要设置 `Content-Type` 和 `Content-Length` 头

### 示例 3：使用 cURL 进行 GET 请求

```php
<?php
declare(strict_types=1);

// 初始化 cURL
$ch = curl_init('https://api.example.com/data');
if ($ch === false) {
    throw new RuntimeException('Cannot initialize cURL');
}

// 设置选项
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);  // 返回响应内容而不是直接输出
curl_setopt($ch, CURLOPT_TIMEOUT, 30);          // 超时时间
curl_setopt($ch, CURLOPT_FOLLOWLOCATION, true); // 跟随重定向
curl_setopt($ch, CURLOPT_MAXREDIRS, 5);        // 最大重定向次数

// 执行请求
$response = curl_exec($ch);
if ($response === false) {
    $error = curl_error($ch);
    curl_close($ch);
    throw new RuntimeException("cURL error: {$error}");
}

// 获取 HTTP 状态码
$httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
echo "HTTP Status: {$httpCode}\n";
echo "Response: {$response}\n";

// 关闭 cURL
curl_close($ch);
```

**说明**：
- cURL 提供更强大的功能和更好的错误处理
- `CURLOPT_RETURNTRANSFER` 使响应内容返回而不是直接输出
- 可以获取详细的请求信息

### 示例 4：使用 cURL 进行 POST 请求

```php
<?php
declare(strict_types=1);

$data = json_encode(['name' => 'John', 'age' => 30]);

$ch = curl_init('https://api.example.com/users');
if ($ch === false) {
    throw new RuntimeException('Cannot initialize cURL');
}

// 设置 POST 选项
curl_setopt($ch, CURLOPT_POST, true);
curl_setopt($ch, CURLOPT_POSTFIELDS, $data);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_HTTPHEADER, [
    'Content-Type: application/json',
    'Content-Length: ' . strlen($data),
]);

$response = curl_exec($ch);
if ($response === false) {
    $error = curl_error($ch);
    curl_close($ch);
    throw new RuntimeException("cURL error: {$error}");
}

$httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
curl_close($ch);

if ($httpCode >= 200 && $httpCode < 300) {
    echo "Success: {$response}\n";
} else {
    echo "Error (HTTP {$httpCode}): {$response}\n";
}
```

**说明**：
- 使用 `CURLOPT_POST` 和 `CURLOPT_POSTFIELDS` 设置 POST 请求
- 可以设置自定义请求头

### 示例 5：使用 cURL 发送 JSON 数据

```php
<?php
declare(strict_types=1);

function sendJsonRequest(string $url, array $data, string $method = 'POST'): array
{
    $jsonData = json_encode($data);
    
    $ch = curl_init($url);
    if ($ch === false) {
        throw new RuntimeException('Cannot initialize cURL');
    }
    
    curl_setopt_array($ch, [
        CURLOPT_CUSTOMREQUEST => $method,
        CURLOPT_POSTFIELDS => $jsonData,
        CURLOPT_RETURNTRANSFER => true,
        CURLOPT_HTTPHEADER => [
            'Content-Type: application/json',
            'Content-Length: ' . strlen($jsonData),
        ],
        CURLOPT_TIMEOUT => 30,
    ]);
    
    $response = curl_exec($ch);
    if ($response === false) {
        $error = curl_error($ch);
        curl_close($ch);
        throw new RuntimeException("cURL error: {$error}");
    }
    
    $httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
    curl_close($ch);
    
    return [
        'status_code' => $httpCode,
        'body' => $response,
        'data' => json_decode($response, true),
    ];
}

// 使用
$result = sendJsonRequest('https://api.example.com/users', ['name' => 'John']);
print_r($result);
```

**说明**：
- 封装了 JSON 请求的发送逻辑
- 使用 `curl_setopt_array()` 批量设置选项
- 返回结构化的响应数据

### 示例 6：使用 cURL 处理 HTTPS 请求

```php
<?php
declare(strict_types=1);

function sendSecureRequest(string $url): string
{
    $ch = curl_init($url);
    if ($ch === false) {
        throw new RuntimeException('Cannot initialize cURL');
    }
    
    curl_setopt_array($ch, [
        CURLOPT_RETURNTRANSFER => true,
        CURLOPT_SSL_VERIFYPEER => true,   // 验证 SSL 证书
        CURLOPT_SSL_VERIFYHOST => 2,      // 验证主机名
        CURLOPT_CAINFO => '/path/to/cacert.pem',  // CA 证书路径（可选）
        CURLOPT_TIMEOUT => 30,
    ]);
    
    $response = curl_exec($ch);
    if ($response === false) {
        $error = curl_error($ch);
        curl_close($ch);
        throw new RuntimeException("cURL error: {$error}");
    }
    
    curl_close($ch);
    return $response;
}

// 使用
$content = sendSecureRequest('https://api.example.com/secure-data');
echo $content;
```

**说明**：
- 对于 HTTPS 请求，应该验证 SSL 证书
- `CURLOPT_SSL_VERIFYPEER` 验证证书
- `CURLOPT_SSL_VERIFYHOST` 验证主机名

### 示例 7：使用 cURL 设置请求头

```php
<?php
declare(strict_types=1);

$ch = curl_init('https://api.example.com/data');
if ($ch === false) {
    throw new RuntimeException('Cannot initialize cURL');
}

curl_setopt_array($ch, [
    CURLOPT_RETURNTRANSFER => true,
    CURLOPT_HTTPHEADER => [
        'User-Agent: MyApp/1.0',
        'Accept: application/json',
        'Authorization: Bearer token123',
        'X-Custom-Header: custom-value',
    ],
]);

$response = curl_exec($ch);
curl_close($ch);
```

**说明**：
- 使用 `CURLOPT_HTTPHEADER` 设置请求头
- 每个请求头是一个数组元素

### 示例 8：使用 Guzzle HTTP 客户端（需要安装）

```php
<?php
declare(strict_types=1);

// 需要先安装：composer require guzzlehttp/guzzle
use GuzzleHttp\Client;

// 创建客户端
$client = new Client([
    'base_uri' => 'https://api.example.com',
    'timeout' => 5.0,
]);

// GET 请求
$response = $client->get('/users/1');
$statusCode = $response->getStatusCode();
$body = $response->getBody()->getContents();
$data = json_decode($body, true);

// POST 请求
$response = $client->post('/users', [
    'json' => ['name' => 'John', 'age' => 30],
    'headers' => [
        'Authorization' => 'Bearer token123',
    ],
]);
```

**说明**：
- Guzzle 提供了更简洁的 API
- 自动处理 JSON 编码和解码
- 提供更好的错误处理

## 使用场景

### 场景 1：调用第三方 API

调用第三方服务提供的 API。

**示例**：

```php
<?php
declare(strict_types=1);

function callThirdPartyAPI(string $endpoint, array $data = []): array
{
    $ch = curl_init($endpoint);
    if ($ch === false) {
        throw new RuntimeException('Cannot initialize cURL');
    }
    
    curl_setopt_array($ch, [
        CURLOPT_RETURNTRANSFER => true,
        CURLOPT_POST => !empty($data),
        CURLOPT_POSTFIELDS => !empty($data) ? json_encode($data) : null,
        CURLOPT_HTTPHEADER => [
            'Content-Type: application/json',
            'Authorization: Bearer ' . getenv('API_TOKEN'),
        ],
        CURLOPT_TIMEOUT => 30,
    ]);
    
    $response = curl_exec($ch);
    if ($response === false) {
        $error = curl_error($ch);
        curl_close($ch);
        throw new RuntimeException("API call failed: {$error}");
    }
    
    $httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
    curl_close($ch);
    
    if ($httpCode >= 200 && $httpCode < 300) {
        return json_decode($response, true);
    }
    
    throw new RuntimeException("API returned error: HTTP {$httpCode}");
}
```

### 场景 2：网页内容抓取

抓取网页内容进行分析。

**示例**：

```php
<?php
declare(strict_types=1);

function fetchWebPage(string $url): string
{
    $ch = curl_init($url);
    if ($ch === false) {
        throw new RuntimeException('Cannot initialize cURL');
    }
    
    curl_setopt_array($ch, [
        CURLOPT_RETURNTRANSFER => true,
        CURLOPT_FOLLOWLOCATION => true,
        CURLOPT_USERAGENT => 'Mozilla/5.0 (compatible; MyBot/1.0)',
        CURLOPT_TIMEOUT => 30,
    ]);
    
    $content = curl_exec($ch);
    if ($content === false) {
        $error = curl_error($ch);
        curl_close($ch);
        throw new RuntimeException("Failed to fetch page: {$error}");
    }
    
    curl_close($ch);
    return $content;
}
```

## 注意事项

### allow_url_fopen 配置

使用 `file_get_contents()` 进行 HTTP 请求需要启用 `allow_url_fopen`。

**示例**：

```php
<?php
declare(strict_types=1);

if (!ini_get('allow_url_fopen')) {
    throw new RuntimeException('allow_url_fopen is disabled. Use cURL instead.');
}

$content = file_get_contents('https://api.example.com/data');
```

### SSL 证书验证

对于 HTTPS 请求，应该验证 SSL 证书以确保安全。

**示例**：

```php
<?php
declare(strict_types=1);

// cURL 默认验证证书（推荐）
curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, true);
curl_setopt($ch, CURLOPT_SSL_VERIFYHOST, 2);

// 仅在开发环境禁用验证（不推荐用于生产）
// curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, false);
```

### 超时设置

设置合理的超时时间，避免请求无限等待。

**示例**：

```php
<?php
declare(strict_types=1);

// cURL 超时设置
curl_setopt($ch, CURLOPT_TIMEOUT, 30);        // 总超时时间
curl_setopt($ch, CURLOPT_CONNECTTIMEOUT, 10); // 连接超时时间
```

## 常见问题

### 问题 1：file_get_contents() 和 cURL 的区别是什么？

**回答**：
- `file_get_contents()`：简单易用，适合简单的 GET 请求，需要 `allow_url_fopen`
- cURL：功能强大，支持所有 HTTP 方法、自定义请求头、SSL 验证等，推荐用于复杂请求

**示例**：

```php
<?php
declare(strict_types=1);

// file_get_contents()：简单但功能有限
$content = file_get_contents('https://api.example.com/data');

// cURL：功能强大
$ch = curl_init('https://api.example.com/data');
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_POST, true);
curl_setopt($ch, CURLOPT_POSTFIELDS, json_encode($data));
$response = curl_exec($ch);
```

### 问题 2：如何发送 POST 请求？

**回答**：使用 cURL 的 `CURLOPT_POST` 和 `CURLOPT_POSTFIELDS` 选项。

**示例**：见"示例 4：使用 cURL 进行 POST 请求"

### 问题 3：如何处理 HTTPS 请求？

**回答**：使用 cURL 并设置 SSL 验证选项，或使用 `file_get_contents()` 配合流上下文。

**示例**：见"示例 6：使用 cURL 处理 HTTPS 请求"

### 问题 4：如何设置请求超时？

**回答**：使用 cURL 的 `CURLOPT_TIMEOUT` 和 `CURLOPT_CONNECTTIMEOUT` 选项。

**示例**：

```php
<?php
declare(strict_types=1);

curl_setopt($ch, CURLOPT_TIMEOUT, 30);        // 总超时 30 秒
curl_setopt($ch, CURLOPT_CONNECTTIMEOUT, 10); // 连接超时 10 秒
```

## 最佳实践

### 1. 优先使用 cURL 进行复杂请求

对于需要自定义请求头、POST 数据、SSL 验证等的请求，使用 cURL。

**示例**：见"示例 3：使用 cURL 进行 GET 请求"

### 2. 设置合理的超时时间

避免请求无限等待，设置合适的超时时间。

**示例**：见"超时设置"部分

### 3. 验证 SSL 证书

对于 HTTPS 请求，始终验证 SSL 证书以确保安全。

**示例**：见"示例 6：使用 cURL 处理 HTTPS 请求"

### 4. 使用 Guzzle 简化操作

对于复杂的 HTTP 客户端需求，考虑使用 Guzzle 库。

**示例**：见"示例 8：使用 Guzzle HTTP 客户端"

### 5. 错误处理

检查每个操作的返回值，处理错误情况。

**示例**：

```php
<?php
declare(strict_types=1);

$response = curl_exec($ch);
if ($response === false) {
    $error = curl_error($ch);
    $errno = curl_errno($ch);
    curl_close($ch);
    throw new RuntimeException("cURL error ({$errno}): {$error}");
}

$httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
if ($httpCode >= 400) {
    curl_close($ch);
    throw new RuntimeException("HTTP error: {$httpCode}");
}
```

## 对比分析

### file_get_contents() vs cURL

| 特性         | file_get_contents()         | cURL                        |
|:-------------|:----------------------------|:----------------------------|
| **简单性**   | ✅ 非常简单                | ⚠️ 需要更多配置             |
| **功能**     | ⚠️ 功能有限                | ✅ 功能强大                 |
| **配置要求** | ⚠️ 需要 allow_url_fopen    | ✅ 无需特殊配置             |
| **错误处理** | ⚠️ 错误信息有限            | ✅ 详细的错误信息           |
| **适用场景** | 简单的 GET 请求            | 复杂的 HTTP 请求            |

### cURL vs Guzzle

| 特性         | cURL                        | Guzzle                      |
|:-------------|:----------------------------|:----------------------------|
| **API 设计**  | ⚠️ 函数式，较底层          | ✅ 面向对象，更高级         |
| **易用性**    | ⚠️ 需要手动配置很多选项    | ✅ 简洁的 API               |
| **功能**      | ✅ 功能完整                 | ✅ 功能完整，更易用         |
| **依赖**      | ✅ PHP 内置扩展             | ⚠️ 需要安装 Composer 包     |
| **适用场景**  | 原生 PHP 环境               | 现代 PHP 项目               |

## 练习任务

1. **HTTP 客户端工具类**：创建一个 HTTP 客户端工具类，封装常用的 HTTP 请求操作。

2. **API 调用封装**：实现一个工具，封装第三方 API 的调用，包括错误处理和重试机制。

3. **网页内容抓取工具**：编写一个工具，抓取网页内容并提取特定信息。

4. **HTTPS 请求工具**：创建一个工具，安全地处理 HTTPS 请求，包括证书验证。

5. **请求重试机制**：实现一个工具，在请求失败时自动重试，支持指数退避策略。

## 相关章节

- **[4.3.1 Socket 编程基础](section-01-socket-basics.md)**：了解底层网络编程
- **[4.3.3 网络协议理解](section-03-network-protocols.md)**：理解网络协议的基本原理
- **[4.2.9 流处理](../chapter-02-filesystem/section-09-streaming.md)**：了解流处理的相关内容
