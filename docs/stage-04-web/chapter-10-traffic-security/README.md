# 4.10 流量治理与安全

## 目标

- 理解 API Rate Limiting 的作用与实现方式。
- 掌握固定窗口、滑动窗口等限流算法。
- 实现防止重复提交的幂等性机制。
- 了解请求签名（Timestamp + HMAC）的实现。

## API Rate Limiting

### 为什么需要限流

- **防止滥用**：限制单个用户或 IP 的请求频率。
- **保护服务**：防止 DDoS 攻击和资源耗尽。
- **公平使用**：确保所有用户公平使用资源。
- **成本控制**：控制 API 调用成本。

### 固定窗口限流

```php
<?php
declare(strict_types=1);

class FixedWindowRateLimiter
{
    private \Redis $redis;
    private int $maxRequests;
    private int $windowSeconds;

    public function __construct(\Redis $redis, int $maxRequests, int $windowSeconds)
    {
        $this->redis = $redis;
        $this->maxRequests = $maxRequests;
        $this->windowSeconds = $windowSeconds;
    }

    public function isAllowed(string $key): bool
    {
        $currentWindow = (int) (time() / $this->windowSeconds);
        $windowKey = "rate_limit:{$key}:{$currentWindow}";
        
        $current = $this->redis->incr($windowKey);
        
        if ($current === 1) {
            // 设置过期时间
            $this->redis->expire($windowKey, $this->windowSeconds);
        }
        
        return $current <= $this->maxRequests;
    }

    public function getRemaining(string $key): int
    {
        $currentWindow = (int) (time() / $this->windowSeconds);
        $windowKey = "rate_limit:{$key}:{$currentWindow}";
        
        $current = (int) $this->redis->get($windowKey);
        return max(0, $this->maxRequests - $current);
    }
}

// 使用
$redis = new \Redis();
$redis->connect('127.0.0.1', 6379);

$limiter = new FixedWindowRateLimiter($redis, 100, 60); // 每分钟 100 次

$clientId = $_SERVER['REMOTE_ADDR'];
if (!$limiter->isAllowed($clientId)) {
    http_response_code(429);
    header('X-RateLimit-Limit: 100');
    header('X-RateLimit-Remaining: 0');
    echo json_encode(['error' => 'Too many requests']);
    exit;
}

header('X-RateLimit-Limit: 100');
header('X-RateLimit-Remaining: ' . $limiter->getRemaining($clientId));
```

### 滑动窗口限流

```php
<?php
declare(strict_types=1);

class SlidingWindowRateLimiter
{
    private \Redis $redis;
    private int $maxRequests;
    private int $windowSeconds;

    public function __construct(\Redis $redis, int $maxRequests, int $windowSeconds)
    {
        $this->redis = $redis;
        $this->maxRequests = $maxRequests;
        $this->windowSeconds = $windowSeconds;
    }

    public function isAllowed(string $key): bool
    {
        $now = time();
        $windowKey = "rate_limit:{$key}";
        
        // 使用有序集合存储请求时间戳
        $this->redis->zremrangebyscore($windowKey, 0, $now - $this->windowSeconds);
        
        $count = $this->redis->zcard($windowKey);
        
        if ($count >= $this->maxRequests) {
            return false;
        }
        
        // 添加当前请求
        $this->redis->zadd($windowKey, $now, $now);
        $this->redis->expire($windowKey, $this->windowSeconds);
        
        return true;
    }

    public function getRemaining(string $key): int
    {
        $now = time();
        $windowKey = "rate_limit:{$key}";
        
        $this->redis->zremrangebyscore($windowKey, 0, $now - $this->windowSeconds);
        $count = $this->redis->zcard($windowKey);
        
        return max(0, $this->maxRequests - $count);
    }
}

// 使用
$limiter = new SlidingWindowRateLimiter($redis, 100, 60);
$clientId = $_SERVER['REMOTE_ADDR'];

if (!$limiter->isAllowed($clientId)) {
    http_response_code(429);
    echo json_encode(['error' => 'Too many requests']);
    exit;
}
```

### 令牌桶算法

```php
<?php
declare(strict_types=1);

class TokenBucketRateLimiter
{
    private \Redis $redis;
    private int $capacity;
    private int $refillRate;  // 每秒补充的令牌数

    public function __construct(\Redis $redis, int $capacity, int $refillRate)
    {
        $this->redis = $redis;
        $this->capacity = $capacity;
        $this->refillRate = $refillRate;
    }

    public function isAllowed(string $key): bool
    {
        $bucketKey = "token_bucket:{$key}";
        $now = microtime(true);
        
        // 获取当前令牌数和最后更新时间
        $data = $this->redis->hmget($bucketKey, ['tokens', 'last_refill']);
        $tokens = (float) ($data[0] ?? $this->capacity);
        $lastRefill = (float) ($data[1] ?? $now);
        
        // 计算需要补充的令牌
        $elapsed = $now - $lastRefill;
        $tokens = min($this->capacity, $tokens + $elapsed * $this->refillRate);
        
        if ($tokens < 1) {
            // 令牌不足
            $this->redis->hset($bucketKey, 'tokens', $tokens);
            $this->redis->hset($bucketKey, 'last_refill', $now);
            $this->redis->expire($bucketKey, 3600);
            return false;
        }
        
        // 消耗一个令牌
        $tokens -= 1;
        $this->redis->hset($bucketKey, 'tokens', $tokens);
        $this->redis->hset($bucketKey, 'last_refill', $now);
        $this->redis->expire($bucketKey, 3600);
        
        return true;
    }
}

// 使用
$limiter = new TokenBucketRateLimiter($redis, 100, 10); // 容量 100，每秒补充 10 个
```

### 限流中间件

```php
<?php
declare(strict_types=1);

class RateLimitMiddleware
{
    private FixedWindowRateLimiter $limiter;
    private string $identifierKey;

    public function __construct(FixedWindowRateLimiter $limiter, string $identifierKey = 'ip')
    {
        $this->limiter = $limiter;
        $this->identifierKey = $identifierKey;
    }

    public function handle(): void
    {
        $identifier = $this->getIdentifier();
        
        if (!$this->limiter->isAllowed($identifier)) {
            http_response_code(429);
            header('Content-Type: application/json');
            header('Retry-After: 60');
            echo json_encode([
                'error' => 'Too many requests',
                'message' => 'Rate limit exceeded. Please try again later.',
            ]);
            exit;
        }
        
        // 设置响应头
        header('X-RateLimit-Limit: ' . $this->limiter->maxRequests);
        header('X-RateLimit-Remaining: ' . $this->limiter->getRemaining($identifier));
    }

    private function getIdentifier(): string
    {
        return match ($this->identifierKey) {
            'ip' => $_SERVER['REMOTE_ADDR'] ?? 'unknown',
            'user' => getCurrentUserId() ?? 'anonymous',
            'api_key' => $_SERVER['HTTP_X_API_KEY'] ?? 'unknown',
            default => 'unknown',
        };
    }
}

// 使用
$limiter = new FixedWindowRateLimiter($redis, 100, 60);
$middleware = new RateLimitMiddleware($limiter, 'ip');
$middleware->handle();
```

## 防止重复提交

### 幂等性 Token

```php
<?php
declare(strict_types=1);

class IdempotencyManager
{
    private \Redis $redis;
    private int $ttl = 3600;  // 1 小时

    public function __construct(\Redis $redis)
    {
        $this->redis = $redis;
    }

    public function generateToken(): string
    {
        return bin2hex(random_bytes(16));
    }

    public function checkAndSet(string $token, string $response): bool
    {
        $key = "idempotency:{$token}";
        
        // 尝试设置，如果已存在则返回 false
        $result = $this->redis->set($key, $response, ['nx', 'ex' => $this->ttl]);
        
        return $result !== false;
    }

    public function getResponse(string $token): ?string
    {
        $key = "idempotency:{$token}";
        $response = $this->redis->get($key);
        return $response !== false ? $response : null;
    }
}

// 使用
$idempotency = new IdempotencyManager($redis);

// 客户端请求时携带 Idempotency-Key
$token = $_SERVER['HTTP_IDEMPOTENCY_KEY'] ?? null;

if ($token === null) {
    // 首次请求，生成 Token
    $token = $idempotency->generateToken();
    header("X-Idempotency-Key: {$token}");
    
    // 处理请求
    $response = json_encode(['success' => true, 'id' => 123]);
    
    // 保存响应
    $idempotency->checkAndSet($token, $response);
    
    echo $response;
} else {
    // 重复请求，返回缓存的响应
    $cachedResponse = $idempotency->getResponse($token);
    
    if ($cachedResponse !== null) {
        echo $cachedResponse;
    } else {
        http_response_code(400);
        echo json_encode(['error' => 'Invalid idempotency key']);
    }
}
```

### 请求去重

```php
<?php
declare(strict_types=1);

class RequestDeduplicator
{
    private \Redis $redis;
    private int $ttl = 300;  // 5 分钟

    public function __construct(\Redis $redis)
    {
        $this->redis = $redis;
    }

    public function generateRequestHash(array $data): string
    {
        // 生成请求的唯一哈希
        $normalized = json_encode($data, JSON_SORT_KEYS);
        return hash('sha256', $normalized);
    }

    public function isDuplicate(string $key, string $requestHash): bool
    {
        $cacheKey = "request:{$key}:{$requestHash}";
        
        // 检查是否已处理
        if ($this->redis->exists($cacheKey)) {
            return true;
        }
        
        // 标记为已处理
        $this->redis->setex($cacheKey, $this->ttl, '1');
        
        return false;
    }
}

// 使用
$deduplicator = new RequestDeduplicator($redis);

$requestData = [
    'user_id' => 1,
    'amount' => 100,
    'timestamp' => time(),
];

$requestHash = $deduplicator->generateRequestHash($requestData);
$clientId = $_SERVER['REMOTE_ADDR'];

if ($deduplicator->isDuplicate($clientId, $requestHash)) {
    http_response_code(409);
    echo json_encode(['error' => 'Duplicate request']);
    exit;
}

// 处理请求
processPayment($requestData);
```

## 请求签名

### HMAC 签名

```php
<?php
declare(strict_types=1);

class RequestSigner
{
    private string $secret;

    public function __construct(string $secret)
    {
        $this->secret = $secret;
    }

    public function sign(string $method, string $path, string $body, int $timestamp): string
    {
        // 构建签名字符串
        $message = "{$method}\n{$path}\n{$body}\n{$timestamp}";
        
        // 生成 HMAC 签名
        return hash_hmac('sha256', $message, $this->secret);
    }

    public function verify(string $method, string $path, string $body, int $timestamp, string $signature): bool
    {
        // 验证时间戳（防止重放攻击）
        $now = time();
        if (abs($now - $timestamp) > 300) {  // 5 分钟有效期
            return false;
        }
        
        // 验证签名
        $expectedSignature = $this->sign($method, $path, $body, $timestamp);
        return hash_equals($expectedSignature, $signature);
    }
}

// 客户端生成签名
$signer = new RequestSigner('secret-key');
$method = 'POST';
$path = '/api/users';
$body = json_encode(['name' => 'Alice']);
$timestamp = time();
$signature = $signer->sign($method, $path, $body, $timestamp);

// 发送请求时携带签名
$headers = [
    'X-Timestamp: ' . $timestamp,
    'X-Signature: ' . $signature,
];

// 服务器验证
$serverSigner = new RequestSigner('secret-key');
$method = $_SERVER['REQUEST_METHOD'];
$path = parse_url($_SERVER['REQUEST_URI'], PHP_URL_PATH);
$body = file_get_contents('php://input');
$timestamp = (int) ($_SERVER['HTTP_X_TIMESTAMP'] ?? 0);
$signature = $_SERVER['HTTP_X_SIGNATURE'] ?? '';

if (!$serverSigner->verify($method, $path, $body, $timestamp, $signature)) {
    http_response_code(401);
    echo json_encode(['error' => 'Invalid signature']);
    exit;
}
```

### 签名中间件

```php
<?php
declare(strict_types=1);

class SignatureMiddleware
{
    private RequestSigner $signer;
    private int $timestampTolerance;

    public function __construct(RequestSigner $signer, int $timestampTolerance = 300)
    {
        $this->signer = $signer;
        $this->timestampTolerance = $timestampTolerance;
    }

    public function handle(): void
    {
        $method = $_SERVER['REQUEST_METHOD'];
        $path = parse_url($_SERVER['REQUEST_URI'], PHP_URL_PATH);
        $body = file_get_contents('php://input');
        
        $timestamp = (int) ($_SERVER['HTTP_X_TIMESTAMP'] ?? 0);
        $signature = $_SERVER['HTTP_X_SIGNATURE'] ?? '';
        
        // 验证时间戳
        $now = time();
        if (abs($now - $timestamp) > $this->timestampTolerance) {
            http_response_code(401);
            echo json_encode(['error' => 'Request expired']);
            exit;
        }
        
        // 验证签名
        if (!$this->signer->verify($method, $path, $body, $timestamp, $signature)) {
            http_response_code(401);
            echo json_encode(['error' => 'Invalid signature']);
            exit;
        }
    }
}

// 使用
$signer = new RequestSigner('secret-key');
$middleware = new SignatureMiddleware($signer);
$middleware->handle();
```

## 综合安全中间件

```php
<?php
declare(strict_types=1);

class SecurityMiddleware
{
    private RateLimitMiddleware $rateLimit;
    private ?SignatureMiddleware $signature;
    private IdempotencyManager $idempotency;

    public function __construct(
        RateLimitMiddleware $rateLimit,
        ?SignatureMiddleware $signature = null,
        IdempotencyManager $idempotency
    ) {
        $this->rateLimit = $rateLimit;
        $this->signature = $signature;
        $this->idempotency = $idempotency;
    }

    public function handle(): void
    {
        // 1. 限流检查
        $this->rateLimit->handle();
        
        // 2. 签名验证（如果启用）
        if ($this->signature !== null) {
            $this->signature->handle();
        }
        
        // 3. 幂等性检查
        $idempotencyKey = $_SERVER['HTTP_IDEMPOTENCY_KEY'] ?? null;
        if ($idempotencyKey !== null) {
            $cachedResponse = $this->idempotency->getResponse($idempotencyKey);
            if ($cachedResponse !== null) {
                echo $cachedResponse;
                exit;
            }
        }
    }
}

// 使用
$redis = new \Redis();
$redis->connect('127.0.0.1', 6379);

$rateLimiter = new FixedWindowRateLimiter($redis, 100, 60);
$rateLimitMiddleware = new RateLimitMiddleware($rateLimiter);

$signer = new RequestSigner('secret-key');
$signatureMiddleware = new SignatureMiddleware($signer);

$idempotency = new IdempotencyManager($redis);

$security = new SecurityMiddleware(
    $rateLimitMiddleware,
    $signatureMiddleware,
    $idempotency
);

$security->handle();
```

## 最佳实践

### 1. 限流策略

- 根据 API 类型设置不同的限流规则。
- 对认证用户和匿名用户使用不同的限制。
- 提供清晰的错误信息和重试建议。

### 2. 幂等性

- 对写操作（POST、PUT、DELETE）实现幂等性。
- 使用 Idempotency-Key Header。
- 缓存响应并返回相同结果。

### 3. 请求签名

- 使用强密钥。
- 包含时间戳防止重放攻击。
- 验证签名和时间戳的有效性。

### 4. 监控和日志

- 记录所有限流事件。
- 监控 API 使用情况。
- 设置告警阈值。

## 练习

1. 实现一个固定窗口限流器，支持按 IP 或用户 ID 限流。

2. 创建一个滑动窗口限流器，使用 Redis 有序集合实现。

3. 实现一个幂等性管理系统，支持请求去重和响应缓存。

4. 编写一个请求签名验证系统，支持 HMAC 签名和时间戳验证。

5. 设计一个综合安全中间件，整合限流、签名验证和幂等性检查。

6. 创建一个限流配置系统，支持不同 API 端点的不同限流规则。
