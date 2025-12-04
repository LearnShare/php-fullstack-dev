# 4.11.1 Guzzle 基础

## 概述

Guzzle 是 PHP 中最流行的 HTTP 客户端库。本节详细介绍为什么需要 HTTP 客户端、Guzzle 安装、基础使用、请求方法、响应处理，以及完整示例。

## 为什么需要 HTTP 客户端

### 原生 PHP 的限制

```php
<?php
// 原生 curl 使用复杂
$ch = curl_init('https://api.example.com/users');
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_HTTPHEADER, [
    'Authorization: Bearer token',
    'Content-Type: application/json',
]);
$response = curl_exec($ch);
$statusCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
curl_close($ch);
```

### Guzzle 的优势

- **简单易用**：链式 API，代码更清晰
- **功能完整**：支持认证、重试、中间件等
- **错误处理**：自动处理 HTTP 错误
- **PSR-7 兼容**：符合 PSR-7 标准
- **异步支持**：支持并发请求

## 安装和配置

### 安装

```bash
composer require guzzlehttp/guzzle
```

### 基础使用

```php
<?php
declare(strict_types=1);

use GuzzleHttp\Client;

// 创建客户端
$client = new Client([
    'base_uri' => 'https://api.example.com',
    'timeout' => 5.0,
]);

// 发送 GET 请求
$response = $client->get('/users/1');

// 获取响应
$statusCode = $response->getStatusCode(); // 200
$body = $response->getBody()->getContents(); // JSON 字符串
$data = json_decode($body, true); // 数组
```

## 请求方法

### GET 请求

```php
<?php
use GuzzleHttp\Client;

$client = new Client();

// 简单 GET
$response = $client->get('https://api.example.com/users');

// 带查询参数
$response = $client->get('https://api.example.com/users', [
    'query' => [
        'page' => 1,
        'limit' => 10,
    ],
]);

// 带 Headers
$response = $client->get('https://api.example.com/users', [
    'headers' => [
        'Authorization' => 'Bearer token',
        'Accept' => 'application/json',
    ],
]);
```

### POST 请求

```php
<?php
// JSON 请求
$response = $client->post('https://api.example.com/users', [
    'json' => [
        'name' => 'Alice',
        'email' => 'alice@example.com',
    ],
]);

// Form 数据
$response = $client->post('https://api.example.com/users', [
    'form_params' => [
        'name' => 'Alice',
        'email' => 'alice@example.com',
    ],
]);
```

## 响应处理

### 获取响应内容

```php
<?php
$response = $client->get('https://api.example.com/users/1');

// 获取状态码
$statusCode = $response->getStatusCode();

// 获取响应头
$headers = $response->getHeaders();

// 获取响应体
$body = $response->getBody()->getContents();

// 解析 JSON
$data = json_decode($body, true);
```

## 完整示例

```php
<?php
declare(strict_types=1);

use GuzzleHttp\Client;

class ApiClient
{
    private Client $client;

    public function __construct(string $baseUri, string $token = null)
    {
        $config = [
            'base_uri' => $baseUri,
            'timeout' => 5.0,
        ];

        if ($token !== null) {
            $config['headers'] = [
                'Authorization' => "Bearer {$token}",
            ];
        }

        $this->client = new Client($config);
    }

    public function get(string $path, array $query = []): array
    {
        $response = $this->client->get($path, ['query' => $query]);
        return json_decode($response->getBody()->getContents(), true);
    }

    public function post(string $path, array $data): array
    {
        $response = $this->client->post($path, ['json' => $data]);
        return json_decode($response->getBody()->getContents(), true);
    }
}
```

## 注意事项

1. **超时设置**：合理设置请求超时时间
2. **错误处理**：使用 try-catch 处理异常
3. **资源管理**：及时关闭响应流
4. **配置复用**：使用 Client 实例复用配置

## 练习

1. 创建一个 HTTP 客户端封装类，简化常用操作。

2. 实现一个 API 客户端，支持认证和错误处理。

3. 编写一个 HTTP 请求工具类，封装常用请求方法。

4. 实现请求日志记录功能，记录所有 HTTP 请求。
