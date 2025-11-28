# 4.8 会话与状态管理

## 目标

- 理解 Web 的无状态特性。
- 掌握 Cookie、Session、Token/JWT 等状态管理机制。
- 熟悉 Session 的工作原理与配置。
- 了解 Redis Session Store 的使用场景。

## Web 无状态特性

### HTTP 无状态协议

- HTTP 协议本身是无状态的，每个请求独立处理。
- 服务器不保存客户端的状态信息。
- 需要状态管理机制来维持用户会话。

### 状态管理方案

| 方案        | 存储位置     | 特点                           | 适用场景           |
| :---------- | :----------- | :----------------------------- | :----------------- |
| Cookie      | 客户端       | 简单，但容量小，安全性较低     | 简单状态、偏好设置 |
| Session     | 服务器       | 安全，但需要存储空间           | 用户登录、购物车   |
| Token/JWT   | 客户端       | 无状态，可扩展，但需注意安全   | API 认证、微服务   |
| Redis Store | 服务器（Redis） | 高性能，可共享                | 分布式系统         |

## Cookie

### 基础用法

```php
<?php
declare(strict_types=1);

// 设置 Cookie
setcookie('username', 'alice', time() + 3600, '/', '', false, true);
// 参数：名称、值、过期时间、路径、域名、secure、httponly

// 读取 Cookie
$username = $_COOKIE['username'] ?? '';

// 删除 Cookie
setcookie('username', '', time() - 3600, '/');
```

### Cookie 参数详解

```php
<?php
declare(strict_types=1);

function setSecureCookie(
    string $name,
    string $value,
    int $expires = 3600,
    string $path = '/',
    ?string $domain = null,
    bool $secure = true,
    bool $httpOnly = true,
    string $sameSite = 'Strict'
): bool {
    $options = [
        'expires' => time() + $expires,
        'path' => $path,
        'secure' => $secure,
        'httponly' => $httpOnly,
        'samesite' => $sameSite,
    ];
    
    if ($domain !== null) {
        $options['domain'] = $domain;
    }
    
    return setcookie($name, $value, $options);
}

// 使用
setSecureCookie('session_id', 'abc123', 7200);
```

### SameSite 属性

```php
<?php
// SameSite=None: 允许跨站请求携带 Cookie（需要 Secure）
setcookie('token', 'value', [
    'samesite' => 'None',
    'secure' => true,
]);

// SameSite=Lax: 默认值，GET 请求可跨站，POST 不可
setcookie('token', 'value', ['samesite' => 'Lax']);

// SameSite=Strict: 完全禁止跨站请求携带 Cookie
setcookie('token', 'value', ['samesite' => 'Strict']);
```

## Session

### Session 基础

```php
<?php
declare(strict_types=1);

// 启动 Session
session_start();

// 设置 Session 数据
$_SESSION['user_id'] = 1;
$_SESSION['username'] = 'alice';

// 读取 Session 数据
$userId = $_SESSION['user_id'] ?? null;

// 删除 Session 数据
unset($_SESSION['user_id']);

// 销毁 Session
session_destroy();
```

### Session 工作原理

```
1. 客户端请求
   ↓
2. 服务器创建 Session ID（如果不存在）
   ↓
3. 通过 Set-Cookie 返回 Session ID
   ↓
4. 客户端保存 Session ID Cookie
   ↓
5. 后续请求携带 Session ID
   ↓
6. 服务器根据 Session ID 读取 Session 数据
```

### Session 配置

```php
<?php
// php.ini 配置
ini_set('session.save_handler', 'files');
ini_set('session.save_path', '/tmp/sessions');
ini_set('session.name', 'PHPSESSID');
ini_set('session.cookie_lifetime', 0);  // 浏览器关闭时过期
ini_set('session.cookie_path', '/');
ini_set('session.cookie_domain', '');
ini_set('session.cookie_secure', true);  // 仅 HTTPS
ini_set('session.cookie_httponly', true);
ini_set('session.cookie_samesite', 'Strict');
ini_set('session.gc_maxlifetime', 3600);  // Session 过期时间
```

### Session 安全

```php
<?php
declare(strict_types=1);

function startSecureSession(): void
{
    // 配置安全选项
    ini_set('session.cookie_secure', '1');
    ini_set('session.cookie_httponly', '1');
    ini_set('session.cookie_samesite', 'Strict');
    ini_set('session.use_strict_mode', '1');  // 只接受服务器生成的 Session ID
    
    // 启动 Session
    session_start();
    
    // 定期重新生成 Session ID（防止会话固定攻击）
    if (!isset($_SESSION['created'])) {
        session_regenerate_id(true);
        $_SESSION['created'] = time();
    } elseif (time() - $_SESSION['created'] > 1800) {
        // 30 分钟后重新生成
        session_regenerate_id(true);
        $_SESSION['created'] = time();
    }
}

startSecureSession();
```

### Session 数据管理

```php
<?php
declare(strict_types=1);

class SessionManager
{
    public static function start(): void
    {
        if (session_status() === PHP_SESSION_NONE) {
            session_start();
        }
    }

    public static function get(string $key, mixed $default = null): mixed
    {
        self::start();
        return $_SESSION[$key] ?? $default;
    }

    public static function set(string $key, mixed $value): void
    {
        self::start();
        $_SESSION[$key] = $value;
    }

    public static function has(string $key): bool
    {
        self::start();
        return isset($_SESSION[$key]);
    }

    public static function remove(string $key): void
    {
        self::start();
        unset($_SESSION[$key]);
    }

    public static function flash(string $key, mixed $value): void
    {
        self::start();
        $_SESSION['_flash'][$key] = $value;
    }

    public static function getFlash(string $key, mixed $default = null): mixed
    {
        self::start();
        $value = $_SESSION['_flash'][$key] ?? $default;
        unset($_SESSION['_flash'][$key]);
        return $value;
    }

    public static function destroy(): void
    {
        self::start();
        $_SESSION = [];
        if (isset($_COOKIE[session_name()])) {
            setcookie(session_name(), '', time() - 3600, '/');
        }
        session_destroy();
    }
}

// 使用
SessionManager::set('user_id', 1);
$userId = SessionManager::get('user_id');

// 闪存消息（一次性消息）
SessionManager::flash('success', '操作成功');
$message = SessionManager::getFlash('success');
```

## Redis Session Store

### 配置 Redis Session

```php
<?php
declare(strict_types=1);

// 使用 Redis 存储 Session
ini_set('session.save_handler', 'redis');
ini_set('session.save_path', 'tcp://127.0.0.1:6379?auth=password');

// 或使用 Redis 集群
ini_set('session.save_path', 'tcp://127.0.0.1:6379?auth=password&database=0');

session_start();
```

### 自定义 Session Handler

```php
<?php
declare(strict_types=1);

class RedisSessionHandler implements SessionHandlerInterface
{
    private \Redis $redis;
    private string $prefix = 'session:';
    private int $ttl = 3600;

    public function __construct(\Redis $redis, int $ttl = 3600)
    {
        $this->redis = $redis;
        $this->ttl = $ttl;
    }

    public function open(string $path, string $name): bool
    {
        return true;
    }

    public function close(): bool
    {
        return true;
    }

    public function read(string $id): string
    {
        $data = $this->redis->get($this->prefix . $id);
        return $data !== false ? $data : '';
    }

    public function write(string $id, string $data): bool
    {
        return $this->redis->setex($this->prefix . $id, $this->ttl, $data);
    }

    public function destroy(string $id): bool
    {
        return $this->redis->del($this->prefix . $id) > 0;
    }

    public function gc(int $max_lifetime): int|false
    {
        // Redis 自动过期，无需手动清理
        return 0;
    }
}

// 使用
$redis = new \Redis();
$redis->connect('127.0.0.1', 6379);

$handler = new RedisSessionHandler($redis, 3600);
session_set_save_handler($handler, true);
session_start();
```

## Token / JWT

### 简单 Token

```php
<?php
declare(strict_types=1);

class TokenManager
{
    private string $secret;

    public function __construct(string $secret)
    {
        $this->secret = $secret;
    }

    public function generate(int $userId, int $expiresIn = 3600): string
    {
        $payload = [
            'user_id' => $userId,
            'exp' => time() + $expiresIn,
            'iat' => time(),
        ];
        
        $token = base64_encode(json_encode($payload));
        $signature = hash_hmac('sha256', $token, $this->secret);
        
        return $token . '.' . $signature;
    }

    public function validate(string $token): ?array
    {
        $parts = explode('.', $token);
        if (count($parts) !== 2) {
            return null;
        }
        
        [$payload, $signature] = $parts;
        
        // 验证签名
        $expectedSignature = hash_hmac('sha256', $payload, $this->secret);
        if (!hash_equals($expectedSignature, $signature)) {
            return null;
        }
        
        $data = json_decode(base64_decode($payload), true);
        
        // 验证过期时间
        if (isset($data['exp']) && $data['exp'] < time()) {
            return null;
        }
        
        return $data;
    }
}

// 使用
$tokenManager = new TokenManager('your-secret-key');
$token = $tokenManager->generate(1, 7200);

// 验证 Token
$data = $tokenManager->validate($token);
if ($data !== null) {
    $userId = $data['user_id'];
}
```

### JWT（JSON Web Token）

```php
<?php
// 使用 firebase/php-jwt 库
require __DIR__ . '/vendor/autoload.php';

use Firebase\JWT\JWT;
use Firebase\JWT\Key;

$key = 'your-secret-key';
$payload = [
    'user_id' => 1,
    'email' => 'alice@example.com',
    'exp' => time() + 3600,
];

// 生成 JWT
$jwt = JWT::encode($payload, $key, 'HS256');

// 解码 JWT
try {
    $decoded = JWT::decode($jwt, new Key($key, 'HS256'));
    $userId = $decoded->user_id;
} catch (Exception $e) {
    // Token 无效
}
```

## 会话方式对比

### Cookie vs Session vs Token

| 特性           | Cookie              | Session             | Token/JWT           |
| :------------- | :------------------ | :------------------ | :------------------ |
| 存储位置       | 客户端              | 服务器              | 客户端              |
| 安全性         | 较低                | 较高                | 中等（需正确实现）  |
| 可扩展性       | 低                  | 中等（需共享存储）  | 高                  |
| 容量限制       | 4KB                 | 无限制              | 无限制              |
| 服务器负载     | 低                  | 高（需存储）        | 低                  |
| 跨域支持       | 有限                | 有限                | 好                  |
| 适用场景       | 简单状态、偏好      | 用户登录、购物车    | API 认证、微服务    |

## 实际应用示例

### 登录系统

```php
<?php
declare(strict_types=1);

session_start();

// 登录
function login(string $username, string $password): bool
{
    // 验证用户名密码
    $user = authenticate($username, $password);
    
    if ($user === null) {
        return false;
    }
    
    // 设置 Session
    session_regenerate_id(true);
    $_SESSION['user_id'] = $user['id'];
    $_SESSION['username'] = $user['username'];
    $_SESSION['logged_in'] = true;
    
    return true;
}

// 检查登录状态
function isLoggedIn(): bool
{
    return isset($_SESSION['logged_in']) && $_SESSION['logged_in'] === true;
}

// 获取当前用户
function getCurrentUser(): ?array
{
    if (!isLoggedIn()) {
        return null;
    }
    
    return [
        'id' => $_SESSION['user_id'],
        'username' => $_SESSION['username'],
    ];
}

// 登出
function logout(): void
{
    $_SESSION = [];
    if (isset($_COOKIE[session_name()])) {
        setcookie(session_name(), '', time() - 3600, '/');
    }
    session_destroy();
}
```

### API Token 认证

```php
<?php
declare(strict_types=1);

function getAuthToken(): ?string
{
    // 从 Header 获取
    $headers = getallheaders();
    if (isset($headers['Authorization'])) {
        $auth = $headers['Authorization'];
        if (preg_match('/Bearer\s+(.*)$/i', $auth, $matches)) {
            return $matches[1];
        }
    }
    
    // 从 Cookie 获取
    return $_COOKIE['token'] ?? null;
}

function authenticateRequest(): ?array
{
    $token = getAuthToken();
    
    if ($token === null) {
        return null;
    }
    
    $tokenManager = new TokenManager('secret-key');
    $data = $tokenManager->validate($token);
    
    return $data;
}

// 使用
$user = authenticateRequest();
if ($user === null) {
    http_response_code(401);
    echo json_encode(['error' => 'Unauthorized']);
    exit;
}

// 继续处理请求
$userId = $user['user_id'];
```

## 最佳实践

### 1. Session 安全

- 使用 `session_regenerate_id()` 防止会话固定攻击。
- 设置安全的 Cookie 选项（secure、httponly、samesite）。
- 定期清理过期 Session。

### 2. Token 安全

- 使用强密钥。
- 设置合理的过期时间。
- 不要在 Token 中存储敏感信息。
- 使用 HTTPS 传输。

### 3. 选择合适方案

- Web 应用：使用 Session。
- API 服务：使用 Token/JWT。
- 分布式系统：使用 Redis Session Store。

## 练习

1. 创建一个 Session 管理类，提供 get、set、has、remove、flash 等方法。

2. 实现一个安全的登录系统，使用 Session 存储用户信息，并实现会话固定攻击防护。

3. 编写一个 Token 生成和验证系统，支持自定义过期时间和签名验证。

4. 实现一个 Redis Session Handler，将 Session 数据存储到 Redis。

5. 创建一个 API 认证中间件，从 Header 或 Cookie 读取 Token 并验证。

6. 设计一个多设备登录管理系统，支持同一用户在不同设备上登录，并能管理所有活跃会话。
