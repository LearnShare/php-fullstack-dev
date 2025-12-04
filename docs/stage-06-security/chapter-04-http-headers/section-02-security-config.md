# 6.4.2 安全配置

## 概述

本节介绍如何在实际应用中配置安全 HTTP 头，包括中间件实现、Nginx 配置、安全测试等内容。

## 中间件实现

### PHP 中间件

```php
<?php
declare(strict_types=1);

class SecurityHeadersMiddleware
{
    private array $config;
    
    public function __construct(array $config = [])
    {
        $this->config = array_merge([
            'csp' => [],
            'hsts' => [],
            'x-frame-options' => 'SAMEORIGIN',
            'referrer-policy' => 'strict-origin-when-cross-origin',
            'remove-powered-by' => true,
        ], $config);
    }
    
    public function __invoke($request, $response, $next)
    {
        // 设置安全头
        $this->setSecurityHeaders();
        
        // 继续处理请求
        return $next($request, $response);
    }
    
    private function setSecurityHeaders(): void
    {
        // CSP
        if (!empty($this->config['csp'])) {
            SecurityHeaders::setCSP($this->config['csp']);
        }
        
        // HSTS
        if ($this->isHttps() && !empty($this->config['hsts'])) {
            HSTSConfig::set(
                $this->config['hsts']['max-age'] ?? 31536000,
                $this->config['hsts']['include-subdomains'] ?? true,
                $this->config['hsts']['preload'] ?? false
            );
        }
        
        // X-Frame-Options
        header("X-Frame-Options: {$this->config['x-frame-options']}");
        
        // X-Content-Type-Options
        header("X-Content-Type-Options: nosniff");
        
        // Referrer-Policy
        header("Referrer-Policy: {$this->config['referrer-policy']}");
        
        // 移除 X-Powered-By
        if ($this->config['remove-powered-by'] && function_exists('header_remove')) {
            header_remove('X-Powered-By');
        }
    }
    
    private function isHttps(): bool
    {
        return (!empty($_SERVER['HTTPS']) && $_SERVER['HTTPS'] !== 'off')
            || $_SERVER['SERVER_PORT'] == 443;
    }
}
```

### Laravel 中间件

```php
<?php
declare(strict_types=1);

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;

class SecurityHeaders
{
    public function handle(Request $request, Closure $next)
    {
        $response = $next($request);
        
        // 设置安全头
        $response->headers->set('X-Content-Type-Options', 'nosniff');
        $response->headers->set('X-Frame-Options', 'SAMEORIGIN');
        $response->headers->set('X-XSS-Protection', '1; mode=block');
        $response->headers->set('Referrer-Policy', 'strict-origin-when-cross-origin');
        
        // CSP
        $csp = "default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'";
        $response->headers->set('Content-Security-Policy', $csp);
        
        // HSTS（仅 HTTPS）
        if ($request->secure()) {
            $response->headers->set('Strict-Transport-Security', 'max-age=31536000; includeSubDomains');
        }
        
        // 移除 X-Powered-By
        $response->headers->remove('X-Powered-By');
        
        return $response;
    }
}
```

## Nginx 配置

### 基础配置

```nginx
# 安全 HTTP 头
add_header X-Content-Type-Options "nosniff" always;
add_header X-Frame-Options "SAMEORIGIN" always;
add_header X-XSS-Protection "1; mode=block" always;
add_header Referrer-Policy "strict-origin-when-cross-origin" always;

# CSP
add_header Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'" always;

# HSTS（仅 HTTPS）
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;

# 移除 Server 头
server_tokens off;
```

### 完整 Nginx 配置示例

```nginx
server {
    listen 443 ssl http2;
    server_name example.com;
    
    # SSL 配置
    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    
    # 安全头
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
    add_header Permissions-Policy "camera=(), microphone=(), geolocation=(self)" always;
    
    # CSP
    add_header Content-Security-Policy "default-src 'self'; script-src 'self' https://cdn.example.com; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com; img-src 'self' data: https:; font-src 'self' https://fonts.gstatic.com; connect-src 'self' https://api.example.com" always;
    
    # HSTS
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;
    
    # 移除敏感头
    server_tokens off;
    more_clear_headers "X-Powered-By";
    
    # PHP-FPM
    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }
}
```

## Apache 配置

### .htaccess 配置

```apache
# 安全 HTTP 头
<IfModule mod_headers.c>
    Header set X-Content-Type-Options "nosniff"
    Header set X-Frame-Options "SAMEORIGIN"
    Header set X-XSS-Protection "1; mode=block"
    Header set Referrer-Policy "strict-origin-when-cross-origin"
    
    # CSP
    Header set Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'"
    
    # HSTS（仅 HTTPS）
    Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"
    
    # 移除 X-Powered-By
    Header unset X-Powered-By
</IfModule>

# 移除 Server 信息
ServerTokens Prod
```

## 安全测试

### 使用 securityheaders.com

访问 https://securityheaders.com 测试安全头配置。

### 使用 curl 测试

```bash
# 测试安全头
curl -I https://example.com

# 检查特定头
curl -I https://example.com | grep -i "content-security-policy"
curl -I https://example.com | grep -i "strict-transport-security"
```

### PHP 测试脚本

```php
<?php
declare(strict_types=1);

class SecurityHeadersTest
{
    private string $url;
    private array $headers = [];
    
    public function __construct(string $url)
    {
        $this->url = $url;
    }
    
    public function test(): array
    {
        $ch = curl_init($this->url);
        curl_setopt_array($ch, [
            CURLOPT_RETURNTRANSFER => true,
            CURLOPT_HEADER => true,
            CURLOPT_NOBODY => true,
            CURLOPT_FOLLOWLOCATION => true,
        ]);
        
        $response = curl_exec($ch);
        curl_close($ch);
        
        $this->parseHeaders($response);
        
        return [
            'csp' => $this->hasHeader('Content-Security-Policy'),
            'hsts' => $this->hasHeader('Strict-Transport-Security'),
            'x-frame-options' => $this->hasHeader('X-Frame-Options'),
            'x-content-type-options' => $this->hasHeader('X-Content-Type-Options'),
            'referrer-policy' => $this->hasHeader('Referrer-Policy'),
            'x-powered-by' => !$this->hasHeader('X-Powered-By'),
        ];
    }
    
    private function parseHeaders(string $response): void
    {
        $lines = explode("\r\n", $response);
        foreach ($lines as $line) {
            if (strpos($line, ':') !== false) {
                [$key, $value] = explode(':', $line, 2);
                $this->headers[trim($key)] = trim($value);
            }
        }
    }
    
    private function hasHeader(string $name): bool
    {
        return isset($this->headers[$name]);
    }
}

// 使用
$test = new SecurityHeadersTest('https://example.com');
$results = $test->test();

foreach ($results as $header => $present) {
    echo "{$header}: " . ($present ? '✓' : '✗') . "\n";
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

// 1. 配置安全头
$securityConfig = [
    'csp' => [
        'default-src' => ["'self'"],
        'script-src' => ["'self'", "https://cdn.example.com"],
        'style-src' => ["'self'", "'unsafe-inline'"],
        'img-src' => ["'self'", "data:", "https:"],
    ],
    'hsts' => [
        'max-age' => 31536000,
        'include-subdomains' => true,
    ],
];

// 2. 应用安全头
SecurityHeadersManager::setAll($securityConfig);

// 3. 测试安全头
$test = new SecurityHeadersTest('https://example.com');
$results = $test->test();

// 4. 记录结果
error_log("Security headers test: " . json_encode($results));
```

## 最佳实践

1. **统一配置**：使用中间件或配置文件统一管理安全头
2. **环境区分**：开发环境可以放宽 CSP，生产环境严格配置
3. **定期测试**：定期检查安全头是否正确设置
4. **监控告警**：监控安全头配置变化

## 注意事项

1. Nginx 的 `add_header` 需要 `always` 参数才能在错误响应中生效
2. Apache 需要启用 `mod_headers` 模块
3. CSP 配置需要仔细测试，避免影响正常功能
4. HSTS 预加载需要提交到浏览器厂商

## 练习

1. 实现一个安全头中间件，支持配置所有常见的安全 HTTP 头。

2. 配置 Nginx，为所有响应添加安全头。

3. 创建一个安全头测试工具，检查网站的安全头配置。

4. 实现 CSP 报告功能，将违规报告记录并分析。
