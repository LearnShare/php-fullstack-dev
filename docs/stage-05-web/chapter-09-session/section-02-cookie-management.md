# 5.9.2 Cookie 管理

## 概述

Cookie 是客户端状态存储的机制，用于在客户端（浏览器）存储小型数据。理解 Cookie 的工作原理、掌握 Cookie 的设置和读取方法、正确配置 Cookie 安全选项，对于实现"记住我"功能、用户偏好设置、跟踪用户行为等功能至关重要。本节详细介绍 Cookie 的概念、`setcookie()` 函数的使用、`$_COOKIE` 超全局变量的操作、Cookie 属性、安全选项等内容，帮助零基础学员掌握 Cookie 管理技术。

Cookie 是存储在客户端的小型数据，每次请求会自动发送到服务器。Cookie 有大小限制（通常 4KB），有过期时间，可以设置作用域（域名、路径）。理解这些特性对于正确使用 Cookie 至关重要。

**主要内容**：
- Cookie 概念（什么是 Cookie、Cookie 的作用、Cookie 的存储位置、Cookie 的限制）
- `setcookie()` 函数的语法、参数和使用方法
- `$_COOKIE` 超全局变量的使用（读取 Cookie、检查 Cookie 是否存在）
- Cookie 属性（expires 过期时间、path 路径、domain 域名、secure 安全传输、httponly HTTP 访问、samesite SameSite 属性）
- Cookie 读取和删除
- 安全选项（Secure 标志、HttpOnly 标志、SameSite 属性、敏感数据处理）
- 实际应用示例和最佳实践

## 特性

- **客户端存储**：数据存储在客户端（浏览器）
- **自动发送**：每次请求自动发送到服务器
- **大小限制**：通常限制为 4KB
- **过期控制**：可以设置过期时间
- **作用域控制**：可以设置域名和路径

## Cookie 概念

### 什么是 Cookie

Cookie 是服务器发送到客户端并存储在客户端的小型数据。客户端在后续请求中会自动将 Cookie 发送回服务器。

### Cookie 的作用

1. **用户偏好**：存储用户偏好设置（主题、语言等）
2. **记住登录**：实现"记住我"功能
3. **跟踪用户**：跟踪用户行为和会话
4. **临时数据**：存储临时数据

### Cookie 的存储位置

**浏览器存储**：Cookie 存储在浏览器的 Cookie 存储中。

**存储格式**：
```
名称=值; 过期时间; 路径; 域名; Secure; HttpOnly; SameSite
```

### Cookie 的限制

1. **大小限制**：通常限制为 4KB
2. **数量限制**：每个域名通常限制为 20-50 个 Cookie
3. **域名限制**：Cookie 只能被设置它的域名访问
4. **路径限制**：Cookie 只能被设置它的路径及其子路径访问

## setcookie() 函数

### 语法

**语法**：`setcookie(string $name, string $value = "", int $expires_or_options = 0, string $path = "", string $domain = "", bool $secure = false, bool $httponly = false): bool`

**PHP 7.3+ 语法**：`setcookie(string $name, string $value = "", array $options = []): bool`

### 参数

**传统参数**：
- `$name`：Cookie 名称
- `$value`：Cookie 值（可选）
- `$expires_or_options`：过期时间（Unix 时间戳）或选项数组（PHP 7.3+）
- `$path`：Cookie 路径（可选）
- `$domain`：Cookie 域名（可选）
- `$secure`：是否仅通过 HTTPS 传输（可选）
- `$httponly`：是否仅通过 HTTP 访问（可选）

**PHP 7.3+ 选项数组**：
- `expires`：过期时间（Unix 时间戳）
- `path`：Cookie 路径
- `domain`：Cookie 域名
- `secure`：是否仅通过 HTTPS 传输
- `httponly`：是否仅通过 HTTP 访问
- `samesite`：SameSite 属性（`Strict`、`Lax`、`None`）

### 返回值

成功返回 `true`，失败返回 `false`。

### 基本用法

**示例 1：基本 Cookie**
```php
<?php
declare(strict_types=1);

// 设置 Cookie（浏览器关闭时过期）
setcookie('username', 'john_doe');
```

**示例 2：带过期时间的 Cookie**
```php
<?php
declare(strict_types=1);

// 设置 Cookie（1 小时后过期）
setcookie('theme', 'dark', time() + 3600);
```

**示例 3：完整参数 Cookie**
```php
<?php
declare(strict_types=1);

// 设置 Cookie（1 小时后过期，路径 /，域名 example.com，仅 HTTPS，仅 HTTP 访问）
setcookie(
    'session_id',
    'abc123',
    time() + 3600,
    '/',
    'example.com',
    true,   // 仅 HTTPS
    true    // 仅 HTTP 访问
);
```

**示例 4：使用选项数组（PHP 7.3+）**
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

### 设置时机

**重要**：`setcookie()` 必须在任何输出之前调用。

**正确**：
```php
<?php
declare(strict_types=1);

setcookie('username', 'john');
echo "Output";
```

**错误**：
```php
<?php
declare(strict_types=1);

echo "Output";  // 错误：在 setcookie() 之前有输出
setcookie('username', 'john');  // 会失败
```

## $_COOKIE 超全局变量

### 读取 Cookie

**示例**：
```php
<?php
declare(strict_types=1);

// 读取 Cookie
$username = $_COOKIE['username'] ?? 'guest';
$theme = $_COOKIE['theme'] ?? 'light';
```

### 检查 Cookie 是否存在

**示例**：
```php
<?php
declare(strict_types=1);

// 检查 Cookie 是否存在
if (isset($_COOKIE['username'])) {
    $username = $_COOKIE['username'];
    echo "用户名: {$username}\n";
} else {
    echo "Cookie 不存在\n";
}
```

### 读取所有 Cookie

**示例**：
```php
<?php
declare(strict_types=1);

// 读取所有 Cookie
foreach ($_COOKIE as $name => $value) {
    echo "{$name}: {$value}\n";
}
```

## Cookie 属性

### expires（过期时间）

**作用**：指定 Cookie 的过期时间。

**格式**：Unix 时间戳

**示例**：
```php
<?php
declare(strict_types=1);

// 1 小时后过期
setcookie('token', 'abc123', time() + 3600);

// 1 天后过期
setcookie('preference', 'value', time() + 86400);

// 1 个月后过期
setcookie('remember', 'yes', time() + 86400 * 30);

// 浏览器关闭时过期（设置为 0 或过去的时间）
setcookie('session', 'data', 0);
```

### path（路径）

**作用**：指定 Cookie 的作用路径。

**示例**：
```php
<?php
declare(strict_types=1);

// 整个网站可用（推荐）
setcookie('global', 'value', time() + 3600, '/');

// 仅 /admin 路径可用
setcookie('admin', 'value', time() + 3600, '/admin');

// 仅当前目录可用
setcookie('local', 'value', time() + 3600, './');
```

### domain（域名）

**作用**：指定 Cookie 的作用域名。

**示例**：
```php
<?php
declare(strict_types=1);

// 仅 example.com 可用
setcookie('cookie1', 'value', time() + 3600, '/', 'example.com');

// example.com 及其所有子域名可用
setcookie('cookie2', 'value', time() + 3600, '/', '.example.com');
```

### secure（安全传输）

**作用**：指定 Cookie 是否仅通过 HTTPS 传输。

**示例**：
```php
<?php
declare(strict_types=1);

// 仅 HTTPS 传输
setcookie('secure', 'value', time() + 3600, '/', '', true);
```

### httponly（HTTP 访问）

**作用**：指定 Cookie 是否仅通过 HTTP 访问（防止 JavaScript 访问）。

**示例**：
```php
<?php
declare(strict_types=1);

// 防止 JavaScript 访问
setcookie('httponly', 'value', time() + 3600, '/', '', false, true);
```

### samesite（SameSite 属性）

**作用**：防止 CSRF 攻击。

**值**：
- `Strict`：最严格，完全禁止跨站请求携带 Cookie
- `Lax`：允许 GET 请求携带 Cookie
- `None`：允许所有跨站请求携带 Cookie（需要 Secure）

**示例**：
```php
<?php
declare(strict_types=1);

// PHP 7.3+
setcookie('samesite', 'value', [
    'expires' => time() + 3600,
    'path' => '/',
    'samesite' => 'Strict',
]);
```

## Cookie 读取和删除

### 读取 Cookie

**示例**：
```php
<?php
declare(strict_types=1);

// 读取 Cookie
$username = $_COOKIE['username'] ?? 'guest';
$theme = $_COOKIE['theme'] ?? 'light';

// 安全读取（验证和过滤）
function getCookie(string $name, mixed $default = null): mixed
{
    if (!isset($_COOKIE[$name])) {
        return $default;
    }
    
    $value = $_COOKIE[$name];
    
    // 验证和过滤（根据需求）
    // ...
    
    return $value;
}
```

### 删除 Cookie

**方法**：设置过期时间为过去的时间。

**示例**：
```php
<?php
declare(strict_types=1);

// 删除 Cookie
setcookie('username', '', time() - 3600);
setcookie('theme', '', time() - 3600, '/');

// 完整删除（包括所有属性）
setcookie('session_id', '', time() - 3600, '/', 'example.com', true, true);
```

### 更新 Cookie

**方法**：重新设置 Cookie（使用相同的名称和路径）。

**示例**：
```php
<?php
declare(strict_types=1);

// 更新 Cookie
setcookie('theme', 'light', time() + 3600, '/');
```

## 安全选项

### Secure 标志

**作用**：仅通过 HTTPS 传输 Cookie。

**示例**：
```php
<?php
declare(strict_types=1);

// 仅 HTTPS 传输
setcookie('secure_cookie', 'value', time() + 3600, '/', '', true);
```

### HttpOnly 标志

**作用**：防止 JavaScript 访问 Cookie（防止 XSS 攻击）。

**示例**：
```php
<?php
declare(strict_types=1);

// 防止 JavaScript 访问
setcookie('httponly_cookie', 'value', time() + 3600, '/', '', false, true);
```

### SameSite 属性

**作用**：防止 CSRF 攻击。

**示例**：
```php
<?php
declare(strict_types=1);

// PHP 7.3+
setcookie('samesite_cookie', 'value', [
    'expires' => time() + 3600,
    'path' => '/',
    'samesite' => 'Strict',  // 或 'Lax'、'None'
]);
```

### 敏感数据处理

**原则**：不要在 Cookie 中存储敏感信息。

**安全做法**：
```php
<?php
declare(strict_types=1);

// 不推荐：存储敏感信息
setcookie('password', 'secret123');  // 危险！

// 推荐：存储 Token 或 ID
setcookie('session_token', 'hashed_token', time() + 3600, '/', '', true, true);
```

## 使用场景

### 用户偏好设置

- 主题设置
- 语言设置
- 显示偏好

### 记住登录状态

- "记住我"功能
- 长期登录状态
- Token 存储

### 跟踪用户行为

- 用户行为分析
- 访问统计
- 个性化推荐

### 临时数据存储

- 表单数据临时存储
- 搜索结果
- 临时配置

## 注意事项

### Cookie 大小限制

- **4KB 限制**：Cookie 大小通常限制为 4KB
- **避免大数据**：不要存储大量数据
- **考虑替代方案**：大数据考虑使用 Session 或数据库

### 域名和路径限制

- **域名限制**：Cookie 只能被设置它的域名访问
- **路径限制**：Cookie 只能被设置它的路径及其子路径访问
- **子域名**：使用 `.example.com` 可以让所有子域名访问

### 安全选项设置

- **Secure**：生产环境使用 HTTPS 时设置
- **HttpOnly**：防止 XSS 攻击，推荐设置
- **SameSite**：防止 CSRF 攻击，推荐设置

### 过期时间管理

- **合理设置**：根据需求设置合理的过期时间
- **及时清理**：删除不需要的 Cookie
- **长期 Cookie**：谨慎使用长期 Cookie

## 常见问题

### 如何设置 Cookie？

使用 `setcookie()` 函数：

```php
<?php
declare(strict_types=1);

setcookie('name', 'value', time() + 3600);
```

### Cookie 的属性有什么作用？

- **expires**：过期时间
- **path**：作用路径
- **domain**：作用域名
- **secure**：仅 HTTPS 传输
- **httponly**：防止 JavaScript 访问
- **samesite**：防止 CSRF 攻击

### 如何删除 Cookie？

设置过期时间为过去的时间：

```php
<?php
declare(strict_types=1);

setcookie('name', '', time() - 3600);
```

### 如何安全地使用 Cookie？

1. **使用 Secure**：仅通过 HTTPS 传输
2. **使用 HttpOnly**：防止 JavaScript 访问
3. **使用 SameSite**：防止 CSRF 攻击
4. **不存储敏感信息**：避免存储密码等敏感信息

## 最佳实践

### 设置合理的过期时间

- 根据需求设置过期时间
- 短期 Cookie 用于临时数据
- 长期 Cookie 用于用户偏好

### 使用 Secure 和 HttpOnly 标志

- 生产环境使用 Secure（HTTPS）
- 始终使用 HttpOnly（防止 XSS）
- 使用 SameSite（防止 CSRF）

### 限制 Cookie 的作用域

- 使用最小权限原则
- 限制路径和域名
- 避免全局 Cookie

### 避免存储敏感信息

- 不要存储密码
- 不要存储敏感数据
- 使用 Token 或 ID 代替

## 相关章节

- **[5.9.1 Session 基础](section-01-session-basics.md)**：了解 Session 的详细内容
- **[5.3.2 $_SERVER、$_SESSION 与 $_COOKIE](../chapter-03-superglobals/section-02-server-session-cookie.md)**：了解 $_COOKIE 的基础使用
- **[5.10.1 认证基础（AuthN）](../chapter-10-auth/section-01-authentication.md)**：了解 Cookie 在认证中的应用

## 练习任务

1. **实现 Cookie 管理类**
   - 创建 Cookie 管理类
   - 封装 Cookie 操作
   - 提供安全配置

2. **实现用户偏好设置**
   - 使用 Cookie 存储用户偏好
   - 实现偏好读取和更新
   - 管理 Cookie 过期时间

3. **实现"记住我"功能**
   - 使用 Cookie 存储登录 Token
   - 实现自动登录
   - 管理 Token 过期

4. **实现 Cookie 安全配置**
   - 配置 Secure 标志
   - 配置 HttpOnly 标志
   - 配置 SameSite 属性

5. **实现完整的 Cookie 管理系统**
   - Cookie 设置和读取
   - Cookie 删除和更新
   - 安全配置和管理
