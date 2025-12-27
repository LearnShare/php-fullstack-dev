# 4.6.1 密码哈希

## 概述

密码哈希是安全存储密码的标准方法。密码哈希使用单向哈希函数将密码转换为不可逆的哈希值，即使哈希值泄露，攻击者也无法直接还原出原始密码。在实际应用中，我们永远不应该存储明文密码，而应该存储密码的哈希值。

PHP 提供了 `password_hash()` 和 `password_verify()` 函数来安全地处理密码哈希。理解密码哈希的原理、使用方法，以及如何选择哈希算法，对于构建安全的应用至关重要。

**主要内容**：
- 密码哈希概述（为什么需要哈希、哈希与加密的区别）
- `password_hash()` 函数（创建密码哈希）
- `password_verify()` 函数（验证密码）
- `password_needs_rehash()` 函数（检查是否需要重新哈希）
- 哈希算法（PASSWORD_DEFAULT、PASSWORD_BCRYPT、PASSWORD_ARGON2ID）
- 密码策略
- 实际应用场景和最佳实践

## 特性

- **单向加密**：密码哈希是单向的，无法还原原始密码
- **自动加盐**：`password_hash()` 自动生成和添加盐值
- **算法更新**：支持算法更新，保持安全性
- **安全可靠**：使用经过验证的安全算法

## 语法/定义

### password_hash() 函数

**语法**：`password_hash(string $password, string|int|null $algo, array $options = []): string`

**参数**：
- `$password`：要哈希的密码字符串
- `$algo`：哈希算法（`PASSWORD_DEFAULT`、`PASSWORD_BCRYPT`、`PASSWORD_ARGON2ID` 等）
- `$options`：可选，算法选项数组

**返回值**：成功返回哈希字符串，失败返回 `false`。

### password_verify() 函数

**语法**：`password_verify(string $password, string $hash): bool`

**参数**：
- `$password`：要验证的密码
- `$hash`：之前使用 `password_hash()` 生成的哈希值

**返回值**：密码匹配返回 `true`，不匹配返回 `false`。

### password_needs_rehash() 函数

**语法**：`password_needs_rehash(string $hash, string|int|null $algo, array $options = []): bool`

**参数**：
- `$hash`：现有的哈希值
- `$algo`：当前使用的算法
- `$options`：算法选项

**返回值**：如果需要重新哈希返回 `true`，否则返回 `false`。

## 基本用法

### 示例 1：创建密码哈希

```php
<?php
declare(strict_types=1);

// 使用默认算法创建密码哈希
$password = 'my_secret_password';
$hash = password_hash($password, PASSWORD_DEFAULT);

if ($hash !== false) {
    echo "密码哈希: {$hash}\n";
    // 存储 $hash 到数据库，不要存储原始密码
} else {
    echo "密码哈希失败\n";
}
```

**说明**：
- `PASSWORD_DEFAULT` 使用当前 PHP 版本推荐的默认算法
- 哈希值包含算法、成本参数和盐值
- 每次调用 `password_hash()` 生成的哈希值都不同（因为自动加盐）

### 示例 2：验证密码

```php
<?php
declare(strict_types=1);

// 存储的哈希值（从数据库获取）
$storedHash = '$2y$10$abcdefghijklmnopqrstuvwxyz1234567890ABCDEFGHIJKLMNOPQRST';

// 用户输入的密码
$userPassword = 'my_secret_password';

// 验证密码
if (password_verify($userPassword, $storedHash)) {
    echo "密码正确\n";
    // 登录成功
} else {
    echo "密码错误\n";
    // 登录失败
}
```

**说明**：
- `password_verify()` 自动从哈希值中提取盐值和算法参数
- 验证过程是安全的，防止时序攻击

### 示例 3：使用不同的哈希算法

```php
<?php
declare(strict_types=1);

$password = 'my_password';

// 使用默认算法（推荐）
$hash1 = password_hash($password, PASSWORD_DEFAULT);

// 使用 bcrypt 算法（PHP 5.5+）
$hash2 = password_hash($password, PASSWORD_BCRYPT, ['cost' => 12]);

// 使用 Argon2ID 算法（PHP 7.2+，推荐）
if (defined('PASSWORD_ARGON2ID')) {
    $hash3 = password_hash($password, PASSWORD_ARGON2ID, [
        'memory_cost' => 65536,  // 64 MB
        'time_cost' => 4,        // 4 次迭代
        'threads' => 3,          // 3 个线程
    ]);
} else {
    echo "Argon2ID 不支持，使用默认算法\n";
}
```

**说明**：
- `PASSWORD_DEFAULT`：推荐使用，会自动使用最佳算法
- `PASSWORD_BCRYPT`：bcrypt 算法，成本参数控制迭代次数
- `PASSWORD_ARGON2ID`：Argon2ID 算法，更安全但需要 PHP 7.2+

### 示例 4：检查是否需要重新哈希

```php
<?php
declare(strict_types=1);

function updatePasswordHashIfNeeded(string $password, string $oldHash): string
{
    // 验证密码
    if (!password_verify($password, $oldHash)) {
        throw new InvalidArgumentException('Invalid password');
    }
    
    // 检查是否需要重新哈希（算法更新、参数更新等）
    if (password_needs_rehash($oldHash, PASSWORD_DEFAULT)) {
        // 使用新算法重新哈希
        $newHash = password_hash($password, PASSWORD_DEFAULT);
        
        // 更新数据库中的哈希值
        // updateUserPasswordHash($userId, $newHash);
        
        return $newHash;
    }
    
    return $oldHash;
}

// 使用
$oldHash = '$2y$10$oldhash...';
$password = 'my_password';

$newHash = updatePasswordHashIfNeeded($password, $oldHash);
echo "哈希值: {$newHash}\n";
```

**说明**：
- `password_needs_rehash()` 检查哈希是否需要更新
- 可以在用户登录时检查并更新旧的哈希值

### 示例 5：完整的密码管理类

```php
<?php
declare(strict_types=1);

class PasswordManager
{
    public static function hash(string $password): string
    {
        $hash = password_hash($password, PASSWORD_DEFAULT);
        if ($hash === false) {
            throw new RuntimeException('Password hashing failed');
        }
        return $hash;
    }
    
    public static function verify(string $password, string $hash): bool
    {
        return password_verify($password, $hash);
    }
    
    public static function needsRehash(string $hash): bool
    {
        return password_needs_rehash($hash, PASSWORD_DEFAULT);
    }
    
    public static function updateHash(string $password, string $oldHash): string
    {
        if (!self::verify($password, $oldHash)) {
            throw new InvalidArgumentException('Invalid password');
        }
        
        if (self::needsRehash($oldHash)) {
            return self::hash($password);
        }
        
        return $oldHash;
    }
}

// 使用
// 注册用户
$password = 'user_password';
$hash = PasswordManager::hash($password);
// 存储 $hash 到数据库

// 验证密码
$userInput = 'user_password';
if (PasswordManager::verify($userInput, $hash)) {
    echo "密码正确\n";
    
    // 检查是否需要更新哈希
    if (PasswordManager::needsRehash($hash)) {
        $newHash = PasswordManager::updateHash($userInput, $hash);
        // 更新数据库中的哈希值
    }
} else {
    echo "密码错误\n";
}
```

**说明**：
- 封装密码管理功能
- 提供哈希、验证、更新等方法

## 使用场景

### 场景 1：用户注册

用户注册时创建密码哈希。

**示例**：

```php
<?php
declare(strict_types=1);

function registerUser(string $username, string $password): void
{
    // 验证密码强度（应该实现）
    // validatePasswordStrength($password);
    
    // 创建密码哈希
    $hash = password_hash($password, PASSWORD_DEFAULT);
    
    // 存储到数据库（不要存储原始密码）
    // $db->insert('users', ['username' => $username, 'password_hash' => $hash]);
}
```

### 场景 2：用户登录

用户登录时验证密码。

**示例**：

```php
<?php
declare(strict_types=1);

function loginUser(string $username, string $password): bool
{
    // 从数据库获取哈希值
    // $hash = $db->getPasswordHash($username);
    
    // 验证密码
    if (password_verify($password, $hash)) {
        // 检查是否需要重新哈希
        if (password_needs_rehash($hash, PASSWORD_DEFAULT)) {
            $newHash = password_hash($password, PASSWORD_DEFAULT);
            // 更新数据库
        }
        return true;
    }
    
    return false;
}
```

## 注意事项

### 永远不要存储明文密码

永远不要存储原始密码，只存储哈希值。

**示例**：

```php
<?php
declare(strict_types=1);

// ❌ 错误：存储明文密码
// $db->insert('users', ['password' => $password]);

// ✅ 正确：存储哈希值
$hash = password_hash($password, PASSWORD_DEFAULT);
$db->insert('users', ['password_hash' => $hash]);
```

### 使用 PASSWORD_DEFAULT

推荐使用 `PASSWORD_DEFAULT`，它会自动使用最佳算法。

**示例**：

```php
<?php
declare(strict_types=1);

// ✅ 推荐
$hash = password_hash($password, PASSWORD_DEFAULT);

// ⚠️ 也可以指定算法，但不推荐（除非有特殊需求）
// $hash = password_hash($password, PASSWORD_BCRYPT);
```

### 密码强度要求

实施密码强度策略，要求用户使用强密码。

**示例**：

```php
<?php
declare(strict_types=1);

function validatePasswordStrength(string $password): bool
{
    // 至少 8 个字符
    if (strlen($password) < 8) {
        return false;
    }
    
    // 包含大小写字母、数字
    if (!preg_match('/[a-z]/', $password) ||
        !preg_match('/[A-Z]/', $password) ||
        !preg_match('/[0-9]/', $password)) {
        return false;
    }
    
    return true;
}
```

## 常见问题

### 问题 1：为什么不能存储明文密码？

**回答**：如果数据库泄露，明文密码会直接暴露，用户账户可能被盗用。哈希值即使泄露，也无法直接还原原始密码。

**示例**：

```php
<?php
declare(strict_types=1);

// ❌ 危险：如果数据库泄露，密码直接暴露
// $db->store($username, $password);

// ✅ 安全：即使数据库泄露，也无法还原密码
$hash = password_hash($password, PASSWORD_DEFAULT);
$db->store($username, $hash);
```

### 问题 2：password_hash() 和 md5() 的区别？

**回答**：
- `password_hash()`：使用安全的算法（bcrypt、Argon2），自动加盐，设计用于密码
- `md5()`：不安全的哈希算法，容易被破解，不应该用于密码

**示例**：

```php
<?php
declare(strict_types=1);

// ❌ 错误：使用 md5()
// $hash = md5($password);  // 不安全，容易被破解

// ✅ 正确：使用 password_hash()
$hash = password_hash($password, PASSWORD_DEFAULT);  // 安全
```

### 问题 3：如何选择哈希算法？

**回答**：推荐使用 `PASSWORD_DEFAULT`，它会自动使用当前 PHP 版本推荐的最佳算法。

**示例**：

```php
<?php
declare(strict_types=1);

// ✅ 推荐：使用默认算法
$hash = password_hash($password, PASSWORD_DEFAULT);

// 如果需要特定算法（通常不需要）
if (defined('PASSWORD_ARGON2ID')) {
    $hash = password_hash($password, PASSWORD_ARGON2ID);
} else {
    $hash = password_hash($password, PASSWORD_BCRYPT);
}
```

### 问题 4：如何更新旧密码哈希？

**回答**：在用户登录时，使用 `password_needs_rehash()` 检查，如果需要则重新哈希。

**示例**：见"示例 4：检查是否需要重新哈希"

## 最佳实践

### 1. 始终使用 password_hash()

永远使用 `password_hash()` 而不是 `md5()`、`sha1()` 等弱哈希算法。

**示例**：

```php
<?php
declare(strict_types=1);

// ✅ 正确
$hash = password_hash($password, PASSWORD_DEFAULT);

// ❌ 错误
// $hash = md5($password);
// $hash = sha1($password);
```

### 2. 使用 PASSWORD_DEFAULT

使用 `PASSWORD_DEFAULT` 让 PHP 自动选择最佳算法。

**示例**：

```php
<?php
declare(strict_types=1);

$hash = password_hash($password, PASSWORD_DEFAULT);
```

### 3. 设置合理的成本参数

对于 `PASSWORD_BCRYPT`，设置合理的成本参数（10-12 是推荐值）。

**示例**：

```php
<?php
declare(strict_types=1);

// 成本参数 12（推荐）
$hash = password_hash($password, PASSWORD_BCRYPT, ['cost' => 12]);
```

### 4. 实现密码强度策略

要求用户使用强密码（至少 8 个字符，包含大小写字母、数字等）。

**示例**：见"密码强度要求"部分

### 5. 定期更新哈希

在用户登录时检查并更新旧的哈希值。

**示例**：见"示例 4：检查是否需要重新哈希"

## 对比分析

### password_hash() vs md5()/sha1()

| 特性         | password_hash()              | md5()/sha1()                 |
|:-------------|:-----------------------------|:-----------------------------|
| **安全性**   | ✅ 安全（bcrypt、Argon2）    | ❌ 不安全（容易被破解）      |
| **盐值**     | ✅ 自动加盐                  | ❌ 需要手动加盐              |
| **设计目的** | ✅ 专为密码设计              | ❌ 通用哈希，不适合密码      |
| **推荐使用** | ✅ 推荐                      | ❌ 不推荐                    |

### PASSWORD_DEFAULT vs PASSWORD_BCRYPT vs PASSWORD_ARGON2ID

| 算法              | 可用版本 | 安全性 | 性能 | 推荐使用 |
|:------------------|:---------|:-------|:-----|:---------|
| **PASSWORD_DEFAULT** | PHP 5.5+ | ✅ 高（自动选择最佳） | ✅ 良好 | ✅ 推荐 |
| **PASSWORD_BCRYPT**  | PHP 5.5+ | ✅ 高   | ✅ 良好 | ⚠️ 可选 |
| **PASSWORD_ARGON2ID**| PHP 7.2+ | ✅ 最高 | ⚠️ 较慢 | ✅ 推荐（PHP 7.2+）|

## 练习任务

1. **密码管理工具类**：创建一个完整的密码管理工具类，封装所有密码相关操作。

2. **密码强度验证工具**：实现一个工具，验证密码强度并提供改进建议。

3. **密码哈希迁移工具**：创建一个工具，将旧的密码哈希迁移到新的算法。

4. **密码策略实施工具**：编写一个工具，实施和验证密码策略。

5. **用户认证系统**：实现一个完整的用户认证系统，包括注册、登录、密码重置等功能。

## 相关章节

- **[4.6.2 数据加密](section-02-data-encryption.md)**：了解数据加密的相关内容
- **[4.6.3 哈希函数](section-03-hash-functions.md)**：了解哈希函数的使用
