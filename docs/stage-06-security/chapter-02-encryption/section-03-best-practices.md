# 6.2.3 加密最佳实践

## 概述

加密最佳实践包括密钥管理、敏感数据脱敏、加密存储、安全传输等。本节详细介绍这些最佳实践和完整实现。

## 密钥管理

### 密钥存储

```php
<?php
declare(strict_types=1);

// 从环境变量读取密钥
$encryptionKey = $_ENV['ENCRYPTION_KEY'] ?? '';

if (empty($encryptionKey)) {
    throw new RuntimeException('Encryption key not configured');
}

// 使用密钥管理服务（如 AWS KMS）
class KeyManager
{
    public function getKey(string $keyId): string
    {
        // 从密钥管理服务获取密钥
        // ...
    }
}
```

## 敏感数据脱敏

### 数据脱敏方法

```php
<?php
declare(strict_types=1);

class DataMasking
{
    // 邮箱脱敏
    public static function maskEmail(string $email): string
    {
        if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
            return $email;
        }
        
        [$local, $domain] = explode('@', $email);
        $maskedLocal = substr($local, 0, 2) . str_repeat('*', strlen($local) - 2);
        return "{$maskedLocal}@{$domain}";
    }
    
    // 手机号脱敏
    public static function maskPhone(string $phone): string
    {
        if (strlen($phone) < 4) {
            return str_repeat('*', strlen($phone));
        }
        
        return substr($phone, 0, 3) . str_repeat('*', strlen($phone) - 6) . substr($phone, -3);
    }
    
    // 银行卡号脱敏
    public static function maskCardNumber(string $cardNumber): string
    {
        if (strlen($cardNumber) < 8) {
            return str_repeat('*', strlen($cardNumber));
        }
        
        return substr($cardNumber, 0, 4) . str_repeat('*', strlen($cardNumber) - 8) . substr($cardNumber, -4);
    }
}
```

## 加密存储

```php
<?php
declare(strict_types=1);

class SecureStorage
{
    private EncryptionService $encryption;

    public function store(string $key, string $data): void
    {
        $encrypted = $this->encryption->encrypt($data);
        // 存储加密数据
        file_put_contents($key . '.enc', $encrypted);
    }

    public function retrieve(string $key): string
    {
        $encrypted = file_get_contents($key . '.enc');
        return $this->encryption->decrypt($encrypted);
    }
}
```

## 安全传输

```php
<?php
declare(strict_types=1);

// 使用 HTTPS
if (!isset($_SERVER['HTTPS']) || $_SERVER['HTTPS'] !== 'on') {
    header('Location: https://' . $_SERVER['HTTP_HOST'] . $_SERVER['REQUEST_URI']);
    exit;
}

// 设置安全头
header('Strict-Transport-Security: max-age=31536000; includeSubDomains');
```

## 完整示例

```php
<?php
declare(strict_types=1);

class SecurityService
{
    private EncryptionService $encryption;
    private DataMasking $masking;

    public function storeSensitiveData(string $key, string $data): void
    {
        $encrypted = $this->encryption->encrypt($data);
        // 存储加密数据
    }

    public function displaySensitiveData(string $data, string $type): string
    {
        return $this->masking->mask($data, $type);
    }
}
```

## 注意事项

1. **密钥安全**：密钥不能硬编码，使用环境变量或密钥管理服务
2. **数据脱敏**：显示敏感数据时进行脱敏
3. **传输安全**：使用 HTTPS 传输敏感数据
4. **存储安全**：加密存储敏感数据

## 练习

1. 实现完整的密钥管理系统。

2. 创建一个敏感数据脱敏工具。

3. 编写加密存储系统。

4. 实现安全传输机制。
