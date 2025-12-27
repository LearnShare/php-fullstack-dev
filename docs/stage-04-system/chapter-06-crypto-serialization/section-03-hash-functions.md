# 4.6.3 哈希函数

## 概述

哈希函数是将任意长度的数据映射为固定长度哈希值的函数。哈希函数用于数据完整性验证、消息认证、唯一标识等场景。与密码哈希不同，通用哈希函数（如 MD5、SHA256）不适合用于密码存储，但可以用于其他目的，如文件完整性验证、数据去重等。

PHP 提供了 `hash()` 和 `hash_hmac()` 等函数来实现哈希功能。理解哈希函数的使用方法、不同算法的特点，以及如何用于数据完整性验证，对于开发安全的 PHP 应用很重要。

**主要内容**：
- 哈希函数概述（什么是哈希函数、哈希函数的特性）
- `hash()` 函数（计算哈希值）
- `hash_hmac()` 函数（带密钥的哈希）
- 常见哈希算法（MD5、SHA1、SHA256、SHA512 等）
- 数据完整性验证
- 文件哈希计算
- 实际应用场景和注意事项

## 特性

- **固定长度**：无论输入多长，输出长度固定
- **单向性**：从哈希值无法还原原始数据
- **确定性**：相同输入总是产生相同输出
- **快速计算**：计算速度快
- **雪崩效应**：输入微小变化导致输出巨大变化

## 语法/定义

### hash() 函数

**语法**：`hash(string $algo, string $data, bool $binary = false): string|false`

**参数**：
- `$algo`：哈希算法名称（如 `'md5'`, `'sha256'`, `'sha512'` 等）
- `$data`：要哈希的数据
- `$binary`：可选，如果为 `true` 返回原始二进制数据，否则返回十六进制字符串

**返回值**：成功返回哈希值，失败返回 `false`。

### hash_hmac() 函数

**语法**：`hash_hmac(string $algo, string $data, string $key, bool $binary = false): string|false`

**参数**：
- `$algo`：哈希算法名称
- `$data`：要哈希的数据
- `$key`：密钥
- `$binary`：可选，如果为 `true` 返回原始二进制数据

**返回值**：成功返回 HMAC 哈希值，失败返回 `false`。

### hash_algos() 函数

**语法**：`hash_algos(): array`

**参数**：无参数

**返回值**：返回支持的哈希算法列表。

## 基本用法

### 示例 1：基本哈希计算

```php
<?php
declare(strict_types=1);

$data = 'Hello, World!';

// 计算 MD5 哈希
$md5 = hash('md5', $data);
echo "MD5: {$md5}\n";

// 计算 SHA256 哈希
$sha256 = hash('sha256', $data);
echo "SHA256: {$sha256}\n";

// 计算 SHA512 哈希
$sha512 = hash('sha512', $data);
echo "SHA512: {$sha512}\n";

// 返回二进制数据
$binary = hash('sha256', $data, true);
echo "SHA256 (binary): " . bin2hex($binary) . "\n";
```

**说明**：
- `hash()` 函数计算数据的哈希值
- 支持多种哈希算法
- 默认返回十六进制字符串

### 示例 2：使用 hash_hmac() 计算带密钥的哈希

```php
<?php
declare(strict_types=1);

$data = '重要数据';
$key = 'secret-key';

// 计算 HMAC-SHA256
$hmac = hash_hmac('sha256', $data, $key);
echo "HMAC-SHA256: {$hmac}\n";

// 验证 HMAC
function verifyHMAC(string $data, string $key, string $expectedHMAC): bool
{
    $calculatedHMAC = hash_hmac('sha256', $data, $key);
    return hash_equals($calculatedHMAC, $expectedHMAC);
}

$data = '重要数据';
$key = 'secret-key';
$hmac = hash_hmac('sha256', $data, $key);

if (verifyHMAC($data, $key, $hmac)) {
    echo "HMAC 验证成功\n";
} else {
    echo "HMAC 验证失败\n";
}
```

**说明**：
- `hash_hmac()` 使用密钥计算哈希值
- 适合用于消息认证
- 使用 `hash_equals()` 进行安全比较（防止时序攻击）

### 示例 3：文件哈希计算

```php
<?php
declare(strict_types=1);

function calculateFileHash(string $filename, string $algorithm = 'sha256'): string|false
{
    if (!file_exists($filename)) {
        return false;
    }
    
    // 方法1：使用 hash_file()
    return hash_file($algorithm, $filename);
}

// 使用
$filename = __DIR__ . '/data.txt';
$hash = calculateFileHash($filename, 'sha256');
if ($hash !== false) {
    echo "文件哈希: {$hash}\n";
}

// 方法2：使用 hash_init() 和 hash_update_file()
function calculateFileHashStreaming(string $filename, string $algorithm = 'sha256'): string|false
{
    if (!file_exists($filename)) {
        return false;
    }
    
    $context = hash_init($algorithm);
    hash_update_file($context, $filename);
    return hash_final($context);
}

$hash = calculateFileHashStreaming($filename, 'sha256');
echo "文件哈希（流式）: {$hash}\n";
```

**说明**：
- `hash_file()` 直接计算文件哈希
- `hash_init()`、`hash_update_file()`、`hash_final()` 用于流式计算
- 适合大文件

### 示例 4：数据完整性验证

```php
<?php
declare(strict_types=1);

class DataIntegrity
{
    public static function calculateHash(string $data): string
    {
        return hash('sha256', $data);
    }
    
    public static function verifyIntegrity(string $data, string $expectedHash): bool
    {
        $calculatedHash = self::calculateHash($data);
        return hash_equals($calculatedHash, $expectedHash);
    }
    
    public static function calculateHMAC(string $data, string $key): string
    {
        return hash_hmac('sha256', $data, $key);
    }
    
    public static function verifyHMAC(string $data, string $key, string $expectedHMAC): bool
    {
        $calculatedHMAC = self::calculateHMAC($data, $key);
        return hash_equals($calculatedHMAC, $expectedHMAC);
    }
}

// 使用
$data = '重要数据';
$hash = DataIntegrity::calculateHash($data);
echo "数据哈希: {$hash}\n";

// 传输后验证
if (DataIntegrity::verifyIntegrity($data, $hash)) {
    echo "数据完整性验证成功\n";
} else {
    echo "数据可能被篡改\n";
}

// 使用 HMAC（需要密钥）
$key = 'secret-key';
$hmac = DataIntegrity::calculateHMAC($data, $key);
echo "HMAC: {$hmac}\n";

if (DataIntegrity::verifyHMAC($data, $key, $hmac)) {
    echo "HMAC 验证成功\n";
}
```

**说明**：
- 计算数据的哈希值
- 验证数据完整性
- 使用 HMAC 提供更强的安全性

### 示例 5：列出支持的哈希算法

```php
<?php
declare(strict_types=1);

// 获取支持的哈希算法列表
$algorithms = hash_algos();

echo "支持的哈希算法:\n";
foreach ($algorithms as $algo) {
    echo "  - {$algo}\n";
}

// 测试不同算法
$data = 'test data';
foreach (['md5', 'sha1', 'sha256', 'sha512'] as $algo) {
    if (in_array($algo, $algorithms, true)) {
        $hash = hash($algo, $data);
        echo "{$algo}: {$hash}\n";
    }
}
```

**说明**：
- `hash_algos()` 返回支持的哈希算法列表
- 可以检查算法是否支持

### 示例 6：安全比较哈希值

```php
<?php
declare(strict_types=1);

// ❌ 不安全：使用 == 比较（容易受到时序攻击）
function unsafeCompare(string $a, string $b): bool
{
    return $a === $b;  // 不安全
}

// ✅ 安全：使用 hash_equals()（防止时序攻击）
function safeCompare(string $a, string $b): bool
{
    return hash_equals($a, $b);  // 安全
}

$hash1 = hash('sha256', 'data');
$hash2 = hash('sha256', 'data');

// 使用安全比较
if (hash_equals($hash1, $hash2)) {
    echo "哈希值匹配\n";
} else {
    echo "哈希值不匹配\n";
}
```

**说明**：
- `hash_equals()` 进行安全比较，防止时序攻击
- 比较哈希值或 HMAC 时应该使用 `hash_equals()`

## 使用场景

### 场景 1：文件完整性验证

验证文件是否被修改。

**示例**：

```php
<?php
declare(strict_types=1);

// 计算文件哈希
$fileHash = hash_file('sha256', 'download.zip');

// 存储哈希值
file_put_contents('download.zip.sha256', $fileHash);

// 验证文件完整性
function verifyFileIntegrity(string $filename): bool
{
    $hashFile = $filename . '.sha256';
    if (!file_exists($hashFile)) {
        return false;
    }
    
    $expectedHash = trim(file_get_contents($hashFile));
    $actualHash = hash_file('sha256', $filename);
    
    return hash_equals($expectedHash, $actualHash);
}

if (verifyFileIntegrity('download.zip')) {
    echo "文件完整性验证成功\n";
} else {
    echo "文件可能被篡改\n";
}
```

### 场景 2：消息认证

使用 HMAC 进行消息认证。

**示例**：

```php
<?php
declare(strict_types=1);

function signMessage(string $message, string $key): string
{
    return hash_hmac('sha256', $message, $key);
}

function verifyMessage(string $message, string $key, string $signature): bool
{
    $calculatedSignature = signMessage($message, $key);
    return hash_equals($calculatedSignature, $signature);
}

// 使用
$message = '重要消息';
$key = 'secret-key';
$signature = signMessage($message, $key);

// 验证
if (verifyMessage($message, $key, $signature)) {
    echo "消息认证成功\n";
}
```

## 注意事项

### 不要用于密码存储

MD5、SHA1、SHA256 等通用哈希函数不适合用于密码存储，应该使用 `password_hash()`。

**示例**：

```php
<?php
declare(strict_types=1);

// ❌ 错误：使用 MD5 存储密码
// $passwordHash = hash('md5', $password);

// ✅ 正确：使用 password_hash()
$passwordHash = password_hash($password, PASSWORD_DEFAULT);
```

### 使用安全的比较方法

比较哈希值时使用 `hash_equals()`，防止时序攻击。

**示例**：见"示例 6：安全比较哈希值"

### 算法选择

根据需求选择合适的算法：
- 文件完整性：SHA256、SHA512
- 消息认证：HMAC-SHA256、HMAC-SHA512
- 避免使用：MD5、SHA1（已被认为不安全）

**示例**：

```php
<?php
declare(strict_types=1);

// ✅ 推荐：SHA256、SHA512
$hash = hash('sha256', $data);

// ⚠️ 不推荐：MD5、SHA1
// $hash = hash('md5', $data);  // 不安全
```

## 常见问题

### 问题 1：hash() 和 password_hash() 的区别？

**回答**：
- `hash()`：通用哈希函数，不适合密码存储
- `password_hash()`：专为密码设计的哈希函数，更安全

**示例**：

```php
<?php
declare(strict_types=1);

// hash() - 通用哈希（不适合密码）
$hash = hash('sha256', 'password');

// password_hash() - 密码哈希（推荐用于密码）
$passwordHash = password_hash('password', PASSWORD_DEFAULT);
```

### 问题 2：如何选择哈希算法？

**回答**：根据用途选择：
- 文件完整性：SHA256、SHA512
- 消息认证：HMAC-SHA256、HMAC-SHA512
- 密码存储：password_hash()（不是 hash()）

**示例**：见"算法选择"部分

### 问题 3：MD5 和 SHA1 为什么不安全？

**回答**：MD5 和 SHA1 已被发现存在碰撞攻击，不再安全，应该使用 SHA256、SHA512 等更安全的算法。

**示例**：

```php
<?php
declare(strict_types=1);

// ❌ 不推荐：MD5、SHA1
// $hash = hash('md5', $data);
// $hash = hash('sha1', $data);

// ✅ 推荐：SHA256、SHA512
$hash = hash('sha256', $data);
$hash = hash('sha512', $data);
```

### 问题 4：HMAC 的作用是什么？

**回答**：HMAC（Hash-based Message Authentication Code）使用密钥计算哈希值，提供消息认证，确保消息来源和完整性。

**示例**：见"示例 2：使用 hash_hmac() 计算带密钥的哈希"

## 最佳实践

### 1. 使用安全的哈希算法

使用 SHA256、SHA512 等安全算法，避免使用 MD5、SHA1。

**示例**：

```php
<?php
declare(strict_types=1);

// ✅ 推荐
$hash = hash('sha256', $data);
$hash = hash('sha512', $data);

// ❌ 不推荐
// $hash = hash('md5', $data);
// $hash = hash('sha1', $data);
```

### 2. 使用 hash_equals() 比较

使用 `hash_equals()` 安全比较哈希值，防止时序攻击。

**示例**：见"示例 6：安全比较哈希值"

### 3. 使用 HMAC 进行消息认证

需要消息认证时，使用 HMAC 而不是普通哈希。

**示例**：见"示例 2：使用 hash_hmac() 计算带密钥的哈希"

### 4. 不要用于密码存储

不要使用 `hash()` 存储密码，使用 `password_hash()`。

**示例**：

```php
<?php
declare(strict_types=1);

// ❌ 错误
// $passwordHash = hash('sha256', $password);

// ✅ 正确
$passwordHash = password_hash($password, PASSWORD_DEFAULT);
```

### 5. 文件哈希使用流式处理

对于大文件，使用流式处理计算哈希。

**示例**：见"示例 3：文件哈希计算"

## 对比分析

### hash() vs password_hash()

| 特性         | hash()                        | password_hash()               |
|:-------------|:------------------------------|:------------------------------|
| **用途**     | 通用哈希（文件完整性等）      | 密码哈希（密码存储）          |
| **安全性**   | ⚠️ 不适合密码                 | ✅ 专为密码设计               |
| **盐值**     | ❌ 需要手动加盐               | ✅ 自动加盐                   |
| **使用场景** | 文件完整性、消息认证          | 密码存储                      |

### MD5/SHA1 vs SHA256/SHA512

| 算法     | 安全性 | 输出长度 | 推荐使用 |
|:---------|:-------|:---------|:---------|
| **MD5**  | ❌ 不安全（碰撞攻击） | 128 位 | ❌ 不推荐 |
| **SHA1** | ❌ 不安全（碰撞攻击） | 160 位 | ❌ 不推荐 |
| **SHA256**| ✅ 安全 | 256 位 | ✅ 推荐 |
| **SHA512**| ✅ 安全 | 512 位 | ✅ 推荐 |

### 普通哈希 vs HMAC

| 特性         | 普通哈希（hash()）            | HMAC（hash_hmac()）           |
|:-------------|:------------------------------|:------------------------------|
| **密钥**     | ❌ 不需要密钥                 | ✅ 需要密钥                   |
| **安全性**   | ⚠️ 较低（无法验证来源）       | ✅ 较高（可以验证来源）       |
| **使用场景** | 数据完整性验证                | 消息认证                      |

## 练习任务

1. **哈希工具类**：创建一个工具类，封装哈希计算和验证功能。

2. **文件完整性验证工具**：实现一个工具，计算和验证文件的哈希值。

3. **消息认证工具**：创建一个工具，使用 HMAC 进行消息认证。

4. **哈希算法对比工具**：编写一个工具，对比不同哈希算法的特点。

5. **数据完整性系统**：实现一个完整的系统，用于数据的完整性验证。

## 相关章节

- **[4.6.1 密码哈希](section-01-password-hashing.md)**：了解密码哈希的相关内容
- **[4.6.2 数据加密](section-02-data-encryption.md)**：了解数据加密的相关内容
- **[4.6.5 数据持久化](section-05-data-persistence.md)**：了解数据持久化的相关内容
