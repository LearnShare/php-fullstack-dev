# 5.9.1 Session 基础

## 概述

Session 是服务器端会话管理的基础机制，用于在多个请求之间保持用户状态。理解 Session 的工作原理、掌握 Session 的使用方法、正确配置 Session 安全选项，对于实现用户登录、购物车、用户偏好等功能至关重要。本节详细介绍 Session 的概念、`session_start()` 函数的使用、`$_SESSION` 超全局变量的操作、会话配置、会话安全、会话销毁等内容，帮助零基础学员掌握 Session 会话管理技术。

Session 通过 Session ID 关联客户端和服务器端的会话数据。客户端通过 Cookie 或 URL 参数传递 Session ID，服务器根据 Session ID 查找对应的会话数据。理解这个机制对于正确使用 Session 至关重要。

**主要内容**：
- Session 概念（什么是 Session、Session 的作用、Session ID 的传递、服务器端存储）
- `session_start()` 函数的语法、参数和使用方法
- `$_SESSION` 超全局变量的使用（数据存储、读取、修改、删除）
- 会话数据操作（存储、读取、更新、删除）
- 会话配置（php.ini 配置、运行时配置、安全配置）
- 会话安全（Session 固定攻击防护、Session 劫持防护、安全配置）
- 会话销毁（`session_destroy()`、`session_unset()`、数据清理）
- 实际应用示例和最佳实践

## 特性

- **服务器端存储**：数据存储在服务器端，相对安全
- **跨请求保持**：在多个请求之间保持状态
- **自动管理**：PHP 自动管理 Session ID 和会话数据
- **灵活配置**：支持多种配置选项
- **安全机制**：提供安全配置选项

## Session 概念

### 什么是 Session

Session（会话）是在多个 HTTP 请求之间保持用户状态的机制。Session 数据存储在服务器端，通过 Session ID 关联客户端和服务器端的会话数据。

### Session 的作用

1. **用户认证**：保持用户登录状态
2. **数据存储**：存储临时数据（如购物车）
3. **用户偏好**：存储用户偏好设置
4. **状态保持**：在无状态 HTTP 协议中保持状态

### Session ID 的传递

**方式一：Cookie（默认）**
```
Set-Cookie: PHPSESSID=abc123; Path=/
```

**方式二：URL 参数**
```
https://example.com/page.php?PHPSESSID=abc123
```

**推荐**：使用 Cookie 传递 Session ID（更安全、更优雅）。

### 服务器端存储

**默认存储**：文件系统（`session.save_path` 指定的目录）

**存储格式**：
```
文件名：sess_{session_id}
内容：序列化的会话数据
```

**其他存储方式**：
- 数据库
- Redis
- Memcached

## session_start() 函数

### 语法

**语法**：`session_start(array $options = []): bool`

### 参数

- `$options`：可选，会话配置选项数组

### 返回值

成功返回 `true`，失败返回 `false`。

### 基本用法

**示例 1：基本启动**
```php
<?php
declare(strict_types=1);

session_start();
```

**示例 2：带配置启动**
```php
<?php
declare(strict_types=1);

session_start([
    'cookie_lifetime' => 3600,      // Cookie 生命周期（秒）
    'cookie_secure' => true,        // 仅 HTTPS
    'cookie_httponly' => true,      // 仅 HTTP 访问
    'cookie_samesite' => 'Strict',  // SameSite 属性
]);
```

### 调用时机

**重要**：`session_start()` 必须在任何输出之前调用。

**正确**：
```php
<?php
declare(strict_types=1);

session_start();
echo "Output";
```

**错误**：
```php
<?php
declare(strict_types=1);

echo "Output";  // 错误：在 session_start() 之前有输出
session_start();  // 会失败
```

**解决方案**：使用输出缓冲
```php
<?php
declare(strict_types=1);

ob_start();
echo "Output";
session_start();  // 可以成功
ob_end_flush();
```

### 会话 ID 生成

**自动生成**：`session_start()` 会自动生成 Session ID（如果不存在）。

**手动设置**：使用 `session_id()` 函数。

**示例**：
```php
<?php
declare(strict_types=1);

// 获取当前 Session ID
$sessionId = session_id();
echo "Session ID: {$sessionId}\n";

// 设置 Session ID（必须在 session_start() 之前）
session_id('custom-session-id');
session_start();
```

## $_SESSION 超全局变量

### 数据存储

**示例**：
```php
<?php
declare(strict_types=1);

session_start();

// 存储简单数据
$_SESSION['user_id'] = 123;
$_SESSION['username'] = 'john_doe';

// 存储数组
$_SESSION['preferences'] = [
    'theme' => 'dark',
    'language' => 'zh-CN',
];

// 存储对象（需要类可序列化）
$_SESSION['user'] = new stdClass();
$_SESSION['user']->name = 'John';
```

### 数据读取

**示例**：
```php
<?php
declare(strict_types=1);

session_start();

// 读取数据
$userId = $_SESSION['user_id'] ?? null;
$username = $_SESSION['username'] ?? 'guest';

// 安全读取（提供默认值）
$theme = $_SESSION['preferences']['theme'] ?? 'light';
```

### 数据修改

**示例**：
```php
<?php
declare(strict_types=1);

session_start();

// 修改数据
$_SESSION['username'] = 'new_username';

// 修改数组数据
$_SESSION['preferences']['theme'] = 'light';
```

### 数据删除

**示例**：
```php
<?php
declare(strict_types=1);

session_start();

// 删除单个键
unset($_SESSION['username']);

// 清空所有会话数据
$_SESSION = [];

// 销毁会话
session_destroy();
```

## 会话数据操作

### 存储数据

**示例**：
```php
<?php
declare(strict_types=1);

session_start();

// 存储用户信息
$_SESSION['user'] = [
    'id' => 123,
    'name' => 'John Doe',
    'email' => 'john@example.com',
];

// 存储登录时间
$_SESSION['login_time'] = time();

// 存储访问次数
if (!isset($_SESSION['visit_count'])) {
    $_SESSION['visit_count'] = 0;
}
$_SESSION['visit_count']++;
```

### 读取数据

**示例**：
```php
<?php
declare(strict_types=1);

session_start();

// 检查数据是否存在
if (isset($_SESSION['user'])) {
    $user = $_SESSION['user'];
    echo "用户: {$user['name']}\n";
}

// 使用默认值
$theme = $_SESSION['theme'] ?? 'light';
```

### 更新数据

**示例**：
```php
<?php
declare(strict_types=1);

session_start();

// 更新用户信息
if (isset($_SESSION['user'])) {
    $_SESSION['user']['name'] = 'Jane Doe';
    $_SESSION['user']['updated_at'] = time();
}
```

### 删除数据

**示例**：
```php
<?php
declare(strict_types=1);

session_start();

// 删除特定数据
unset($_SESSION['temp_data']);

// 清空所有数据
$_SESSION = [];
```

## 会话配置

### php.ini 配置

**主要配置项**：
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

; SameSite 属性
session.cookie_samesite = Strict

; 会话垃圾回收概率
session.gc_probability = 1
session.gc_divisor = 1000

; 会话过期时间（秒）
session.gc_maxlifetime = 1440

; 使用严格模式（防止 Session 固定攻击）
session.use_strict_mode = 1
```

### 运行时配置

**示例**：
```php
<?php
declare(strict_types=1);

// 配置会话
ini_set('session.cookie_lifetime', 3600);      // 1 小时
ini_set('session.cookie_secure', 1);            // 仅 HTTPS
ini_set('session.cookie_httponly', 1);          // 仅 HTTP 访问
ini_set('session.cookie_samesite', 'Strict');   // SameSite 属性
ini_set('session.use_strict_mode', 1);          // 严格模式

session_start();
```

### 安全配置

**示例**：
```php
<?php
declare(strict_types=1);

// 安全配置
session_start([
    'cookie_lifetime' => 0,                      // 浏览器关闭时过期
    'cookie_secure' => true,                    // 仅 HTTPS
    'cookie_httponly' => true,                  // 防止 JavaScript 访问
    'cookie_samesite' => 'Strict',              // 防止 CSRF
    'use_strict_mode' => true,                   // 防止 Session 固定攻击
]);

// 防止 Session 固定攻击
if (!isset($_SESSION['created'])) {
    session_regenerate_id(true);  // 重新生成 Session ID
    $_SESSION['created'] = time();
}
```

## 会话安全

### Session 固定攻击防护

**问题**：攻击者获取用户的 Session ID，使用该 ID 访问用户账户。

**防护方法**：
1. **使用严格模式**：`session.use_strict_mode = 1`
2. **重新生成 Session ID**：登录后重新生成 Session ID
3. **验证 Session ID**：验证 Session ID 的有效性

**示例**：
```php
<?php
declare(strict_types=1);

session_start([
    'use_strict_mode' => true,
]);

// 登录后重新生成 Session ID
function login(int $userId): void
{
    session_regenerate_id(true);  // 重新生成并删除旧 Session ID
    
    $_SESSION['user_id'] = $userId;
    $_SESSION['authenticated'] = true;
    $_SESSION['login_time'] = time();
}
```

### Session 劫持防护

**问题**：攻击者窃取用户的 Session ID，冒充用户。

**防护方法**：
1. **使用 HTTPS**：加密传输 Session ID
2. **HttpOnly Cookie**：防止 JavaScript 访问 Session ID
3. **验证用户信息**：验证 IP 地址、User-Agent 等

**示例**：
```php
<?php
declare(strict_types=1);

session_start();

// 验证 Session 是否被劫持
function validateSession(): bool
{
    if (!isset($_SESSION['ip_address']) || !isset($_SESSION['user_agent'])) {
        $_SESSION['ip_address'] = $_SERVER['REMOTE_ADDR'] ?? '';
        $_SESSION['user_agent'] = $_SERVER['HTTP_USER_AGENT'] ?? '';
        return true;
    }
    
    $currentIp = $_SERVER['REMOTE_ADDR'] ?? '';
    $currentUserAgent = $_SERVER['HTTP_USER_AGENT'] ?? '';
    
    if ($_SESSION['ip_address'] !== $currentIp || 
        $_SESSION['user_agent'] !== $currentUserAgent) {
        // Session 可能被劫持，销毁会话
        session_destroy();
        return false;
    }
    
    return true;
}
```

### 安全配置

**完整安全配置**：
```php
<?php
declare(strict_types=1);

// 安全配置
ini_set('session.cookie_secure', 1);        // 仅 HTTPS
ini_set('session.cookie_httponly', 1);      // 防止 JavaScript 访问
ini_set('session.cookie_samesite', 'Strict'); // 防止 CSRF
ini_set('session.use_strict_mode', 1);      // 严格模式
ini_set('session.cookie_lifetime', 0);      // 浏览器关闭时过期

session_start();

// 防止 Session 固定攻击
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

## 会话销毁

### session_destroy() 函数

**语法**：`session_destroy(): bool`

**作用**：销毁当前会话，删除会话数据文件。

**注意**：不会删除 `$_SESSION` 数组，需要手动清空。

**示例**：
```php
<?php
declare(strict_types=1);

session_start();

// 清空会话数据
$_SESSION = [];

// 销毁会话
session_destroy();
```

### session_unset() 函数

**语法**：`session_unset(): bool`

**作用**：清空 `$_SESSION` 数组中的所有数据。

**注意**：不会销毁会话，Session ID 仍然有效。

**示例**：
```php
<?php
declare(strict_types=1);

session_start();

// 清空会话数据
session_unset();

// 会话仍然存在，可以继续使用
$_SESSION['new_data'] = 'value';
```

### 完整销毁流程

**示例**：
```php
<?php
declare(strict_types=1);

function destroySession(): void
{
    session_start();
    
    // 1. 清空会话数据
    $_SESSION = [];
    
    // 2. 删除 Session Cookie
    if (isset($_COOKIE[session_name()])) {
        setcookie(session_name(), '', time() - 3600, '/');
    }
    
    // 3. 销毁会话
    session_destroy();
}

// 使用
destroySession();
```

## 使用场景

### 用户登录状态

- 保持用户登录状态
- 存储用户信息
- 管理用户会话

### 购物车数据

- 存储购物车商品
- 临时数据存储
- 跨请求保持数据

### 临时数据存储

- 表单数据临时存储
- 临时计算结果
- 用户操作记录

### 用户偏好设置

- 主题设置
- 语言设置
- 显示偏好

## 注意事项

### session_start() 必须在输出前调用

- **问题**：在输出后调用 `session_start()` 会失败
- **解决**：使用输出缓冲或确保在输出前调用
- **检查**：使用 `headers_sent()` 检查

### 会话安全配置

- **使用 HTTPS**：加密传输 Session ID
- **HttpOnly Cookie**：防止 JavaScript 访问
- **SameSite 属性**：防止 CSRF 攻击
- **严格模式**：防止 Session 固定攻击

### 会话数据大小

- **限制大小**：不要存储过多数据
- **性能影响**：大数据的 Session 影响性能
- **存储位置**：考虑使用外部存储（Redis、数据库）

### 会话过期处理

- **设置过期时间**：设置合理的过期时间
- **检查过期**：定期检查会话是否过期
- **自动清理**：使用垃圾回收机制

## 常见问题

### 如何启动 Session？

使用 `session_start()` 函数：

```php
<?php
declare(strict_types=1);

session_start();
```

### Session 数据存储在哪里？

默认存储在文件系统中（`session.save_path` 指定的目录），也可以存储在数据库、Redis 等。

### 如何销毁 Session？

1. 清空 `$_SESSION` 数组
2. 调用 `session_destroy()`
3. 删除 Session Cookie

### Session 安全如何配置？

1. 使用 HTTPS（`session.cookie_secure = 1`）
2. 使用 HttpOnly（`session.cookie_httponly = 1`）
3. 使用 SameSite（`session.cookie_samesite = Strict`）
4. 使用严格模式（`session.use_strict_mode = 1`）

## 最佳实践

### 在输出前启动 Session

- 确保在输出前调用 `session_start()`
- 使用 `headers_sent()` 检查
- 使用输出缓冲避免问题

### 配置安全的 Session Cookie

- 使用 HTTPS 传输
- 使用 HttpOnly 防止 JavaScript 访问
- 使用 SameSite 防止 CSRF
- 设置合理的过期时间

### 及时销毁不需要的 Session

- 用户退出时销毁会话
- 会话过期时自动清理
- 定期清理过期会话

### 限制 Session 数据大小

- 不要存储过多数据
- 考虑使用外部存储
- 定期清理不需要的数据

## 相关章节

- **[5.9.2 Cookie 管理](section-02-cookie-management.md)**：了解 Cookie 管理的详细内容
- **[5.3.2 $_SERVER、$_SESSION 与 $_COOKIE](../chapter-03-superglobals/section-02-server-session-cookie.md)**：了解 $_SESSION 的基础使用
- **[5.10.1 认证基础（AuthN）](../chapter-10-auth/section-01-authentication.md)**：了解 Session 在认证中的应用

## 练习任务

1. **实现 Session 管理类**
   - 创建 Session 管理类
   - 封装 Session 操作
   - 提供安全配置

2. **实现 Session 安全防护**
   - 防止 Session 固定攻击
   - 防止 Session 劫持
   - 实现会话过期检查

3. **实现用户登录系统**
   - 使用 Session 保持登录状态
   - 实现登录和退出功能
   - 实现会话管理

4. **实现购物车功能**
   - 使用 Session 存储购物车数据
   - 实现购物车操作
   - 管理购物车数据

5. **实现完整的 Session 管理系统**
   - Session 配置和管理
   - 安全防护机制
   - 会话数据操作
