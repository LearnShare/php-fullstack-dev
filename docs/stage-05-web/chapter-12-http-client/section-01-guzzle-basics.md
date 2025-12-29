# 5.12.1 Guzzle 基础

## 概述

Guzzle 是 PHP 最流行的 HTTP 客户端库，提供了简洁的 API 和强大的功能。理解 Guzzle 的基本使用方法、掌握请求和响应处理、配置请求选项，对于进行 API 调用、服务间通信、数据抓取等场景至关重要。本节详细介绍 Guzzle 的概述、安装配置、基本请求（GET、POST、PUT、DELETE）、请求选项、响应处理等内容，帮助零基础学员掌握 Guzzle 的使用。

Guzzle 提供了统一的接口来处理 HTTP 请求，支持同步和异步请求，具有强大的错误处理和重试机制。掌握 Guzzle 的使用对于构建健壮的 HTTP 客户端至关重要。

**主要内容**：
- Guzzle 概述（Guzzle 的功能、Guzzle 的优势、使用场景）
- 安装配置（Composer 安装、Client 创建、基本配置）
- 基本请求（GET 请求、POST 请求、PUT 请求、DELETE 请求）
- 请求选项（headers 请求头、query 查询参数、json JSON 数据、timeout 超时设置、auth 认证）
- 响应处理（响应对象、状态码获取、响应体获取、响应头获取、JSON 解析）
- 实际应用示例和最佳实践

## 特性

- **简洁 API**：提供简洁易用的 API
- **功能强大**：支持多种 HTTP 方法和选项
- **错误处理**：强大的错误处理机制
- **异步支持**：支持异步请求
- **可扩展**：易于扩展和定制

## Guzzle 概述

### Guzzle 的功能

1. **HTTP 请求**：支持所有 HTTP 方法
2. **请求选项**：丰富的请求配置选项
3. **响应处理**：便捷的响应处理方法
4. **异步请求**：支持异步和并发请求
5. **中间件**：支持中间件机制

### Guzzle 的优势

1. **易用性**：简洁的 API 设计
2. **功能完整**：功能完整且强大
3. **性能优秀**：性能表现优秀
4. **社区活跃**：社区活跃，文档完善
5. **标准兼容**：遵循 PSR-7 标准

### 使用场景

1. **API 调用**：调用第三方 API
2. **服务间通信**：微服务间通信
3. **数据抓取**：网页数据抓取
4. **Webhook**：发送 Webhook 请求

## 安装配置

### Composer 安装

**安装命令**：
```bash
composer require guzzlehttp/guzzle
```

### Client 创建

**示例**：
```php
<?php
declare(strict_types=1);

require __DIR__ . '/vendor/autoload.php';

use GuzzleHttp\Client;

// 创建客户端
$client = new Client([
    'base_uri' => 'https://api.example.com',
    'timeout' => 30,
]);
```

### 基本配置

**示例**：
```php
<?php
declare(strict_types=1);

use GuzzleHttp\Client;

$client = new Client([
    'base_uri' => 'https://api.example.com',
    'timeout' => 30,
    'connect_timeout' => 10,
    'verify' => true,  // SSL 验证
    'http_errors' => true,  // 抛出 HTTP 错误异常
]);
```

## 基本请求

### GET 请求

**示例**：
```php
<?php
declare(strict_types=1);

use GuzzleHttp\Client;

$client = new Client();

// 基本 GET 请求
$response = $client->get('https://api.example.com/users');

// 带查询参数的 GET 请求
$response = $client->get('https://api.example.com/users', [
    'query' => [
        'page' => 1,
        'limit' => 10,
    ],
]);

// 获取响应内容
$body = $response->getBody()->getContents();
$data = json_decode($body, true);
```

### POST 请求

**示例**：
```php
<?php
declare(strict_types=1);

use GuzzleHttp\Client;

$client = new Client();

// 表单数据 POST 请求
$response = $client->post('https://api.example.com/users', [
    'form_params' => [
        'name' => 'John Doe',
        'email' => 'john@example.com',
    ],
]);

// JSON 数据 POST 请求
$response = $client->post('https://api.example.com/users', [
    'json' => [
        'name' => 'John Doe',
        'email' => 'john@example.com',
    ],
]);

// 原始数据 POST 请求
$response = $client->post('https://api.example.com/users', [
    'body' => 'raw data',
    'headers' => [
        'Content-Type' => 'application/json',
    ],
]);
```

### PUT 请求

**示例**：
```php
<?php
declare(strict_types=1);

use GuzzleHttp\Client;

$client = new Client();

// PUT 请求
$response = $client->put('https://api.example.com/users/1', [
    'json' => [
        'name' => 'Jane Doe',
        'email' => 'jane@example.com',
    ],
]);
```

### DELETE 请求

**示例**：
```php
<?php
declare(strict_types=1);

use GuzzleHttp\Client;

$client = new Client();

// DELETE 请求
$response = $client->delete('https://api.example.com/users/1');
```

## 请求选项

### headers（请求头）

**示例**：
```php
<?php
declare(strict_types=1);

use GuzzleHttp\Client;

$client = new Client();

$response = $client->get('https://api.example.com/users', [
    'headers' => [
        'Authorization' => 'Bearer token123',
        'User-Agent' => 'MyApp/1.0',
        'Accept' => 'application/json',
    ],
]);
```

### query（查询参数）

**示例**：
```php
<?php
declare(strict_types=1);

use GuzzleHttp\Client;

$client = new Client();

$response = $client->get('https://api.example.com/users', [
    'query' => [
        'page' => 1,
        'limit' => 10,
        'sort' => 'name',
        'order' => 'asc',
    ],
]);
```

### json（JSON 数据）

**示例**：
```php
<?php
declare(strict_types=1);

use GuzzleHttp\Client;

$client = new Client();

$response = $client->post('https://api.example.com/users', [
    'json' => [
        'name' => 'John Doe',
        'email' => 'john@example.com',
    ],
]);
```

### timeout（超时设置）

**示例**：
```php
<?php
declare(strict_types=1);

use GuzzleHttp\Client;

$client = new Client();

$response = $client->get('https://api.example.com/users', [
    'timeout' => 30,  // 总超时时间（秒）
    'connect_timeout' => 10,  // 连接超时时间（秒）
]);
```

### auth（认证）

**示例**：
```php
<?php
declare(strict_types=1);

use GuzzleHttp\Client;

$client = new Client();

// Basic 认证
$response = $client->get('https://api.example.com/users', [
    'auth' => ['username', 'password'],
]);

// Bearer Token 认证
$response = $client->get('https://api.example.com/users', [
    'headers' => [
        'Authorization' => 'Bearer token123',
    ],
]);
```

### multipart（文件上传）

**示例**：
```php
<?php
declare(strict_types=1);

use GuzzleHttp\Client;

$client = new Client();

$response = $client->post('https://api.example.com/upload', [
    'multipart' => [
        [
            'name' => 'file',
            'contents' => fopen('/path/to/file.jpg', 'r'),
            'filename' => 'file.jpg',
        ],
        [
            'name' => 'description',
            'contents' => 'File description',
        ],
    ],
]);
```

## 响应处理

### 响应对象

**示例**：
```php
<?php
declare(strict_types=1);

use GuzzleHttp\Client;

$client = new Client();
$response = $client->get('https://api.example.com/users');

// 响应对象的方法
$statusCode = $response->getStatusCode();  // 状态码
$headers = $response->getHeaders();  // 响应头
$body = $response->getBody();  // 响应体（流对象）
```

### 状态码获取

**示例**：
```php
<?php
declare(strict_types=1);

use GuzzleHttp\Client;

$client = new Client();
$response = $client->get('https://api.example.com/users');

$statusCode = $response->getStatusCode();

if ($statusCode === 200) {
    // 成功
} elseif ($statusCode === 404) {
    // 未找到
} elseif ($statusCode >= 500) {
    // 服务器错误
}
```

### 响应体获取

**示例**：
```php
<?php
declare(strict_types=1);

use GuzzleHttp\Client;

$client = new Client();
$response = $client->get('https://api.example.com/users');

// 获取响应体内容（字符串）
$body = $response->getBody()->getContents();

// 获取响应体内容（流对象）
$body = $response->getBody();
$content = $body->read(1024);  // 读取 1024 字节

// 重置流位置
$body->rewind();
$content = $body->getContents();
```

### 响应头获取

**示例**：
```php
<?php
declare(strict_types=1);

use GuzzleHttp\Client;

$client = new Client();
$response = $client->get('https://api.example.com/users');

// 获取所有响应头
$headers = $response->getHeaders();

// 获取特定响应头
$contentType = $response->getHeader('Content-Type');
$contentType = $response->getHeaderLine('Content-Type');  // 字符串形式

// 检查响应头是否存在
if ($response->hasHeader('Content-Type')) {
    // 存在
}
```

### JSON 解析

**示例**：
```php
<?php
declare(strict_types=1);

use GuzzleHttp\Client;

$client = new Client();
$response = $client->get('https://api.example.com/users');

// 解析 JSON 响应
$data = json_decode($response->getBody()->getContents(), true);

// 使用 Guzzle 的 json() 方法（Guzzle 7.0+）
$data = $response->json();
```

## 使用场景

### API 调用

- 调用第三方 API
- RESTful API 调用
- GraphQL API 调用

### 服务间通信

- 微服务间通信
- 服务发现
- 服务调用

### 数据抓取

- 网页数据抓取
- API 数据获取
- 数据同步

### 第三方服务集成

- 支付接口调用
- 短信服务调用
- 邮件服务调用

## 注意事项

### 错误处理

- **HTTP 错误**：Guzzle 默认会抛出异常
- **网络错误**：处理网络连接错误
- **超时错误**：设置合理的超时时间
- **异常捕获**：使用 try-catch 捕获异常

### 超时设置

- **总超时**：设置合理的总超时时间
- **连接超时**：设置连接超时时间
- **不同场景**：不同场景使用不同超时时间

### 请求重试

- **重试机制**：实现请求重试机制
- **重试条件**：定义重试条件
- **重试次数**：限制重试次数

### 资源释放

- **流对象**：及时关闭流对象
- **连接复用**：使用 Client 实例复用连接
- **内存管理**：注意内存使用

## 常见问题

### 如何安装 Guzzle？

使用 Composer 安装：
```bash
composer require guzzlehttp/guzzle
```

### 如何发送 POST 请求？

使用 `post()` 方法：
```php
$response = $client->post('https://api.example.com/users', [
    'json' => ['name' => 'John'],
]);
```

### 如何设置请求头？

使用 `headers` 选项：
```php
$response = $client->get('https://api.example.com/users', [
    'headers' => ['Authorization' => 'Bearer token'],
]);
```

### 如何处理响应？

获取响应内容：
```php
$body = $response->getBody()->getContents();
$data = json_decode($body, true);
```

## 最佳实践

### 使用 Client 实例复用

- 创建 Client 实例并复用
- 避免每次请求都创建新实例
- 提高性能和资源利用率

### 设置合理的超时时间

- 根据实际需求设置超时时间
- 不同场景使用不同超时时间
- 考虑网络延迟

### 实现错误处理

- 使用 try-catch 捕获异常
- 处理不同类型的错误
- 提供清晰的错误信息

### 使用请求选项简化配置

- 使用 `json` 选项发送 JSON 数据
- 使用 `query` 选项添加查询参数
- 使用 `headers` 选项设置请求头

## 相关章节

- **[5.12.2 异步请求](section-02-async-requests.md)**：了解异步请求的详细内容
- **[5.12.3 错误处理与重试](section-03-error-handling-retry.md)**：了解错误处理和重试的详细内容
- **[5.4.3 查询字符串处理](../chapter-04-url-handling/section-03-query-string.md)**：了解 URL 处理的详细内容

## 练习任务

1. **实现基本的 HTTP 客户端**
   - 创建 Guzzle Client
   - 发送 GET 和 POST 请求
   - 处理响应

2. **实现 API 调用封装**
   - 封装 API 调用方法
   - 处理认证
   - 错误处理

3. **实现文件上传功能**
   - 使用 multipart 上传文件
   - 处理上传响应
   - 错误处理

4. **实现请求配置管理**
   - 统一请求配置
   - 请求选项管理
   - 配置复用

5. **实现完整的 HTTP 客户端系统**
   - 多种请求方法
   - 完整的错误处理
   - 响应处理
