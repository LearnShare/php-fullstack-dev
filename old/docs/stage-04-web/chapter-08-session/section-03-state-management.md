# 4.8.3 状态管理最佳实践

## 概述

不同场景需要不同的状态管理方案。本节详细介绍 Token/JWT 使用、状态管理方案对比、分布式 Session、安全最佳实践，以及方案选择指南。

## 状态管理方案对比

### 方案对比表

| 方案 | 存储位置 | 特点 | 适用场景 |
| :--- | :--- | :--- | :--- |
| Cookie | 客户端 | 简单，容量小，安全性较低 | 简单状态、偏好设置 |
| Session | 服务器 | 安全，但需要存储空间 | 用户登录、购物车 |
| Token/JWT | 客户端 | 无状态，可扩展 | API 认证、微服务 |
| Redis Store | 服务器（Redis） | 高性能，可共享 | 分布式系统 |

## Token/JWT 使用

### JWT 基础

```php
<?php
declare(strict_types=1);

require __DIR__ . '/vendor/autoload.php';

use Firebase\JWT\JWT;
use Firebase\JWT\Key;

class JWTAuth
{
    private string $secret;
    private string $algorithm = 'HS256';

    public function __construct(string $secret)
    {
        $this->secret = $secret;
    }

    public function generateToken(int $userId, array $claims = []): string
    {
        $payload = array_merge([
            'user_id' => $userId,
            'iat' => time(),
            'exp' => time() + 3600,
        ], $claims);
        
        return JWT::encode($payload, $this->secret, $this->algorithm);
    }

    public function validateToken(string $token): ?array
    {
        try {
            $decoded = JWT::decode($token, new Key($this->secret, $this->algorithm));
            return (array) $decoded;
        } catch (Exception $e) {
            return null;
        }
    }
}
```

### Token 使用场景

```php
<?php
// 1. 登录后生成 Token
$token = $jwtAuth->generateToken($userId);

// 2. 客户端存储 Token（localStorage 或 Cookie）
// 前端：localStorage.setItem('token', token);

// 3. 请求时携带 Token
// 前端：Authorization: Bearer {token}

// 4. 服务器验证 Token
$authHeader = $_SERVER['HTTP_AUTHORIZATION'] ?? '';
if (preg_match('/Bearer\s+(.*)$/i', $authHeader, $matches)) {
    $token = $matches[1];
    $payload = $jwtAuth->validateToken($token);
    if ($payload !== null) {
        $userId = $payload['user_id'];
    }
}
```

## 分布式 Session

### Redis Session 集群

```php
<?php
declare(strict_types=1);

// 使用 Redis 集群存储 Session
ini_set('session.save_handler', 'redis');
ini_set('session.save_path', 'tcp://redis1:6379?weight=1,tcp://redis2:6379?weight=1');

session_start();
```

### Session 共享

```php
<?php
declare(strict_types=1);

class SharedSession
{
    private \Redis $redis;
    private string $prefix = 'session:';

    public function __construct(\Redis $redis)
    {
        $this->redis = $redis;
    }

    public function get(string $sessionId, string $key): mixed
    {
        $data = $this->redis->hGet($this->prefix . $sessionId, $key);
        return $data !== false ? unserialize($data) : null;
    }

    public function set(string $sessionId, string $key, mixed $value): void
    {
        $this->redis->hSet($this->prefix . $sessionId, $key, serialize($value));
        $this->redis->expire($this->prefix . $sessionId, 3600);
    }
}
```

## 方案选择指南

### 选择 Session 的场景

- 需要服务器端控制
- 需要存储敏感信息
- 单服务器或共享存储

### 选择 Token/JWT 的场景

- 无状态 API
- 微服务架构
- 跨域认证

### 选择 Cookie 的场景

- 简单偏好设置
- 非敏感信息
- 客户端状态

## 完整示例

```php
<?php
declare(strict_types=1);

class StateManager
{
    public function getState(string $type, string $key): mixed
    {
        return match ($type) {
            'session' => $_SESSION[$key] ?? null,
            'cookie' => $_COOKIE[$key] ?? null,
            'token' => $this->getFromToken($key),
            default => null,
        };
    }

    private function getFromToken(string $key): mixed
    {
        $token = $this->extractToken();
        if ($token === null) {
            return null;
        }
        
        $payload = $this->jwtAuth->validateToken($token);
        return $payload[$key] ?? null;
    }
}
```

## 注意事项

1. **安全性**：根据数据敏感性选择方案
2. **性能**：考虑存储和验证的性能开销
3. **扩展性**：考虑分布式部署的需求
4. **兼容性**：考虑客户端支持情况

## 练习

1. 实现一个 JWT 认证系统，包含生成和验证 Token。

2. 创建一个状态管理抽象类，支持多种存储方式。

3. 实现分布式 Session 存储，使用 Redis 集群。

4. 设计一个状态管理方案选择工具，根据需求推荐合适的方案。
