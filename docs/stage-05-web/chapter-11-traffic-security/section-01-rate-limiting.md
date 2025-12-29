# 5.11.1 Rate Limiting

## 概述

Rate Limiting（速率限制）是一种流量控制机制，用于限制客户端在特定时间窗口内的请求次数。Rate Limiting 可以防止 API 滥用、保护服务器资源、防范 DDoS 攻击等。理解 Rate Limiting 的概念、掌握实现方法、选择合适的限制策略，对于构建稳定、安全的 Web 应用至关重要。本节详细介绍 Rate Limiting 的概念、实现方法、基于 IP 的限制、基于用户的限制、Redis 实现、滑动窗口算法等内容，帮助零基础学员掌握流量控制技术。

Rate Limiting 是现代 Web 应用的重要安全措施，可以有效防止恶意请求和资源滥用。理解不同的限制策略和实现方法，对于设计合理的 Rate Limiting 系统至关重要。

**主要内容**：
- Rate Limiting 概念（什么是 Rate Limiting、Rate Limiting 的作用、限制策略）
- 实现方法（内存计数、文件存储、数据库存储、Redis 存储）
- 基于 IP 的限制（IP 识别、计数管理、限制规则、实现示例）
- 基于用户的限制（用户识别、用户计数、限制规则、实现示例）
- Redis 实现（Redis 计数器、过期时间设置、原子操作、分布式限制）
- 滑动窗口算法（固定窗口、滑动窗口、令牌桶、漏桶算法）
- 实际应用示例和最佳实践

## 特性

- **灵活策略**：支持多种限制策略
- **高性能**：使用 Redis 等高性能存储
- **分布式支持**：支持分布式环境
- **可配置**：支持灵活的配置选项
- **易于集成**：易于集成到现有系统

## Rate Limiting 概念

### 什么是 Rate Limiting

Rate Limiting（速率限制）是一种流量控制机制，限制客户端在特定时间窗口内的请求次数。

### Rate Limiting 的作用

1. **防止滥用**：防止 API 被恶意滥用
2. **保护资源**：保护服务器资源不被耗尽
3. **防范攻击**：防范 DDoS 攻击和暴力破解
4. **公平使用**：确保资源的公平使用

### 限制策略

**常见策略**：
- **固定窗口**：固定时间窗口内的请求数限制
- **滑动窗口**：滑动时间窗口内的请求数限制
- **令牌桶**：基于令牌的速率限制
- **漏桶**：基于漏桶的速率限制

## 实现方法

### 内存计数

**示例**：
```php
<?php
declare(strict_types=1);

class InMemoryRateLimiter
{
    private array $counters = [];

    public function checkLimit(string $key, int $limit, int $window): bool
    {
        $now = time();
        $windowStart = $now - ($now % $window);
        $counterKey = "{$key}:{$windowStart}";

        if (!isset($this->counters[$counterKey])) {
            $this->counters[$counterKey] = 0;
        }

        $this->counters[$counterKey]++;

        // 清理过期计数器
        $this->cleanExpired($now, $window);

        return $this->counters[$counterKey] <= $limit;
    }

    private function cleanExpired(int $now, int $window): void
    {
        $currentWindow = $now - ($now % $window);
        foreach (array_keys($this->counters) as $key) {
            $windowStart = (int) explode(':', $key)[1];
            if ($windowStart < $currentWindow - $window) {
                unset($this->counters[$key]);
            }
        }
    }
}
```

### 文件存储

**示例**：
```php
<?php
declare(strict_types=1);

class FileRateLimiter
{
    private string $storageDir;

    public function __construct(string $storageDir)
    {
        $this->storageDir = $storageDir;
        if (!is_dir($storageDir)) {
            mkdir($storageDir, 0755, true);
        }
    }

    public function checkLimit(string $key, int $limit, int $window): bool
    {
        $file = $this->storageDir . '/' . md5($key) . '.json';
        $now = time();

        if (file_exists($file)) {
            $data = json_decode(file_get_contents($file), true);
            if ($data['expires'] > $now) {
                if ($data['count'] >= $limit) {
                    return false;
                }
                $data['count']++;
            } else {
                $data = ['count' => 1, 'expires' => $now + $window];
            }
        } else {
            $data = ['count' => 1, 'expires' => $now + $window];
        }

        file_put_contents($file, json_encode($data));
        return true;
    }
}
```

### 数据库存储

**示例**：
```php
<?php
declare(strict_types=1);

class DatabaseRateLimiter
{
    public function checkLimit(string $key, int $limit, int $window): bool
    {
        $now = time();
        $windowStart = $now - ($now % $window);

        // 获取当前计数
        $stmt = $this->db->prepare(
            'SELECT count FROM rate_limits 
             WHERE `key` = ? AND window_start = ?'
        );
        $stmt->execute([$key, $windowStart]);
        $row = $stmt->fetch();

        if ($row && $row['count'] >= $limit) {
            return false;
        }

        // 更新或插入计数
        $stmt = $this->db->prepare(
            'INSERT INTO rate_limits (`key`, window_start, count, expires_at) 
             VALUES (?, ?, 1, ?)
             ON DUPLICATE KEY UPDATE count = count + 1'
        );
        $stmt->execute([$key, $windowStart, $now + $window]);

        return true;
    }
}
```

### Redis 存储（推荐）

**示例**：
```php
<?php
declare(strict_types=1);

class RedisRateLimiter
{
    private Redis $redis;

    public function __construct(Redis $redis)
    {
        $this->redis = $redis;
    }

    public function checkLimit(string $key, int $limit, int $window): bool
    {
        $redisKey = "rate_limit:{$key}";
        
        // 使用 INCR 原子操作
        $current = $this->redis->incr($redisKey);
        
        // 如果是第一次，设置过期时间
        if ($current === 1) {
            $this->redis->expire($redisKey, $window);
        }
        
        return $current <= $limit;
    }

    public function getRemaining(string $key, int $limit): int
    {
        $redisKey = "rate_limit:{$key}";
        $current = (int) $this->redis->get($redisKey);
        return max(0, $limit - $current);
    }

    public function getResetTime(string $key): ?int
    {
        $redisKey = "rate_limit:{$key}";
        $ttl = $this->redis->ttl($redisKey);
        return $ttl > 0 ? time() + $ttl : null;
    }
}
```

## 基于 IP 的限制

### IP 识别

**示例**：
```php
<?php
declare(strict_types=1);

function getClientIp(): string
{
    $ipKeys = [
        'HTTP_CF_CONNECTING_IP',  // Cloudflare
        'HTTP_X_FORWARDED_FOR',
        'HTTP_X_REAL_IP',
        'REMOTE_ADDR',
    ];

    foreach ($ipKeys as $key) {
        if (!empty($_SERVER[$key])) {
            $ip = $_SERVER[$key];
            // 处理多个 IP（取第一个）
            if (str_contains($ip, ',')) {
                $ip = trim(explode(',', $ip)[0]);
            }
            if (filter_var($ip, FILTER_VALIDATE_IP)) {
                return $ip;
            }
        }
    }

    return '0.0.0.0';
}
```

### 计数管理

**示例**：
```php
<?php
declare(strict_types=1);

class IpRateLimiter
{
    private RedisRateLimiter $limiter;

    public function __construct(RedisRateLimiter $limiter)
    {
        $this->limiter = $limiter;
    }

    public function checkIpLimit(int $limit, int $window): bool
    {
        $ip = getClientIp();
        $key = "ip:{$ip}";
        
        return $this->limiter->checkLimit($key, $limit, $window);
    }

    public function getIpLimitInfo(): array
    {
        $ip = getClientIp();
        $key = "ip:{$ip}";
        $limit = 100;  // 配置的限制
        
        return [
            'ip' => $ip,
            'remaining' => $this->limiter->getRemaining($key, $limit),
            'reset_time' => $this->limiter->getResetTime($key),
        ];
    }
}
```

### 限制规则

**示例**：
```php
<?php
declare(strict_types=1);

class RateLimitConfig
{
    // 不同端点的不同限制
    private const LIMITS = [
        '/api/login' => ['limit' => 5, 'window' => 300],      // 5 次/5 分钟
        '/api/register' => ['limit' => 3, 'window' => 3600],   // 3 次/小时
        '/api/users' => ['limit' => 100, 'window' => 60],     // 100 次/分钟
    ];

    public static function getLimit(string $endpoint): array
    {
        return self::LIMITS[$endpoint] ?? ['limit' => 60, 'window' => 60];
    }
}
```

## 基于用户的限制

### 用户识别

**示例**：
```php
<?php
declare(strict_types=1);

class UserRateLimiter
{
    private RedisRateLimiter $limiter;

    public function __construct(RedisRateLimiter $limiter)
    {
        $this->limiter = $limiter;
    }

    public function checkUserLimit(int $userId, int $limit, int $window): bool
    {
        $key = "user:{$userId}";
        return $this->limiter->checkLimit($key, $limit, $window);
    }

    public function checkAuthenticatedUserLimit(int $limit, int $window): bool
    {
        session_start();
        $userId = $_SESSION['user_id'] ?? null;
        
        if ($userId === null) {
            // 未认证用户使用 IP 限制
            return $this->checkIpLimit($limit, $window);
        }
        
        // 认证用户使用用户 ID 限制
        return $this->checkUserLimit($userId, $limit, $window);
    }
}
```

## Redis 实现

### Redis 计数器

**示例**：
```php
<?php
declare(strict_types=1);

class RedisRateLimiter
{
    private Redis $redis;

    public function __construct(Redis $redis)
    {
        $this->redis = $redis;
    }

    public function checkLimit(string $key, int $limit, int $window): bool
    {
        $redisKey = "rate_limit:{$key}";
        
        // 使用管道提高性能
        $pipe = $this->redis->pipeline();
        $pipe->incr($redisKey);
        $pipe->expire($redisKey, $window);
        $results = $pipe->execute();
        
        $current = $results[0];
        return $current <= $limit;
    }
}
```

### 过期时间设置

**示例**：
```php
<?php
declare(strict_types=1);

public function checkLimit(string $key, int $limit, int $window): bool
{
    $redisKey = "rate_limit:{$key}";
    
    // 使用 Lua 脚本保证原子性
    $lua = "
        local key = KEYS[1]
        local limit = tonumber(ARGV[1])
        local window = tonumber(ARGV[2])
        
        local current = redis.call('INCR', key)
        if current == 1 then
            redis.call('EXPIRE', key, window)
        end
        
        return current <= limit
    ";
    
    $result = $this->redis->eval($lua, [$redisKey, $limit, $window], 1);
    return (bool) $result;
}
```

### 原子操作

**示例**：
```php
<?php
declare(strict_types=1);

// 使用 Lua 脚本保证原子性
$lua = "
    local key = KEYS[1]
    local limit = tonumber(ARGV[1])
    local window = tonumber(ARGV[2])
    
    local current = redis.call('INCR', key)
    if current == 1 then
        redis.call('EXPIRE', key, window)
    end
    
    if current > limit then
        return {0, current, redis.call('TTL', key)}
    else
        return {1, current, redis.call('TTL', key)}
    end
";

$result = $redis->eval($lua, [$key, $limit, $window], 1);
[$allowed, $current, $ttl] = $result;
```

### 分布式限制

**示例**：
```php
<?php
declare(strict_types=1);

// Redis 天然支持分布式
// 多个服务器共享同一个 Redis 实例
// 所有服务器使用相同的限制计数
```

## 滑动窗口算法

### 固定窗口

**示例**：
```php
<?php
declare(strict_types=1);

class FixedWindowRateLimiter
{
    public function checkLimit(string $key, int $limit, int $window): bool
    {
        $now = time();
        $windowStart = $now - ($now % $window);  // 固定窗口开始时间
        $redisKey = "rate_limit:{$key}:{$windowStart}";
        
        $current = $this->redis->incr($redisKey);
        if ($current === 1) {
            $this->redis->expire($redisKey, $window);
        }
        
        return $current <= $limit;
    }
}
```

### 滑动窗口

**示例**：
```php
<?php
declare(strict_types=1);

class SlidingWindowRateLimiter
{
    public function checkLimit(string $key, int $limit, int $window): bool
    {
        $now = time();
        $redisKey = "rate_limit:{$key}";
        
        // 使用有序集合存储时间戳
        $this->redis->zremrangebyscore($redisKey, 0, $now - $window);
        
        $count = $this->redis->zcard($redisKey);
        
        if ($count >= $limit) {
            return false;
        }
        
        // 添加当前时间戳
        $this->redis->zadd($redisKey, $now, (string) $now);
        $this->redis->expire($redisKey, $window);
        
        return true;
    }
}
```

### 令牌桶算法

**示例**：
```php
<?php
declare(strict_types=1);

class TokenBucketRateLimiter
{
    public function checkLimit(string $key, int $capacity, int $refillRate): bool
    {
        $redisKey = "token_bucket:{$key}";
        $now = time();
        
        // 获取当前令牌数
        $data = $this->redis->hgetall($redisKey);
        $tokens = (int) ($data['tokens'] ?? $capacity);
        $lastRefill = (int) ($data['last_refill'] ?? $now);
        
        // 计算需要补充的令牌数
        $elapsed = $now - $lastRefill;
        $tokens = min($capacity, $tokens + ($elapsed * $refillRate));
        
        if ($tokens < 1) {
            return false;  // 没有令牌
        }
        
        // 消耗一个令牌
        $tokens--;
        $this->redis->hmset($redisKey, [
            'tokens' => $tokens,
            'last_refill' => $now,
        ]);
        $this->redis->expire($redisKey, 3600);
        
        return true;
    }
}
```

## 使用场景

### API 保护

- 防止 API 滥用
- 保护 API 资源
- 控制 API 使用频率

### 防止暴力破解

- 登录尝试限制
- 密码重置限制
- 验证码请求限制

### 防止 DDoS 攻击

- 限制请求频率
- 识别异常流量
- 自动封禁

### 资源保护

- 数据库查询限制
- 文件上传限制
- 计算资源限制

## 注意事项

### 限制策略设计

- **合理限制**：设置合理的限制值
- **不同端点**：不同端点使用不同限制
- **用户区分**：认证用户和匿名用户不同限制

### 误杀处理

- **白名单**：提供白名单机制
- **错误信息**：提供清晰的错误信息
- **重试建议**：提供重试时间建议

### 分布式环境

- **共享存储**：使用 Redis 等共享存储
- **一致性**：保证限制的一致性
- **性能考虑**：考虑网络延迟

### 性能影响

- **存储选择**：选择合适的存储方案
- **缓存策略**：使用缓存提高性能
- **原子操作**：使用原子操作保证准确性

## 常见问题

### 如何实现 Rate Limiting？

主要方法：
1. **内存计数**：简单但不支持分布式
2. **Redis**：推荐，支持分布式
3. **数据库**：支持但性能较低

### 如何选择限制策略？

- **固定窗口**：简单，但可能在窗口边界有突发
- **滑动窗口**：更平滑，但实现复杂
- **令牌桶**：适合突发流量

### 如何处理分布式环境？

使用 Redis 等共享存储：
- 所有服务器共享同一个 Redis
- 使用原子操作保证准确性
- 考虑网络延迟

### Rate Limiting 的性能影响？

- **Redis**：性能影响小
- **数据库**：性能影响较大
- **内存**：性能最好但不支持分布式

## 最佳实践

### 使用 Redis 实现

- 使用 Redis 作为存储后端
- 使用 Lua 脚本保证原子性
- 设置合理的过期时间

### 设置合理的限制

- 根据实际需求设置限制
- 不同端点使用不同限制
- 提供配置接口

### 提供清晰的错误信息

- 返回剩余请求数
- 返回重置时间
- 提供重试建议

### 考虑白名单机制

- 提供白名单功能
- 管理员 IP 白名单
- 内部服务白名单

## 相关章节

- **[5.11.2 请求签名与验证](section-02-request-signing.md)**：了解请求签名的详细内容
- **[5.11.3 安全最佳实践](section-03-security-best-practices.md)**：了解安全最佳实践的详细内容

## 练习任务

1. **实现 Rate Limiting 类**
   - 使用 Redis 实现
   - 支持固定窗口和滑动窗口
   - 提供限制信息查询

2. **实现基于 IP 的限制**
   - IP 识别和限制
   - 不同端点不同限制
   - 白名单机制

3. **实现基于用户的限制**
   - 用户识别和限制
   - 认证用户和匿名用户区分
   - 用户限制信息

4. **实现滑动窗口算法**
   - 滑动窗口实现
   - 令牌桶算法
   - 性能优化

5. **实现完整的 Rate Limiting 系统**
   - 多种限制策略
   - 配置管理
   - 监控和日志
