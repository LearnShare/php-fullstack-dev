# 2.12.2 $_SERVER、$_SESSION 与 $_COOKIE

## 概述

`$_SERVER`、`$_SESSION` 和 `$_COOKIE` 是 PHP 的重要超级全局变量，分别用于获取服务器信息、管理会话状态和处理 Cookie 数据。

## $_SERVER - 服务器信息

### 基本概念

`$_SERVER` 包含服务器和执行环境的信息，如请求方法、URI、HTTP 头等。

### 常用键

| 键名                | 说明                     |
| :------------------ | :----------------------- |
| `REQUEST_METHOD`    | HTTP 请求方法（GET、POST） |
| `REQUEST_URI`        | 请求的 URI               |
| `SCRIPT_NAME`       | 当前脚本的路径           |
| `HTTP_HOST`         | 主机名                   |
| `HTTP_USER_AGENT`   | 用户代理字符串           |
| `REMOTE_ADDR`        | 客户端 IP 地址           |
| `SERVER_NAME`        | 服务器名称               |
| `SERVER_PORT`        | 服务器端口               |

### 基本用法

```php
<?php
declare(strict_types=1);

// 获取请求方法
$method = $_SERVER['REQUEST_METHOD'] ?? 'GET';
echo "Method: {$method}\n";

// 获取请求 URI
$uri = $_SERVER['REQUEST_URI'] ?? '/';
echo "URI: {$uri}\n";

// 获取客户端 IP
$ip = $_SERVER['REMOTE_ADDR'] ?? 'unknown';
echo "IP: {$ip}\n";
```

### 检查 HTTPS

```php
<?php
declare(strict_types=1);

function isHttps(): bool
{
    return isset($_SERVER['HTTPS']) && $_SERVER['HTTPS'] === 'on';
}

if (isHttps()) {
    echo "Secure connection\n";
}
```

## $_SESSION - 会话管理

### 基本概念

`$_SESSION` 用于在多个请求之间存储用户数据。Session 数据存储在服务器端，客户端只保存 Session ID。

### 启动 Session

```php
<?php
declare(strict_types=1);

// 启动 Session（必须在输出之前）
session_start();

// 设置 Session 数据
$_SESSION['user_id'] = 1;
$_SESSION['username'] = 'Alice';
```

### 读取 Session 数据

```php
<?php
declare(strict_types=1);

session_start();

$userId = $_SESSION['user_id'] ?? null;
$username = $_SESSION['username'] ?? 'Guest';

echo "User ID: {$userId}, Username: {$username}\n";
```

### 修改和删除

```php
<?php
declare(strict_types=1);

session_start();

// 修改
$_SESSION['username'] = 'Bob';

// 删除单个键
unset($_SESSION['username']);

// 清空所有 Session 数据
$_SESSION = [];

// 销毁 Session
session_destroy();
```

### Session 配置

```php
<?php
declare(strict_types=1);

// 设置 Session 参数
ini_set('session.cookie_httponly', '1');  // 防止 XSS
ini_set('session.cookie_secure', '1');    // 仅 HTTPS
ini_set('session.use_strict_mode', '1');   // 严格模式

session_start();
```

## $_COOKIE - Cookie 数据

### 基本概念

`$_COOKIE` 用于读取客户端发送的 Cookie 数据。

### 读取 Cookie

```php
<?php
declare(strict_types=1);

$username = $_COOKIE['username'] ?? 'Guest';
echo "Username: {$username}\n";
```

### 设置 Cookie

使用 `setcookie()` 函数设置 Cookie：

**语法**：`setcookie(string $name, string $value = "", int $expires = 0, string $path = "", string $domain = "", bool $secure = false, bool $httponly = false): bool`

```php
<?php
declare(strict_types=1);

// 基本用法
setcookie('username', 'Alice', time() + 3600);  // 1 小时后过期

// 完整参数
setcookie(
    'username',           // 名称
    'Alice',             // 值
    time() + 3600,       // 过期时间
    '/',                 // 路径
    'example.com',       // 域名
    true,                // 仅 HTTPS
    true                 // 仅 HTTP（防止 JavaScript 访问）
);
```

### 删除 Cookie

```php
<?php
declare(strict_types=1);

// 设置过期时间为过去的时间
setcookie('username', '', time() - 3600);
```

## 完整示例

```php
<?php
declare(strict_types=1);

class ServerSessionCookie
{
    public static function demonstrateServer(): void
    {
        echo "=== \$_SERVER ===\n";
        echo "Method: " . ($_SERVER['REQUEST_METHOD'] ?? 'N/A') . "\n";
        echo "URI: " . ($_SERVER['REQUEST_URI'] ?? 'N/A') . "\n";
        echo "IP: " . ($_SERVER['REMOTE_ADDR'] ?? 'N/A') . "\n";
    }
    
    public static function demonstrateSession(): void
    {
        echo "\n=== \$_SESSION ===\n";
        session_start();
        
        // 设置
        $_SESSION['user_id'] = 1;
        $_SESSION['username'] = 'Alice';
        
        // 读取
        echo "User ID: " . ($_SESSION['user_id'] ?? 'N/A') . "\n";
        echo "Username: " . ($_SESSION['username'] ?? 'N/A') . "\n";
    }
    
    public static function demonstrateCookie(): void
    {
        echo "\n=== \$_COOKIE ===\n";
        // 设置 Cookie
        setcookie('theme', 'dark', time() + 3600);
        
        // 读取
        $theme = $_COOKIE['theme'] ?? 'light';
        echo "Theme: {$theme}\n";
    }
}

ServerSessionCookie::demonstrateServer();
ServerSessionCookie::demonstrateSession();
ServerSessionCookie::demonstrateCookie();
```

## 安全建议

1. **Session 安全**：
   - 使用 `session_regenerate_id()` 定期更新 Session ID
   - 设置 `session.cookie_httponly` 防止 XSS
   - 设置 `session.cookie_secure` 仅 HTTPS

2. **Cookie 安全**：
   - 敏感数据不要存储在 Cookie 中
   - 使用 `httponly` 标志防止 JavaScript 访问
   - 使用 `secure` 标志仅 HTTPS 传输

3. **输入验证**：始终验证 `$_SERVER`、`$_SESSION` 和 `$_COOKIE` 中的数据。

4. **CSRF 防护**：使用 CSRF token 保护表单提交。

5. **Session 固定**：使用 `session_regenerate_id()` 防止 Session 固定攻击。

## 注意事项

1. **Session 启动**：`session_start()` 必须在输出之前调用。

2. **Cookie 设置**：`setcookie()` 必须在输出之前调用。

3. **数据来源**：不要信任 `$_SERVER` 中的所有数据，某些值可能被伪造。

4. **Session 存储**：默认存储在文件系统，生产环境考虑使用 Redis 或数据库。

5. **Cookie 大小**：Cookie 有大小限制（通常 4KB），不要存储大量数据。

## 练习

1. 创建一个函数，安全地获取客户端 IP 地址（考虑代理）。

2. 编写一个 Session 管理类，提供安全的 Session 操作。

3. 实现一个 Cookie 工具类，安全地设置和读取 Cookie。

4. 创建一个函数，检测请求是否来自移动设备。

5. 编写一个函数，实现 Session 的自动过期和清理。
