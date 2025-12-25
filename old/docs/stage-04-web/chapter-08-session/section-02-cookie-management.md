# 4.8.2 Cookie 管理

## 概述

Cookie 是客户端状态存储的重要机制。本节详细介绍 Cookie 基础、Cookie 参数详解、SameSite 属性、安全 Cookie 设置，以及 Cookie 与 Session 的配合使用。

## Cookie 基础

### 基础用法

```php
<?php
declare(strict_types=1);

// 设置 Cookie
setcookie('username', 'alice', time() + 3600, '/', '', false, true);

// 读取 Cookie
$username = $_COOKIE['username'] ?? '';

// 删除 Cookie
setcookie('username', '', time() - 3600, '/');
```

### Cookie 参数

```php
<?php
setcookie(
    'name',           // Cookie 名称
    'value',          // Cookie 值
    time() + 3600,    // 过期时间（Unix 时间戳）
    '/',              // 路径
    'example.com',    // 域名
    true,             // secure（仅 HTTPS）
    true              // httponly（防止 JS 访问）
);
```

## Cookie 参数详解

### 过期时间

```php
<?php
// 1 小时后过期
setcookie('token', 'value', time() + 3600);

// 1 天后过期
setcookie('token', 'value', time() + 86400);

// 会话 Cookie（浏览器关闭后删除）
setcookie('token', 'value', 0);
```

### 路径和域名

```php
<?php
// 路径：只有 /admin 路径可以访问
setcookie('admin_token', 'value', time() + 3600, '/admin');

// 域名：所有子域名都可以访问
setcookie('token', 'value', time() + 3600, '/', '.example.com');
```

## SameSite 属性

### SameSite 值

- **Strict**：完全禁止跨站请求携带 Cookie
- **Lax**：GET 请求可跨站，POST 不可（默认）
- **None**：允许跨站，但需要 `Secure` 属性

```php
<?php
// SameSite=Strict（最安全）
setcookie('token', 'value', [
    'expires' => time() + 3600,
    'path' => '/',
    'samesite' => 'Strict',
    'secure' => true,
    'httponly' => true,
]);

// SameSite=Lax（平衡）
setcookie('token', 'value', [
    'samesite' => 'Lax',
    'secure' => true,
]);

// SameSite=None（需要 Secure）
setcookie('token', 'value', [
    'samesite' => 'None',
    'secure' => true,  // 必需
]);
```

## 安全 Cookie 设置

### 安全函数

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
    $options = [
        'expires' => time() + $expires,
        'path' => $path,
        'secure' => true,      // 仅 HTTPS
        'httponly' => true,    // 防止 JavaScript 访问
        'samesite' => $sameSite,  // 防止 CSRF
    ];
    
    if ($domain !== null) {
        $options['domain'] = $domain;
    }
    
    return setcookie($name, $value, $options);
}

// 使用
setSecureCookie('session_id', 'abc123', 7200);
```

## Cookie 与 Session 配合

### Session ID 通过 Cookie 传递

```php
<?php
// Session 自动通过 Cookie 传递 Session ID
session_start();

// Session ID 存储在 Cookie 中
// Cookie 名称：PHPSESSID（可通过 session_name() 修改）
// Cookie 值：Session ID
```

### 自定义 Session Cookie

```php
<?php
// 自定义 Session Cookie 名称
session_name('MY_SESSION_ID');

// 自定义 Session Cookie 参数
ini_set('session.cookie_lifetime', 3600);
ini_set('session.cookie_path', '/');
ini_set('session.cookie_domain', '.example.com');
ini_set('session.cookie_secure', 1);
ini_set('session.cookie_httponly', 1);
ini_set('session.cookie_samesite', 'Strict');

session_start();
```

## 完整示例

```php
<?php
declare(strict_types=1);

class CookieManager
{
    public static function set(
        string $name,
        string $value,
        int $expires = 3600,
        array $options = []
    ): bool {
        $defaults = [
            'expires' => time() + $expires,
            'path' => '/',
            'secure' => true,
            'httponly' => true,
            'samesite' => 'Strict',
        ];
        
        return setcookie($name, $value, array_merge($defaults, $options));
    }

    public static function get(string $name, string $default = ''): string
    {
        return $_COOKIE[$name] ?? $default;
    }

    public static function delete(string $name, string $path = '/'): bool
    {
        return setcookie($name, '', time() - 3600, $path);
    }
}
```

## 注意事项

1. **安全设置**：使用 `secure`、`httponly`、`samesite` 属性
2. **过期时间**：合理设置过期时间，避免过长
3. **路径和域名**：根据需求设置，避免过于宽泛
4. **大小限制**：Cookie 有大小限制（约 4KB）

## 练习

1. 创建一个安全的 Cookie 设置函数，包含所有安全选项。

2. 实现 Cookie 管理类，提供设置、获取、删除方法。

3. 实现 Session 和 Cookie 的配合使用，自定义 Session Cookie 参数。

4. 创建一个 Cookie 配置类，统一管理 Cookie 配置。
