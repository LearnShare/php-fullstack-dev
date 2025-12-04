# 4.3.2 $_SERVER、$_SESSION 与 $_COOKIE

## 概述

`$_SERVER`、`$_SESSION` 和 `$_COOKIE` 是处理服务器信息、会话管理和客户端状态的重要超全局变量。本节详细介绍服务器信息获取、Header 处理、Session 管理、Cookie 安全设置，以及客户端 IP 获取等实用技巧。

## $_SERVER - 服务器信息

### 常用变量

| 变量名 | 说明 | 示例 |
| :--- | :--- | :--- |
| `REQUEST_METHOD` | HTTP 请求方法 | `GET`、`POST`、`PUT`、`DELETE` |
| `REQUEST_URI` | 请求的 URI | `/users/1?page=2` |
| `SCRIPT_NAME` | 当前脚本路径 | `/index.php` |
| `QUERY_STRING` | 查询字符串 | `id=1&page=2` |
| `HTTP_HOST` | 主机名 | `example.com` |
| `HTTP_USER_AGENT` | 用户代理 | `Mozilla/5.0...` |
| `REMOTE_ADDR` | 客户端 IP 地址 | `192.168.1.1` |
| `HTTPS` | 是否 HTTPS | `on` 或空 |
| `CONTENT_TYPE` | 请求内容类型 | `application/json` |

### 实用函数

```php
<?php
declare(strict_types=1);

function getRequestMethod(): string
{
    return $_SERVER['REQUEST_METHOD'] ?? 'GET';
}

function getRequestUri(): string
{
    return $_SERVER['REQUEST_URI'] ?? '/';
}

function getBaseUrl(): string
{
    $protocol = (!empty($_SERVER['HTTPS']) && $_SERVER['HTTPS'] !== 'off') ? 'https' : 'http';
    $host = $_SERVER['HTTP_HOST'] ?? 'localhost';
    return "{$protocol}://{$host}";
}

function getClientIp(): string
{
    // 考虑代理情况
    if (!empty($_SERVER['HTTP_X_FORWARDED_FOR'])) {
        $ips = explode(',', $_SERVER['HTTP_X_FORWARDED_FOR']);
        return trim($ips[0]);
    }
    
    if (!empty($_SERVER['HTTP_X_REAL_IP'])) {
        return $_SERVER['HTTP_X_REAL_IP'];
    }
    
    return $_SERVER['REMOTE_ADDR'] ?? '0.0.0.0';
}

function isAjaxRequest(): bool
{
    return isset($_SERVER['HTTP_X_REQUESTED_WITH']) 
        && strtolower($_SERVER['HTTP_X_REQUESTED_WITH']) === 'xmlhttprequest';
}
```

### 获取所有 Headers

```php
<?php
function getAllHeaders(): array
{
    if (function_exists('getallheaders')) {
        return getallheaders();
    }
    
    // 手动构建
    $headers = [];
    foreach ($_SERVER as $key => $value) {
        if (strpos($key, 'HTTP_') === 0) {
            $header = str_replace(' ', '-', ucwords(strtolower(str_replace('_', ' ', substr($key, 5)))));
            $headers[$header] = $value;
        }
    }
    return $headers;
}
```

## $_SESSION - 会话管理

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

### Session 配置

```php
<?php
// 配置 Session（在 session_start() 之前）
ini_set('session.cookie_httponly', 1);
ini_set('session.cookie_secure', 1);  // 仅 HTTPS
ini_set('session.use_strict_mode', 1);
ini_set('session.cookie_samesite', 'Strict');

session_start();
```

### Session 存储到 Redis

```php
<?php
// 使用 Redis 存储 Session
ini_set('session.save_handler', 'redis');
ini_set('session.save_path', 'tcp://127.0.0.1:6379?prefix=session:');

session_start();
```

### Session 安全

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
        ini_set('session.gc_maxlifetime', 3600);  // 1 小时

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

    public static function set(string $key, mixed $value): void
    {
        $_SESSION[$key] = $value;
    }

    public static function get(string $key, mixed $default = null): mixed
    {
        return $_SESSION[$key] ?? $default;
    }

    public static function destroy(): void
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

## $_COOKIE - Cookie 数据

### 基础用法

```php
<?php
// 读取 Cookie
$userId = $_COOKIE['user_id'] ?? null;
$theme = $_COOKIE['theme'] ?? 'light';

// 设置 Cookie
setcookie('user_id', '123', time() + 3600, '/', '', true, true);

// 删除 Cookie
setcookie('user_id', '', time() - 3600, '/');
```

### 安全设置

```php
<?php
declare(strict_types=1);

function setSecureCookie(
    string $name,
    string $value,
    int $expires = 3600,
    string $path = '/',
    ?string $domain = null,
    string $sameSite = 'Strict'
): bool {
    return setcookie(
        $name,
        $value,
        [
            'expires' => time() + $expires,
            'path' => $path,
            'domain' => $domain,
            'secure' => true,      // 仅 HTTPS
            'httponly' => true,    // 防止 JavaScript 访问
            'samesite' => $sameSite  // 防止 CSRF
        ]
    );
}

// 使用
setSecureCookie('session_id', 'abc123', 7200);
```

### Cookie 参数说明

| 参数 | 说明 | 推荐值 |
| :--- | :--- | :--- |
| `expires` | 过期时间（Unix 时间戳） | `time() + 3600` |
| `path` | Cookie 路径 | `/` |
| `domain` | Cookie 域名 | 当前域名 |
| `secure` | 仅 HTTPS | `true` |
| `httponly` | 防止 JS 访问 | `true` |
| `samesite` | 防止 CSRF | `Strict` 或 `Lax` |

## 完整示例

```php
<?php
declare(strict_types=1);

class WebRequest
{
    public function getMethod(): string
    {
        return $_SERVER['REQUEST_METHOD'] ?? 'GET';
    }

    public function getUri(): string
    {
        return $_SERVER['REQUEST_URI'] ?? '/';
    }

    public function getClientIp(): string
    {
        if (!empty($_SERVER['HTTP_X_FORWARDED_FOR'])) {
            $ips = explode(',', $_SERVER['HTTP_X_FORWARDED_FOR']);
            return trim($ips[0]);
        }
        return $_SERVER['REMOTE_ADDR'] ?? '0.0.0.0';
    }

    public function getHeader(string $name): ?string
    {
        $key = 'HTTP_' . strtoupper(str_replace('-', '_', $name));
        return $_SERVER[$key] ?? null;
    }

    public function isAjax(): bool
    {
        return $this->getHeader('X-Requested-With') === 'XMLHttpRequest';
    }
}

// Session 管理
class SessionManager
{
    public function start(): void
    {
        SecureSession::start();
    }

    public function set(string $key, mixed $value): void
    {
        SecureSession::set($key, $value);
    }

    public function get(string $key, mixed $default = null): mixed
    {
        return SecureSession::get($key, $default);
    }
}
```

## 注意事项

1. **Session 安全**：使用安全配置，防止会话固定和劫持
2. **Cookie 安全**：设置 `secure`、`httponly`、`samesite` 属性
3. **IP 获取**：考虑代理和负载均衡的情况
4. **Header 处理**：注意 Header 名称的大小写和格式

## 练习

1. 编写一个函数获取客户端真实 IP 地址，考虑代理和负载均衡的情况。

2. 创建一个安全的 Session 管理类，包含启动、设置、获取、销毁等方法。

3. 创建一个安全的 Cookie 设置函数，包含所有安全选项。

4. 实现一个请求信息封装类，提供获取方法、URI、Headers 等便捷方法。
