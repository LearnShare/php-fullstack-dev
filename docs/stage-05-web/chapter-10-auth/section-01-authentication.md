# 5.10.1 认证基础（AuthN）

## 概述

认证（Authentication，AuthN）是验证用户身份的过程，是 Web 应用安全的基础。理解认证的概念、掌握密码验证方法、实现 Session 认证和 Token 认证，对于构建安全的 Web 应用至关重要。本节详细介绍认证的基本概念和方法，包括密码验证、Session 认证、Token 认证、认证流程、安全考虑等内容，帮助零基础学员掌握用户认证技术。

认证是确认"你是谁"的过程，与授权（Authorization，AuthZ）不同，授权是确认"你能做什么"的过程。理解这个区别对于正确实现认证和授权系统至关重要。

**主要内容**：
- 认证概念（什么是认证、认证的目的、认证方式）
- 密码验证（密码哈希存储、`password_hash()` 和 `password_verify()` 函数、密码强度要求、密码重置）
- Session 认证（Session 认证流程、登录状态保持、会话管理、登出处理）
- Token 认证（Token 生成、Token 验证、Token 存储、Token 刷新）
- 认证流程（登录流程、登出流程、认证状态检查）
- 安全考虑（密码安全、会话安全、Token 安全、认证状态检查）
- 实际应用示例和最佳实践

## 特性

- **多种方式**：支持密码、Session、Token 等多种认证方式
- **安全可靠**：使用安全的密码哈希和认证机制
- **灵活配置**：支持多种认证配置
- **易于扩展**：支持自定义认证逻辑
- **标准协议**：遵循标准认证协议

## 认证概念

### 什么是认证

认证（Authentication）是验证用户身份的过程，确认用户是否是他所声称的身份。

### 认证的目的

1. **身份验证**：确认用户身份
2. **访问控制**：控制资源访问
3. **安全保护**：保护用户数据
4. **审计追踪**：记录用户操作

### 认证方式

**主要方式**：
1. **密码认证**：用户名和密码
2. **Session 认证**：基于 Session 的认证
3. **Token 认证**：基于 Token 的认证（JWT）
4. **OAuth2**：第三方认证
5. **多因素认证**：密码 + 短信验证码等

## 密码验证

### 密码哈希存储

**重要原则**：永远不要存储明文密码。

**使用 `password_hash()`**：
```php
<?php
declare(strict_types=1);

// 生成密码哈希
$password = 'user_password';
$hash = password_hash($password, PASSWORD_DEFAULT);

// 存储到数据库
// INSERT INTO users (username, password_hash) VALUES ('john', ?)
```

### password_hash() 函数

**语法**：`password_hash(string $password, string|int|null $algo, array $options = []): string`

**参数**：
- `$password`：要哈希的密码
- `$algo`：哈希算法（`PASSWORD_DEFAULT`、`PASSWORD_BCRYPT`、`PASSWORD_ARGON2ID`）
- `$options`：可选，算法选项

**返回值**：返回哈希后的密码字符串。

**示例**：
```php
<?php
declare(strict_types=1);

// 使用默认算法（推荐）
$hash = password_hash('password123', PASSWORD_DEFAULT);

// 使用 BCRYPT 算法
$hash = password_hash('password123', PASSWORD_BCRYPT, [
    'cost' => 12,  // 计算成本（4-31）
]);

// 使用 ARGON2ID 算法（PHP 7.2+）
$hash = password_hash('password123', PASSWORD_ARGON2ID, [
    'memory_cost' => 65536,  // 内存成本（KB）
    'time_cost' => 4,        // 时间成本
    'threads' => 3,          // 线程数
]);
```

### password_verify() 函数

**语法**：`password_verify(string $password, string $hash): bool`

**参数**：
- `$password`：要验证的密码
- `$hash`：存储的密码哈希

**返回值**：密码匹配返回 `true`，否则返回 `false`。

**示例**：
```php
<?php
declare(strict_types=1);

// 验证密码
$password = 'user_password';
$hash = '$2y$10$...';  // 从数据库获取

if (password_verify($password, $hash)) {
    echo "密码正确\n";
} else {
    echo "密码错误\n";
}
```

### 完整登录示例

**示例**：
```php
<?php
declare(strict_types=1);

class AuthService
{
    public function login(string $username, string $password): bool
    {
        // 1. 从数据库获取用户
        $user = $this->getUserByUsername($username);
        if ($user === null) {
            return false;  // 用户不存在
        }

        // 2. 验证密码
        if (!password_verify($password, $user['password_hash'])) {
            return false;  // 密码错误
        }

        // 3. 创建会话
        session_start();
        session_regenerate_id(true);  // 防止 Session 固定攻击
        
        $_SESSION['user_id'] = $user['id'];
        $_SESSION['username'] = $user['username'];
        $_SESSION['authenticated'] = true;
        $_SESSION['login_time'] = time();

        return true;
    }

    private function getUserByUsername(string $username): ?array
    {
        // 从数据库查询用户
        // ...
        return null;
    }
}
```

### 密码强度要求

**示例**：
```php
<?php
declare(strict_types=1);

function validatePasswordStrength(string $password): array
{
    $errors = [];

    // 最小长度
    if (strlen($password) < 8) {
        $errors[] = '密码长度至少 8 个字符';
    }

    // 包含大写字母
    if (!preg_match('/[A-Z]/', $password)) {
        $errors[] = '密码必须包含至少一个大写字母';
    }

    // 包含小写字母
    if (!preg_match('/[a-z]/', $password)) {
        $errors[] = '密码必须包含至少一个小写字母';
    }

    // 包含数字
    if (!preg_match('/[0-9]/', $password)) {
        $errors[] = '密码必须包含至少一个数字';
    }

    // 包含特殊字符
    if (!preg_match('/[^A-Za-z0-9]/', $password)) {
        $errors[] = '密码必须包含至少一个特殊字符';
    }

    return $errors;
}

// 使用
$password = 'UserPass123!';
$errors = validatePasswordStrength($password);
if (empty($errors)) {
    $hash = password_hash($password, PASSWORD_DEFAULT);
} else {
    foreach ($errors as $error) {
        echo $error . "\n";
    }
}
```

### 密码重置

**示例**：
```php
<?php
declare(strict_types=1);

class PasswordResetService
{
    public function requestReset(string $email): void
    {
        // 1. 查找用户
        $user = $this->getUserByEmail($email);
        if ($user === null) {
            return;  // 用户不存在，不泄露信息
        }

        // 2. 生成重置 Token
        $token = bin2hex(random_bytes(32));
        $expires = time() + 3600;  // 1 小时后过期

        // 3. 存储重置 Token
        $this->saveResetToken($user['id'], $token, $expires);

        // 4. 发送重置邮件
        $this->sendResetEmail($email, $token);
    }

    public function resetPassword(string $token, string $newPassword): bool
    {
        // 1. 验证 Token
        $reset = $this->getResetByToken($token);
        if ($reset === null || $reset['expires'] < time()) {
            return false;  // Token 无效或过期
        }

        // 2. 验证密码强度
        $errors = validatePasswordStrength($newPassword);
        if (!empty($errors)) {
            return false;
        }

        // 3. 更新密码
        $hash = password_hash($newPassword, PASSWORD_DEFAULT);
        $this->updatePassword($reset['user_id'], $hash);

        // 4. 删除重置 Token
        $this->deleteResetToken($token);

        return true;
    }
}
```

## Session 认证

### Session 认证流程

**流程**：
1. 用户提交登录表单
2. 验证用户名和密码
3. 创建 Session，存储用户信息
4. 返回 Session ID（通过 Cookie）
5. 后续请求使用 Session ID 验证身份

**示例**：
```php
<?php
declare(strict_types=1);

class SessionAuth
{
    public function login(string $username, string $password): bool
    {
        // 验证用户
        $user = $this->validateUser($username, $password);
        if ($user === null) {
            return false;
        }

        // 启动会话
        session_start();
        session_regenerate_id(true);  // 防止 Session 固定攻击

        // 存储用户信息
        $_SESSION['user_id'] = $user['id'];
        $_SESSION['username'] = $user['username'];
        $_SESSION['authenticated'] = true;
        $_SESSION['login_time'] = time();
        $_SESSION['ip_address'] = $_SERVER['REMOTE_ADDR'] ?? '';
        $_SESSION['user_agent'] = $_SERVER['HTTP_USER_AGENT'] ?? '';

        return true;
    }

    public function isAuthenticated(): bool
    {
        session_start();
        
        if (!isset($_SESSION['authenticated']) || !$_SESSION['authenticated']) {
            return false;
        }

        // 检查会话是否过期（30 分钟）
        if (isset($_SESSION['last_activity']) && (time() - $_SESSION['last_activity'] > 1800)) {
            $this->logout();
            return false;
        }

        // 验证 IP 和 User-Agent（防止 Session 劫持）
        $currentIp = $_SERVER['REMOTE_ADDR'] ?? '';
        $currentUserAgent = $_SERVER['HTTP_USER_AGENT'] ?? '';
        
        if (isset($_SESSION['ip_address']) && $_SESSION['ip_address'] !== $currentIp) {
            $this->logout();
            return false;
        }

        if (isset($_SESSION['user_agent']) && $_SESSION['user_agent'] !== $currentUserAgent) {
            $this->logout();
            return false;
        }

        $_SESSION['last_activity'] = time();
        return true;
    }

    public function logout(): void
    {
        session_start();
        $_SESSION = [];
        
        if (isset($_COOKIE[session_name()])) {
            setcookie(session_name(), '', time() - 3600, '/');
        }
        
        session_destroy();
    }

    private function validateUser(string $username, string $password): ?array
    {
        // 验证用户逻辑
        // ...
        return null;
    }
}
```

### 登录状态保持

**示例**：
```php
<?php
declare(strict_types=1);

// 检查登录状态
function requireAuth(): void
{
    session_start();
    
    if (!isset($_SESSION['authenticated']) || !$_SESSION['authenticated']) {
        http_response_code(401);
        header('Location: /login.php');
        exit;
    }
}

// 使用
requireAuth();
// 后续代码只有认证用户才能访问
```

### 会话管理

**示例**：
```php
<?php
declare(strict_types=1);

class SessionManager
{
    public function startSession(): void
    {
        session_start([
            'cookie_lifetime' => 0,                      // 浏览器关闭时过期
            'cookie_secure' => true,                    // 仅 HTTPS
            'cookie_httponly' => true,                  // 防止 JavaScript 访问
            'cookie_samesite' => 'Strict',              // 防止 CSRF
            'use_strict_mode' => true,                   // 防止 Session 固定攻击
        ]);
    }

    public function checkSession(): bool
    {
        if (!isset($_SESSION['authenticated'])) {
            return false;
        }

        // 检查过期时间
        if (isset($_SESSION['expires']) && $_SESSION['expires'] < time()) {
            $this->destroySession();
            return false;
        }

        return true;
    }

    public function destroySession(): void
    {
        $_SESSION = [];
        
        if (isset($_COOKIE[session_name()])) {
            setcookie(session_name(), '', time() - 3600, '/');
        }
        
        session_destroy();
    }
}
```

### 登出处理

**示例**：
```php
<?php
declare(strict_types=1);

function logout(): void
{
    session_start();
    
    // 清空会话数据
    $_SESSION = [];
    
    // 删除 Session Cookie
    if (isset($_COOKIE[session_name()])) {
        setcookie(session_name(), '', time() - 3600, '/');
    }
    
    // 销毁会话
    session_destroy();
    
    // 重定向到登录页
    header('Location: /login.php');
    exit;
}
```

## Token 认证

### Token 生成

**示例**：
```php
<?php
declare(strict_types=1);

class TokenAuth
{
    private string $secretKey;

    public function __construct(string $secretKey)
    {
        $this->secretKey = $secretKey;
    }

    public function generateToken(int $userId, int $expiresIn = 3600): string
    {
        $payload = [
            'user_id' => $userId,
            'iat' => time(),           // 签发时间
            'exp' => time() + $expiresIn,  // 过期时间
        ];

        // 使用 base64 编码（简单示例，实际应使用 JWT 库）
        $header = base64_encode(json_encode(['typ' => 'JWT', 'alg' => 'HS256']));
        $payload = base64_encode(json_encode($payload));
        $signature = hash_hmac('sha256', "{$header}.{$payload}", $this->secretKey, true);
        $signature = base64_encode($signature);

        return "{$header}.{$payload}.{$signature}";
    }
}
```

### Token 验证

**示例**：
```php
<?php
declare(strict_types=1);

class TokenAuth
{
    public function validateToken(string $token): ?array
    {
        $parts = explode('.', $token);
        if (count($parts) !== 3) {
            return null;  // Token 格式错误
        }

        [$header, $payload, $signature] = $parts;

        // 验证签名
        $expectedSignature = hash_hmac('sha256', "{$header}.{$payload}", $this->secretKey, true);
        $expectedSignature = base64_encode($expectedSignature);

        if ($signature !== $expectedSignature) {
            return null;  // 签名验证失败
        }

        // 解析 Payload
        $payloadData = json_decode(base64_decode($payload), true);
        if ($payloadData === null) {
            return null;  // Payload 解析失败
        }

        // 检查过期时间
        if (isset($payloadData['exp']) && $payloadData['exp'] < time()) {
            return null;  // Token 已过期
        }

        return $payloadData;
    }
}
```

### Token 存储

**客户端存储**：
- **Cookie**：HttpOnly Cookie（推荐）
- **LocalStorage**：JavaScript 可访问（不推荐存储敏感 Token）
- **SessionStorage**：会话期间有效

**示例**：
```php
<?php
declare(strict_types=1);

// 存储 Token 到 Cookie
function setAuthToken(string $token): void
{
    setcookie('auth_token', $token, [
        'expires' => time() + 3600,
        'path' => '/',
        'secure' => true,
        'httponly' => true,
        'samesite' => 'Strict',
    ]);
}

// 从 Cookie 读取 Token
function getAuthToken(): ?string
{
    return $_COOKIE['auth_token'] ?? null;
}
```

### Token 刷新

**示例**：
```php
<?php
declare(strict_types=1);

class TokenAuth
{
    public function refreshToken(string $refreshToken): ?string
    {
        // 验证刷新 Token
        $payload = $this->validateToken($refreshToken);
        if ($payload === null) {
            return null;  // 刷新 Token 无效
        }

        // 检查刷新 Token 类型
        if (!isset($payload['type']) || $payload['type'] !== 'refresh') {
            return null;  // 不是刷新 Token
        }

        // 生成新的访问 Token
        return $this->generateToken($payload['user_id'], 3600);
    }

    public function generateTokenPair(int $userId): array
    {
        $accessToken = $this->generateToken($userId, 3600);  // 1 小时
        $refreshToken = $this->generateToken($userId, 86400 * 7);  // 7 天
        $refreshTokenPayload = json_decode(base64_decode(explode('.', $refreshToken)[1]), true);
        $refreshTokenPayload['type'] = 'refresh';
        // 重新生成刷新 Token（包含 type）
        
        return [
            'access_token' => $accessToken,
            'refresh_token' => $refreshToken,
            'expires_in' => 3600,
        ];
    }
}
```

## 认证流程

### 登录流程

**示例**：
```php
<?php
declare(strict_types=1);

class AuthController
{
    public function login(): void
    {
        $username = $_POST['username'] ?? '';
        $password = $_POST['password'] ?? '';

        // 验证输入
        if (empty($username) || empty($password)) {
            http_response_code(400);
            echo json_encode(['error' => '用户名和密码不能为空']);
            exit;
        }

        // 验证用户
        $authService = new AuthService();
        if (!$authService->login($username, $password)) {
            http_response_code(401);
            echo json_encode(['error' => '用户名或密码错误']);
            exit;
        }

        // 登录成功
        http_response_code(200);
        echo json_encode(['message' => '登录成功']);
    }
}
```

### 登出流程

**示例**：
```php
<?php
declare(strict_types=1);

class AuthController
{
    public function logout(): void
    {
        // Session 认证
        session_start();
        session_destroy();

        // Token 认证：删除 Token Cookie
        setcookie('auth_token', '', time() - 3600, '/', '', true, true);

        http_response_code(200);
        echo json_encode(['message' => '登出成功']);
    }
}
```

### 认证状态检查

**示例**：
```php
<?php
declare(strict_types=1);

class AuthMiddleware
{
    public function requireAuth(): void
    {
        // 方法 1：Session 认证
        session_start();
        if (!isset($_SESSION['authenticated']) || !$_SESSION['authenticated']) {
            http_response_code(401);
            echo json_encode(['error' => '未认证']);
            exit;
        }

        // 方法 2：Token 认证
        $token = $_SERVER['HTTP_AUTHORIZATION'] ?? '';
        $token = str_replace('Bearer ', '', $token);
        
        $tokenAuth = new TokenAuth($secretKey);
        $payload = $tokenAuth->validateToken($token);
        
        if ($payload === null) {
            http_response_code(401);
            echo json_encode(['error' => 'Token 无效或已过期']);
            exit;
        }
    }
}
```

## 使用场景

### 用户登录

- 用户名密码登录
- 邮箱密码登录
- 手机号验证码登录

### API 认证

- Token 认证
- API Key 认证
- OAuth2 认证

### 第三方认证

- OAuth2 登录
- 社交账号登录
- 单点登录（SSO）

## 注意事项

### 密码安全存储

- **使用 password_hash()**：永远不要存储明文密码
- **使用强算法**：使用 `PASSWORD_DEFAULT` 或 `PASSWORD_ARGON2ID`
- **验证密码强度**：要求强密码

### 会话安全

- **Session 固定攻击防护**：登录后重新生成 Session ID
- **Session 劫持防护**：验证 IP 和 User-Agent
- **会话过期**：设置合理的过期时间

### Token 安全

- **强密钥**：使用强密钥签名 Token
- **过期时间**：设置合理的过期时间
- **Token 刷新**：实现 Token 刷新机制

### 认证状态检查

- **中间件检查**：使用中间件检查认证状态
- **及时验证**：每次请求验证认证状态
- **错误处理**：提供清晰的错误信息

## 常见问题

### 如何验证密码？

使用 `password_verify()` 函数：

```php
<?php
declare(strict_types=1);

$password = 'user_password';
$hash = '$2y$10$...';  // 从数据库获取

if (password_verify($password, $hash)) {
    // 密码正确
}
```

### Session 认证和 Token 认证的区别？

| 特性 | Session 认证 | Token 认证 |
|:-----|:-------------|:------------|
| 存储位置 | 服务器端 | 客户端 |
| 无状态 | ❌ 有状态 | ✅ 无状态 |
| 可扩展性 | ⚠️ 需要共享存储 | ✅ 易于扩展 |
| 安全性 | ✅ 相对安全 | ⚠️ 需要安全传输 |

### 如何实现登录功能？

1. 验证用户名和密码
2. 创建 Session 或生成 Token
3. 存储用户信息
4. 返回认证信息

### 如何检查认证状态？

**Session 认证**：
```php
<?php
declare(strict_types=1);

session_start();
$isAuthenticated = isset($_SESSION['authenticated']) && $_SESSION['authenticated'];
```

**Token 认证**：
```php
<?php
declare(strict_types=1);

$token = $_SERVER['HTTP_AUTHORIZATION'] ?? '';
$payload = validateToken($token);
$isAuthenticated = $payload !== null;
```

## 最佳实践

### 使用 password_hash() 存储密码

- 使用 `password_hash()` 生成哈希
- 使用 `password_verify()` 验证密码
- 使用强算法（`PASSWORD_DEFAULT`）

### 使用 password_verify() 验证密码

- 始终使用 `password_verify()` 验证
- 不要手动比较哈希
- 处理验证失败的情况

### 实现会话超时

- 设置合理的会话过期时间
- 检查最后活动时间
- 自动清理过期会话

### 使用 HTTPS 传输认证信息

- 使用 HTTPS 传输密码
- 使用 HTTPS 传输 Token
- 使用 Secure Cookie

## 相关章节

- **[5.10.2 授权模型（AuthZ）](section-02-authorization.md)**：了解授权的详细内容
- **[5.9.1 Session 基础](../chapter-09-session/section-01-session-basics.md)**：了解 Session 的详细内容
- **[5.10.3 JWT 与 Token](section-03-jwt-token.md)**：了解 JWT Token 的详细内容

## 练习任务

1. **实现密码验证系统**
   - 使用 password_hash() 存储密码
   - 使用 password_verify() 验证密码
   - 实现密码强度验证

2. **实现 Session 认证**
   - 实现登录功能
   - 实现认证状态检查
   - 实现登出功能

3. **实现 Token 认证**
   - 生成和验证 Token
   - 实现 Token 刷新
   - 管理 Token 生命周期

4. **实现认证中间件**
   - 创建认证中间件
   - 检查认证状态
   - 处理认证错误

5. **实现完整的认证系统**
   - 登录和登出
   - 认证状态管理
   - 安全防护机制
