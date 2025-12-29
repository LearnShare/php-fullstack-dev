# 5.12.2 异步请求

## 概述

异步请求能够提高并发处理能力，在等待一个请求响应时可以处理其他请求。理解异步请求的概念、掌握 Promise 的使用、实现并发请求处理，对于提高应用性能、处理大量请求、优化用户体验至关重要。本节详细介绍异步请求的概念、Promise 机制、并发请求、Promise 处理等内容，帮助零基础学员掌握异步 HTTP 请求技术。

异步请求是现代 Web 应用的重要特性，可以显著提高应用的并发处理能力。理解 Promise 机制和异步处理模式，对于构建高性能的 HTTP 客户端至关重要。

**主要内容**：
- 异步请求概念（什么是异步请求、异步请求的优势、使用场景）
- Promise 概念（Promise 是什么、Promise 状态、Promise 处理、Promise 链）
- 并发请求（多个请求并发、Promise 数组、等待所有请求、部分成功处理）
- Promise 处理（then 处理、catch 错误处理、finally 最终处理、Promise 组合）
- 异步处理（非阻塞执行、回调处理、结果收集、性能优势）
- 实际应用示例和最佳实践

## 特性

- **非阻塞**：请求不会阻塞主线程
- **并发处理**：可以同时处理多个请求
- **性能提升**：显著提高处理性能
- **资源利用**：更好地利用系统资源
- **用户体验**：提升用户体验

## 异步请求概念

### 什么是异步请求

异步请求是非阻塞的 HTTP 请求，发送请求后不需要等待响应就可以继续执行其他操作。

### 异步请求的优势

1. **提高并发**：可以同时处理多个请求
2. **提高性能**：减少等待时间
3. **资源利用**：更好地利用系统资源
4. **用户体验**：提升用户体验

### 使用场景

1. **并发 API 调用**：同时调用多个 API
2. **数据聚合**：从多个源聚合数据
3. **性能优化**：优化应用性能
4. **批量处理**：批量处理请求

## Promise 概念

### Promise 是什么

Promise 是异步操作的表示，表示一个可能在未来完成的操作。

### Promise 状态

**三种状态**：
- **Pending**：进行中
- **Fulfilled**：已完成
- **Rejected**：已拒绝

### Promise 处理

**示例**：
```php
<?php
declare(strict_types=1);

use GuzzleHttp\Client;
use GuzzleHttp\Promise;

$client = new Client();

// 发送异步请求
$promise = $client->getAsync('https://api.example.com/users');

// 处理 Promise
$promise->then(
    function ($response) {
        // 成功处理
        echo "Success: " . $response->getStatusCode() . "\n";
        return $response;
    },
    function ($exception) {
        // 错误处理
        echo "Error: " . $exception->getMessage() . "\n";
        throw $exception;
    }
);
```

### Promise 链

**示例**：
```php
<?php
declare(strict_types=1);

use GuzzleHttp\Client;

$client = new Client();

$promise = $client->getAsync('https://api.example.com/users')
    ->then(function ($response) {
        // 第一个请求成功
        $data = json_decode($response->getBody()->getContents(), true);
        $userId = $data[0]['id'];
        
        // 发起第二个请求
        return $client->getAsync("https://api.example.com/users/{$userId}");
    })
    ->then(function ($response) {
        // 第二个请求成功
        $user = json_decode($response->getBody()->getContents(), true);
        echo "User: " . $user['name'] . "\n";
        return $user;
    })
    ->otherwise(function ($exception) {
        // 任何阶段的错误
        echo "Error: " . $exception->getMessage() . "\n";
        throw $exception;
    });
```

## 并发请求

### 多个请求并发

**示例**：
```php
<?php
declare(strict_types=1);

use GuzzleHttp\Client;
use GuzzleHttp\Promise;

$client = new Client();

// 创建多个 Promise
$promises = [
    'users' => $client->getAsync('https://api.example.com/users'),
    'posts' => $client->getAsync('https://api.example.com/posts'),
    'comments' => $client->getAsync('https://api.example.com/comments'),
];

// 等待所有请求完成
$results = Promise\Utils::settle($promises)->wait();

// 处理结果
foreach ($results as $key => $result) {
    if ($result['state'] === 'fulfilled') {
        $response = $result['value'];
        $data = json_decode($response->getBody()->getContents(), true);
        echo "{$key}: " . count($data) . " items\n";
    } else {
        $exception = $result['reason'];
        echo "{$key}: Error - " . $exception->getMessage() . "\n";
    }
}
```

### Promise 数组

**示例**：
```php
<?php
declare(strict_types=1);

use GuzzleHttp\Client;
use GuzzleHttp\Promise;

$client = new Client();

// 创建 Promise 数组
$promises = [];
for ($i = 1; $i <= 10; $i++) {
    $promises["user_{$i}"] = $client->getAsync("https://api.example.com/users/{$i}");
}

// 等待所有请求完成
$results = Promise\Utils::settle($promises)->wait();
```

### 等待所有请求

**示例**：
```php
<?php
declare(strict_types=1);

use GuzzleHttp\Client;
use GuzzleHttp\Promise;

$client = new Client();

$promises = [
    $client->getAsync('https://api.example.com/users'),
    $client->getAsync('https://api.example.com/posts'),
];

// 等待所有请求完成（全部成功或全部失败）
$results = Promise\Utils::all($promises)->wait();

// 处理结果
foreach ($results as $response) {
    $data = json_decode($response->getBody()->getContents(), true);
    echo count($data) . " items\n";
}
```

### 部分成功处理

**示例**：
```php
<?php
declare(strict_types=1);

use GuzzleHttp\Client;
use GuzzleHttp\Promise;

$client = new Client();

$promises = [
    'users' => $client->getAsync('https://api.example.com/users'),
    'posts' => $client->getAsync('https://api.example.com/posts'),
    'invalid' => $client->getAsync('https://api.example.com/invalid'),
];

// 使用 settle 处理部分成功
$results = Promise\Utils::settle($promises)->wait();

$data = [];
foreach ($results as $key => $result) {
    if ($result['state'] === 'fulfilled') {
        $response = $result['value'];
        $data[$key] = json_decode($response->getBody()->getContents(), true);
    } else {
        echo "{$key} failed: " . $result['reason']->getMessage() . "\n";
    }
}
```

## Promise 处理

### then 处理

**示例**：
```php
<?php
declare(strict_types=1);

use GuzzleHttp\Client;

$client = new Client();

$promise = $client->getAsync('https://api.example.com/users')
    ->then(function ($response) {
        // 成功处理
        $data = json_decode($response->getBody()->getContents(), true);
        return $data;
    });
```

### catch 错误处理

**示例**：
```php
<?php
declare(strict_types=1);

use GuzzleHttp\Client;
use GuzzleHttp\Exception\RequestException;

$client = new Client();

$promise = $client->getAsync('https://api.example.com/users')
    ->then(function ($response) {
        return $response;
    })
    ->otherwise(function ($exception) {
        if ($exception instanceof RequestException) {
            echo "Request failed: " . $exception->getMessage() . "\n";
        } else {
            echo "Error: " . $exception->getMessage() . "\n";
        }
        throw $exception;
    });
```

### finally 最终处理

**示例**：
```php
<?php
declare(strict_types=1);

use GuzzleHttp\Client;

$client = new Client();

$promise = $client->getAsync('https://api.example.com/users')
    ->then(function ($response) {
        return $response;
    })
    ->otherwise(function ($exception) {
        throw $exception;
    })
    ->wait();  // 等待完成

// 无论成功或失败都会执行
echo "Request completed\n";
```

### Promise 组合

**示例**：
```php
<?php
declare(strict_types=1);

use GuzzleHttp\Client;
use GuzzleHttp\Promise;

$client = new Client();

// 组合多个 Promise
$promise1 = $client->getAsync('https://api.example.com/users');
$promise2 = $client->getAsync('https://api.example.com/posts');

// 等待所有完成
$results = Promise\Utils::all([$promise1, $promise2])->wait();

// 或者等待任意一个完成
$result = Promise\Utils::any([$promise1, $promise2])->wait();
```

## 异步处理

### 非阻塞执行

**示例**：
```php
<?php
declare(strict_types=1);

use GuzzleHttp\Client;

$client = new Client();

// 发送异步请求（不阻塞）
$promise = $client->getAsync('https://api.example.com/users');

// 继续执行其他操作
echo "Request sent, continuing...\n";
doSomethingElse();

// 稍后等待结果
$response = $promise->wait();
```

### 回调处理

**示例**：
```php
<?php
declare(strict_types=1);

use GuzzleHttp\Client;

$client = new Client();

$promise = $client->getAsync('https://api.example.com/users')
    ->then(function ($response) {
        // 成功回调
        handleSuccess($response);
    })
    ->otherwise(function ($exception) {
        // 错误回调
        handleError($exception);
    });
```

### 结果收集

**示例**：
```php
<?php
declare(strict_types=1);

use GuzzleHttp\Client;
use GuzzleHttp\Promise;

$client = new Client();

$promises = [];
for ($i = 1; $i <= 100; $i++) {
    $promises["item_{$i}"] = $client->getAsync("https://api.example.com/items/{$i}");
}

// 收集所有结果
$results = Promise\Utils::settle($promises)->wait();

$successCount = 0;
$failureCount = 0;

foreach ($results as $key => $result) {
    if ($result['state'] === 'fulfilled') {
        $successCount++;
    } else {
        $failureCount++;
    }
}

echo "Success: {$successCount}, Failed: {$failureCount}\n";
```

### 性能优势

**示例**：
```php
<?php
declare(strict_types=1);

use GuzzleHttp\Client;
use GuzzleHttp\Promise;

$client = new Client();

// 同步请求（慢）
$start = microtime(true);
for ($i = 1; $i <= 10; $i++) {
    $response = $client->get("https://api.example.com/items/{$i}");
}
$syncTime = microtime(true) - $start;

// 异步请求（快）
$start = microtime(true);
$promises = [];
for ($i = 1; $i <= 10; $i++) {
    $promises[] = $client->getAsync("https://api.example.com/items/{$i}");
}
Promise\Utils::all($promises)->wait();
$asyncTime = microtime(true) - $start;

echo "Sync: {$syncTime}s, Async: {$asyncTime}s\n";
```

## 使用场景

### 并发 API 调用

- 同时调用多个 API
- 聚合多个数据源
- 提高响应速度

### 数据聚合

- 从多个源获取数据
- 合并数据结果
- 数据同步

### 性能优化

- 减少等待时间
- 提高并发处理能力
- 优化用户体验

### 批量处理

- 批量请求处理
- 批量数据获取
- 批量操作

## 注意事项

### Promise 处理

- **及时处理**：及时处理 Promise 结果
- **错误处理**：正确处理错误
- **资源释放**：及时释放资源

### 错误处理

- **异常捕获**：捕获所有异常
- **错误分类**：区分不同类型的错误
- **错误恢复**：实现错误恢复机制

### 资源管理

- **连接管理**：管理 HTTP 连接
- **内存使用**：注意内存使用
- **超时控制**：设置合理的超时时间

### 超时控制

- **请求超时**：设置请求超时
- **总体超时**：设置总体超时
- **超时处理**：处理超时情况

## 常见问题

### 如何发送异步请求？

使用 `getAsync()`、`postAsync()` 等方法：
```php
$promise = $client->getAsync('https://api.example.com/users');
```

### Promise 如何处理？

使用 `then()` 和 `otherwise()` 方法：
```php
$promise->then(function ($response) {
    // 成功处理
})->otherwise(function ($exception) {
    // 错误处理
});
```

### 如何等待多个请求？

使用 `Promise\Utils::all()` 或 `settle()`：
```php
$results = Promise\Utils::all($promises)->wait();
```

### 异步请求的性能优势？

异步请求可以并发处理多个请求，显著减少总等待时间，提高应用性能。

## 最佳实践

### 使用异步请求提高并发

- 对于多个独立请求，使用异步请求
- 利用并发提高性能
- 注意资源限制

### 正确处理 Promise

- 使用 `then()` 处理成功情况
- 使用 `otherwise()` 处理错误
- 及时等待 Promise 完成

### 实现错误处理

- 捕获所有异常
- 提供清晰的错误信息
- 实现错误恢复机制

### 控制并发数量

- 限制并发请求数量
- 使用队列管理请求
- 避免资源耗尽

## 相关章节

- **[5.12.1 Guzzle 基础](section-01-guzzle-basics.md)**：了解 Guzzle 基础的详细内容
- **[5.12.3 错误处理与重试](section-03-error-handling-retry.md)**：了解错误处理和重试的详细内容

## 练习任务

1. **实现异步请求处理**
   - 发送异步请求
   - 处理 Promise
   - 错误处理

2. **实现并发请求**
   - 多个请求并发
   - 结果收集
   - 部分成功处理

3. **实现 Promise 链**
   - 链式请求
   - 依赖处理
   - 错误传播

4. **实现批量请求处理**
   - 批量请求
   - 结果聚合
   - 性能优化

5. **实现完整的异步 HTTP 客户端**
   - 异步请求封装
   - 并发控制
   - 错误处理和重试
