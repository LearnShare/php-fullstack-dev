# 4.6.5 数据持久化

## 概述

数据持久化是将数据安全地存储到持久化介质（如文件、数据库）的过程。在实际应用中，我们经常需要持久化数据，包括用户数据、配置信息、缓存数据等。理解如何安全地持久化数据，特别是敏感数据的处理，对于构建安全的应用至关重要。

本节总结数据持久化的最佳实践，包括存储格式选择、敏感数据处理、加密存储、数据完整性验证等，帮助零基础学员掌握安全的数据持久化方法。

**主要内容**：
- 数据持久化概述（什么是持久化、为什么需要持久化）
- 存储格式选择（JSON、XML、序列化、数据库）
- 敏感数据处理（密码哈希、数据加密）
- 加密存储策略
- 数据完整性验证（哈希验证、数字签名）
- 访问控制
- 备份策略
- 实际应用场景和最佳实践

## 特性

- **安全存储**：提供安全的存储策略
- **格式多样**：支持多种存储格式
- **完整性保证**：提供数据完整性验证方法
- **访问控制**：支持访问控制机制

## 语法/定义

### 数据持久化要点

数据持久化没有单一的语法定义，而是综合性的策略和实践方法。

## 基本用法

### 示例 1：安全存储密码

```php
<?php
declare(strict_types=1);

class SecurePasswordStorage
{
    public static function storePassword(string $username, string $password): void
    {
        // 使用 password_hash() 创建密码哈希
        $hash = password_hash($password, PASSWORD_DEFAULT);
        if ($hash === false) {
            throw new RuntimeException('Password hashing failed');
        }
        
        // 存储哈希值（不要存储原始密码）
        // $db->insert('users', ['username' => $username, 'password_hash' => $hash]);
    }
    
    public static function verifyPassword(string $username, string $password): bool
    {
        // 从数据库获取哈希值
        // $hash = $db->getPasswordHash($username);
        
        // 验证密码
        return password_verify($password, $hash);
    }
}
```

**说明**：
- 使用 `password_hash()` 存储密码哈希
- 永远不要存储明文密码

### 示例 2：加密存储敏感数据

```php
<?php
declare(strict_types=1);

class SecureDataStorage
{
    private string $encryptionKey;
    
    public function __construct(string $encryptionKey)
    {
        $this->encryptionKey = $encryptionKey;
    }
    
    public function storeSensitiveData(string $key, string $data): void
    {
        // 加密数据
        $encrypted = $this->encrypt($data);
        
        // 存储加密数据
        // $db->insert('encrypted_data', ['key' => $key, 'data' => $encrypted]);
    }
    
    public function retrieveSensitiveData(string $key): ?string
    {
        // 获取加密数据
        // $encrypted = $db->getEncryptedData($key);
        
        // 解密数据
        return $this->decrypt($encrypted);
    }
    
    private function encrypt(string $data): string
    {
        $method = 'aes-256-cbc';
        $ivLength = openssl_cipher_iv_length($method);
        $iv = openssl_random_pseudo_bytes($ivLength);
        
        $encrypted = openssl_encrypt($data, $method, $this->encryptionKey, 0, $iv);
        if ($encrypted === false) {
            throw new RuntimeException('Encryption failed');
        }
        
        return base64_encode($iv . $encrypted);
    }
    
    private function decrypt(string $encryptedData): string
    {
        $method = 'aes-256-cbc';
        $data = base64_decode($encryptedData, true);
        if ($data === false) {
            throw new InvalidArgumentException('Invalid encrypted data');
        }
        
        $ivLength = openssl_cipher_iv_length($method);
        $iv = substr($data, 0, $ivLength);
        $encrypted = substr($data, $ivLength);
        
        $decrypted = openssl_decrypt($encrypted, $method, $this->encryptionKey, 0, $iv);
        if ($decrypted === false) {
            throw new RuntimeException('Decryption failed');
        }
        
        return $decrypted;
    }
}
```

**说明**：
- 使用加密存储敏感数据
- 密钥应该安全管理

### 示例 3：数据完整性验证

```php
<?php
declare(strict_types=1);

class DataIntegrityStorage
{
    public static function storeWithIntegrity(string $filename, string $data): void
    {
        // 存储数据
        file_put_contents($filename, $data);
        
        // 计算哈希值
        $hash = hash_file('sha256', $filename);
        
        // 存储哈希值
        file_put_contents($filename . '.hash', $hash);
    }
    
    public static function verifyIntegrity(string $filename): bool
    {
        if (!file_exists($filename) || !file_exists($filename . '.hash')) {
            return false;
        }
        
        // 计算当前哈希值
        $currentHash = hash_file('sha256', $filename);
        
        // 读取存储的哈希值
        $storedHash = trim(file_get_contents($filename . '.hash'));
        
        // 比较哈希值（使用 hash_equals() 防止时序攻击）
        return hash_equals($currentHash, $storedHash);
    }
}

// 使用
DataIntegrityStorage::storeWithIntegrity('data.txt', '重要数据');

if (DataIntegrityStorage::verifyIntegrity('data.txt')) {
    echo "数据完整性验证成功\n";
} else {
    echo "数据可能被篡改\n";
}
```

**说明**：
- 使用哈希值验证数据完整性
- 使用 `hash_equals()` 安全比较

### 示例 4：配置文件安全存储

```php
<?php
declare(strict_types=1);

class SecureConfigStorage
{
    private string $encryptionKey;
    
    public function __construct(string $encryptionKey)
    {
        $this->encryptionKey = $encryptionKey;
    }
    
    public function saveConfig(array $config, string $filename): void
    {
        // 序列化配置
        $serialized = json_encode($config, JSON_PRETTY_PRINT);
        
        // 加密配置（如果包含敏感信息）
        $encrypted = $this->encrypt($serialized);
        
        // 存储配置
        file_put_contents($filename, $encrypted);
        
        // 设置文件权限（只允许所有者读写）
        chmod($filename, 0600);
    }
    
    public function loadConfig(string $filename): array
    {
        if (!file_exists($filename)) {
            throw new RuntimeException("Config file not found: {$filename}");
        }
        
        // 读取加密配置
        $encrypted = file_get_contents($filename);
        
        // 解密配置
        $serialized = $this->decrypt($encrypted);
        
        // 反序列化
        $config = json_decode($serialized, true);
        if ($config === null) {
            throw new RuntimeException('Invalid config format');
        }
        
        return $config;
    }
    
    private function encrypt(string $data): string
    {
        // 实现加密逻辑（见前面的示例）
        // ...
    }
    
    private function decrypt(string $encrypted): string
    {
        // 实现解密逻辑（见前面的示例）
        // ...
    }
}
```

**说明**：
- 敏感配置应该加密存储
- 设置适当的文件权限

### 示例 5：数据持久化最佳实践总结

```php
<?php
declare(strict_types=1);

/**
 * 数据持久化最佳实践
 */
class DataPersistenceBestPractices
{
    /**
     * 1. 密码存储：使用 password_hash()
     */
    public static function storePassword(string $password): string
    {
        return password_hash($password, PASSWORD_DEFAULT);
    }
    
    /**
     * 2. 敏感数据：加密存储
     */
    public static function storeSensitiveData(string $data, string $key): string
    {
        $method = 'aes-256-cbc';
        $iv = openssl_random_pseudo_bytes(openssl_cipher_iv_length($method));
        $encrypted = openssl_encrypt($data, $method, $key, 0, $iv);
        return base64_encode($iv . $encrypted);
    }
    
    /**
     * 3. 数据完整性：计算并存储哈希值
     */
    public static function storeWithHash(string $data): array
    {
        $hash = hash('sha256', $data);
        return ['data' => $data, 'hash' => $hash];
    }
    
    /**
     * 4. 配置存储：使用 JSON（可读性好）
     */
    public static function storeConfig(array $config, string $filename): void
    {
        $json = json_encode($config, JSON_PRETTY_PRINT | JSON_UNESCAPED_UNICODE);
        file_put_contents($filename, $json);
        chmod($filename, 0600);  // 设置文件权限
    }
    
    /**
     * 5. 访问控制：检查权限
     */
    public static function checkAccess(string $filename): bool
    {
        // 检查文件权限
        if (!is_readable($filename)) {
            return false;
        }
        
        // 检查用户权限
        // ...
        
        return true;
    }
}
```

**说明**：
- 总结数据持久化的最佳实践
- 提供各种场景的存储方法

## 使用场景

### 场景 1：用户数据存储

安全存储用户数据（密码、个人信息等）。

**示例**：见"示例 1：安全存储密码"和"示例 2：加密存储敏感数据"

### 场景 2：配置信息存储

存储应用配置信息。

**示例**：见"示例 4：配置文件安全存储"

### 场景 3：数据备份

实现数据备份和恢复。

**示例**：

```php
<?php
declare(strict_types=1);

function backupData(string $source, string $backup): void
{
    // 复制数据
    copy($source, $backup);
    
    // 计算备份文件的哈希值
    $hash = hash_file('sha256', $backup);
    file_put_contents($backup . '.hash', $hash);
}

function verifyBackup(string $backup): bool
{
    if (!file_exists($backup) || !file_exists($backup . '.hash')) {
        return false;
    }
    
    $currentHash = hash_file('sha256', $backup);
    $storedHash = trim(file_get_contents($backup . '.hash'));
    
    return hash_equals($currentHash, $storedHash);
}
```

## 注意事项

### 敏感数据处理

敏感数据（密码、个人信息等）应该加密或哈希存储。

**示例**：见"示例 1：安全存储密码"和"示例 2：加密存储敏感数据"

### 文件权限

设置适当的文件权限，保护存储的数据。

**示例**：

```php
<?php
declare(strict_types=1);

// 设置文件权限（只允许所有者读写）
chmod($filename, 0600);

// 或使用更严格的权限
chmod($filename, 0400);  // 只读
```

### 密钥管理

加密密钥应该安全存储，不要硬编码在代码中。

**示例**：

```php
<?php
declare(strict_types=1);

// ❌ 错误：硬编码密钥
// $key = 'hardcoded-key';

// ✅ 正确：从环境变量读取
$key = getenv('ENCRYPTION_KEY') ?: throw new RuntimeException('Encryption key not set');
```

### 数据完整性

使用哈希值验证数据完整性，确保数据未被篡改。

**示例**：见"示例 3：数据完整性验证"

## 常见问题

### 问题 1：如何安全地存储敏感数据？

**回答**：使用加密存储敏感数据，使用密码哈希存储密码，设置适当的文件权限。

**示例**：见"示例 2：加密存储敏感数据"

### 问题 2：存储格式如何选择？

**回答**：
- JSON：配置文件、API 数据交换（可读性好）
- 序列化：PHP 内部数据存储（保留对象方法）
- 数据库：结构化数据（查询、索引）

**示例**：

```php
<?php
declare(strict_types=1);

// JSON：配置文件
$config = ['db_host' => 'localhost'];
file_put_contents('config.json', json_encode($config));

// 序列化：PHP 内部数据
$data = ['complex' => new stdClass()];
file_put_contents('data.dat', serialize($data));

// 数据库：结构化数据
// $db->insert('users', $userData);
```

### 问题 3：如何保证数据完整性？

**回答**：计算并存储数据的哈希值，读取时验证哈希值。

**示例**：见"示例 3：数据完整性验证"

### 问题 4：如何管理加密密钥？

**回答**：密钥应该存储在安全的位置，如环境变量、密钥管理服务，不要硬编码在代码中。

**示例**：见"密钥管理"部分

## 最佳实践

### 1. 敏感数据加密存储

敏感数据应该加密存储，使用强加密算法。

**示例**：见"示例 2：加密存储敏感数据"

### 2. 密码哈希存储

使用 `password_hash()` 存储密码，永远不要存储明文密码。

**示例**：见"示例 1：安全存储密码"

### 3. 数据完整性验证

使用哈希值验证数据完整性，使用 `hash_equals()` 安全比较。

**示例**：见"示例 3：数据完整性验证"

### 4. 设置文件权限

设置适当的文件权限，保护存储的数据。

**示例**：

```php
<?php
declare(strict_types=1);

// 只允许所有者读写
chmod($filename, 0600);
```

### 5. 安全管理密钥

密钥应该存储在安全的位置，不要硬编码。

**示例**：见"密钥管理"部分

## 对比分析

### 存储格式对比

| 格式       | 可读性 | 跨语言 | 对象支持 | 使用场景           |
|:-----------|:-------|:-------|:---------|:-------------------|
| **JSON**   | ✅ 好  | ✅ 是  | ❌ 否    | 配置文件、API      |
| **序列化** | ⚠️ 差  | ❌ 否  | ✅ 是    | PHP 内部数据存储   |
| **数据库** | ✅ 好  | ✅ 是  | ⚠️ 部分  | 结构化数据         |

### 加密 vs 哈希

| 特性         | 加密                           | 哈希                           |
|:-------------|:-------------------------------|:-------------------------------|
| **方向性**   | 双向（可解密）                 | 单向（不可逆）                 |
| **用途**     | 保护需要恢复的数据             | 保护不需要恢复的数据（如密码） |
| **密钥**     | ✅ 需要密钥                    | ❌ 不需要密钥                  |
| **使用场景** | 敏感数据存储                   | 密码存储、数据完整性验证       |

## 练习任务

1. **安全数据存储工具类**：创建一个工具类，封装安全的数据存储功能。

2. **配置管理系统**：实现一个配置管理系统，支持加密存储配置。

3. **数据备份工具**：创建一个工具，实现数据备份和完整性验证。

4. **密钥管理系统**：编写一个密钥管理系统，安全地管理加密密钥。

5. **数据持久化框架**：实现一个完整的数据持久化框架，支持多种存储格式和安全策略。

## 相关章节

- **[4.6.1 密码哈希](section-01-password-hashing.md)**：了解密码哈希的相关内容
- **[4.6.2 数据加密](section-02-data-encryption.md)**：了解数据加密的相关内容
- **[4.6.4 序列化与反序列化](section-04-serialization.md)**：了解序列化的相关内容
- **[6.1 数据库操作](../../stage-06-database/chapter-01-basics/readme.md)**：了解数据库存储的相关内容
