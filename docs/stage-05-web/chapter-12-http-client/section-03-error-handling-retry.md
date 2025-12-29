# 5.12.3 错误处理与重试

## 概述

错误处理和重试机制是构建健壮 HTTP 客户端的关键。理解 Guzzle 的异常类型、掌握错误处理方法、实现重试机制，对于构建可靠的 HTTP 客户端、处理网络不稳定、提高请求成功率至关重要。本节详细介绍错误处理概述、Guzzle 异常类型、异常处理、重试机制、重试策略、指数退避等内容，帮助零基础学员实现可靠的 HTTP 客户端。

网络请求可能因为各种原因失败，如网络问题、服务器错误、超时等。实现完善的错误处理和重试机制，对于构建可靠的 HTTP 客户端至关重要。

**主要内容**：
- 错误处理概述（错误类型、错误分类、错误响应处理）
- Guzzle 异常类型（RequestException、ClientException、ServerException、ConnectException）
- 异常处理（异常捕获、错误分类、错误响应处理、错误日志）
- 重试机制（重试条件、重试次数、重试间隔、重试策略）
- 重试策略（固定间隔重试、指数退避、条件重试、最大重试次数）
- 指数退避（指数退避算法、退避时间计算、最大退避时间）
- 实际应用示例和最佳实践

## 特性

- **错误分类**：区分不同类型的错误
- **智能重试**：根据错误类型决定是否重试
- **指数退避**：使用指数退避避免服务器压力
- **可配置**：支持灵活的配置选项
- **易于使用**：提供简洁的 API

## 错误处理概述

### 错误类型

**常见错误类型**：
- **网络错误**：连接失败、超时等
- **HTTP 错误**：4xx、5xx 状态码
- **客户端错误**：请求格式错误等
- **服务器错误**：服务器内部错误等

### 错误分类

**示例**：
```php
<?php
declare(strict_types=1);

use GuzzleHttp\Exception\RequestException;
use GuzzleHttp\Exception\ClientException;
use GuzzleHttp\Exception\ServerException;
use GuzzleHttp\Exception\ConnectException;

function classifyError(\Exception $exception): string
{
    if ($exception instanceof ConnectException) {
        return 'network_error';
    } elseif ($exception instanceof ClientException) {
        return 'client_error';
    } elseif ($exception instanceof ServerException) {
        return 'server_error';
    } elseif ($exception instanceof RequestException) {
        return 'request_error';
    }
    return 'unknown_error';
}
```

### 错误响应处理

**示例**：
```php
<?php
declare(strict_types=1);

use GuzzleHttp\Exception\ClientException;

try {
    $response = $client->get('https://api.example.com/users');
} catch (ClientException $e) {
    $response = $e->getResponse();
    $statusCode = $response->getStatusCode();
    $body = $response->getBody()->getContents();
    
    echo "Error {$statusCode}: {$body}\n";
}
```

## Guzzle 异常类型

### RequestException

**RequestException**：所有请求异常的基类。

**示例**：
```php
<?php
declare(strict_types=1);

use GuzzleHttp\Exception\RequestException;

try {
    $response = $client->get('https://api.example.com/users');
} catch (RequestException $e) {
    echo "Request failed: " . $e->getMessage() . "\n";
    
    if ($e->hasResponse()) {
        $response = $e->getResponse();
        echo "Status: " . $response->getStatusCode() . "\n";
    }
}
```

### ClientException（4xx）

**ClientException**：客户端错误（4xx 状态码）。

**示例**：
```php
<?php
declare(strict_types=1);

use GuzzleHttp\Exception\ClientException;

try {
    $response = $client->get('https://api.example.com/users/999');
} catch (ClientException $e) {
    $statusCode = $e->getResponse()->getStatusCode();
    
    if ($statusCode === 404) {
        echo "Resource not found\n";
    } elseif ($statusCode === 401) {
        echo "Unauthorized\n";
    } elseif ($statusCode === 403) {
        echo "Forbidden\n";
    }
}
```

### ServerException（5xx）

**ServerException**：服务器错误（5xx 状态码）。

**示例**：
```php
<?php
declare(strict_types=1);

use GuzzleHttp\Exception\ServerException;

try {
    $response = $client->get('https://api.example.com/users');
} catch (ServerException $e) {
    $statusCode = $e->getResponse()->getStatusCode();
    
    if ($statusCode === 500) {
        echo "Internal server error\n";
    } elseif ($statusCode === 503) {
        echo "Service unavailable\n";
    }
}
```

### ConnectException

**ConnectException**：连接异常（网络问题）。

**示例**：
```php
<?php
declare(strict_types=1);

use GuzzleHttp\Exception\ConnectException;

try {
    $response = $client->get('https://api.example.com/users', [
        'timeout' => 5,
    ]);
} catch (ConnectException $e) {
    echo "Connection failed: " . $e->getMessage() . "\n";
    // 可能是网络问题，可以重试
}
```

## 异常处理

### 异常捕获

**示例**：
```php
<?php
declare(strict_types=1);

use GuzzleHttp\Exception\RequestException;

try {
    $response = $client->get('https://api.example.com/users');
    $data = json_decode($response->getBody()->getContents(), true);
} catch (RequestException $e) {
    // 处理请求异常
    handleRequestError($e);
} catch (\Exception $e) {
    // 处理其他异常
    handleGenericError($e);
}
```

### 错误分类

**示例**：
```php
<?php
declare(strict_types=1);

use GuzzleHttp\Exception\RequestException;
use GuzzleHttp\Exception\ClientException;
use GuzzleHttp\Exception\ServerException;
use GuzzleHttp\Exception\ConnectException;

function handleError(\Exception $exception): void
{
    if ($exception instanceof ConnectException) {
        // 网络错误，可以重试
        echo "Network error, will retry\n";
    } elseif ($exception instanceof ServerException) {
        // 服务器错误，可以重试
        echo "Server error, will retry\n";
    } elseif ($exception instanceof ClientException) {
        // 客户端错误，通常不需要重试
        echo "Client error, no retry\n";
    } else {
        echo "Unknown error\n";
    }
}
```

### 错误响应处理

**示例**：
```php
<?php
declare(strict_types=1);

use GuzzleHttp\Exception\ClientException;

try {
    $response = $client->post('https://api.example.com/users', [
        'json' => ['name' => 'John'],
    ]);
} catch (ClientException $e) {
    $response = $e->getResponse();
    $statusCode = $response->getStatusCode();
    $body = json_decode($response->getBody()->getContents(), true);
    
    if (isset($body['error'])) {
        echo "API Error: " . $body['error']['message'] . "\n";
    } else {
        echo "HTTP Error {$statusCode}\n";
    }
}
```

### 错误日志

**示例**：
```php
<?php
declare(strict_types=1);

use GuzzleHttp\Exception\RequestException;

function logError(RequestException $e, array $context = []): void
{
    $log = [
        'message' => $e->getMessage(),
        'code' => $e->getCode(),
        'file' => $e->getFile(),
        'line' => $e->getLine(),
        'context' => $context,
    ];
    
    if ($e->hasResponse()) {
        $response = $e->getResponse();
        $log['status_code'] = $response->getStatusCode();
        $log['response_body'] = $response->getBody()->getContents();
    }
    
    error_log(json_encode($log));
}
```

## 重试机制

### 重试条件

**示例**：
```php
<?php
declare(strict_types=1);

use GuzzleHttp\Exception\RequestException;
use GuzzleHttp\Exception\ServerException;
use GuzzleHttp\Exception\ConnectException;

function shouldRetry(RequestException $e, int $retries): bool
{
    // 超过最大重试次数
    if ($retries >= 3) {
        return false;
    }
    
    // 网络错误，可以重试
    if ($e instanceof ConnectException) {
        return true;
    }
    
    // 服务器错误，可以重试
    if ($e instanceof ServerException) {
        $statusCode = $e->getResponse()->getStatusCode();
        return in_array($statusCode, [500, 502, 503, 504], true);
    }
    
    // 客户端错误，通常不重试
    return false;
}
```

### 重试次数

**示例**：
```php
<?php
declare(strict_types=1);

function retryRequest(callable $request, int $maxRetries = 3): mixed
{
    $retries = 0;
    
    while ($retries < $maxRetries) {
        try {
            return $request();
        } catch (RequestException $e) {
            $retries++;
            
            if ($retries >= $maxRetries || !shouldRetry($e, $retries)) {
                throw $e;
            }
            
            // 等待后重试
            sleep(pow(2, $retries));  // 指数退避
        }
    }
    
    throw new \RuntimeException('Max retries exceeded');
}
```

### 重试间隔

**示例**：
```php
<?php
declare(strict_types=1);

function retryWithBackoff(callable $request, int $maxRetries = 3): mixed
{
    $retries = 0;
    $baseDelay = 1;  // 基础延迟（秒）
    
    while ($retries < $maxRetries) {
        try {
            return $request();
        } catch (RequestException $e) {
            $retries++;
            
            if ($retries >= $maxRetries || !shouldRetry($e, $retries)) {
                throw $e;
            }
            
            // 指数退避
            $delay = $baseDelay * pow(2, $retries - 1);
            sleep($delay);
        }
    }
    
    throw new \RuntimeException('Max retries exceeded');
}
```

### 重试策略

**示例**：
```php
<?php
declare(strict_types=1);

class RetryStrategy
{
    public function __construct(
        private int $maxRetries = 3,
        private int $baseDelay = 1,
        private int $maxDelay = 60
    ) {}

    public function retry(callable $request): mixed
    {
        $retries = 0;
        
        while ($retries < $this->maxRetries) {
            try {
                return $request();
            } catch (RequestException $e) {
                $retries++;
                
                if ($retries >= $this->maxRetries || !$this->shouldRetry($e)) {
                    throw $e;
                }
                
                $delay = $this->calculateDelay($retries);
                sleep($delay);
            }
        }
        
        throw new \RuntimeException('Max retries exceeded');
    }

    private function shouldRetry(RequestException $e): bool
    {
        if ($e instanceof ConnectException) {
            return true;
        }
        
        if ($e instanceof ServerException) {
            $statusCode = $e->getResponse()->getStatusCode();
            return in_array($statusCode, [500, 502, 503, 504], true);
        }
        
        return false;
    }

    private function calculateDelay(int $retries): int
    {
        $delay = $this->baseDelay * pow(2, $retries - 1);
        return min($delay, $this->maxDelay);
    }
}
```

## 重试策略

### 固定间隔重试

**示例**：
```php
<?php
declare(strict_types=1);

function retryWithFixedDelay(callable $request, int $maxRetries = 3, int $delay = 1): mixed
{
    $retries = 0;
    
    while ($retries < $maxRetries) {
        try {
            return $request();
        } catch (RequestException $e) {
            $retries++;
            
            if ($retries >= $maxRetries || !shouldRetry($e, $retries)) {
                throw $e;
            }
            
            sleep($delay);  // 固定延迟
        }
    }
    
    throw new \RuntimeException('Max retries exceeded');
}
```

### 指数退避

**示例**：
```php
<?php
declare(strict_types=1);

function retryWithExponentialBackoff(callable $request, int $maxRetries = 3): mixed
{
    $retries = 0;
    $baseDelay = 1;
    
    while ($retries < $maxRetries) {
        try {
            return $request();
        } catch (RequestException $e) {
            $retries++;
            
            if ($retries >= $maxRetries || !shouldRetry($e, $retries)) {
                throw $e;
            }
            
            // 指数退避：1s, 2s, 4s, 8s...
            $delay = $baseDelay * pow(2, $retries - 1);
            sleep($delay);
        }
    }
    
    throw new \RuntimeException('Max retries exceeded');
}
```

### 条件重试

**示例**：
```php
<?php
declare(strict_types=1);

function retryWithCondition(callable $request, callable $shouldRetry, int $maxRetries = 3): mixed
{
    $retries = 0;
    
    while ($retries < $maxRetries) {
        try {
            return $request();
        } catch (RequestException $e) {
            $retries++;
            
            if ($retries >= $maxRetries || !$shouldRetry($e, $retries)) {
                throw $e;
            }
            
            sleep(pow(2, $retries - 1));
        }
    }
    
    throw new \RuntimeException('Max retries exceeded');
}
```

### 最大重试次数

**示例**：
```php
<?php
declare(strict_types=1);

class RetryConfig
{
    public function __construct(
        private int $maxRetries = 3,
        private array $retryableStatusCodes = [500, 502, 503, 504],
        private bool $retryOnTimeout = true,
        private bool $retryOnConnectionError = true
    ) {}

    public function shouldRetry(RequestException $e, int $retries): bool
    {
        if ($retries >= $this->maxRetries) {
            return false;
        }
        
        if ($e instanceof ConnectException && $this->retryOnConnectionError) {
            return true;
        }
        
        if ($e instanceof ServerException) {
            $statusCode = $e->getResponse()->getStatusCode();
            return in_array($statusCode, $this->retryableStatusCodes, true);
        }
        
        return false;
    }
}
```

## 指数退避

### 指数退避算法

**示例**：
```php
<?php
declare(strict_types=1);

function exponentialBackoff(int $retry, int $baseDelay = 1, int $maxDelay = 60): int
{
    $delay = $baseDelay * pow(2, $retry - 1);
    return min($delay, $maxDelay);
}

// 使用
for ($i = 1; $i <= 5; $i++) {
    $delay = exponentialBackoff($i);
    echo "Retry {$i}: wait {$delay} seconds\n";
}
// 输出：
// Retry 1: wait 1 seconds
// Retry 2: wait 2 seconds
// Retry 3: wait 4 seconds
// Retry 4: wait 8 seconds
// Retry 5: wait 16 seconds
```

### 退避时间计算

**示例**：
```php
<?php
declare(strict_types=1);

class ExponentialBackoff
{
    public function __construct(
        private int $baseDelay = 1,
        private int $maxDelay = 60,
        private float $multiplier = 2.0
    ) {}

    public function calculateDelay(int $retry): int
    {
        $delay = $this->baseDelay * pow($this->multiplier, $retry - 1);
        return min((int) $delay, $this->maxDelay);
    }
}
```

### 最大退避时间

**示例**：
```php
<?php
declare(strict_types=1);

function calculateBackoffDelay(int $retry, int $maxDelay = 60): int
{
    $delay = pow(2, $retry - 1);  // 1, 2, 4, 8, 16...
    return min($delay, $maxDelay);  // 限制最大延迟
}
```

## 使用 Guzzle 中间件实现重试

**示例**：
```php
<?php
declare(strict_types=1);

use GuzzleHttp\Client;
use GuzzleHttp\HandlerStack;
use GuzzleHttp\Middleware;
use GuzzleHttp\Psr7\Request;
use GuzzleHttp\Psr7\Response;

$stack = HandlerStack::create();

// 添加重试中间件
$stack->push(Middleware::retry(function ($retries, Request $request, Response $response = null, \Exception $exception = null) {
    // 最大重试 3 次
    if ($retries >= 3) {
        return false;
    }
    
    // 服务器错误或连接错误时重试
    if ($exception instanceof \GuzzleHttp\Exception\ConnectException) {
        return true;
    }
    
    if ($response && $response->getStatusCode() >= 500) {
        return true;
    }
    
    return false;
}, function ($retries) {
    // 指数退避
    return pow(2, $retries) * 1000;  // 毫秒
}));

$client = new Client(['handler' => $stack]);
```

## 使用场景

### 不稳定的网络环境

- 网络连接不稳定
- 间歇性网络错误
- 移动网络环境

### 第三方服务调用

- 第三方 API 不稳定
- 服务临时不可用
- 限流和降级

### 关键操作重试

- 支付操作
- 重要数据同步
- 关键业务操作

### 提高可靠性

- 提高请求成功率
- 减少失败影响
- 改善用户体验

## 注意事项

### 重试条件设计

- **可重试错误**：网络错误、服务器错误
- **不可重试错误**：客户端错误（4xx）
- **幂等性**：确保重试操作是幂等的

### 重试次数限制

- **合理限制**：设置合理的重试次数
- **避免无限重试**：避免无限重试导致资源浪费
- **快速失败**：某些错误应该快速失败

### 幂等性考虑

- **GET 请求**：通常是幂等的
- **POST 请求**：可能不是幂等的
- **PUT/DELETE**：通常是幂等的
- **设计幂等接口**：设计幂等的 API 接口

### 性能影响

- **延迟影响**：重试会增加响应时间
- **资源消耗**：重试会消耗更多资源
- **平衡考虑**：平衡可靠性和性能

## 常见问题

### 如何处理 HTTP 错误？

使用 try-catch 捕获异常：
```php
try {
    $response = $client->get('https://api.example.com/users');
} catch (ClientException $e) {
    // 处理客户端错误
} catch (ServerException $e) {
    // 处理服务器错误
}
```

### 如何实现重试机制？

使用重试函数或中间件：
```php
$result = retryWithBackoff(function() use ($client) {
    return $client->get('https://api.example.com/users');
});
```

### 如何选择重试策略？

- **固定间隔**：简单但可能增加服务器压力
- **指数退避**：推荐，避免服务器压力
- **条件重试**：根据错误类型决定是否重试

### 重试的性能影响？

- **延迟增加**：重试会增加响应时间
- **资源消耗**：重试会消耗更多资源
- **需要平衡**：平衡可靠性和性能

## 最佳实践

### 区分可重试和不可重试错误

- **可重试**：网络错误、服务器错误（5xx）
- **不可重试**：客户端错误（4xx）
- **条件判断**：根据错误类型决定是否重试

### 实现指数退避

- **避免服务器压力**：使用指数退避
- **设置最大延迟**：限制最大退避时间
- **随机抖动**：添加随机抖动避免雷群效应

### 限制重试次数

- **合理限制**：设置合理的重试次数（通常 3-5 次）
- **快速失败**：某些错误应该快速失败
- **避免无限重试**：避免无限重试

### 记录重试日志

- **记录重试信息**：记录重试次数、原因等
- **监控重试率**：监控重试率，发现问题
- **分析重试原因**：分析重试原因，优化系统

## 相关章节

- **[5.12.1 Guzzle 基础](section-01-guzzle-basics.md)**：了解 Guzzle 基础的详细内容
- **[5.12.2 异步请求](section-02-async-requests.md)**：了解异步请求的详细内容

## 练习任务

1. **实现错误处理系统**
   - 异常分类和处理
   - 错误响应处理
   - 错误日志记录

2. **实现重试机制**
   - 重试条件判断
   - 指数退避算法
   - 重试策略配置

3. **实现重试中间件**
   - Guzzle 中间件实现
   - 可配置重试策略
   - 重试日志记录

4. **实现完整的错误处理和重试系统**
   - 错误分类和处理
   - 智能重试机制
   - 性能监控和优化

5. **实现健壮的 HTTP 客户端**
   - 完整的错误处理
   - 智能重试机制
   - 性能优化
