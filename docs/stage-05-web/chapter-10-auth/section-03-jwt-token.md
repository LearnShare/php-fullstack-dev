# 5.10.3 JWT 与 Token

## 概述

JWT（JSON Web Token）是现代 Web 应用中广泛使用的无状态认证方案。理解 JWT 的结构、掌握 JWT 的生成和验证方法、实现 Token 刷新机制，对于构建可扩展的 API 服务和微服务架构至关重要。本节详细介绍 JWT 的概念、JWT 结构（Header、Payload、Signature）、JWT 生成和验证、Token 刷新、安全考虑等内容，帮助零基础学员掌握 JWT 认证技术。

JWT 是一种开放标准（RFC 7519），用于在各方之间安全地传输信息。JWT 是自包含的，包含所有必要的信息，不需要在服务器端存储会话状态，非常适合无状态架构。

**主要内容**：
- JWT 概念（什么是 JWT、JWT 的优势、JWT 的应用场景）
- JWT 结构（Header 头部、Payload 载荷、Signature 签名、Base64 编码）
- JWT 生成（库的选择 firebase/php-jwt、Payload 设计、签名算法、过期时间设置）
- JWT 验证（签名验证、过期时间检查、载荷解析、错误处理）
- Token 刷新（刷新 Token 机制、Token 对生成、刷新流程）
- 安全考虑（密钥安全、Token 过期、Token 刷新、签名验证）
- 实际应用示例和最佳实践

## 特性

- **无状态**：不需要服务器端存储
- **自包含**：包含所有必要信息
- **可验证**：通过签名验证完整性
- **可扩展**：易于扩展和集成
- **标准协议**：遵循 JWT 标准（RFC 7519）

## JWT 概念

### 什么是 JWT

JWT（JSON Web Token）是一种紧凑且自包含的方式，用于在各方之间安全地传输信息。JWT 由三部分组成：Header、Payload、Signature。

### JWT 的优势

1. **无状态**：不需要服务器端存储会话
2. **可扩展**：易于水平扩展
3. **跨域支持**：可以跨域使用
4. **自包含**：包含所有必要信息
5. **标准化**：遵循标准协议

### JWT 的应用场景

1. **API 认证**：RESTful API 认证
2. **单点登录**：SSO 系统
3. **微服务认证**：服务间认证
4. **移动应用**：移动应用后端认证

## JWT 结构

### JWT 组成

JWT 由三部分组成，用 `.` 分隔：

```
Header.Payload.Signature
```

**示例**：
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX2lkIjoxMjMsImV4cCI6MTcwMzEyMzQ1Nn0.signature
```

### Header（头部）

**内容**：算法和类型信息。

**示例**：
```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

**编码**：Base64URL 编码。

### Payload（载荷）

**内容**：用户信息和声明（Claims）。

**标准声明**：
- `iss`（Issuer）：签发者
- `sub`（Subject）：主题
- `aud`（Audience）：受众
- `exp`（Expiration Time）：过期时间
- `iat`（Issued At）：签发时间
- `nbf`（Not Before）：生效时间

**示例**：
```json
{
  "user_id": 123,
  "username": "john_doe",
  "iat": 1703123456,
  "exp": 1703127056
}
```

**编码**：Base64URL 编码。

### Signature（签名）

**作用**：验证 Token 的完整性和真实性。

**生成方式**：
```
HMACSHA256(
  base64UrlEncode(header) + "." + base64UrlEncode(payload),
  secret
)
```

**验证方式**：使用相同的密钥和算法重新计算签名，与 Token 中的签名比较。

## JWT 生成

### 使用 firebase/php-jwt 库

**安装**：
```bash
composer require firebase/php-jwt
```

**基本用法**：
```php
<?php
declare(strict_types=1);

require __DIR__ . '/vendor/autoload.php';

use Firebase\JWT\JWT;
use Firebase\JWT\Key;

$secretKey = 'your-secret-key';
$payload = [
    'user_id' => 123,
    'username' => 'john_doe',
    'iat' => time(),
    'exp' => time() + 3600,  // 1 小时后过期
];

$token = JWT::encode($payload, $secretKey, 'HS256');
echo $token;
```

### Payload 设计

**示例**：
```php
<?php
declare(strict_types=1);

function generateToken(int $userId, array $additionalClaims = []): string
{
    $payload = array_merge([
        'user_id' => $userId,
        'iat' => time(),
        'exp' => time() + 3600,  // 1 小时
        'iss' => 'your-app-name',  // 签发者
    ], $additionalClaims);

    return JWT::encode($payload, $secretKey, 'HS256');
}
```

### 签名算法

**常用算法**：
- **HS256**：HMAC SHA256（对称加密，推荐）
- **RS256**：RSA SHA256（非对称加密）
- **ES256**：ECDSA SHA256（椭圆曲线）

**示例**：
```php
<?php
declare(strict_types=1);

// HS256（对称加密）
$token = JWT::encode($payload, $secretKey, 'HS256');

// RS256（非对称加密）
$privateKey = file_get_contents('private.pem');
$token = JWT::encode($payload, $privateKey, 'RS256');
```

### 过期时间设置

**示例**：
```php
<?php
declare(strict_types=1);

// 短期 Token（1 小时）
$accessToken = [
    'user_id' => 123,
    'exp' => time() + 3600,
];

// 长期 Token（7 天）
$refreshToken = [
    'user_id' => 123,
    'type' => 'refresh',
    'exp' => time() + 86400 * 7,
];
```

## JWT 验证

### 签名验证

**示例**：
```php
<?php
declare(strict_types=1);

use Firebase\JWT\JWT;
use Firebase\JWT\Key;
use Firebase\JWT\ExpiredException;
use Firebase\JWT\SignatureInvalidException;

function validateToken(string $token): ?array
{
    try {
        $decoded = JWT::decode($token, new Key($secretKey, 'HS256'));
        return (array) $decoded;
    } catch (ExpiredException $e) {
        // Token 已过期
        return null;
    } catch (SignatureInvalidException $e) {
        // 签名验证失败
        return null;
    } catch (Exception $e) {
        // 其他错误
        return null;
    }
}
```

### 过期时间检查

**自动检查**：`JWT::decode()` 会自动检查 `exp` 声明。

**手动检查**：
```php
<?php
declare(strict_types=1);

function validateToken(string $token): ?array
{
    try {
        $decoded = JWT::decode($token, new Key($secretKey, 'HS256'));
        $payload = (array) $decoded;
        
        // 手动检查过期时间
        if (isset($payload['exp']) && $payload['exp'] < time()) {
            return null;  // 已过期
        }
        
        return $payload;
    } catch (Exception $e) {
        return null;
    }
}
```

### 载荷解析

**示例**：
```php
<?php
declare(strict_types=1);

function getUserIdFromToken(string $token): ?int
{
    $payload = validateToken($token);
    if ($payload === null) {
        return null;
    }
    
    return $payload['user_id'] ?? null;
}
```

### 错误处理

**示例**：
```php
<?php
declare(strict_types=1);

class JWTValidator
{
    public function validate(string $token): array
    {
        try {
            $decoded = JWT::decode($token, new Key($this->secretKey, 'HS256'));
            return [
                'valid' => true,
                'payload' => (array) $decoded,
            ];
        } catch (ExpiredException $e) {
            return [
                'valid' => false,
                'error' => 'Token expired',
                'code' => 'EXPIRED',
            ];
        } catch (SignatureInvalidException $e) {
            return [
                'valid' => false,
                'error' => 'Invalid signature',
                'code' => 'INVALID_SIGNATURE',
            ];
        } catch (Exception $e) {
            return [
                'valid' => false,
                'error' => $e->getMessage(),
                'code' => 'UNKNOWN',
            ];
        }
    }
}
```

## Token 刷新

### 刷新 Token 机制

**策略**：使用两个 Token
- **Access Token**：短期 Token（1 小时），用于 API 访问
- **Refresh Token**：长期 Token（7 天），用于刷新 Access Token

**示例**：
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
            'expires_in' => 3600,  // Access Token 过期时间（秒）
        ];
    }

    private function generateAccessToken(int $userId): string
    {
        $payload = [
            'user_id' => $userId,
            'type' => 'access',
            'iat' => time(),
            'exp' => time() + 3600,  // 1 小时
        ];
        
        return JWT::encode($payload, $this->secretKey, 'HS256');
    }

    private function generateRefreshToken(int $userId): string
    {
        $payload = [
            'user_id' => $userId,
            'type' => 'refresh',
            'iat' => time(),
            'exp' => time() + 86400 * 7,  // 7 天
        ];
        
        return JWT::encode($payload, $this->refreshSecretKey, 'HS256');
    }

    public function refreshAccessToken(string $refreshToken): ?string
    {
        try {
            $decoded = JWT::decode($refreshToken, new Key($this->refreshSecretKey, 'HS256'));
            $payload = (array) $decoded;
            
            // 验证 Token 类型
            if (($payload['type'] ?? '') !== 'refresh') {
                return null;
            }
            
            // 生成新的 Access Token
            return $this->generateAccessToken($payload['user_id']);
        } catch (Exception $e) {
            return null;
        }
    }
}
```

### Token 对生成

**示例**：
```php
<?php
declare(strict_types=1);

// 登录时生成 Token 对
function login(string $username, string $password): array
{
    // 验证用户
    $user = validateUser($username, $password);
    if ($user === null) {
        return ['error' => 'Invalid credentials'];
    }

    // 生成 Token 对
    $tokenManager = new TokenManager();
    $tokens = $tokenManager->generateTokenPair($user['id']);
    
    return [
        'access_token' => $tokens['access_token'],
        'refresh_token' => $tokens['refresh_token'],
        'expires_in' => $tokens['expires_in'],
    ];
}
```

### 刷新流程

**示例**：
```php
<?php
declare(strict_types=1);

// 刷新 Token 端点
function refreshToken(): void
{
    $refreshToken = $_POST['refresh_token'] ?? '';
    
    if (empty($refreshToken)) {
        http_response_code(400);
        echo json_encode(['error' => 'Refresh token required']);
        exit;
    }

    $tokenManager = new TokenManager();
    $newAccessToken = $tokenManager->refreshAccessToken($refreshToken);
    
    if ($newAccessToken === null) {
        http_response_code(401);
        echo json_encode(['error' => 'Invalid refresh token']);
        exit;
    }

    http_response_code(200);
    echo json_encode([
        'access_token' => $newAccessToken,
        'expires_in' => 3600,
    ]);
}
```

## 使用场景

### API 认证

- RESTful API 认证
- 无状态认证
- 跨域认证

### 无状态认证

- 微服务架构
- 分布式系统
- 水平扩展

### 微服务认证

- 服务间认证
- API 网关认证
- 服务发现认证

### 单点登录

- SSO 系统
- 多应用认证
- 统一认证

## 注意事项

### 密钥安全

- **强密钥**：使用强随机密钥
- **密钥管理**：安全存储密钥
- **密钥轮换**：定期轮换密钥

### Token 过期

- **合理过期时间**：设置合理的过期时间
- **过期检查**：及时检查过期时间
- **刷新机制**：实现 Token 刷新

### Token 刷新

- **刷新 Token 安全**：使用不同的密钥
- **刷新频率限制**：限制刷新频率
- **刷新 Token 撤销**：支持撤销刷新 Token

### 签名验证

- **始终验证**：始终验证签名
- **算法验证**：验证签名算法
- **密钥验证**：使用正确的密钥

## 常见问题

### JWT 的结构是什么？

JWT 由三部分组成：
- **Header**：算法和类型
- **Payload**：用户信息和声明
- **Signature**：签名

### 如何生成 JWT？

使用 `firebase/php-jwt` 库：

```php
<?php
declare(strict_types=1);

use Firebase\JWT\JWT;

$payload = ['user_id' => 123, 'exp' => time() + 3600];
$token = JWT::encode($payload, $secretKey, 'HS256');
```

### 如何验证 JWT？

使用 `JWT::decode()` 函数：

```php
<?php
declare(strict_types=1);

use Firebase\JWT\JWT;
use Firebase\JWT\Key;

$decoded = JWT::decode($token, new Key($secretKey, 'HS256'));
```

### JWT 的安全问题？

1. **密钥泄露**：保护密钥安全
2. **Token 泄露**：使用 HTTPS 传输
3. **过期时间**：设置合理的过期时间
4. **签名验证**：始终验证签名

## 最佳实践

### 使用强密钥

- 使用随机生成的强密钥
- 密钥长度至少 32 字节
- 安全存储密钥

### 设置合理的过期时间

- Access Token：短期（1 小时）
- Refresh Token：长期（7 天）
- 根据需求调整

### 实现 Token 刷新机制

- 使用 Token 对（Access + Refresh）
- 实现刷新端点
- 限制刷新频率

### 验证签名和过期时间

- 始终验证签名
- 检查过期时间
- 处理验证错误

## 相关章节

- **[5.10.1 认证基础（AuthN）](section-01-authentication.md)**：了解认证的详细内容
- **[5.10.4 OAuth2 实现](section-04-oauth2.md)**：了解 OAuth2 的详细内容
- **[5.9.3 状态管理最佳实践](../chapter-09-session/section-03-state-management.md)**：了解无状态设计的详细内容

## 练习任务

1. **实现 JWT 生成和验证**
   - 安装 firebase/php-jwt
   - 实现 Token 生成
   - 实现 Token 验证

2. **实现 Token 刷新机制**
   - 生成 Token 对
   - 实现刷新端点
   - 管理 Token 生命周期

3. **实现 JWT 中间件**
   - 创建 JWT 认证中间件
   - 验证 Token
   - 提取用户信息

4. **实现 Token 管理**
   - Token 存储
   - Token 撤销
   - Token 黑名单

5. **实现完整的 JWT 认证系统**
   - 登录生成 Token
   - Token 验证中间件
   - Token 刷新机制
