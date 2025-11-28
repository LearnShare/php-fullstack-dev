# 4.9 鉴权与授权模型（AuthN & AuthZ）

## 目标

- 理解鉴权（Authentication）与授权（Authorization）的区别。
- 掌握基础 Token 鉴权、JWT 鉴权的实现。
- 了解 OAuth2 流程与第三方登录。
- 熟悉 RBAC/ABAC 权限模型的设计与实现。

## 鉴权 vs 授权

### 概念区别

- **鉴权（Authentication）**：验证用户身份（"你是谁"）。
- **授权（Authorization）**：验证用户权限（"你能做什么"）。

```
用户登录 → 鉴权（验证身份） → 获取 Token
用户请求 → 授权（检查权限） → 允许/拒绝访问
```

## 基础 Token 鉴权

### 实现示例

```php
<?php
declare(strict_types=1);

class TokenAuth
{
    private string $secret;
    private \PDO $db;

    public function __construct(string $secret, \PDO $db)
    {
        $this->secret = $secret;
        $this->db = $db;
    }

    public function login(string $username, string $password): ?string
    {
        // 验证用户
        $stmt = $this->db->prepare('SELECT id, password_hash FROM users WHERE username = ?');
        $stmt->execute([$username]);
        $user = $stmt->fetch(\PDO::FETCH_ASSOC);
        
        if ($user === false) {
            return null;
        }
        
        if (!password_verify($password, $user['password_hash'])) {
            return null;
        }
        
        // 生成 Token
        $token = $this->generateToken($user['id']);
        
        // 保存 Token 到数据库
        $this->saveToken($user['id'], $token);
        
        return $token;
    }

    private function generateToken(int $userId): string
    {
        $payload = [
            'user_id' => $userId,
            'exp' => time() + 3600,
            'iat' => time(),
        ];
        
        return base64_encode(json_encode($payload));
    }

    private function saveToken(int $userId, string $token): void
    {
        $stmt = $this->db->prepare(
            'INSERT INTO user_tokens (user_id, token, expires_at) VALUES (?, ?, ?)'
        );
        $stmt->execute([$userId, $token, date('Y-m-d H:i:s', time() + 3600)]);
    }

    public function validateToken(string $token): ?int
    {
        // 从数据库验证 Token
        $stmt = $this->db->prepare(
            'SELECT user_id FROM user_tokens WHERE token = ? AND expires_at > NOW()'
        );
        $stmt->execute([$token]);
        $result = $stmt->fetch(\PDO::FETCH_ASSOC);
        
        if ($result === false) {
            return null;
        }
        
        return (int) $result['user_id'];
    }
}

// 使用
$auth = new TokenAuth('secret-key', $db);
$token = $auth->login('alice', 'password123');

// 验证 Token
$userId = $auth->validateToken($token);
```

## JWT 鉴权

### JWT 结构

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

    public function refreshToken(string $token): ?string
    {
        $payload = $this->validateToken($token);
        
        if ($payload === null) {
            return null;
        }
        
        // 生成新 Token
        unset($payload['iat'], $payload['exp']);
        return $this->generateToken($payload['user_id'], $payload);
    }
}

// 使用
$jwtAuth = new JWTAuth('your-secret-key');
$token = $jwtAuth->generateToken(1, ['role' => 'admin']);

// 验证
$payload = $jwtAuth->validateToken($token);
if ($payload !== null) {
    $userId = $payload['user_id'];
    $role = $payload['role'] ?? 'user';
}
```

### Access Token / Refresh Token

```php
<?php
declare(strict_types=1);

class TokenPair
{
    public function __construct(
        public string $accessToken,
        public string $refreshToken
    ) {
    }
}

class DualTokenAuth
{
    private JWTAuth $jwtAuth;
    private \PDO $db;

    public function __construct(string $secret, \PDO $db)
    {
        $this->jwtAuth = new JWTAuth($secret);
        $this->db = $db;
    }

    public function login(int $userId): TokenPair
    {
        // Access Token（短期，15 分钟）
        $accessToken = $this->jwtAuth->generateToken($userId, [
            'type' => 'access',
            'exp' => time() + 900,  // 15 分钟
        ]);
        
        // Refresh Token（长期，7 天）
        $refreshToken = bin2hex(random_bytes(32));
        $this->saveRefreshToken($userId, $refreshToken);
        
        return new TokenPair($accessToken, $refreshToken);
    }

    public function refreshAccessToken(string $refreshToken): ?string
    {
        // 验证 Refresh Token
        $stmt = $this->db->prepare(
            'SELECT user_id FROM refresh_tokens WHERE token = ? AND expires_at > NOW()'
        );
        $stmt->execute([$refreshToken]);
        $result = $stmt->fetch(\PDO::FETCH_ASSOC);
        
        if ($result === false) {
            return null;
        }
        
        $userId = (int) $result['user_id'];
        
        // 生成新的 Access Token
        return $this->jwtAuth->generateToken($userId, [
            'type' => 'access',
            'exp' => time() + 900,
        ]);
    }

    private function saveRefreshToken(int $userId, string $token): void
    {
        $stmt = $this->db->prepare(
            'INSERT INTO refresh_tokens (user_id, token, expires_at) VALUES (?, ?, ?)'
        );
        $stmt->execute([$userId, $token, date('Y-m-d H:i:s', time() + 604800)]); // 7 天
    }
}
```

## OAuth2

### OAuth2 流程

```
1. 客户端请求授权
   ↓
2. 用户授权
   ↓
3. 授权服务器返回授权码
   ↓
4. 客户端用授权码换取 Access Token
   ↓
5. 使用 Access Token 访问资源
```

### OAuth2 服务器实现（简化版）

```php
<?php
declare(strict_types=1);

class OAuth2Server
{
    private \PDO $db;

    public function __construct(\PDO $db)
    {
        $this->db = $db;
    }

    public function authorize(string $clientId, string $redirectUri, string $state): string
    {
        // 验证客户端
        $client = $this->getClient($clientId);
        if ($client === null || $client['redirect_uri'] !== $redirectUri) {
            throw new InvalidArgumentException('Invalid client');
        }
        
        // 生成授权码
        $code = bin2hex(random_bytes(16));
        $this->saveAuthorizationCode($clientId, $code, $redirectUri);
        
        // 返回授权码（实际应该重定向到授权页面）
        return $code;
    }

    public function exchangeToken(string $code, string $clientId, string $clientSecret): ?array
    {
        // 验证客户端
        $client = $this->getClient($clientId);
        if ($client === null || $client['secret'] !== $clientSecret) {
            return null;
        }
        
        // 验证授权码
        $stmt = $this->db->prepare(
            'SELECT * FROM authorization_codes WHERE code = ? AND expires_at > NOW()'
        );
        $stmt->execute([$code]);
        $authCode = $stmt->fetch(\PDO::FETCH_ASSOC);
        
        if ($authCode === false) {
            return null;
        }
        
        // 生成 Access Token
        $accessToken = bin2hex(random_bytes(32));
        $refreshToken = bin2hex(random_bytes(32));
        
        $this->saveToken($clientId, $accessToken, $refreshToken);
        
        // 删除已使用的授权码
        $this->db->prepare('DELETE FROM authorization_codes WHERE code = ?')->execute([$code]);
        
        return [
            'access_token' => $accessToken,
            'refresh_token' => $refreshToken,
            'expires_in' => 3600,
            'token_type' => 'Bearer',
        ];
    }

    private function getClient(string $clientId): ?array
    {
        $stmt = $this->db->prepare('SELECT * FROM oauth_clients WHERE client_id = ?');
        $stmt->execute([$clientId]);
        return $stmt->fetch(\PDO::FETCH_ASSOC) ?: null;
    }

    private function saveAuthorizationCode(string $clientId, string $code, string $redirectUri): void
    {
        $stmt = $this->db->prepare(
            'INSERT INTO authorization_codes (client_id, code, redirect_uri, expires_at) 
             VALUES (?, ?, ?, ?)'
        );
        $stmt->execute([
            $clientId,
            $code,
            $redirectUri,
            date('Y-m-d H:i:s', time() + 600),  // 10 分钟过期
        ]);
    }

    private function saveToken(string $clientId, string $accessToken, string $refreshToken): void
    {
        $stmt = $this->db->prepare(
            'INSERT INTO oauth_tokens (client_id, access_token, refresh_token, expires_at) 
             VALUES (?, ?, ?, ?)'
        );
        $stmt->execute([
            $clientId,
            $accessToken,
            $refreshToken,
            date('Y-m-d H:i:s', time() + 3600),
        ]);
    }
}
```

## RBAC（基于角色的访问控制）

### 数据库设计

```sql
-- 用户表
CREATE TABLE users (
    id INT PRIMARY KEY,
    username VARCHAR(255),
    email VARCHAR(255)
);

-- 角色表
CREATE TABLE roles (
    id INT PRIMARY KEY,
    name VARCHAR(255) UNIQUE
);

-- 权限表
CREATE TABLE permissions (
    id INT PRIMARY KEY,
    name VARCHAR(255) UNIQUE,
    resource VARCHAR(255),
    action VARCHAR(255)
);

-- 用户角色关联
CREATE TABLE user_roles (
    user_id INT,
    role_id INT,
    PRIMARY KEY (user_id, role_id)
);

-- 角色权限关联
CREATE TABLE role_permissions (
    role_id INT,
    permission_id INT,
    PRIMARY KEY (role_id, permission_id)
);
```

### RBAC 实现

```php
<?php
declare(strict_types=1);

class RBAC
{
    private \PDO $db;

    public function __construct(\PDO $db)
    {
        $this->db = $db;
    }

    public function assignRole(int $userId, string $roleName): void
    {
        $roleId = $this->getRoleId($roleName);
        if ($roleId === null) {
            throw new InvalidArgumentException("Role {$roleName} not found");
        }
        
        $stmt = $this->db->prepare(
            'INSERT IGNORE INTO user_roles (user_id, role_id) VALUES (?, ?)'
        );
        $stmt->execute([$userId, $roleId]);
    }

    public function hasPermission(int $userId, string $resource, string $action): bool
    {
        $stmt = $this->db->prepare(
            'SELECT COUNT(*) FROM permissions p
             INNER JOIN role_permissions rp ON p.id = rp.permission_id
             INNER JOIN user_roles ur ON rp.role_id = ur.role_id
             WHERE ur.user_id = ? AND p.resource = ? AND p.action = ?'
        );
        $stmt->execute([$userId, $resource, $action]);
        
        return $stmt->fetchColumn() > 0;
    }

    public function getUserRoles(int $userId): array
    {
        $stmt = $this->db->prepare(
            'SELECT r.name FROM roles r
             INNER JOIN user_roles ur ON r.id = ur.role_id
             WHERE ur.user_id = ?'
        );
        $stmt->execute([$userId]);
        
        return $stmt->fetchAll(\PDO::FETCH_COLUMN);
    }

    public function getUserPermissions(int $userId): array
    {
        $stmt = $this->db->prepare(
            'SELECT DISTINCT p.name, p.resource, p.action FROM permissions p
             INNER JOIN role_permissions rp ON p.id = rp.permission_id
             INNER JOIN user_roles ur ON rp.role_id = ur.role_id
             WHERE ur.user_id = ?'
        );
        $stmt->execute([$userId]);
        
        return $stmt->fetchAll(\PDO::FETCH_ASSOC);
    }

    private function getRoleId(string $roleName): ?int
    {
        $stmt = $this->db->prepare('SELECT id FROM roles WHERE name = ?');
        $stmt->execute([$roleName]);
        $result = $stmt->fetchColumn();
        return $result !== false ? (int) $result : null;
    }
}

// 使用
$rbac = new RBAC($db);

// 分配角色
$rbac->assignRole(1, 'admin');
$rbac->assignRole(1, 'editor');

// 检查权限
if ($rbac->hasPermission(1, 'users', 'create')) {
    // 允许创建用户
}

// 获取用户角色
$roles = $rbac->getUserRoles(1); // ['admin', 'editor']
```

## ABAC（基于属性的访问控制）

### ABAC 实现

```php
<?php
declare(strict_types=1);

class ABAC
{
    public function checkAccess(
        array $userAttributes,
        string $resource,
        string $action,
        array $resourceAttributes = []
    ): bool {
        // 规则：管理员可以访问所有资源
        if (in_array('admin', $userAttributes['roles'] ?? [])) {
            return true;
        }
        
        // 规则：资源所有者可以访问自己的资源
        if ($action === 'read' && 
            isset($userAttributes['id']) && 
            isset($resourceAttributes['owner_id']) &&
            $userAttributes['id'] === $resourceAttributes['owner_id']) {
            return true;
        }
        
        // 规则：部门成员可以访问同部门的资源
        if (isset($userAttributes['department_id']) &&
            isset($resourceAttributes['department_id']) &&
            $userAttributes['department_id'] === $resourceAttributes['department_id']) {
            return true;
        }
        
        // 规则：工作时间外禁止访问敏感资源
        if ($resource === 'sensitive_data' && !$this->isBusinessHours()) {
            return false;
        }
        
        return false;
    }

    private function isBusinessHours(): bool
    {
        $hour = (int) date('H');
        return $hour >= 9 && $hour < 18;
    }
}

// 使用
$abac = new ABAC();

$user = [
    'id' => 1,
    'roles' => ['user'],
    'department_id' => 5,
];

$resource = [
    'id' => 10,
    'owner_id' => 1,
    'department_id' => 5,
];

$allowed = $abac->checkAccess($user, 'document', 'read', $resource);
```

## 权限中间件

```php
<?php
declare(strict_types=1);

class PermissionMiddleware
{
    private RBAC $rbac;
    private int $userId;

    public function __construct(RBAC $rbac, int $userId)
    {
        $this->rbac = $rbac;
        $this->userId = $userId;
    }

    public function requirePermission(string $resource, string $action): void
    {
        if (!$this->rbac->hasPermission($this->userId, $resource, $action)) {
            http_response_code(403);
            echo json_encode(['error' => 'Forbidden']);
            exit;
        }
    }

    public function requireRole(string $role): void
    {
        $roles = $this->rbac->getUserRoles($this->userId);
        if (!in_array($role, $roles, true)) {
            http_response_code(403);
            echo json_encode(['error' => 'Forbidden']);
            exit;
        }
    }
}

// 使用
$rbac = new RBAC($db);
$userId = getCurrentUserId();

$middleware = new PermissionMiddleware($rbac, $userId);

// 检查权限
$middleware->requirePermission('users', 'create');

// 检查角色
$middleware->requireRole('admin');
```

## 最佳实践

### 1. Token 安全

- 使用强密钥。
- 设置合理的过期时间。
- 使用 HTTPS 传输。
- 实现 Token 刷新机制。

### 2. 权限设计

- 使用最小权限原则。
- 定期审查权限分配。
- 记录权限变更日志。

### 3. 密码安全

- 使用 `password_hash()` 存储密码。
- 实现密码强度要求。
- 支持密码重置功能。

## 练习

1. 实现一个完整的 JWT 认证系统，包括 Token 生成、验证和刷新。

2. 创建一个 RBAC 系统，实现角色和权限的分配、检查功能。

3. 设计一个 OAuth2 授权服务器，支持授权码流程。

4. 实现一个权限中间件，根据用户角色和权限控制 API 访问。

5. 创建一个多因素认证（MFA）系统，支持短信验证码或 TOTP。

6. 设计一个权限审计系统，记录所有权限检查操作，便于安全审计。
