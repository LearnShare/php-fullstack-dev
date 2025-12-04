# 4.11.2 异步请求

## 概述

异步请求可以显著提升并发性能。本节详细介绍并发请求、Promise 使用、异步处理、性能优化，以及完整示例。

## 并发请求

### Promise 使用

```php
<?php
declare(strict_types=1);

use GuzzleHttp\Client;
use GuzzleHttp\Promise;

$client = new Client();

// 创建多个 Promise
$promises = [
    'user' => $client->getAsync('https://api.example.com/users/1'),
    'posts' => $client->getAsync('https://api.example.com/users/1/posts'),
    'comments' => $client->getAsync('https://api.example.com/users/1/comments'),
];

// 等待所有请求完成
$results = Promise\Utils::settle($promises)->wait();

// 处理结果
foreach ($results as $key => $result) {
    if ($result['state'] === 'fulfilled') {
        $data = json_decode($result['value']->getBody()->getContents(), true);
        echo "{$key}: " . json_encode($data) . "\n";
    } else {
        echo "{$key}: Error\n";
    }
}
```

## 异步处理

### 使用 Promise

```php
<?php
declare(strict_types=1);

use GuzzleHttp\Client;
use GuzzleHttp\Promise\PromiseInterface;

$client = new Client();

$promise = $client->getAsync('https://api.example.com/users');

$promise->then(
    function ($response) {
        // 成功处理
        $data = json_decode($response->getBody()->getContents(), true);
        return $data;
    },
    function ($exception) {
        // 错误处理
        echo "Error: " . $exception->getMessage();
    }
);

// 等待完成
$promise->wait();
```

## 性能优化

### 并发限制

```php
<?php
declare(strict_types=1);

use GuzzleHttp\Client;
use GuzzleHttp\Pool;

$client = new Client();

$requests = function () {
    for ($i = 1; $i <= 100; $i++) {
        yield function () use ($i) {
            return $client->getAsync("https://api.example.com/users/{$i}");
        };
    }
};

// 并发 10 个请求
$pool = new Pool($client, $requests(), [
    'concurrency' => 10,
    'fulfilled' => function ($response, $index) {
        // 处理成功响应
    },
    'rejected' => function ($reason, $index) {
        // 处理失败
    },
]);

$promise = $pool->promise();
$promise->wait();
```

## 完整示例

```php
<?php
declare(strict_types=1);

class AsyncApiClient
{
    private Client $client;

    public function fetchMultiple(array $urls): array
    {
        $promises = [];
        foreach ($urls as $key => $url) {
            $promises[$key] = $this->client->getAsync($url);
        }

        $results = Promise\Utils::settle($promises)->wait();
        
        $data = [];
        foreach ($results as $key => $result) {
            if ($result['state'] === 'fulfilled') {
                $data[$key] = json_decode($result['value']->getBody()->getContents(), true);
            }
        }

        return $data;
    }
}
```

## 注意事项

1. **并发控制**：合理控制并发数量
2. **错误处理**：处理异步请求的错误
3. **资源管理**：及时清理 Promise
4. **性能监控**：监控异步请求的性能

## 练习

1. 实现并发请求功能，同时请求多个 API。

2. 创建一个异步 HTTP 客户端，支持 Promise。

3. 实现请求池管理，控制并发数量。

4. 编写性能测试，对比同步和异步请求的性能。
