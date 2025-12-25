# 4.10.1 Rate Limiting

## 概述

Rate Limiting（限流）是保护 API 和服务的重要手段。本节详细介绍限流的作用、固定窗口限流、滑动窗口限流、令牌桶算法、漏桶算法，以及完整实现。

## 为什么需要限流

- **防止滥用**：限制单个用户或 IP 的请求频率
- **保护服务**：防止 DDoS 攻击和资源耗尽
- **公平使用**：确保所有用户公平使用资源
- **成本控制**：控制 API 调用成本

## 固定窗口限流

### 实现

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
            $this->redis->expire($windowKey, $this->windowSeconds);
        }
        
        return $current <= $this->maxRequests;
    }
}
```

## 滑动窗口限流

### 实现

```php
<?php
declare(strict_types=1);

class SlidingWindowRateLimiter
{
    private \Redis $redis;
    private int $maxRequests;
    private int $windowSeconds;

    public function isAllowed(string $key): bool
    {
        $now = time();
        $windowStart = $now - $this->windowSeconds;
        
        // 使用有序集合存储请求时间戳
        $zsetKey = "rate_limit:sliding:{$key}";
        
        // 移除过期记录
        $this->redis->zRemRangeByScore($zsetKey, 0, $windowStart);
        
        // 获取当前窗口内的请求数
        $count = $this->redis->zCard($zsetKey);
        
        if ($count >= $this->maxRequests) {
            return false;
        }
        
        // 添加当前请求
        $this->redis->zAdd($zsetKey, $now, (string) $now);
        $this->redis->expire($zsetKey, $this->windowSeconds);
        
        return true;
    }
}
```

## 令牌桶算法

### 实现

```php
<?php
declare(strict_types=1);

class TokenBucketRateLimiter
{
    private \Redis $redis;
    private int $capacity;
    private int $refillRate;  // 每秒补充的令牌数

    public function isAllowed(string $key): bool
    {
        $bucketKey = "token_bucket:{$key}";
        
        // 获取当前令牌数和最后更新时间
        $data = $this->redis->hMGet($bucketKey, ['tokens', 'last_refill']);
        
        $tokens = (float) ($data['tokens'] ?? $this->capacity);
        $lastRefill = (int) ($data['last_refill'] ?? time());
        
        // 计算需要补充的令牌
        $now = time();
        $elapsed = $now - $lastRefill;
        $tokens = min($this->capacity, $tokens + $elapsed * $this->refillRate);
        
        if ($tokens < 1) {
            return false;
        }
        
        // 消耗一个令牌
        $tokens -= 1;
        
        // 更新
        $this->redis->hMSet($bucketKey, [
            'tokens' => $tokens,
            'last_refill' => $now,
        ]);
        $this->redis->expire($bucketKey, 3600);
        
        return true;
    }
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

class RateLimiterMiddleware
{
    private RateLimiter $limiter;

    public function handle(string $key): void
    {
        if (!$this->limiter->isAllowed($key)) {
            http_response_code(429);
            header('X-RateLimit-Limit: ' . $this->limiter->getLimit());
            header('X-RateLimit-Remaining: 0');
            echo json_encode(['error' => 'Too many requests']);
            exit;
        }
        
        header('X-RateLimit-Remaining: ' . $this->limiter->getRemaining($key));
    }
}
```

## 注意事项

1. **算法选择**：根据需求选择合适的限流算法
2. **存储选择**：使用 Redis 等外部存储支持分布式
3. **错误处理**：限流时返回 429 状态码
4. **响应头**：返回限流信息给客户端

## 练习

1. 实现固定窗口和滑动窗口限流算法。

2. 创建一个限流中间件，自动限制请求频率。

3. 实现令牌桶算法，支持动态调整速率。

4. 设计一个多级限流系统，支持不同用户的不同限制。
