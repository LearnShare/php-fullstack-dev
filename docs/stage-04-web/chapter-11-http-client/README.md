# 4.11 HTTP 客户端：Guzzle 详细使用

## 目标

- 理解为什么需要 HTTP 客户端库。
- 掌握 Guzzle HTTP 客户端的基础使用。
- 熟悉请求、响应、错误处理、中间件等高级功能。
- 能够处理认证、重试、超时等常见场景。

## 为什么需要 HTTP 客户端

### 原生 PHP 的限制

```php
<?php
declare(strict_types=1);

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

// 错误处理复杂
if ($response === false) {
    $error = curl_error($ch);
    // ...
}
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
declare(strict_types=1);

use GuzzleHttp\Client;

$client = new Client();

// 简单 GET
$response = $client->get('https://api.example.com/users');

// 带查询参数
$response = $client->get('https://api.example.com/users', [
    'query' => [
        'page' => 1,
        'limit' => 10,
        'status' => 'active',
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
declare(strict_types=1);

use GuzzleHttp\Client;

$client = new Client();

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

// 原始数据
$response = $client->post('https://api.example.com/users', [
    'body' => 'raw data',
    'headers' => [
        'Content-Type' => 'application/xml',
    ],
]);
```

### PUT 和 PATCH 请求

```php
<?php
declare(strict_types=1);

use GuzzleHttp\Client;

$client = new Client();

// PUT 请求（完整更新）
$response = $client->put('https://api.example.com/users/1', [
    'json' => [
        'name' => 'Alice Updated',
        'email' => 'alice@example.com',
    ],
]);

// PATCH 请求（部分更新）
$response = $client->patch('https://api.example.com/users/1', [
    'json' => [
        'name' => 'Alice Updated',
    ],
]);
```

### DELETE 请求

```php
<?php
declare(strict_types=1);

use GuzzleHttp\Client;

$client = new Client();

$response = $client->delete('https://api.example.com/users/1');
```

## 响应处理

### 获取响应内容

```php
<?php
declare(strict_types=1);

use GuzzleHttp\Client;

$client = new Client();
$response = $client->get('https://api.example.com/users/1');

// 获取状态码
$statusCode = $response->getStatusCode(); // 200

// 获取响应头
$headers = $response->getHeaders();
$contentType = $response->getHeader('Content-Type')[0];

// 获取响应体（字符串）
$body = $response->getBody()->getContents();

// 获取响应体（流）
$stream = $response->getBody();
$stream->read(1024); // 读取 1024 字节

// JSON 响应
$data = json_decode($response->getBody()->getContents(), true);
```

### 响应类型判断

```php
<?php
declare(strict_types=1);

use GuzzleHttp\Client;

$client = new Client();
$response = $client->get('https://api.example.com/users/1');

// 判断状态码
if ($response->getStatusCode() === 200) {
    // 成功
}

// 判断内容类型
$contentType = $response->getHeaderLine('Content-Type');
if (str_contains($contentType, 'application/json')) {
    $data = json_decode($response->getBody()->getContents(), true);
}
```

## 错误处理

### 异常处理

```php
<?php
declare(strict_types=1);

use GuzzleHttp\Client;
use GuzzleHttp\Exception\ClientException;
use GuzzleHttp\Exception\ServerException;
use GuzzleHttp\Exception\RequestException;

$client = new Client();

try {
    $response = $client->get('https://api.example.com/users/1');
} catch (ClientException $e) {
    // 4xx 错误（客户端错误）
    $statusCode = $e->getResponse()->getStatusCode();
    $body = $e->getResponse()->getBody()->getContents();
    error_log("Client error: {$statusCode} - {$body}");
} catch (ServerException $e) {
    // 5xx 错误（服务器错误）
    $statusCode = $e->getResponse()->getStatusCode();
    error_log("Server error: {$statusCode}");
} catch (RequestException $e) {
    // 请求异常（网络错误等）
    error_log("Request failed: " . $e->getMessage());
}
```

### HTTP 错误选项

```php
<?php
declare(strict_types=1);

use GuzzleHttp\Client;

$client = new Client();

// 不抛出异常，返回响应
$response = $client->get('https://api.example.com/users/1', [
    'http_errors' => false, // 不抛出异常
]);

// 检查状态码
if ($response->getStatusCode() >= 400) {
    // 处理错误
}
```

## 认证

### Bearer Token

```php
<?php
declare(strict_types=1);

use GuzzleHttp\Client;

$client = new Client([
    'headers' => [
        'Authorization' => 'Bearer your-token',
    ],
]);

// 或每次请求指定
$response = $client->get('https://api.example.com/users', [
    'headers' => [
        'Authorization' => 'Bearer ' . $token,
    ],
]);
```

### Basic 认证

```php
<?php
declare(strict_types=1);

use GuzzleHttp\Client;

$client = new Client();

$response = $client->get('https://api.example.com/users', [
    'auth' => ['username', 'password'],
]);

// 或使用配置
$client = new Client([
    'auth' => ['username', 'password'],
]);
```

### API Key

```php
<?php
declare(strict_types=1);

use GuzzleHttp\Client;

$client = new Client();

// Query 参数
$response = $client->get('https://api.example.com/users', [
    'query' => [
        'api_key' => 'your-api-key',
    ],
]);

// Header
$response = $client->get('https://api.example.com/users', [
    'headers' => [
        'X-API-Key' => 'your-api-key',
    ],
]);
```

## 超时和重试

### 超时设置

```php
<?php
declare(strict_types=1);

use GuzzleHttp\Client;

$client = new Client([
    'timeout' => 5.0,        // 总超时时间（秒）
    'connect_timeout' => 2.0, // 连接超时时间（秒）
]);

// 或每次请求指定
$response = $client->get('https://api.example.com/users', [
    'timeout' => 10.0,
]);
```

### 重试机制

```php
<?php
declare(strict_types=1);

use GuzzleHttp\Client;
use GuzzleHttp\HandlerStack;
use GuzzleHttp\Middleware;
use GuzzleHttp\Psr7\Request;
use GuzzleHttp\Psr7\Response;

// 创建重试中间件
$stack = HandlerStack::create();
$stack->push(Middleware::retry(
    function ($retries, Request $request, Response $response = null, $exception = null) {
        // 最多重试 3 次
        if ($retries >= 3) {
            return false;
        }
        
        // 5xx 错误或异常时重试
        if ($exception || ($response && $response->getStatusCode() >= 500)) {
            return true;
        }
        
        return false;
    },
    function ($retries) {
        // 指数退避：1s, 2s, 4s
        return 1000 * (2 ** $retries);
    }
));

$client = new Client(['handler' => $stack]);
```

## 并发请求

### 异步请求

```php
<?php
declare(strict_types=1);

use GuzzleHttp\Client;
use GuzzleHttp\Promise;

$client = new Client();

// 创建多个请求
$promises = [
    'user' => $client->getAsync('https://api.example.com/users/1'),
    'posts' => $client->getAsync('https://api.example.com/posts?user_id=1'),
    'comments' => $client->getAsync('https://api.example.com/comments?user_id=1'),
];

// 等待所有请求完成
$results = Promise\Utils::settle($promises)->wait();

// 处理结果
foreach ($results as $key => $result) {
    if ($result['state'] === 'fulfilled') {
        $response = $result['value'];
        $data = json_decode($response->getBody()->getContents(), true);
        echo "{$key}: " . json_encode($data) . "\n";
    } else {
        echo "{$key}: Failed\n";
    }
}
```

### 并发限制

```php
<?php
declare(strict_types=1);

use GuzzleHttp\Client;
use GuzzleHttp\Pool;
use GuzzleHttp\Psr7\Request;

$client = new Client();

// 创建请求生成器
$requests = function () {
    for ($i = 1; $i <= 100; $i++) {
        yield new Request('GET', "https://api.example.com/users/{$i}");
    }
};

// 并发处理（最多 10 个并发）
$pool = new Pool($client, $requests(), [
    'concurrency' => 10,
    'fulfilled' => function ($response, $index) {
        $data = json_decode($response->getBody()->getContents(), true);
        echo "Request {$index} completed\n";
    },
    'rejected' => function ($reason, $index) {
        echo "Request {$index} failed: {$reason}\n";
    },
]);

$promise = $pool->promise();
$promise->wait();
```

## 中间件

### 日志中间件

```php
<?php
declare(strict_types=1);

use GuzzleHttp\Client;
use GuzzleHttp\HandlerStack;
use GuzzleHttp\Middleware;
use Psr\Http\Message\RequestInterface;
use Psr\Http\Message\ResponseInterface;

$stack = HandlerStack::create();

// 添加日志中间件
$stack->push(Middleware::mapRequest(function (RequestInterface $request) {
    error_log(sprintf(
        '[Guzzle] %s %s',
        $request->getMethod(),
        $request->getUri()
    ));
    return $request;
}));

$stack->push(Middleware::mapResponse(function (ResponseInterface $response) {
    error_log(sprintf(
        '[Guzzle] Response: %d',
        $response->getStatusCode()
    ));
    return $response;
}));

$client = new Client(['handler' => $stack]);
```

## 实际应用示例

### API 客户端封装

```php
<?php
declare(strict_types=1);

use GuzzleHttp\Client;
use GuzzleHttp\Exception\GuzzleException;

class ApiClient
{
    private Client $client;
    
    public function __construct(string $baseUri, string $token)
    {
        $this->client = new Client([
            'base_uri' => $baseUri,
            'headers' => [
                'Authorization' => "Bearer {$token}",
                'Accept' => 'application/json',
            ],
            'timeout' => 10.0,
        ]);
    }
    
    public function getUsers(array $filters = []): array
    {
        try {
            $response = $this->client->get('/users', [
                'query' => $filters,
            ]);
            
            return json_decode($response->getBody()->getContents(), true);
        } catch (GuzzleException $e) {
            throw new RuntimeException('Failed to fetch users', 0, $e);
        }
    }
    
    public function createUser(array $data): array
    {
        try {
            $response = $this->client->post('/users', [
                'json' => $data,
            ]);
            
            return json_decode($response->getBody()->getContents(), true);
        } catch (GuzzleException $e) {
            throw new RuntimeException('Failed to create user', 0, $e);
        }
    }
}

// 使用
$client = new ApiClient('https://api.example.com', 'your-token');
$users = $client->getUsers(['status' => 'active']);
```

## 练习

1. 使用 Guzzle 创建一个简单的 HTTP 客户端，发送 GET 请求并处理响应。

2. 实现一个 API 客户端类，封装常用的 API 操作（GET、POST、PUT、DELETE）。

3. 实现重试机制，在请求失败时自动重试（最多 3 次）。

4. 使用并发请求，同时获取多个资源（用户、文章、评论）。

5. 实现请求日志记录，记录所有 HTTP 请求和响应。

6. 创建一个带认证的 API 客户端，支持 Token 刷新机制。
