# 6.4 安全 HTTP 头

## 目标

- 理解安全 HTTP 头的作用。
- 掌握 CSP（内容安全策略）的配置。
- 熟悉 HSTS、X-Frame-Options 等安全头。
- 了解 Referrer-Policy、Permissions-Policy 的配置。

## CSP（内容安全策略）

### 基础配置

```php
<?php
// 基础 CSP
header("Content-Security-Policy: default-src 'self'");

// 详细配置
header("Content-Security-Policy: "
    . "default-src 'self'; "
    . "script-src 'self' 'unsafe-inline' https://cdn.example.com; "
    . "style-src 'self' 'unsafe-inline'; "
    . "img-src 'self' data: https:; "
    . "font-src 'self' https://fonts.googleapis.com; "
    . "connect-src 'self' https://api.example.com; "
    . "frame-ancestors 'none'; "
    . "base-uri 'self'; "
    . "form-action 'self'"
);
```

### CSP 指令说明

| 指令              | 说明                           | 示例                           |
| :---------------- | :----------------------------- | :----------------------------- |
| `default-src`     | 默认策略                       | `'self'`                       |
| `script-src`      | 允许的脚本来源                 | `'self' 'unsafe-inline'`       |
| `style-src`       | 允许的样式来源                 | `'self' 'unsafe-inline'`       |
| `img-src`         | 允许的图片来源                 | `'self' data: https:`          |
| `font-src`        | 允许的字体来源                 | `'self' https://fonts.googleapis.com` |
| `connect-src`     | 允许的连接来源（AJAX、WebSocket） | `'self' https://api.example.com` |
| `frame-ancestors` | 允许嵌入的框架                 | `'none'` 或 `'self'`           |
| `base-uri`        | 允许的 base 标签 URI           | `'self'`                       |
| `form-action`     | 允许的表单提交目标             | `'self'`                       |

### CSP 报告

```php
<?php
// 启用 CSP 报告
header("Content-Security-Policy: default-src 'self'; report-uri /csp-report");

// 或使用 report-to（新标准）
header("Content-Security-Policy: default-src 'self'; report-to csp-endpoint");
header("Report-To: {\"group\":\"csp-endpoint\",\"max_age\":10886400,\"endpoints\":[{\"url\":\"/csp-report\"}]}");
```

## HSTS（HTTP 严格传输安全）

### 配置 HSTS

```php
<?php
// 基础 HSTS（1 年）
header('Strict-Transport-Security: max-age=31536000');

// 包含子域名
header('Strict-Transport-Security: max-age=31536000; includeSubDomains');

// 预加载（需要先满足其他条件）
header('Strict-Transport-Security: max-age=31536000; includeSubDomains; preload');
```

## X-Frame-Options

### 防止点击劫持

```php
<?php
// 禁止嵌入
header('X-Frame-Options: DENY');

// 只允许同源嵌入
header('X-Frame-Options: SAMEORIGIN');

// 允许指定源嵌入（已废弃，使用 CSP frame-ancestors）
// header('X-Frame-Options: ALLOW-FROM https://example.com');
```

## 其他安全头

### X-Content-Type-Options

```php
<?php
// 禁止 MIME 类型嗅探
header('X-Content-Type-Options: nosniff');
```

### Referrer-Policy

```php
<?php
// 不发送 Referrer
header('Referrer-Policy: no-referrer');

// 同源发送完整 Referrer，跨源不发送
header('Referrer-Policy: same-origin');

// 只发送源（不包含路径）
header('Referrer-Policy: origin');

// 严格源（HTTPS->HTTPS 发送源，其他不发送）
header('Referrer-Policy: strict-origin');
```

### Permissions-Policy

```php
<?php
// 禁用某些浏览器功能
header("Permissions-Policy: "
    . "geolocation=(), "
    . "microphone=(), "
    . "camera=(), "
    . "payment=('self'), "
    . "usb=()"
);
```

## 完整安全头配置

```php
<?php
declare(strict_types=1);

class SecurityHeaders
{
    public static function setAll(): void
    {
        // CSP
        header("Content-Security-Policy: "
            . "default-src 'self'; "
            . "script-src 'self' 'unsafe-inline'; "
            . "style-src 'self' 'unsafe-inline'; "
            . "img-src 'self' data: https:; "
            . "font-src 'self'; "
            . "connect-src 'self'; "
            . "frame-ancestors 'none'; "
            . "base-uri 'self'; "
            . "form-action 'self'"
        );
        
        // HSTS
        if (isset($_SERVER['HTTPS']) && $_SERVER['HTTPS'] === 'on') {
            header('Strict-Transport-Security: max-age=31536000; includeSubDomains');
        }
        
        // X-Frame-Options
        header('X-Frame-Options: DENY');
        
        // X-Content-Type-Options
        header('X-Content-Type-Options: nosniff');
        
        // Referrer-Policy
        header('Referrer-Policy: strict-origin-when-cross-origin');
        
        // Permissions-Policy
        header("Permissions-Policy: "
            . "geolocation=(), "
            . "microphone=(), "
            . "camera=()"
        );
        
        // 移除服务器信息
        header_remove('X-Powered-By');
    }
}

// 使用
SecurityHeaders::setAll();
```

## 练习

1. 配置完整的 CSP 策略，支持内联脚本和外部资源。

2. 创建一个安全头中间件，统一设置所有安全 HTTP 头。

3. 实现 CSP 报告接收端点，记录和分析 CSP 违规。

4. 设计一个安全头配置系统，支持不同环境使用不同的安全策略。

5. 编写一个安全头测试工具，验证所有安全头是否正确设置。

6. 创建一个安全头文档生成器，自动生成安全头配置说明。
