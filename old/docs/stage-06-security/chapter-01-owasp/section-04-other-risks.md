# 6.1.4 其他安全风险

## 概述

除了注入攻击和认证问题，还有其他重要的安全风险。本节详细介绍访问控制失效、安全配置错误、敏感数据暴露等风险，以及相应的防护措施。

## 访问控制失效

### 问题

- 未授权访问
- 权限绕过
- 水平权限提升
- 垂直权限提升

### 防护措施

```php
<?php
declare(strict_types=1);

class AccessControl
{
    public function checkPermission(int $userId, string $resource, string $action): bool
    {
        // 检查用户权限
        $stmt = $this->pdo->prepare("
            SELECT COUNT(*) FROM user_permissions 
            WHERE user_id = ? AND resource = ? AND action = ?
        ");
        $stmt->execute([$userId, $resource, $action]);
        return $stmt->fetchColumn() > 0;
    }

    public function checkOwnership(int $userId, int $resourceId, string $table): bool
    {
        // 检查资源所有权
        $stmt = $this->pdo->prepare("SELECT user_id FROM {$table} WHERE id = ?");
        $stmt->execute([$resourceId]);
        $ownerId = $stmt->fetchColumn();
        return $ownerId === $userId;
    }
}
```

## 安全配置错误

### 问题

- 默认配置
- 错误信息泄露
- 不必要的功能
- 过时的组件

### 防护措施

```php
<?php
// 1. 关闭错误显示
ini_set('display_errors', '0');
ini_set('log_errors', '1');

// 2. 隐藏服务器信息
header_remove('X-Powered-By');

// 3. 限制文件上传
ini_set('upload_max_filesize', '10M');
ini_set('post_max_size', '10M');

// 4. 安全头
header('X-Content-Type-Options: nosniff');
header('X-Frame-Options: DENY');
header('X-XSS-Protection: 1; mode=block');
```

## 敏感数据暴露

### 问题

- 敏感信息泄露
- 不安全的传输
- 不安全的存储

### 防护措施

```php
<?php
declare(strict_types=1);

// 1. 加密敏感数据
function encryptSensitiveData(string $data, string $key): string
{
    $iv = random_bytes(16);
    $encrypted = openssl_encrypt($data, 'AES-256-CBC', $key, 0, $iv);
    return base64_encode($iv . $encrypted);
}

// 2. 脱敏处理
function maskEmail(string $email): string
{
    $parts = explode('@', $email);
    $username = $parts[0];
    $domain = $parts[1] ?? '';
    
    if (strlen($username) <= 2) {
        return str_repeat('*', strlen($username)) . '@' . $domain;
    }
    
    return substr($username, 0, 2) . str_repeat('*', strlen($username) - 2) . '@' . $domain;
}
```

## SSRF（服务端请求伪造）

### 防护措施

```php
<?php
declare(strict_types=1);

function safeFetchUrl(string $url): string
{
    // 验证 URL
    $parsed = parse_url($url);
    
    // 禁止内网地址
    $forbiddenHosts = ['127.0.0.1', 'localhost', '0.0.0.0'];
    $host = $parsed['host'] ?? '';
    
    foreach ($forbiddenHosts as $forbidden) {
        if (strpos($host, $forbidden) === 0) {
            throw new InvalidArgumentException('Forbidden host');
        }
    }
    
    // 只允许 HTTP/HTTPS
    $scheme = $parsed['scheme'] ?? '';
    if (!in_array($scheme, ['http', 'https'], true)) {
        throw new InvalidArgumentException('Invalid scheme');
    }
    
    // 使用 cURL 并限制重定向
    $ch = curl_init($url);
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
    curl_setopt($ch, CURLOPT_FOLLOWLOCATION, false);
    curl_setopt($ch, CURLOPT_TIMEOUT, 5);
    
    $result = curl_exec($ch);
    curl_close($ch);
    
    return $result;
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

class SecurityService
{
    public function checkAccess(int $userId, string $resource, string $action): bool
    {
        // 检查权限
        return $this->hasPermission($userId, $resource, $action);
    }

    public function sanitizeOutput(string $data): string
    {
        return htmlspecialchars($data, ENT_QUOTES, 'UTF-8');
    }

    public function maskSensitiveData(string $data, string $type): string
    {
        return match ($type) {
            'email' => $this->maskEmail($data),
            'phone' => $this->maskPhone($data),
            default => $data,
        };
    }
}
```

## 注意事项

1. **访问控制**：实现细粒度的访问控制
2. **配置安全**：正确配置安全选项
3. **数据保护**：加密和脱敏敏感数据
4. **SSRF 防护**：验证和限制 URL 访问

## 练习

1. 实现完整的访问控制系统。

2. 创建一个安全配置检查工具。

3. 编写敏感数据脱敏工具。

4. 实现 SSRF 防护机制。
