# 4.9.3 JWT 与 Token

## 概述

JWT（JSON Web Token）是现代 Web 应用中广泛使用的无状态认证方案。本节详细介绍 JWT 结构、JWT 实现、Token 刷新、Token 撤销，以及安全最佳实践。

## JWT 结构

### JWT 组成

```
Header.Payload.Signature

Header: {"alg": "HS256", "typ": "JWT"}
Payload: {"user_id": 1, "exp": 1234567890}
Signature: HMACSHA256(base64UrlEncode(header) + "." + base64UrlEncode(payload), secret)
```

### JWT 实现

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
            'exp' => time() + 3600,  // 1 小时过期
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

## Token 刷新

### 刷新 Token 机制

```php
<?php
declare(strict_types=1);

class TokenManager
{
    public function generateTokenPair(int $userId): array
    {
        $accessToken = $this->generateAccessToken($userId);
        $refreshToken = $this->generateRefreshToken($userId);
        
        return [
            'access_token' => $accessToken,
            'refresh_token' => $refreshToken,
            'expires_in' => 3600,
        ];
    }

    private function generateAccessToken(int $userId): string
    {
        $payload = [
            'user_id' => $userId,
            'type' => 'access',
            'exp' => time() + 3600,  // 1 小时
        ];
        return JWT::encode($payload, $this->secret, 'HS256');
    }

    private function generateRefreshToken(int $userId): string
    {
        $payload = [
            'user_id' => $userId,
            'type' => 'refresh',
            'exp' => time() + 86400 * 7,  // 7 天
        ];
        return JWT::encode($payload, $this->secret, 'HS256');
    }

    public function refreshAccessToken(string $refreshToken): ?string
    {
        $payload = $this->validateToken($refreshToken);
        
        if ($payload === null || ($payload['type'] ?? '') !== 'refresh') {
            return null;
        }
        
        return $this->generateAccessToken($payload['user_id']);
    }
}
```

## Token 撤销

### Token 黑名单

```php
<?php
declare(strict_types=1);

class TokenBlacklist
{
    private \Redis $redis;

    public function __construct(\Redis $redis)
    {
        $this->redis = $redis;
    }

    public function revokeToken(string $token, int $ttl = 3600): void
    {
        $jti = $this->getTokenId($token);
        if ($jti !== null) {
            $this->redis->setex("blacklist:{$jti}", $ttl, '1');
        }
    }

    public function isRevoked(string $token): bool
    {
        $jti = $this->getTokenId($token);
        if ($jti === null) {
            return false;
        }
        
        return $this->redis->exists("blacklist:{$jti}") > 0;
    }

    private function getTokenId(string $token): ?string
    {
        $payload = $this->decodeToken($token);
        return $payload['jti'] ?? null;
    }
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

class JWTService
{
    public function authenticate(string $token): ?int
    {
        $payload = $this->validateToken($token);
        
        if ($payload === null) {
            return null;
        }
        
        // 检查 Token 是否被撤销
        if ($this->blacklist->isRevoked($token)) {
            return null;
        }
        
        return $payload['user_id'] ?? null;
    }

    public function logout(string $token): void
    {
        $this->blacklist->revokeToken($token);
    }
}
```

## 注意事项

1. **密钥安全**：使用强密钥，妥善保管
2. **过期时间**：合理设置 Token 过期时间
3. **Token 撤销**：实现 Token 撤销机制
4. **HTTPS**：生产环境必须使用 HTTPS

## 练习

1. 实现一个完整的 JWT 认证系统，包含生成和验证 Token。

2. 实现 Token 刷新机制，支持 Access Token 和 Refresh Token。

3. 创建 Token 黑名单系统，支持 Token 撤销。

4. 实现 JWT 中间件，自动验证请求中的 Token。
