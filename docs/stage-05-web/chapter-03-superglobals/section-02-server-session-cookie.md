# 5.3.2 $_SERVER、$_SESSION 与 $_COOKIE

## 概述

`$_SERVER`、`$_SESSION` 和 `$_COOKIE` 是 PHP 中重要的超全局变量，分别用于获取服务器和请求信息、管理会话数据、处理 Cookie 数据。理解这些变量的使用方法和应用场景，对于构建功能完整的 Web 应用至关重要。本节详细介绍这些超全局变量的使用方法、会话管理、Cookie 管理、安全注意事项等内容，帮助零基础学员掌握服务器信息和状态管理。

在 Web 开发中，我们经常需要获取服务器信息（如请求 URI、客户端 IP、请求方法等）、管理用户会话（如登录状态、用户偏好等）、处理 Cookie 数据（如记住登录、用户设置等）。PHP 提供了这些超全局变量来简化这些操作。

**主要内容**：
- `$_SERVER` 超全局变量的常用键值和使用方法
- `$_SESSION` 超全局变量的使用方法和会话管理
- `$_COOKIE` 超全局变量的使用方法和 Cookie 管理
- `session_start()` 函数的使用
- `setcookie()` 函数的使用和参数
- 会话和 Cookie 的安全配置
- 实际应用示例和最佳实践

## 特性

- **服务器信息**：`$_SERVER` 提供丰富的服务器和请求信息
- **会话管理**：`$_SESSION` 提供跨请求的状态保持
- **Cookie 处理**：`$_COOKIE` 提供客户端数据访问
- **自动填充**：PHP 自动填充这些变量
- **安全性**：需要正确配置以确保安全

## $_SERVER 超全局变量

### 概述

`$_SERVER` 是一个包含服务器和请求信息的关联数组，提供了关于当前请求的详细信息。

### 常用键值

#### 请求信息

| 键名 | 说明 | 示例 |
|:-----|:-----|:-----|
| `REQUEST_METHOD` | HTTP 请求方法 | `GET`、`POST`、`PUT`、`DELETE` |
| `REQUEST_URI` | 请求的 URI | `/api/users?id=123` |
| `QUERY_STRING` | 查询字符串 | `id=123&name=John` |
| `SCRIPT_NAME` | 当前脚本路径 | `/index.php` |
| `PHP_SELF` | 当前脚本文件名 | `/index.php` |
| `PATH_INFO` | 路径信息 | `/users/123` |

#### 服务器信息

| 键名 | 说明 | 示例 |
|:-----|:-----|:-----|
| `SERVER_NAME` | 服务器主机名 | `example.com` |
| `SERVER_PORT` | 服务器端口 | `80`、`443` |
| `SERVER_PROTOCOL` | 协议版本 | `HTTP/1.1` |
| `SERVER_SOFTWARE` | 服务器软件 | `nginx/1.18.0` |

#### 客户端信息

| 键名 | 说明 | 示例 |
|:-----|:-----|:-----|
| `REMOTE_ADDR` | 客户端 IP 地址 | `192.168.1.100` |
| `HTTP_USER_AGENT` | 用户代理（浏览器信息） | `Mozilla/5.0...` |
| `HTTP_REFERER` | 来源页面 URL | `https://example.com/page` |
| `HTTP_ACCEPT` | 接受的内容类型 | `text/html,application/json` |

#### HTTP 头信息

所有 HTTP 请求头都以 `HTTP_` 前缀存储在 `$_SERVER` 中，键名转换为大写，连字符转换为下划线。

| HTTP 头 | $_SERVER 键名 | 示例 |
|:--------|:--------------|:-----|
| `User-Agent` | `HTTP_USER_AGENT` | `Mozilla/5.0...` |
| `Accept` | `HTTP_ACCEPT` | `text/html,application/json` |
| `Content-Type` | `HTTP_CONTENT_TYPE` | `application/json` |
| `Authorization` | `HTTP_AUTHORIZATION` | `Bearer token123` |

### 基本用法

**示例 1：获取请求方法**
```php
<?php
declare(strict_types=1);

$method = $_SERVER['REQUEST_METHOD'] ?? 'GET';

switch ($method) {
    case 'GET':
        echo "处理 GET 请求\n";
        break;
    case 'POST':
        echo "处理 POST 请求\n";
        break;
    default:
        echo "处理其他请求\n";
}
```

**示例 2：获取请求 URI**
```php
<?php
declare(strict_types=1);

$uri = $_SERVER['REQUEST_URI'] ?? '/';
$queryString = $_SERVER['QUERY_STRING'] ?? '';

echo "URI: {$uri}\n";
echo "查询字符串: {$queryString}\n";
```

**示例 3：获取客户端 IP**
```php
<?php
declare(strict_types=1);

function getClientIp(): string
{
    $ipKeys = [
        'HTTP_CLIENT_IP',
        'HTTP_X_FORWARDED_FOR',
        'HTTP_X_FORWARDED',
        'HTTP_X_CLUSTER_CLIENT_IP',
        'HTTP_FORWARDED_FOR',
        'HTTP_FORWARDED',
        'REMOTE_ADDR',
    ];

    foreach ($ipKeys as $key) {
        if (!empty($_SERVER[$key])) {
            $ips = explode(',', $_SERVER[$key]);
            $ip = trim($ips[0]);
            if (filter_var($ip, FILTER_VALIDATE_IP, FILTER_FLAG_NO_PRIV_RANGE | FILTER_FLAG_NO_RES_RANGE)) {
                return $ip;
            }
        }
    }

    return $_SERVER['REMOTE_ADDR'] ?? '0.0.0.0';
}

$clientIp = getClientIp();
echo "客户端 IP: {$clientIp}\n";
```

**示例 4：获取所有请求头**
```php
<?php
declare(strict_types=1);

function getAllHeaders(): array
{
    $headers = [];
    
    foreach ($_SERVER as $key => $value) {
        if (str_starts_with($key, 'HTTP_')) {
            $headerName = str_replace('_', '-', substr($key, 5));
            $headerName = ucwords(strtolower($headerName), '-');
            $headers[$headerName] = $value;
        }
    }
    
    return $headers;
}

$headers = getAllHeaders();
print_r($headers);
```

## $_SESSION 超全局变量

### 概述

`$_SESSION` 用于在多个请求之间保持用户状态，数据存储在服务器端，通过 Session ID 关联。

### session_start() 函数

**语法**：`session_start(array $options = []): bool`

**参数**：
- `$options`：可选，会话配置选项数组

**返回值**：成功返回 `true`，失败返回 `false`。

**作用**：启动会话，如果会话已存在则恢复会话数据。

### 基本用法

**示例 1：启动会话并存储数据**
```php
<?php
declare(strict_types=1);

// 启动会话
session_start();

// 存储数据
$_SESSION['user_id'] = 123;
$_SESSION['username'] = 'john_doe';
$_SESSION['login_time'] = time();

echo "会话数据已存储\n";
```

**示例 2：读取会话数据**
```php
<?php
declare(strict_types=1);

// 启动会话
session_start();

// 读取数据
$userId = $_SESSION['user_id'] ?? null;
$username = $_SESSION['username'] ?? '游客';

echo "用户 ID: {$userId}\n";
echo "用户名: {$username}\n";
```

**示例 3：更新会话数据**
```php
<?php
declare(strict_types=1);

session_start();

// 更新数据
if (isset($_SESSION['visit_count'])) {
    $_SESSION['visit_count']++;
} else {
    $_SESSION['visit_count'] = 1;
}

echo "访问次数: {$_SESSION['visit_count']}\n";
```

**示例 4：删除会话数据**
```php
<?php
declare(strict_types=1);

session_start();

// 删除单个键
unset($_SESSION['user_id']);

// 删除所有会话数据
$_SESSION = [];

// 销毁会话
session_destroy();
```

### 会话配置

**php.ini 配置**：
```ini
; 会话存储方式
session.save_handler = files

; 会话存储路径
session.save_path = "/var/lib/php/sessions"

; Session ID Cookie 名称
session.name = PHPSESSID

; Session ID Cookie 生命周期（秒）
session.cookie_lifetime = 0

; Session ID Cookie 路径
session.cookie_path = /

; Session ID Cookie 域名
session.cookie_domain = 

; 仅通过 HTTPS 传输 Cookie
session.cookie_secure = 1

; 仅通过 HTTP 访问 Cookie（防止 JavaScript 访问）
session.cookie_httponly = 1

; 会话垃圾回收概率
session.gc_probability = 1
session.gc_divisor = 1000

; 会话过期时间（秒）
session.gc_maxlifetime = 1440
```

**代码中配置**：
```php
<?php
declare(strict_types=1);

// 配置会话
ini_set('session.cookie_lifetime', 3600);  // 1 小时
ini_set('session.cookie_secure', 1);        // 仅 HTTPS
ini_set('session.cookie_httponly', 1);      // 仅 HTTP 访问
ini_set('session.cookie_samesite', 'Strict'); // SameSite 属性

session_start();
```

### 会话安全

**安全配置**：
```php
<?php
declare(strict_types=1);

// 安全配置
ini_set('session.cookie_secure', 1);        // 仅 HTTPS
ini_set('session.cookie_httponly', 1);      // 防止 JavaScript 访问
ini_set('session.cookie_samesite', 'Strict'); // 防止 CSRF
ini_set('session.use_strict_mode', 1);      // 严格模式
ini_set('session.cookie_lifetime', 0);       // 浏览器关闭时过期

session_start();

// 防止会话固定攻击
if (!isset($_SESSION['created'])) {
    session_regenerate_id(true);  // 重新生成 Session ID
    $_SESSION['created'] = time();
}
```

## $_COOKIE 超全局变量

### 概述

`$_COOKIE` 用于获取客户端发送的 Cookie 数据。Cookie 是存储在客户端的小型数据，每次请求会自动发送到服务器。

### 基本用法

**示例 1：读取 Cookie**
```php
<?php
declare(strict_types=1);

// 读取 Cookie
$username = $_COOKIE['username'] ?? '游客';
$theme = $_COOKIE['theme'] ?? 'light';

echo "用户名: {$username}\n";
echo "主题: {$theme}\n";
```

**示例 2：检查 Cookie 是否存在**
```php
<?php
declare(strict_types=1);

if (isset($_COOKIE['user_preference'])) {
    $preference = $_COOKIE['user_preference'];
    echo "用户偏好: {$preference}\n";
} else {
    echo "用户偏好未设置\n";
}
```

### setcookie() 函数

**语法**：`setcookie(string $name, string $value = "", int $expires_or_options = 0, string $path = "", string $domain = "", bool $secure = false, bool $httponly = false): bool`

**参数**：
- `$name`：Cookie 名称
- `$value`：Cookie 值（可选）
- `$expires_or_options`：过期时间（Unix 时间戳）或选项数组（PHP 7.3+）
- `$path`：Cookie 路径（可选）
- `$domain`：Cookie 域名（可选）
- `$secure`：是否仅通过 HTTPS 传输（可选）
- `$httponly`：是否仅通过 HTTP 访问（可选）

**返回值**：成功返回 `true`，失败返回 `false`。

**注意**：`setcookie()` 必须在任何输出之前调用。

### 设置 Cookie

**示例 1：基本 Cookie**
```php
<?php
declare(strict_types=1);

// 设置 Cookie（浏览器关闭时过期）
setcookie('username', 'john_doe');

// 设置带过期时间的 Cookie（1 小时后过期）
setcookie('theme', 'dark', time() + 3600);

// 设置带路径和域名的 Cookie
setcookie('language', 'zh-CN', time() + 86400, '/', 'example.com');
```

**示例 2：安全 Cookie**
```php
<?php
declare(strict_types=1);

// 安全 Cookie（仅 HTTPS，仅 HTTP 访问）
setcookie(
    'session_id',
    'abc123',
    time() + 3600,
    '/',
    'example.com',
    true,   // 仅 HTTPS
    true    // 仅 HTTP 访问（防止 JavaScript 访问）
);
```

**示例 3：使用选项数组（PHP 7.3+）**
```php
<?php
declare(strict_types=1);

setcookie('preference', 'value', [
    'expires' => time() + 3600,
    'path' => '/',
    'domain' => 'example.com',
    'secure' => true,
    'httponly' => true,
    'samesite' => 'Strict',
]);
```

### 删除 Cookie

**示例**：
```php
<?php
declare(strict_types=1);

// 删除 Cookie（设置过期时间为过去的时间）
setcookie('username', '', time() - 3600);
setcookie('theme', '', time() - 3600, '/');
```

## 会话管理

### 会话生命周期

1. **启动会话**：使用 `session_start()` 启动会话
2. **存储数据**：在 `$_SESSION` 中存储数据
3. **读取数据**：从 `$_SESSION` 中读取数据
4. **销毁会话**：使用 `session_destroy()` 销毁会话

### 会话存储

**默认存储**（文件）：
- 存储在 `session.save_path` 指定的目录
- 文件名格式：`sess_{session_id}`
- 每个会话一个文件

**其他存储方式**：
- **数据库**：使用自定义 Session Handler
- **Redis**：使用 Redis Session Handler
- **Memcached**：使用 Memcached Session Handler

### 会话安全

**安全措施**：
1. **HTTPS**：使用 HTTPS 传输 Session ID
2. **HttpOnly**：防止 JavaScript 访问 Session ID Cookie
3. **SameSite**：防止 CSRF 攻击
4. **Session 固定防护**：使用 `session_regenerate_id()` 重新生成 Session ID
5. **过期时间**：设置合理的会话过期时间

**示例**：
```php
<?php
declare(strict_types=1);

// 安全配置
ini_set('session.cookie_secure', 1);
ini_set('session.cookie_httponly', 1);
ini_set('session.cookie_samesite', 'Strict');
ini_set('session.use_strict_mode', 1);

session_start();

// 防止会话固定攻击
if (!isset($_SESSION['created'])) {
    session_regenerate_id(true);
    $_SESSION['created'] = time();
}

// 检查会话是否过期（30 分钟）
if (isset($_SESSION['last_activity']) && (time() - $_SESSION['last_activity'] > 1800)) {
    session_destroy();
    session_start();
}

$_SESSION['last_activity'] = time();
```

## Cookie 管理

### Cookie 属性

| 属性 | 说明 | 示例 |
|:-----|:-----|:-----|
| `name` | Cookie 名称 | `username` |
| `value` | Cookie 值 | `john_doe` |
| `expires` | 过期时间（Unix 时间戳） | `time() + 3600` |
| `path` | Cookie 路径 | `/`、`/admin` |
| `domain` | Cookie 域名 | `example.com`、`.example.com` |
| `secure` | 仅 HTTPS 传输 | `true`、`false` |
| `httponly` | 仅 HTTP 访问 | `true`、`false` |
| `samesite` | SameSite 属性 | `Strict`、`Lax`、`None` |

### Cookie 安全

**安全配置**：
```php
<?php
declare(strict_types=1);

// 安全 Cookie
setcookie(
    'user_token',
    'token123',
    [
        'expires' => time() + 3600,
        'path' => '/',
        'domain' => 'example.com',
        'secure' => true,      // 仅 HTTPS
        'httponly' => true,    // 防止 JavaScript 访问
        'samesite' => 'Strict', // 防止 CSRF
    ]
);
```

### Cookie 和 Session 的区别

| 特性 | Cookie | Session |
|:-----|:-------|:--------|
| 存储位置 | 客户端（浏览器） | 服务器端 |
| 数据大小 | 受限制（通常 4KB） | 无限制（受服务器配置限制） |
| 安全性 | 较低（可被客户端修改） | 较高（服务器端存储） |
| 性能 | 每次请求自动发送 | 需要服务器查找 |
| 过期时间 | 可设置 | 可设置 |
| 适用场景 | 用户偏好、记住登录 | 敏感数据、登录状态 |

## 使用场景

### 获取服务器和请求信息

- 获取请求方法、URI、查询字符串
- 获取客户端 IP、用户代理
- 获取 HTTP 头信息

### 用户会话管理

- 用户登录状态
- 用户偏好设置
- 购物车数据
- 临时数据存储

### 用户偏好设置

- 语言设置
- 主题设置
- 显示偏好
- 记住登录

### 状态保持

- 跨请求保持状态
- 用户认证信息
- 临时数据存储

## 注意事项

### 会话安全配置

- 使用 HTTPS 传输 Session ID
- 设置 HttpOnly 防止 JavaScript 访问
- 设置 SameSite 防止 CSRF 攻击
- 使用严格模式防止会话固定攻击

### Cookie 安全选项

- 使用 Secure 选项（仅 HTTPS）
- 使用 HttpOnly 选项（防止 JavaScript 访问）
- 使用 SameSite 选项（防止 CSRF）
- 设置合理的过期时间

### 数据验证

- 验证所有用户输入
- 验证 Cookie 数据
- 验证 Session 数据
- 防止数据篡改

### 敏感信息处理

- 不要在 Cookie 中存储敏感信息
- 不要在 Session 中存储过多数据
- 及时清理过期数据
- 使用加密存储敏感数据

### setcookie() 调用时机

- 必须在任何输出之前调用
- 必须在 `header()` 之前调用（如果可能）
- 使用输出缓冲可以避免问题

## 常见问题

### $_SERVER 包含哪些信息？

`$_SERVER` 包含服务器和请求的详细信息，包括：
- 请求信息（方法、URI、查询字符串等）
- 服务器信息（主机名、端口、协议等）
- 客户端信息（IP、用户代理等）
- HTTP 头信息（所有请求头）

### 如何启动和管理会话？

1. **启动会话**：使用 `session_start()` 启动会话
2. **存储数据**：在 `$_SESSION` 中存储数据
3. **读取数据**：从 `$_SESSION` 中读取数据
4. **更新数据**：直接修改 `$_SESSION` 中的数据
5. **删除数据**：使用 `unset()` 删除单个键，或使用 `session_destroy()` 销毁会话

### Cookie 和 Session 的区别？

| 特性 | Cookie | Session |
|:-----|:-------|:--------|
| 存储位置 | 客户端 | 服务器端 |
| 数据大小 | 受限（4KB） | 无限制 |
| 安全性 | 较低 | 较高 |
| 性能 | 每次请求发送 | 需要服务器查找 |

### 如何安全地使用 Cookie？

1. **使用 Secure 选项**：仅通过 HTTPS 传输
2. **使用 HttpOnly 选项**：防止 JavaScript 访问
3. **使用 SameSite 选项**：防止 CSRF 攻击
4. **设置合理的过期时间**：避免 Cookie 长期有效
5. **不要存储敏感信息**：敏感信息应存储在服务器端

## 最佳实践

### 使用 $_SERVER 获取请求信息

- 使用 `$_SERVER['REQUEST_METHOD']` 判断请求方法
- 使用 `$_SERVER['REQUEST_URI']` 获取请求 URI
- 使用 `$_SERVER['REMOTE_ADDR']` 获取客户端 IP（注意代理）

### 合理使用会话存储

- 不要存储过多数据
- 及时清理过期数据
- 使用外部存储（Redis、数据库）提高性能

### 设置安全的 Cookie 选项

- 使用 Secure 选项（仅 HTTPS）
- 使用 HttpOnly 选项（防止 JavaScript 访问）
- 使用 SameSite 选项（防止 CSRF）
- 设置合理的过期时间

### 及时销毁会话数据

- 用户退出时销毁会话
- 会话过期时自动清理
- 定期清理过期会话

### 安全配置

- 配置会话安全选项
- 配置 Cookie 安全选项
- 使用 HTTPS 传输敏感数据
- 防止会话固定攻击

## 相关章节

- **[5.3.1 $_GET、$_POST 与 $_REQUEST](section-01-get-post-request.md)**：了解其他超全局变量的使用方法
- **[5.9 会话与状态管理](../chapter-09-session/readme.md)**：了解会话管理的详细内容
- **[5.1.1 HTTP 请求响应流程](../chapter-01-request-response/section-01-http-flow.md)**：了解 HTTP 请求的结构

## 练习任务

1. **获取服务器信息**
   - 创建一个页面，显示所有 `$_SERVER` 信息
   - 提取常用的服务器信息（IP、URI、方法等）
   - 实现获取客户端 IP 的函数

2. **实现会话管理**
   - 创建登录页面，使用 Session 存储登录状态
   - 创建受保护页面，检查 Session 验证登录
   - 实现退出功能，销毁 Session

3. **实现 Cookie 管理**
   - 创建用户偏好设置页面
   - 使用 Cookie 存储用户偏好（语言、主题等）
   - 实现记住登录功能

4. **实现安全的会话和 Cookie**
   - 配置安全的 Session 选项
   - 配置安全的 Cookie 选项
   - 实现会话固定攻击防护

5. **实现会话和 Cookie 的组合使用**
   - 使用 Session 存储登录状态
   - 使用 Cookie 存储用户偏好
   - 实现"记住我"功能
