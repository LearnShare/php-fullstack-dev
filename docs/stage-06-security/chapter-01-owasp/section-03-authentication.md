# 6.1.3 认证与会话管理

## 概述

认证和会话管理是 Web 应用安全的基础。本节详细介绍认证失败、会话管理、密码安全、多因素认证，以及完整的认证系统实现。

## 认证失败

### 常见问题

- 弱密码策略
- 密码明文存储
- 会话管理不当
- 认证绕过漏洞

### 防护措施

#### 1. 强密码策略

```php
<?php
declare(strict_types=1);

function validatePassword(string $password): bool
{
    // 至少 8 位，包含大小写字母、数字和特殊字符
    return preg_match('/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{8,}$/', $password);
}
```

#### 2. 密码哈希

```php
<?php
declare(strict_types=1);

// 使用 Argon2id
$hash = password_hash($password, PASSWORD_ARGON2ID, [
    'memory_cost' => 65536,
    'time_cost' => 4,
    'threads' => 3,
]);

// 验证密码
if (password_verify($password, $hash)) {
    // 密码正确
}
```

#### 3. 登录限制

```php
<?php
declare(strict_types=1);

class LoginLimiter
{
    private Redis $redis;
    
    public function checkLimit(string $ip, string $username): bool
    {
        $key = "login_attempts:{$ip}:{$username}";
        $attempts = (int) $this->redis->get($key);
        
        if ($attempts >= 5) {
            return false;  // 超过限制
        }
        
        return true;
    }
    
    public function recordAttempt(string $ip, string $username): void
    {
        $key = "login_attempts:{$ip}:{$username}";
        $this->redis->incr($key);
        $this->redis->expire($key, 900);  // 15 分钟
    }
}
```

## 会话管理

### 安全配置

```php
<?php
declare(strict_types=1);

// 安全配置
ini_set('session.cookie_httponly', 1);
ini_set('session.cookie_secure', 1);
ini_set('session.use_strict_mode', 1);
ini_set('session.cookie_samesite', 'Strict');

session_start();
```

### 会话固定防护

```php
<?php
declare(strict_types=1);

function secureSessionStart(): void
{
    session_start();
    
    // 防止会话固定
    if (!isset($_SESSION['created'])) {
        session_regenerate_id(true);
        $_SESSION['created'] = true;
    }
}
```

## 多因素认证

### TOTP 实现

```php
<?php
declare(strict_types=1);

require __DIR__ . '/vendor/autoload.php';

use OTPHP\TOTP;

// 生成密钥
$totp = TOTP::create();
$secret = $totp->getSecret();

// 验证
$code = $_POST['code'] ?? '';
if ($totp->verify($code)) {
    // 验证成功
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

class AuthenticationService
{
    private PDO $pdo;
    private Redis $redis;

    public function login(string $username, string $password): bool
    {
        $stmt = $this->pdo->prepare("SELECT * FROM users WHERE username = ?");
        $stmt->execute([$username]);
        $user = $stmt->fetch();
        
        if ($user === false) {
            return false;
        }
        
        if (!password_verify($password, $user['password_hash'])) {
            $this->recordFailedAttempt($username);
            return false;
        }
        
        $this->createSession($user);
        return true;
    }

    private function createSession(array $user): void
    {
        session_start();
        session_regenerate_id(true);
        $_SESSION['user_id'] = $user['id'];
        $_SESSION['username'] = $user['username'];
    }
}
```

## 注意事项

1. **密码安全**：使用强密码哈希算法
2. **会话安全**：正确配置会话安全选项
3. **登录限制**：实现登录尝试限制
4. **多因素认证**：关键操作使用多因素认证

## 练习

1. 实现完整的认证系统，包含登录限制。

2. 创建一个多因素认证系统。

3. 编写会话管理工具。

4. 实现密码策略验证。
