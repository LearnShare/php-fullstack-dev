# 4.9.1 认证基础（AuthN）

## 概述

认证（Authentication）是验证用户身份的过程。本节详细介绍鉴权与授权的区别、基础 Token 鉴权、密码验证、Session 认证，以及完整的认证实现。

## 鉴权 vs 授权

### 概念区别

- **鉴权（Authentication）**：验证用户身份（"你是谁"）
- **授权（Authorization）**：验证用户权限（"你能做什么"）

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

    public function validateToken(string $token): ?int
    {
        // 从数据库验证 Token
        $stmt = $this->db->prepare(
            'SELECT user_id FROM user_tokens WHERE token = ? AND expires_at > NOW()'
        );
        $stmt->execute([$token]);
        $result = $stmt->fetch(\PDO::FETCH_ASSOC);
        
        return $result !== false ? (int) $result['user_id'] : null;
    }
}
```

## 密码验证

### 密码哈希

```php
<?php
declare(strict_types=1);

// 注册时哈希密码
$passwordHash = password_hash($password, PASSWORD_DEFAULT);

// 登录时验证密码
if (password_verify($password, $passwordHash)) {
    // 密码正确
}
```

### 密码策略

```php
<?php
declare(strict_types=1);

function validatePassword(string $password): array
{
    $errors = [];
    
    if (strlen($password) < 8) {
        $errors[] = '密码长度至少 8 位';
    }
    
    if (!preg_match('/[A-Z]/', $password)) {
        $errors[] = '密码必须包含大写字母';
    }
    
    if (!preg_match('/[a-z]/', $password)) {
        $errors[] = '密码必须包含小写字母';
    }
    
    if (!preg_match('/[0-9]/', $password)) {
        $errors[] = '密码必须包含数字';
    }
    
    return $errors;
}
```

## Session 认证

```php
<?php
declare(strict_types=1);

class SessionAuth
{
    public function login(string $username, string $password): bool
    {
        $user = $this->verifyUser($username, $password);
        
        if ($user === null) {
            return false;
        }
        
        session_start();
        session_regenerate_id(true);  // 防止会话固定攻击
        
        $_SESSION['user_id'] = $user['id'];
        $_SESSION['username'] = $user['username'];
        
        return true;
    }

    public function isAuthenticated(): bool
    {
        session_start();
        return isset($_SESSION['user_id']);
    }

    public function logout(): void
    {
        session_start();
        $_SESSION = [];
        session_destroy();
    }
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

class AuthenticationService
{
    public function login(string $username, string $password): array
    {
        $user = $this->verifyCredentials($username, $password);
        
        if ($user === null) {
            return ['success' => false, 'message' => 'Invalid credentials'];
        }
        
        // 生成 Token
        $token = $this->generateToken($user['id']);
        
        return [
            'success' => true,
            'token' => $token,
            'user' => $user,
        ];
    }

    private function verifyCredentials(string $username, string $password): ?array
    {
        // 验证逻辑
        return null;
    }
}
```

## 注意事项

1. **密码安全**：使用 `password_hash()` 和 `password_verify()`
2. **会话安全**：使用安全配置，防止会话劫持
3. **Token 安全**：使用强随机数生成 Token
4. **错误处理**：不泄露用户是否存在的信息

## 练习

1. 实现一个完整的登录系统，包含密码验证和 Token 生成。

2. 创建一个认证中间件，验证请求中的 Token。

3. 实现密码策略验证，确保密码强度。

4. 创建一个认证服务类，统一处理认证逻辑。
