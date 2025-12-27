# 4.6.2 数据加密

## 概述

数据加密是保护敏感数据的重要方法。与密码哈希（单向）不同，数据加密是双向的，可以加密数据也可以解密数据。在实际应用中，我们经常需要加密存储或传输的敏感数据，如用户信息、支付数据等。

PHP 通过 OpenSSL 扩展提供了数据加密功能。理解数据加密的基本概念、加密算法的选择、密钥管理，以及如何使用 `openssl_encrypt()` 和 `openssl_decrypt()` 函数，对于构建安全的应用至关重要。

**主要内容**：
- 数据加密概述（加密与哈希的区别、加密的作用）
- `openssl_encrypt()` 函数（加密数据）
- `openssl_decrypt()` 函数（解密数据）
- 加密算法选择（AES、DES、RSA 等）
- 密钥管理（密钥生成、存储、轮换）
- 初始化向量（IV）的使用
- 实际应用场景和最佳实践

## 特性

- **双向加密**：可以加密和解密数据
- **多种算法**：支持多种加密算法（AES、DES 等）
- **安全可靠**：使用 OpenSSL 库，经过验证
- **灵活配置**：支持不同的加密模式和选项

## 语法/定义

### openssl_encrypt() 函数

**语法**：`openssl_encrypt(string $data, string $cipher_algo, string $passphrase, int $options = 0, string $iv = "", string &$tag = null, string $aad = "", int $tag_length = 16): string|false`

**参数**：
- `$data`：要加密的数据
- `$cipher_algo`：加密算法（如 `'aes-256-cbc'`）
- `$passphrase`：加密密钥
- `$options`：可选，选项标志
- `$iv`：可选，初始化向量
- `$tag`：可选，用于 AEAD 模式
- `$aad`：可选，附加认证数据
- `$tag_length`：可选，标签长度

**返回值**：成功返回加密后的字符串（base64 编码），失败返回 `false`。

### openssl_decrypt() 函数

**语法**：`openssl_decrypt(string $data, string $cipher_algo, string $passphrase, int $options = 0, string $iv = "", ?string $tag = null, string $aad = ""): string|false`

**参数**：
- `$data`：要解密的数据
- `$cipher_algo`：加密算法（必须与加密时相同）
- `$passphrase`：解密密钥（必须与加密时相同）
- `$options`：可选，选项标志
- `$iv`：可选，初始化向量（必须与加密时相同）
- `$tag`：可选，用于 AEAD 模式
- `$aad`：可选，附加认证数据

**返回值**：成功返回解密后的字符串，失败返回 `false`。

## 基本用法

### 示例 1：基本加密和解密

```php
<?php
declare(strict_types=1);

// 加密数据
$data = '敏感数据';
$method = 'aes-256-cbc';
$key = 'your-secret-key-32-characters!';  // 密钥长度必须匹配算法要求

// 生成初始化向量（IV）
$ivLength = openssl_cipher_iv_length($method);
$iv = openssl_random_pseudo_bytes($ivLength);

// 加密
$encrypted = openssl_encrypt($data, $method, $key, 0, $iv);
if ($encrypted === false) {
    throw new RuntimeException('Encryption failed');
}

echo "加密数据: {$encrypted}\n";

// 解密（需要相同的密钥和 IV）
$decrypted = openssl_decrypt($encrypted, $method, $key, 0, $iv);
if ($decrypted === false) {
    throw new RuntimeException('Decryption failed');
}

echo "解密数据: {$decrypted}\n";
```

**说明**：
- 使用 AES-256-CBC 算法加密
- 需要生成初始化向量（IV）
- 解密时需要相同的密钥和 IV

### 示例 2：安全的加密类

```php
<?php
declare(strict_types=1);

class Encryption
{
    private string $method;
    private string $key;
    
    public function __construct(string $key, string $method = 'aes-256-cbc')
    {
        $this->key = $key;
        $this->method = $method;
    }
    
    public function encrypt(string $data): string
    {
        // 生成 IV
        $ivLength = openssl_cipher_iv_length($this->method);
        $iv = openssl_random_pseudo_bytes($ivLength);
        
        // 加密
        $encrypted = openssl_encrypt($data, $this->method, $this->key, 0, $iv);
        if ($encrypted === false) {
            throw new RuntimeException('Encryption failed');
        }
        
        // 将 IV 和加密数据组合（IV 可以公开）
        return base64_encode($iv . $encrypted);
    }
    
    public function decrypt(string $encryptedData): string
    {
        // 解码
        $data = base64_decode($encryptedData, true);
        if ($data === false) {
            throw new InvalidArgumentException('Invalid encrypted data');
        }
        
        // 提取 IV
        $ivLength = openssl_cipher_iv_length($this->method);
        $iv = substr($data, 0, $ivLength);
        $encrypted = substr($data, $ivLength);
        
        // 解密
        $decrypted = openssl_decrypt($encrypted, $this->method, $this->key, 0, $iv);
        if ($decrypted === false) {
            throw new RuntimeException('Decryption failed');
        }
        
        return $decrypted;
    }
}

// 使用
$encryption = new Encryption('your-secret-key-32-characters!');

$data = '敏感数据';
$encrypted = $encryption->encrypt($data);
echo "加密: {$encrypted}\n";

$decrypted = $encryption->decrypt($encrypted);
echo "解密: {$decrypted}\n";
```

**说明**：
- 封装加密和解密功能
- 自动处理 IV 的生成和存储
- IV 可以包含在加密数据中（IV 可以公开）

### 示例 3：密钥生成和管理

```php
<?php
declare(strict_types=1);

class KeyManager
{
    /**
     * 生成随机密钥
     */
    public static function generateKey(int $length = 32): string
    {
        return bin2hex(random_bytes($length));
    }
    
    /**
     * 从密码派生密钥（使用 PBKDF2）
     */
    public static function deriveKeyFromPassword(string $password, string $salt, int $iterations = 10000): string
    {
        return hash_pbkdf2('sha256', $password, $salt, $iterations, 32, true);
    }
    
    /**
     * 生成盐值
     */
    public static function generateSalt(int $length = 16): string
    {
        return random_bytes($length);
    }
}

// 使用
// 方法1：生成随机密钥
$key = KeyManager::generateKey(32);
echo "随机密钥: {$key}\n";

// 方法2：从密码派生密钥
$password = 'user_password';
$salt = KeyManager::generateSalt();
$derivedKey = KeyManager::deriveKeyFromPassword($password, $salt);
echo "派生密钥: " . bin2hex($derivedKey) . "\n";
```

**说明**：
- 生成随机密钥
- 从密码派生密钥（使用 PBKDF2）
- 生成盐值

### 示例 4：使用不同加密算法

```php
<?php
declare(strict_types=1);

$data = '敏感数据';
$key = 'your-secret-key-32-characters!';

// AES-256-CBC（推荐）
$method1 = 'aes-256-cbc';
$iv1 = openssl_random_pseudo_bytes(openssl_cipher_iv_length($method1));
$encrypted1 = openssl_encrypt($data, $method1, $key, 0, $iv1);
echo "AES-256-CBC: {$encrypted1}\n";

// AES-256-GCM（AEAD 模式，更安全，PHP 7.1+）
if (in_array('aes-256-gcm', openssl_get_cipher_methods())) {
    $method2 = 'aes-256-gcm';
    $iv2 = openssl_random_pseudo_bytes(openssl_cipher_iv_length($method2));
    $tag = '';
    $encrypted2 = openssl_encrypt($data, $method2, $key, 0, $iv2, $tag);
    echo "AES-256-GCM: {$encrypted2}\n";
    
    // 解密（需要 tag）
    $decrypted2 = openssl_decrypt($encrypted2, $method2, $key, 0, $iv2, $tag);
    echo "解密: {$decrypted2}\n";
}
```

**说明**：
- AES-256-CBC：常用的加密模式
- AES-256-GCM：AEAD 模式，提供认证加密（PHP 7.1+）

### 示例 5：加密类完整示例

```php
<?php
declare(strict_types=1);

class SecureEncryption
{
    private string $key;
    private string $method;
    
    public function __construct(string $key, string $method = 'aes-256-cbc')
    {
        $this->key = $this->validateKey($key, $method);
        $this->method = $method;
    }
    
    private function validateKey(string $key, string $method): string
    {
        $keyLength = $this->getKeyLength($method);
        if (strlen($key) !== $keyLength) {
            throw new InvalidArgumentException("Key length must be {$keyLength} bytes");
        }
        return $key;
    }
    
    private function getKeyLength(string $method): int
    {
        return match (true) {
            str_contains($method, '256') => 32,
            str_contains($method, '192') => 24,
            str_contains($method, '128') => 16,
            default => 32,
        };
    }
    
    public function encrypt(string $data): string
    {
        $ivLength = openssl_cipher_iv_length($this->method);
        $iv = openssl_random_pseudo_bytes($ivLength);
        
        $encrypted = openssl_encrypt($data, $this->method, $this->key, 0, $iv);
        if ($encrypted === false) {
            throw new RuntimeException('Encryption failed');
        }
        
        // 组合 IV 和加密数据
        return base64_encode($iv . $encrypted);
    }
    
    public function decrypt(string $encryptedData): string
    {
        $data = base64_decode($encryptedData, true);
        if ($data === false) {
            throw new InvalidArgumentException('Invalid encrypted data');
        }
        
        $ivLength = openssl_cipher_iv_length($this->method);
        if (strlen($data) < $ivLength) {
            throw new InvalidArgumentException('Invalid encrypted data format');
        }
        
        $iv = substr($data, 0, $ivLength);
        $encrypted = substr($data, $ivLength);
        
        $decrypted = openssl_decrypt($encrypted, $this->method, $this->key, 0, $iv);
        if ($decrypted === false) {
            throw new RuntimeException('Decryption failed');
        }
        
        return $decrypted;
    }
}

// 使用
$key = random_bytes(32);  // 32 字节密钥（AES-256）
$encryption = new SecureEncryption($key);

$data = '敏感数据';
$encrypted = $encryption->encrypt($data);
$decrypted = $encryption->decrypt($encrypted);

echo "原始: {$data}\n";
echo "加密: {$encrypted}\n";
echo "解密: {$decrypted}\n";
```

**说明**：
- 完整的加密类实现
- 验证密钥长度
- 安全的 IV 处理

## 使用场景

### 场景 1：加密存储敏感数据

加密存储数据库中的敏感数据。

**示例**：

```php
<?php
declare(strict_types=1);

$encryption = new SecureEncryption($key);

// 存储加密数据
$sensitiveData = 'user_credit_card';
$encrypted = $encryption->encrypt($sensitiveData);
// $db->insert('users', ['encrypted_data' => $encrypted]);

// 读取并解密
// $encrypted = $db->getEncryptedData($userId);
$decrypted = $encryption->decrypt($encrypted);
```

### 场景 2：传输加密数据

在网络上传输加密数据。

**示例**：

```php
<?php
declare(strict_types=1);

$encryption = new SecureEncryption($key);

// 加密数据用于传输
$data = json_encode(['user_id' => 123, 'amount' => 100.00]);
$encrypted = $encryption->encrypt($data);

// 传输加密数据
// sendToServer($encrypted);

// 服务器端解密
$decrypted = $encryption->decrypt($encrypted);
$data = json_decode($decrypted, true);
```

## 注意事项

### 密钥管理

密钥应该安全存储，不要硬编码在代码中。

**示例**：

```php
<?php
declare(strict_types=1);

// ❌ 错误：硬编码密钥
// $key = 'hardcoded-key';

// ✅ 正确：从环境变量或配置文件读取
$key = getenv('ENCRYPTION_KEY') ?: throw new RuntimeException('Encryption key not set');
```

### IV 的使用

每次加密都应该使用新的随机 IV，IV 可以公开但必须唯一。

**示例**：

```php
<?php
declare(strict_types=1);

// ✅ 正确：每次生成新的 IV
$iv = openssl_random_pseudo_bytes($ivLength);

// ❌ 错误：重复使用 IV
// $iv = 'fixed-iv';  // 不安全
```

### 算法选择

使用安全的加密算法，推荐 AES-256-CBC 或 AES-256-GCM。

**示例**：

```php
<?php
declare(strict_types=1);

// ✅ 推荐：AES-256-CBC 或 AES-256-GCM
$method = 'aes-256-cbc';
// 或
$method = 'aes-256-gcm';

// ❌ 不推荐：DES（不安全）
// $method = 'des';
```

## 常见问题

### 问题 1：如何生成安全的密钥？

**回答**：使用 `random_bytes()` 生成随机密钥，或从密码派生密钥。

**示例**：

```php
<?php
declare(strict_types=1);

// 生成随机密钥
$key = random_bytes(32);  // 32 字节用于 AES-256

// 或从密码派生
$key = hash_pbkdf2('sha256', $password, $salt, 10000, 32, true);
```

### 问题 2：密钥应该存储在哪里？

**回答**：密钥应该存储在安全的位置，如环境变量、密钥管理服务，不要硬编码在代码中。

**示例**：

```php
<?php
declare(strict_types=1);

// 从环境变量读取
$key = getenv('ENCRYPTION_KEY');
```

### 问题 3：IV 需要保密吗？

**回答**：IV 不需要保密，但必须唯一且随机。IV 可以包含在加密数据中。

**示例**：见"示例 2：安全的加密类"

### 问题 4：如何选择加密算法？

**回答**：推荐使用 AES-256-CBC 或 AES-256-GCM（PHP 7.1+），避免使用 DES 等不安全的算法。

## 最佳实践

### 1. 使用安全的加密算法

使用 AES-256-CBC 或 AES-256-GCM 等安全算法。

**示例**：

```php
<?php
declare(strict_types=1);

// ✅ 推荐
$method = 'aes-256-cbc';
// 或
$method = 'aes-256-gcm';
```

### 2. 安全管理密钥

密钥应该存储在安全的位置，不要硬编码。

**示例**：见"密钥管理"部分

### 3. 每次使用新的 IV

每次加密都使用新的随机 IV。

**示例**：见"IV 的使用"部分

### 4. 验证加密和解密结果

检查加密和解密操作的返回值，处理错误情况。

**示例**：

```php
<?php
declare(strict_types=1);

$encrypted = openssl_encrypt($data, $method, $key, 0, $iv);
if ($encrypted === false) {
    throw new RuntimeException('Encryption failed');
}
```

### 5. 使用经过验证的加密库

使用 PHP 的 OpenSSL 扩展，不要自己实现加密算法。

**示例**：

```php
<?php
declare(strict_types=1);

// ✅ 使用 OpenSSL 扩展
// 不要自己实现加密算法
```

## 对比分析

### 加密 vs 哈希

| 特性         | 加密                           | 哈希                           |
|:-------------|:-------------------------------|:-------------------------------|
| **方向性**   | 双向（可解密）                 | 单向（不可逆）                 |
| **用途**     | 保护需要恢复的数据             | 保护不需要恢复的数据（如密码） |
| **密钥**     | ✅ 需要密钥                    | ❌ 不需要密钥                  |
| **使用场景** | 加密存储、传输数据             | 密码存储、数据完整性验证       |

### AES-256-CBC vs AES-256-GCM

| 特性         | AES-256-CBC                    | AES-256-GCM                    |
|:-------------|:-------------------------------|:-------------------------------|
| **认证**     | ⚠️ 不提供认证                  | ✅ 提供认证（AEAD）            |
| **安全性**   | ✅ 高                          | ✅ 更高                        |
| **PHP 版本** | PHP 5.3+                       | PHP 7.1+                       |
| **性能**     | ✅ 较快                        | ⚠️ 稍慢                        |

## 练习任务

1. **加密工具类**：创建一个完整的加密工具类，封装加密和解密功能。

2. **密钥管理工具**：实现一个密钥管理工具，包括密钥生成、存储、轮换等功能。

3. **数据加密存储工具**：创建一个工具，安全地加密存储敏感数据。

4. **加密通信工具**：编写一个工具，实现客户端和服务器之间的加密通信。

5. **加密最佳实践指南**：创建一个指南，总结数据加密的最佳实践。

## 相关章节

- **[4.6.1 密码哈希](section-01-password-hashing.md)**：了解密码哈希的相关内容
- **[4.6.3 哈希函数](section-03-hash-functions.md)**：了解哈希函数的使用
- **[4.6.5 数据持久化](section-05-data-persistence.md)**：了解数据持久化的安全方法
