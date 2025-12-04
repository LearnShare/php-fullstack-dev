# 4.8.1 Session 基础

## 概述

Session 是服务器端状态管理的核心机制。本节详细介绍 Web 无状态特性、Session 工作原理、Session 配置、Session 存储（文件/Redis），以及 Session 安全设置。

## Web 无状态特性

### HTTP 无状态协议

- HTTP 协议本身是无状态的，每个请求独立处理
- 服务器不保存客户端的状态信息
- 需要状态管理机制来维持用户会话

### 状态管理需求

- **用户登录状态**：保持用户登录
- **购物车**：保存用户购物车内容
- **用户偏好**：保存用户设置和偏好

## Session 工作原理

### 工作流程

```
1. 客户端请求
   ↓
2. 服务器创建 Session，生成 Session ID
   ↓
3. 通过 Cookie 返回 Session ID 给客户端
   ↓
4. 客户端后续请求携带 Session ID
   ↓
5. 服务器根据 Session ID 读取 Session 数据
```

### 基础用法

```php
<?php
// 启动 Session
session_start();

// 设置 Session 数据
$_SESSION['user_id'] = 123;
$_SESSION['username'] = 'alice';

// 读取 Session 数据
$userId = $_SESSION['user_id'] ?? null;

// 删除 Session 数据
unset($_SESSION['user_id']);

// 销毁 Session
session_destroy();
```

## Session 配置

### 安全配置

```php
<?php
// 配置 Session（在 session_start() 之前）
ini_set('session.cookie_httponly', 1);        // 防止 JavaScript 访问
ini_set('session.cookie_secure', 1);           // 仅 HTTPS
ini_set('session.use_strict_mode', 1);         // 严格模式
ini_set('session.cookie_samesite', 'Strict');  // 防止 CSRF
ini_set('session.gc_maxlifetime', 3600);       // 1 小时过期

session_start();
```

### 配置说明

| 配置项 | 说明 | 推荐值 |
| :--- | :--- | :--- |
| `session.cookie_httponly` | 防止 JS 访问 | `1` |
| `session.cookie_secure` | 仅 HTTPS | `1`（生产环境） |
| `session.use_strict_mode` | 严格模式 | `1` |
| `session.cookie_samesite` | 防止 CSRF | `Strict` |
| `session.gc_maxlifetime` | 过期时间（秒） | `3600` |

## Session 存储

### 文件存储（默认）

```php
<?php
// 默认存储在文件系统
ini_set('session.save_path', '/tmp/sessions');
session_start();
```

### Redis 存储

```php
<?php
// 使用 Redis 存储 Session
ini_set('session.save_handler', 'redis');
ini_set('session.save_path', 'tcp://127.0.0.1:6379?prefix=session:');

session_start();
```

### 自定义存储处理器

```php
<?php
declare(strict_types=1);

class RedisSessionHandler implements SessionHandlerInterface
{
    private \Redis $redis;
    private string $prefix = 'session:';

    public function __construct(\Redis $redis)
    {
        $this->redis = $redis;
    }

    public function open(string $save_path, string $name): bool
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
        return $this->redis->setex($this->prefix . $id, 3600, $data);
    }

    public function destroy(string $id): bool
    {
        return $this->redis->del($this->prefix . $id) > 0;
    }

    public function gc(int $max_lifetime): int|false
    {
        return 0;
    }
}

// 使用
$redis = new \Redis();
$redis->connect('127.0.0.1', 6379);
$handler = new RedisSessionHandler($redis);
session_set_save_handler($handler, true);
session_start();
```

## Session 安全

### 防止会话固定攻击

```php
<?php
declare(strict_types=1);

class SecureSession
{
    public static function start(): void
    {
        // 安全配置
        ini_set('session.cookie_httponly', 1);
        ini_set('session.cookie_secure', 1);
        ini_set('session.use_strict_mode', 1);
        ini_set('session.cookie_samesite', 'Strict');

        session_start();

        // 防止会话固定攻击
        if (!isset($_SESSION['created'])) {
            session_regenerate_id(true);
            $_SESSION['created'] = time();
        } elseif (time() - $_SESSION['created'] > 1800) {
            // 30 分钟后重新生成 ID
            session_regenerate_id(true);
            $_SESSION['created'] = time();
        }
    }
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

class SessionManager
{
    public function start(): void
    {
        SecureSession::start();
    }

    public function set(string $key, mixed $value): void
    {
        $_SESSION[$key] = $value;
    }

    public function get(string $key, mixed $default = null): mixed
    {
        return $_SESSION[$key] ?? $default;
    }

    public function has(string $key): bool
    {
        return isset($_SESSION[$key]);
    }

    public function remove(string $key): void
    {
        unset($_SESSION[$key]);
    }

    public function destroy(): void
    {
        $_SESSION = [];
        if (ini_get("session.use_cookies")) {
            $params = session_get_cookie_params();
            setcookie(session_name(), '', time() - 42000,
                $params["path"], $params["domain"],
                $params["secure"], $params["httponly"]
            );
        }
        session_destroy();
    }
}
```

## 注意事项

1. **安全配置**：使用安全配置防止会话劫持和固定攻击
2. **存储选择**：根据需求选择文件或 Redis 存储
3. **过期管理**：合理设置 Session 过期时间
4. **ID 再生**：定期重新生成 Session ID

## 练习

1. 创建一个安全的 Session 管理类，包含启动、设置、获取、销毁等方法。

2. 实现 Redis Session 存储处理器。

3. 实现 Session 安全防护，防止会话固定和劫持攻击。

4. 创建一个 Session 配置类，统一管理 Session 配置。
