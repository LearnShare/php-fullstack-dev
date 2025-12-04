# 6.4.1 安全 HTTP 头

## 概述

安全 HTTP 头是保护 Web 应用免受各种攻击的重要防线。本节介绍常见的安全 HTTP 头，包括 CSP（内容安全策略）、HSTS（HTTP 严格传输安全）、X-Frame-Options、X-Content-Type-Options、Referrer-Policy 等。

## CSP（内容安全策略）

### 什么是 CSP

CSP（Content Security Policy）是一种安全机制，用于防止 XSS 攻击。它通过限制浏览器可以加载的资源来源来工作。

### 基础配置

```php
<?php
declare(strict_types=1);

// 设置 CSP 头
header("Content-Security-Policy: default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'");
```

### 详细配置示例

```php
<?php
declare(strict_types=1);

class SecurityHeaders
{
    /**
     * 设置 CSP 头
     */
    public static function setCSP(array $directives): void
    {
        $policy = [];
        
        // default-src: 默认资源来源
        if (isset($directives['default-src'])) {
            $policy[] = "default-src " . implode(' ', $directives['default-src']);
        }
        
        // script-src: 脚本来源
        if (isset($directives['script-src'])) {
            $policy[] = "script-src " . implode(' ', $directives['script-src']);
        }
        
        // style-src: 样式来源
        if (isset($directives['style-src'])) {
            $policy[] = "style-src " . implode(' ', $directives['style-src']);
        }
        
        // img-src: 图片来源
        if (isset($directives['img-src'])) {
            $policy[] = "img-src " . implode(' ', $directives['img-src']);
        }
        
        // connect-src: 连接来源（AJAX、WebSocket 等）
        if (isset($directives['connect-src'])) {
            $policy[] = "connect-src " . implode(' ', $directives['connect-src']);
        }
        
        // font-src: 字体来源
        if (isset($directives['font-src'])) {
            $policy[] = "font-src " . implode(' ', $directives['font-src']);
        }
        
        // object-src: 对象来源（embed、object 等）
        if (isset($directives['object-src'])) {
            $policy[] = "object-src " . implode(' ', $directives['object-src']);
        }
        
        // media-src: 媒体来源（audio、video 等）
        if (isset($directives['media-src'])) {
            $policy[] = "media-src " . implode(' ', $directives['media-src']);
        }
        
        // frame-src: 框架来源（iframe 等）
        if (isset($directives['frame-src'])) {
            $policy[] = "frame-src " . implode(' ', $directives['frame-src']);
        }
        
        // base-uri: base 标签的 URI
        if (isset($directives['base-uri'])) {
            $policy[] = "base-uri " . implode(' ', $directives['base-uri']);
        }
        
        // form-action: 表单提交目标
        if (isset($directives['form-action'])) {
            $policy[] = "form-action " . implode(' ', $directives['form-action']);
        }
        
        // upgrade-insecure-requests: 升级 HTTP 请求到 HTTPS
        if (isset($directives['upgrade-insecure-requests']) && $directives['upgrade-insecure-requests']) {
            $policy[] = "upgrade-insecure-requests";
        }
        
        header("Content-Security-Policy: " . implode('; ', $policy));
    }
}

// 使用示例
SecurityHeaders::setCSP([
    'default-src' => ["'self'"],
    'script-src' => ["'self'", "'unsafe-inline'", "https://cdn.example.com"],
    'style-src' => ["'self'", "'unsafe-inline'", "https://fonts.googleapis.com"],
    'img-src' => ["'self'", "data:", "https:"],
    'connect-src' => ["'self'", "https://api.example.com"],
    'font-src' => ["'self'", "https://fonts.gstatic.com"],
    'object-src' => ["'none'"],
    'base-uri' => ["'self'"],
    'form-action' => ["'self'"],
    'upgrade-insecure-requests' => true,
]);
```

### CSP 报告

```php
<?php
declare(strict_types=1);

// 启用 CSP 报告
header("Content-Security-Policy: default-src 'self'; report-uri /csp-report");
header("Content-Security-Policy-Report-Only: default-src 'self'; report-uri /csp-report");

// 处理 CSP 报告
if ($_SERVER['REQUEST_METHOD'] === 'POST' && $_SERVER['REQUEST_URI'] === '/csp-report') {
    $report = json_decode(file_get_contents('php://input'), true);
    
    // 记录违规报告
    error_log("CSP Violation: " . json_encode($report));
    
    // 可以存储到数据库或发送到监控服务
    // ...
    
    http_response_code(204);
    exit;
}
```

## HSTS（HTTP 严格传输安全）

### 什么是 HSTS

HSTS（HTTP Strict Transport Security）强制浏览器使用 HTTPS 连接，防止中间人攻击。

### 配置 HSTS

```php
<?php
declare(strict_types=1);

// 基础 HSTS 配置（1 年，包含子域名）
header("Strict-Transport-Security: max-age=31536000; includeSubDomains; preload");

// 详细配置
class HSTSConfig
{
    public static function set(int $maxAge = 31536000, bool $includeSubDomains = true, bool $preload = false): void
    {
        $directives = ["max-age={$maxAge}"];
        
        if ($includeSubDomains) {
            $directives[] = "includeSubDomains";
        }
        
        if ($preload) {
            $directives[] = "preload";
        }
        
        header("Strict-Transport-Security: " . implode('; ', $directives));
    }
}

// 使用
HSTSConfig::set(31536000, true, true);
```

## X-Frame-Options

### 防止点击劫持

```php
<?php
declare(strict_types=1);

// DENY: 禁止在任何框架中显示
header("X-Frame-Options: DENY");

// SAMEORIGIN: 只允许同源框架
header("X-Frame-Options: SAMEORIGIN");

// ALLOW-FROM: 允许指定来源（已废弃，使用 CSP frame-ancestors）
// header("X-Frame-Options: ALLOW-FROM https://example.com");
```

## X-Content-Type-Options

### 防止 MIME 类型嗅探

```php
<?php
declare(strict_types=1);

// 禁止浏览器猜测内容类型
header("X-Content-Type-Options: nosniff");
```

## Referrer-Policy

### 控制 Referrer 信息

```php
<?php
declare(strict_types=1);

// no-referrer: 不发送 Referrer
header("Referrer-Policy: no-referrer");

// no-referrer-when-downgrade: 降级时不发送（默认）
header("Referrer-Policy: no-referrer-when-downgrade");

// origin: 只发送源
header("Referrer-Policy: origin");

// origin-when-cross-origin: 跨域时只发送源
header("Referrer-Policy: origin-when-cross-origin");

// same-origin: 同源时发送完整 Referrer
header("Referrer-Policy: same-origin");

// strict-origin: 只发送源，HTTPS->HTTP 不发送
header("Referrer-Policy: strict-origin");

// strict-origin-when-cross-origin: 跨域时只发送源，HTTPS->HTTP 不发送
header("Referrer-Policy: strict-origin-when-cross-origin");

// unsafe-url: 总是发送完整 Referrer（不推荐）
// header("Referrer-Policy: unsafe-url");
```

## Permissions-Policy

### 控制浏览器功能

```php
<?php
declare(strict_types=1);

// 禁用摄像头和麦克风
header("Permissions-Policy: camera=(), microphone=()");

// 允许同源使用地理位置
header("Permissions-Policy: geolocation=(self)");

// 完整配置示例
header("Permissions-Policy: camera=(), microphone=(), geolocation=(self), payment=(self)");
```

## X-XSS-Protection

### XSS 过滤器（已废弃，但仍在使用）

```php
<?php
declare(strict_types=1);

// 启用 XSS 过滤器（已废弃，现代浏览器使用 CSP）
header("X-XSS-Protection: 1; mode=block");
```

## 完整示例

```php
<?php
declare(strict_types=1);

class SecurityHeadersManager
{
    /**
     * 设置所有安全头
     */
    public static function setAll(array $config = []): void
    {
        // CSP
        self::setCSP($config['csp'] ?? []);
        
        // HSTS（仅 HTTPS）
        if (self::isHttps()) {
            self::setHSTS($config['hsts'] ?? []);
        }
        
        // X-Frame-Options
        self::setXFrameOptions($config['x-frame-options'] ?? 'SAMEORIGIN');
        
        // X-Content-Type-Options
        header("X-Content-Type-Options: nosniff");
        
        // Referrer-Policy
        self::setReferrerPolicy($config['referrer-policy'] ?? 'strict-origin-when-cross-origin');
        
        // Permissions-Policy
        if (isset($config['permissions-policy'])) {
            self::setPermissionsPolicy($config['permissions-policy']);
        }
        
        // 移除 X-Powered-By（如果可能）
        if (function_exists('header_remove')) {
            header_remove('X-Powered-By');
        }
    }
    
    private static function setCSP(array $directives): void
    {
        // ... CSP 设置逻辑（见上文）
    }
    
    private static function setHSTS(array $config): void
    {
        $maxAge = $config['max-age'] ?? 31536000;
        $includeSubDomains = $config['include-subdomains'] ?? true;
        $preload = $config['preload'] ?? false;
        
        HSTSConfig::set($maxAge, $includeSubDomains, $preload);
    }
    
    private static function setXFrameOptions(string $value): void
    {
        header("X-Frame-Options: {$value}");
    }
    
    private static function setReferrerPolicy(string $policy): void
    {
        header("Referrer-Policy: {$policy}");
    }
    
    private static function setPermissionsPolicy(array $directives): void
    {
        $policy = [];
        foreach ($directives as $feature => $allowlist) {
            $policy[] = "{$feature}=(" . implode(' ', $allowlist) . ")";
        }
        header("Permissions-Policy: " . implode(', ', $policy));
    }
    
    private static function isHttps(): bool
    {
        return (!empty($_SERVER['HTTPS']) && $_SERVER['HTTPS'] !== 'off')
            || $_SERVER['SERVER_PORT'] == 443
            || (!empty($_SERVER['HTTP_X_FORWARDED_PROTO']) && $_SERVER['HTTP_X_FORWARDED_PROTO'] === 'https');
    }
}

// 使用
SecurityHeadersManager::setAll([
    'csp' => [
        'default-src' => ["'self'"],
        'script-src' => ["'self'", "https://cdn.example.com"],
        'style-src' => ["'self'", "'unsafe-inline'"],
        'img-src' => ["'self'", "data:", "https:"],
    ],
    'hsts' => [
        'max-age' => 31536000,
        'include-subdomains' => true,
        'preload' => false,
    ],
    'x-frame-options' => 'SAMEORIGIN',
    'referrer-policy' => 'strict-origin-when-cross-origin',
    'permissions-policy' => [
        'camera' => [],
        'microphone' => [],
        'geolocation' => ['self'],
    ],
]);
```

## 最佳实践

1. **CSP**：从严格策略开始，逐步放宽
2. **HSTS**：生产环境必须启用
3. **X-Frame-Options**：使用 SAMEORIGIN 或 DENY
4. **Referrer-Policy**：使用 strict-origin-when-cross-origin
5. **移除敏感头**：移除 X-Powered-By、Server 等

## 注意事项

1. CSP 配置需要仔细测试，避免影响正常功能
2. HSTS 只在 HTTPS 下有效
3. 某些安全头可能影响第三方集成
4. 使用 CSP 报告模式进行测试

## 练习

1. 实现一个安全头管理类，支持配置所有常见的安全 HTTP 头。

2. 配置 CSP 策略，允许从 CDN 加载资源，但禁止内联脚本和样式。

3. 实现 CSP 报告功能，将违规报告记录到日志或数据库。

4. 创建一个中间件，自动为所有响应添加安全头。
