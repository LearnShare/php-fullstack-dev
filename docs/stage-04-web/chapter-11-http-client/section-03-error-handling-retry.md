# 4.11.3 错误处理与重试

## 概述

错误处理和重试机制是构建可靠 HTTP 客户端的关键。本节详细介绍错误处理、重试机制、超时设置、中间件使用，以及完整实现。

## 错误处理

### 异常处理

```php
<?php
declare(strict_types=1);

use GuzzleHttp\Client;
use GuzzleHttp\Exception\RequestException;

$client = new Client();

try {
    $response = $client->get('https://api.example.com/users/1');
    $data = json_decode($response->getBody()->getContents(), true);
} catch (RequestException $e) {
    if ($e->hasResponse()) {
        $statusCode = $e->getResponse()->getStatusCode();
        $body = $e->getResponse()->getBody()->getContents();
        echo "Error {$statusCode}: {$body}\n";
    } else {
        echo "Request failed: " . $e->getMessage() . "\n";
    }
}
```

## 重试机制

### 自动重试

```php
<?php
declare(strict_types=1);

use GuzzleHttp\Client;
use GuzzleHttp\HandlerStack;
use GuzzleHttp\Middleware;
use GuzzleHttp\Psr7\Request;
use GuzzleHttp\Psr7\Response;

// 重试中间件
$stack = HandlerStack::create();
$stack->push(Middleware::retry(function ($retries, Request $request, Response $response = null, $exception = null) {
    // 最多重试 3 次
    if ($retries >= 3) {
        return false;
    }

    // 5xx 错误重试
    if ($response && $response->getStatusCode() >= 500) {
        return true;
    }

    // 网络错误重试
    if ($exception instanceof \GuzzleHttp\Exception\ConnectException) {
        return true;
    }

    return false;
}, function ($retries) {
    // 指数退避
    return 1000 * pow(2, $retries);
}));

$client = new Client(['handler' => $stack]);
```

## 超时设置

### 超时配置

```php
<?php
declare(strict_types=1);

$client = new Client([
    'timeout' => 5.0,           // 总超时时间
    'connect_timeout' => 2.0,    // 连接超时
    'read_timeout' => 3.0,       // 读取超时
]);
```

## 中间件使用

### 日志中间件

```php
<?php
declare(strict_types=1);

use GuzzleHttp\Middleware;
use Psr\Log\LoggerInterface;

$stack = HandlerStack::create();
$stack->push(Middleware::log($logger, new \GuzzleHttp\MessageFormatter()));
$client = new Client(['handler' => $stack]);
```

## 完整示例

```php
<?php
declare(strict_types=1);

class RobustHttpClient
{
    private Client $client;

    public function __construct()
    {
        $stack = HandlerStack::create();
        
        // 添加重试中间件
        $stack->push($this->createRetryMiddleware());
        
        // 添加日志中间件
        $stack->push($this->createLogMiddleware());
        
        $this->client = new Client([
            'handler' => $stack,
            'timeout' => 5.0,
        ]);
    }

    public function request(string $method, string $uri, array $options = []): array
    {
        try {
            $response = $this->client->request($method, $uri, $options);
            return json_decode($response->getBody()->getContents(), true);
        } catch (RequestException $e) {
            // 错误处理
            throw new RuntimeException('HTTP request failed', 0, $e);
        }
    }
}
```

## 注意事项

1. **重试策略**：合理设置重试次数和退避时间
2. **超时设置**：根据场景设置合适的超时时间
3. **错误分类**：区分可重试和不可重试的错误
4. **日志记录**：记录请求和错误信息

## 练习

1. 实现 HTTP 客户端错误处理，包含异常捕获和处理。

2. 创建重试机制，支持指数退避。

3. 实现请求日志中间件，记录所有请求和响应。

4. 设计一个健壮的 HTTP 客户端，包含所有错误处理和重试逻辑。
