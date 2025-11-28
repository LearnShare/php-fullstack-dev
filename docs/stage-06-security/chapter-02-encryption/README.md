# 6.2 密码与加密安全

## 目标

- 掌握 `password_hash` 和 `password_verify` 的使用。
- 理解 Argon2id 参数的最佳实践。
- 熟悉 Sodium 密码库的使用（对称/非对称加密、签名）。
- 了解敏感数据脱敏的方法。

## 密码哈希

### password_hash

- **语法**：`password_hash(string $password, string|int|null $algo, array $options = []): string`
- **参数说明**：
  - `$password`：要哈希的密码字符串。
  - `$algo`：哈希算法，推荐 `PASSWORD_ARGON2ID`（PHP 7.2+），或 `PASSWORD_DEFAULT`（使用当前默认算法）。
  - `$options`：可选参数数组，如 `['memory_cost' => 65536, 'time_cost' => 4, 'threads' => 3]`。
- **返回值**：返回哈希后的密码字符串，失败返回 `false`。
- **异常**：可能抛出 `ValueError`（当参数无效时）。
- 使用安全的哈希算法生成密码哈希。

```php
<?php
declare(strict_types=1);

// 使用默认算法（当前是 bcrypt）
$hash = password_hash('password123', PASSWORD_DEFAULT);

// 使用 Argon2id（推荐，PHP 7.2+）
$hash = password_hash('password123', PASSWORD_ARGON2ID);

// 自定义参数
$hash = password_hash('password123', PASSWORD_ARGON2ID, [
    'memory_cost' => 65536,  // 64 MB
    'time_cost' => 4,        // 4 次迭代
    'threads' => 3,          // 3 个线程
]);
```

### password_verify

- **语法**：`password_verify(string $password, string $hash): bool`
- **参数说明**：
  - `$password`：要验证的密码字符串。
  - `$hash`：之前使用 `password_hash()` 生成的哈希值。
- **返回值**：密码匹配返回 `true`，不匹配返回 `false`。
- **异常**：不会抛出异常，总是返回布尔值。
- 验证密码是否匹配哈希值。

```php
<?php
declare(strict_types=1);

$hash = password_hash('password123', PASSWORD_ARGON2ID);

// 验证密码
if (password_verify('password123', $hash)) {
    echo "Password correct\n";
} else {
    echo "Password incorrect\n";
}
```

### Argon2id 最佳实践

```php
<?php
declare(strict_types=1);

class PasswordHasher
{
    private const MEMORY_COST = 65536;  // 64 MB
    private const TIME_COST = 4;       // 4 次迭代
    private const THREADS = 3;         // 3 个线程
    
    public static function hash(string $password): string
    {
        return password_hash($password, PASSWORD_ARGON2ID, [
            'memory_cost' => self::MEMORY_COST,
            'time_cost' => self::TIME_COST,
            'threads' => self::THREADS,
        ]);
    }
    
    public static function verify(string $password, string $hash): bool
    {
        return password_verify($password, $hash);
    }
    
    public static function needsRehash(string $hash): bool
    {
        return password_needs_rehash($hash, PASSWORD_ARGON2ID, [
            'memory_cost' => self::MEMORY_COST,
            'time_cost' => self::TIME_COST,
            'threads' => self::THREADS,
        ]);
    }
}

// 使用
$hash = PasswordHasher::hash('password123');

// 登录验证
if (PasswordHasher::verify($inputPassword, $storedHash)) {
    // 检查是否需要重新哈希（算法或参数更新）
    if (PasswordHasher::needsRehash($storedHash)) {
        $newHash = PasswordHasher::hash($inputPassword);
        // 更新数据库中的哈希
    }
}
```

## Sodium 密码库

### 对称加密

```php
<?php
declare(strict_types=1);

// 生成密钥
$key = sodium_crypto_secretbox_keygen();

// 加密
function encrypt(string $message, string $key): string
{
    $nonce = random_bytes(SODIUM_CRYPTO_SECRETBOX_NONCEBYTES);
    $ciphertext = sodium_crypto_secretbox($message, $nonce, $key);
    return base64_encode($nonce . $ciphertext);
}

// 解密
function decrypt(string $encrypted, string $key): string
{
    $data = base64_decode($encrypted);
    $nonce = substr($data, 0, SODIUM_CRYPTO_SECRETBOX_NONCEBYTES);
    $ciphertext = substr($data, SODIUM_CRYPTO_SECRETBOX_NONCEBYTES);
    return sodium_crypto_secretbox_open($ciphertext, $nonce, $key);
}

// 使用
$key = sodium_crypto_secretbox_keygen();
$encrypted = encrypt('Sensitive data', $key);
$decrypted = decrypt($encrypted, $key);
```

### 非对称加密

```php
<?php
declare(strict_types=1);

// 生成密钥对
$keypair = sodium_crypto_box_keypair();
$publicKey = sodium_crypto_box_publickey($keypair);
$secretKey = sodium_crypto_box_secretkey($keypair);

// 加密（使用接收方公钥）
function encryptFor(string $message, string $recipientPublicKey, string $senderSecretKey): string
{
    $nonce = random_bytes(SODIUM_CRYPTO_BOX_NONCEBYTES);
    $ciphertext = sodium_crypto_box($message, $nonce, $recipientPublicKey, $senderSecretKey);
    return base64_encode($nonce . $ciphertext);
}

// 解密（使用接收方私钥）
function decryptFrom(string $encrypted, string $senderPublicKey, string $recipientSecretKey): string
{
    $data = base64_decode($encrypted);
    $nonce = substr($data, 0, SODIUM_CRYPTO_BOX_NONCEBYTES);
    $ciphertext = substr($data, SODIUM_CRYPTO_BOX_NONCEBYTES);
    return sodium_crypto_box_open($ciphertext, $nonce, $senderPublicKey, $recipientSecretKey);
}
```

### 数字签名

```php
<?php
declare(strict_types=1);

// 生成签名密钥对
$signKeypair = sodium_crypto_sign_keypair();
$signPublicKey = sodium_crypto_sign_publickey($signKeypair);
$signSecretKey = sodium_crypto_sign_secretkey($signKeypair);

// 签名
function sign(string $message, string $secretKey): string
{
    return base64_encode(sodium_crypto_sign_detached($message, $secretKey));
}

// 验证签名
function verify(string $message, string $signature, string $publicKey): bool
{
    return sodium_crypto_sign_verify_detached(
        base64_decode($signature),
        $message,
        $publicKey
    );
}

// 使用
$message = 'Important data';
$signature = sign($message, $signSecretKey);

if (verify($message, $signature, $signPublicKey)) {
    echo "Signature valid\n";
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
    
    // 身份证号脱敏
    public static function maskIdCard(string $idCard): string
    {
        if (strlen($idCard) < 6) {
            return str_repeat('*', strlen($idCard));
        }
        
        return substr($idCard, 0, 3) . str_repeat('*', strlen($idCard) - 6) . substr($idCard, -3);
    }
}

// 使用
echo DataMasking::maskEmail('alice@example.com');  // al***@example.com
echo DataMasking::maskPhone('13812345678');  // 138****5678
echo DataMasking::maskCardNumber('1234567890123456');  // 1234********3456
```

## 完整示例

### 加密服务类

```php
<?php
declare(strict_types=1);

class EncryptionService
{
    private string $secretKey;
    
    public function __construct(string $secretKey)
    {
        $this->secretKey = $secretKey;
    }
    
    public function encrypt(string $data): string
    {
        $nonce = random_bytes(SODIUM_CRYPTO_SECRETBOX_NONCEBYTES);
        $ciphertext = sodium_crypto_secretbox($data, $nonce, $this->secretKey);
        return base64_encode($nonce . $ciphertext);
    }
    
    public function decrypt(string $encrypted): string
    {
        $data = base64_decode($encrypted);
        $nonce = substr($data, 0, SODIUM_CRYPTO_SECRETBOX_NONCEBYTES);
        $ciphertext = substr($data, SODIUM_CRYPTO_SECRETBOX_NONCEBYTES);
        
        $decrypted = sodium_crypto_secretbox_open($ciphertext, $nonce, $this->secretKey);
        if ($decrypted === false) {
            throw new RuntimeException('Decryption failed');
        }
        
        return $decrypted;
    }
}

// 使用
$encryption = new EncryptionService($secretKey);
$encrypted = $encryption->encrypt('Sensitive data');
$decrypted = $encryption->decrypt($encrypted);
```

## 最佳实践

### 1. 密码存储

- 使用 `password_hash` 和 `PASSWORD_ARGON2ID`。
- 设置合理的成本参数。
- 定期检查是否需要重新哈希。

### 2. 密钥管理

- 密钥存储在安全位置（环境变量、密钥管理服务）。
- 定期轮换密钥。
- 不同环境使用不同密钥。

### 3. 数据脱敏

- 日志中不记录敏感信息。
- API 响应中脱敏敏感字段。
- 数据库查询结果脱敏。

## 练习

1. 实现一个密码管理类，支持哈希、验证和重新哈希。

2. 创建一个加密服务类，使用 Sodium 进行对称加密。

3. 实现一个数据脱敏工具，支持邮箱、手机号、银行卡等脱敏。

4. 编写一个密钥管理类，支持密钥生成、存储和轮换。

5. 设计一个加密配置系统，安全地存储和读取加密的配置项。

6. 创建一个签名验证系统，使用 Sodium 进行数字签名和验证。
