# 6.2.1 密码加密

## 概述

密码安全是 Web 应用的基础。本节详细介绍 `password_hash`、`password_verify`、Argon2id 配置、密码策略，以及完整的密码处理示例。

## password_hash

### 语法

- **语法**：`password_hash(string $password, string|int|null $algo, array $options = []): string`
- **参数说明**：
  - `$password`：要哈希的密码字符串
  - `$algo`：哈希算法，推荐 `PASSWORD_ARGON2ID`（PHP 7.2+），或 `PASSWORD_DEFAULT`
  - `$options`：可选参数数组，如 `['memory_cost' => 65536, 'time_cost' => 4, 'threads' => 3]`
- **返回值**：返回哈希后的密码字符串，失败返回 `false`

### 使用示例

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

## password_verify

### 语法

- **语法**：`password_verify(string $password, string $hash): bool`
- **参数说明**：
  - `$password`：要验证的密码字符串
  - `$hash`：之前使用 `password_hash()` 生成的哈希值
- **返回值**：密码匹配返回 `true`，不匹配返回 `false`

### 使用示例

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

## Argon2id 最佳实践

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
```

## 密码策略

```php
<?php
declare(strict_types=1);

function validatePassword(string $password): bool
{
    // 至少 8 位，包含大小写字母、数字和特殊字符
    return preg_match('/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{8,}$/', $password);
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

class PasswordService
{
    private PDO $pdo;

    public function register(string $username, string $password): int
    {
        if (!validatePassword($password)) {
            throw new InvalidArgumentException('Password does not meet requirements');
        }
        
        $hash = PasswordHasher::hash($password);
        
        $stmt = $this->pdo->prepare("INSERT INTO users (username, password_hash) VALUES (?, ?)");
        $stmt->execute([$username, $hash]);
        
        return (int) $this->pdo->lastInsertId();
    }

    public function login(string $username, string $password): bool
    {
        $stmt = $this->pdo->prepare("SELECT password_hash FROM users WHERE username = ?");
        $stmt->execute([$username]);
        $hash = $stmt->fetchColumn();
        
        if ($hash === false) {
            return false;
        }
        
        if (PasswordHasher::verify($password, $hash)) {
            // 检查是否需要重新哈希
            if (PasswordHasher::needsRehash($hash)) {
                $newHash = PasswordHasher::hash($password);
                $stmt = $this->pdo->prepare("UPDATE users SET password_hash = ? WHERE username = ?");
                $stmt->execute([$newHash, $username]);
            }
            return true;
        }
        
        return false;
    }
}
```

## 注意事项

1. **算法选择**：优先使用 Argon2id
2. **参数配置**：根据服务器性能调整参数
3. **密码策略**：实施强密码策略
4. **重新哈希**：定期检查并更新哈希

## 练习

1. 实现完整的密码处理系统。

2. 创建一个密码策略验证工具。

3. 编写密码哈希迁移工具。

4. 实现密码强度检测。
