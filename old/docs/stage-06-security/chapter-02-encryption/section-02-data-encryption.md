# 6.2.2 数据加密

## 概述

数据加密用于保护敏感数据。本节详细介绍 Sodium 库、对称加密、非对称加密、数字签名，以及完整的加密系统实现。

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

## 完整示例

```php
<?php
declare(strict_types=1);

class EncryptionService
{
    private string $key;

    public function __construct(string $key)
    {
        $this->key = $key;
    }

    public function encrypt(string $data): string
    {
        $nonce = random_bytes(SODIUM_CRYPTO_SECRETBOX_NONCEBYTES);
        $ciphertext = sodium_crypto_secretbox($data, $nonce, $this->key);
        return base64_encode($nonce . $ciphertext);
    }

    public function decrypt(string $encrypted): string
    {
        $data = base64_decode($encrypted);
        $nonce = substr($data, 0, SODIUM_CRYPTO_SECRETBOX_NONCEBYTES);
        $ciphertext = substr($data, SODIUM_CRYPTO_SECRETBOX_NONCEBYTES);
        return sodium_crypto_secretbox_open($ciphertext, $nonce, $this->key);
    }
}
```

## 注意事项

1. **密钥管理**：安全存储和管理密钥
2. **Nonce 使用**：每次加密使用新的 nonce
3. **算法选择**：使用 Sodium 提供的现代算法
4. **性能考虑**：加密可能影响性能

## 练习

1. 实现完整的加密系统，支持对称和非对称加密。

2. 创建一个数字签名系统。

3. 编写密钥管理工具。

4. 实现加密数据存储系统。
