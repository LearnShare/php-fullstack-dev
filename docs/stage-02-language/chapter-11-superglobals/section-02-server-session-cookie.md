# 2.11.2 $_SERVER、$_SESSION 与 $_COOKIE

## 概述

`$_SERVER`、`$_SESSION` 和 `$_COOKIE` 是 PHP 的重要超级全局变量，分别用于获取服务器信息、管理会话状态和处理 Cookie 数据。

## $_SERVER - 服务器信息

### 基本概念

`$_SERVER` 包含服务器和执行环境的信息，如请求方法、URI、HTTP 头等。

### 常用键详解

`$_SERVER` 包含大量服务器和执行环境信息。以下按类别列出常用键及其值类型和说明：

#### 请求相关

| 键名                | 值类型   | 说明                                                                 | 示例值                    |
| :------------------ | :------- | :------------------------------------------------------------------- | :------------------------ |
| `REQUEST_METHOD`    | `string` | HTTP 请求方法                                                        | `"GET"`, `"POST"`, `"PUT"`, `"DELETE"` |
| `REQUEST_URI`        | `string` | 请求的 URI（包含查询字符串）                                         | `"/index.php?id=1"`       |
| `QUERY_STRING`      | `string` | 查询字符串（URL 中 `?` 后的部分）                                    | `"id=1&name=Alice"`       |
| `REQUEST_TIME`      | `int`    | 请求开始的时间戳（Unix 时间戳）                                      | `1704067200`              |
| `REQUEST_TIME_FLOAT` | `float` | 请求开始的时间戳（微秒精度）                                         | `1704067200.123456`       |
| `CONTENT_TYPE`      | `string` | 请求体的内容类型（POST/PUT 请求）                                    | `"application/json"`       |
| `CONTENT_LENGTH`    | `string` | 请求体的长度（字节数）                                               | `"1024"`                  |

#### 服务器信息

| 键名                | 值类型   | 说明                                                                 | 示例值                    |
| :------------------ | :------- | :------------------------------------------------------------------- | :------------------------ |
| `SERVER_NAME`       | `string` | 服务器主机名（来自 HTTP Host 头或服务器配置）                       | `"example.com"`           |
| `SERVER_PORT`       | `string` | 服务器端口号                                                         | `"80"`, `"443"`           |
| `SERVER_SOFTWARE`   | `string` | 服务器软件标识字符串                                                 | `"nginx/1.18.0"`          |
| `SERVER_PROTOCOL`   | `string` | 请求协议版本                                                         | `"HTTP/1.1"`              |
| `SERVER_ADDR`       | `string` | 服务器 IP 地址                                                       | `"192.168.1.100"`         |
| `SERVER_ADMIN`      | `string` | 服务器管理员邮箱（来自服务器配置）                                   | `"admin@example.com"`     |

#### 脚本路径相关

| 键名                | 值类型   | 说明                                                                 | 示例值                    |
| :------------------ | :------- | :------------------------------------------------------------------- | :------------------------ |
| `SCRIPT_NAME`       | `string` | 当前脚本的路径（相对于文档根目录）                                   | `"/index.php"`            |
| `SCRIPT_FILENAME`   | `string` | 当前脚本的完整文件系统路径                                           | `"/var/www/html/index.php"` |
| `PHP_SELF`          | `string` | 当前脚本的文件名（相对于文档根目录，可能包含路径信息）               | `"/index.php"`             |
| `DOCUMENT_ROOT`     | `string` | 文档根目录的完整路径                                                 | `"/var/www/html"`          |
| `PATH_INFO`         | `string` | 路径信息（URL 中脚本名后的部分，如果存在）                            | `"/user/profile"`          |
| `PATH_TRANSLATED`   | `string` | 文件系统路径（PATH_INFO 转换为文件系统路径）                          | `"/var/www/html/user/profile"` |

#### HTTP 请求头

| 键名                | 值类型   | 说明                                                                 | 示例值                    |
| :------------------ | :------- | :------------------------------------------------------------------- | :------------------------ |
| `HTTP_HOST`         | `string` | 主机名（来自 HTTP Host 头）                                          | `"example.com"`           |
| `HTTP_USER_AGENT`   | `string` | 用户代理字符串（浏览器标识）                                         | `"Mozilla/5.0..."`        |
| `HTTP_REFERER`      | `string` | 来源页面 URL（如果存在）                                             | `"https://example.com/page"` |
| `HTTP_ACCEPT`       | `string` | 客户端接受的内容类型                                                 | `"text/html,application/xhtml+xml"` |
| `HTTP_ACCEPT_LANGUAGE` | `string` | 客户端接受的语言                                                     | `"zh-CN,zh;q=0.9,en;q=0.8"` |
| `HTTP_ACCEPT_ENCODING` | `string` | 客户端接受的编码方式                                                 | `"gzip, deflate, br"`     |
| `HTTP_CONNECTION`   | `string` | 连接类型                                                             | `"keep-alive"`            |
| `HTTP_UPGRADE_INSECURE_REQUESTS` | `string` | 是否允许升级到 HTTPS                                                 | `"1"`                      |
| `HTTP_X_FORWARDED_FOR` | `string` | 代理转发的客户端 IP（可能包含多个 IP，逗号分隔）                     | `"192.168.1.1, 10.0.0.1"` |
| `HTTP_X_REAL_IP`    | `string` | 真实客户端 IP（由反向代理设置）                                      | `"192.168.1.1"`           |

#### 客户端信息

| 键名                | 值类型   | 说明                                                                 | 示例值                    |
| :------------------ | :------- | :------------------------------------------------------------------- | :------------------------ |
| `REMOTE_ADDR`       | `string` | 客户端 IP 地址（直接连接的 IP，可能是代理服务器）                    | `"192.168.1.1"`           |
| `REMOTE_PORT`       | `string` | 客户端端口号                                                         | `"54321"`                 |
| `REMOTE_HOST`       | `string` | 客户端主机名（如果可用，通常为空）                                   | `""` 或 `"client.example.com"` |

#### HTTPS 和安全相关

| 键名                | 值类型   | 说明                                                                 | 示例值                    |
| :------------------ | :------- | :------------------------------------------------------------------- | :------------------------ |
| `HTTPS`             | `string` | 如果通过 HTTPS 连接，值为 `"on"`，否则不存在或为空                  | `"on"` 或不存在           |
| `REQUEST_SCHEME`    | `string` | 请求协议方案                                                         | `"http"`, `"https"`       |
| `SERVER_SIGNATURE`  | `string` | 服务器签名（如果启用）                                               | `"Apache/2.4.41 Server..."` |

#### PHP 相关

| 键名                | 值类型   | 说明                                                                 | 示例值                    |
| :------------------ | :------- | :------------------------------------------------------------------- | :------------------------ |
| `GATEWAY_INTERFACE` | `string` | CGI 规范版本                                                         | `"CGI/1.1"`               |
| `REDIRECT_STATUS`   | `string` | 重定向状态码（如果通过重定向访问）                                   | `"200"`                    |

**注意事项**：
- 所有 HTTP 头键名都以 `HTTP_` 前缀开头，原始头名中的连字符（`-`）会被转换为下划线（`_`）
- 某些键可能不存在，取决于服务器配置和请求类型
- 某些值可能被客户端伪造（如 `HTTP_USER_AGENT`、`HTTP_REFERER`），不要完全信任
- `REMOTE_ADDR` 在反向代理环境中可能显示代理服务器 IP，而非真实客户端 IP

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

**重要特性**：
- **类型**：`$_SESSION` 是一个关联数组（`array<string, mixed>`）
- **作用域**：全局可用，无需 `global` 声明
- **存储位置**：服务器端（文件系统、数据库或内存）
- **生命周期**：从 `session_start()` 到脚本结束或 `session_destroy()`
- **数据持久化**：在脚本结束时自动保存到存储介质

**键的特点**：
- 键名必须是字符串
- 值可以是任何可序列化的 PHP 类型（字符串、整数、数组、对象等）
- 键名区分大小写
- 建议使用有意义的键名，避免冲突

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

**重要特性**：
- **类型**：`$_COOKIE` 是一个关联数组（`array<string, string>`）
- **作用域**：全局可用，无需 `global` 声明
- **数据来源**：来自客户端浏览器发送的 HTTP Cookie 头
- **值类型限制**：所有值都是字符串类型（即使设置时是其他类型，也会被转换为字符串）
- **自动 URL 解码**：值会自动进行 URL 解码

**键的特点**：
- 键名必须是字符串
- 值始终是字符串类型
- 键名区分大小写
- 只包含客户端发送的 Cookie，不包含服务器设置的 Cookie（除非客户端已接收并发送回来）

**常用场景**：
- 用户偏好设置（主题、语言等）
- 跟踪标识符（分析、广告等）
- 临时数据（购物车、表单数据等）
- **注意**：不要存储敏感信息（密码、令牌等），应使用 Session

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

**参数**：
- `$name`：Cookie 名称
- `$value`：可选，Cookie 值，默认为空字符串
- `$expires`：可选，过期时间（Unix 时间戳），默认为 0（会话结束时过期）
- `$path`：可选，Cookie 路径，默认为空字符串（当前目录）
- `$domain`：可选，Cookie 域名，默认为空字符串（当前域名）
- `$secure`：可选，是否仅通过 HTTPS 传输，默认为 `false`
- `$httponly`：可选，是否仅允许 HTTP 访问（防止 JavaScript 访问），默认为 `false`

**返回值**：成功返回 `true`，失败返回 `false`。

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
