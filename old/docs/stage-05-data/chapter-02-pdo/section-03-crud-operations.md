# 5.2.3 CRUD 操作

## 概述

CRUD（Create、Read、Update、Delete）是数据库操作的基础。本节详细介绍使用 PDO 进行创建、读取、更新、删除操作，以及各种获取模式的使用。

## Create（创建）

### 插入单条记录

```php
<?php
declare(strict_types=1);

// 插入单条记录
$stmt = $pdo->prepare("INSERT INTO users (username, email, password_hash) VALUES (?, ?, ?)");
$stmt->execute(['alice', 'alice@example.com', password_hash('password', PASSWORD_DEFAULT)]);

$userId = $pdo->lastInsertId();
echo "New user ID: {$userId}\n";
```

### 批量插入

```php
<?php
declare(strict_types=1);

$users = [
    ['bob', 'bob@example.com', password_hash('password', PASSWORD_DEFAULT)],
    ['charlie', 'charlie@example.com', password_hash('password', PASSWORD_DEFAULT)],
];

$stmt = $pdo->prepare("INSERT INTO users (username, email, password_hash) VALUES (?, ?, ?)");
foreach ($users as $user) {
    $stmt->execute($user);
}
```

## Read（读取）

### 查询单条记录

```php
<?php
declare(strict_types=1);

$stmt = $pdo->prepare("SELECT * FROM users WHERE id = ?");
$stmt->execute([1]);
$user = $stmt->fetch();

if ($user === false) {
    echo "User not found\n";
} else {
    echo "User: {$user['username']}\n";
}
```

### 查询多条记录

```php
<?php
declare(strict_types=1);

$stmt = $pdo->prepare("SELECT * FROM users WHERE status = ? ORDER BY created_at DESC");
$stmt->execute(['active']);
$users = $stmt->fetchAll();

foreach ($users as $user) {
    echo "User: {$user['username']}\n";
}
```

### 查询单列

```php
<?php
declare(strict_types=1);

$stmt = $pdo->prepare("SELECT username FROM users");
$stmt->execute();
$usernames = $stmt->fetchAll(PDO::FETCH_COLUMN);
```

### 查询单个值

```php
<?php
declare(strict_types=1);

$stmt = $pdo->prepare("SELECT COUNT(*) FROM users WHERE status = ?");
$stmt->execute(['active']);
$count = $stmt->fetchColumn();
```

## Update（更新）

### 更新记录

```php
<?php
declare(strict_types=1);

// 更新记录
$stmt = $pdo->prepare("UPDATE users SET email = ?, updated_at = NOW() WHERE id = ?");
$stmt->execute(['newemail@example.com', 1]);

$affectedRows = $stmt->rowCount();
echo "Updated {$affectedRows} row(s)\n";
```

### 更新多个字段

```php
<?php
declare(strict_types=1);

$stmt = $pdo->prepare("
    UPDATE users 
    SET username = ?, email = ?, updated_at = NOW() 
    WHERE id = ?
");
$stmt->execute(['newname', 'newemail@example.com', 1]);
```

## Delete（删除）

### 硬删除

```php
<?php
declare(strict_types=1);

// 删除记录
$stmt = $pdo->prepare("DELETE FROM users WHERE id = ?");
$stmt->execute([1]);

$affectedRows = $stmt->rowCount();
echo "Deleted {$affectedRows} row(s)\n";
```

### 软删除（推荐）

```php
<?php
declare(strict_types=1);

// 软删除（推荐）
$stmt = $pdo->prepare("UPDATE users SET deleted_at = NOW() WHERE id = ?");
$stmt->execute([1]);
```

## 获取模式

### 常用获取模式

| 模式 | 说明 | 示例 |
| :--- | :--- | :--- |
| `PDO::FETCH_ASSOC` | 关联数组 | `['id' => 1, 'name' => 'Alice']` |
| `PDO::FETCH_NUM` | 数字索引数组 | `[0 => 1, 1 => 'Alice']` |
| `PDO::FETCH_BOTH` | 同时返回关联和数字索引 | 两者结合 |
| `PDO::FETCH_OBJ` | 对象 | `$user->id, $user->name` |
| `PDO::FETCH_CLASS` | 映射到类 | `new User($data)` |
| `PDO::FETCH_COLUMN` | 单列 | `['Alice', 'Bob']` |

### 使用示例

```php
<?php
declare(strict_types=1);

// FETCH_ASSOC（默认，推荐）
$stmt = $pdo->prepare("SELECT * FROM users WHERE id = ?");
$stmt->execute([1]);
$user = $stmt->fetch(PDO::FETCH_ASSOC);
echo $user['username'];

// FETCH_OBJ
$stmt = $pdo->prepare("SELECT * FROM users WHERE id = ?");
$stmt->execute([1]);
$user = $stmt->fetch(PDO::FETCH_OBJ);
echo $user->username;

// FETCH_CLASS
class User
{
    public int $id;
    public string $username;
    public string $email;
}

$stmt = $pdo->prepare("SELECT * FROM users WHERE id = ?");
$stmt->setFetchMode(PDO::FETCH_CLASS, User::class);
$stmt->execute([1]);
$user = $stmt->fetch();
echo $user->username;
```

## 完整示例

### 用户管理类

```php
<?php
declare(strict_types=1);

class UserRepository
{
    private PDO $pdo;

    public function __construct(PDO $pdo)
    {
        $this->pdo = $pdo;
    }

    public function create(string $username, string $email, string $password): int
    {
        $stmt = $this->pdo->prepare(
            "INSERT INTO users (username, email, password_hash) VALUES (?, ?, ?)"
        );
        $stmt->execute([
            $username,
            $email,
            password_hash($password, PASSWORD_DEFAULT),
        ]);
        
        return (int) $this->pdo->lastInsertId();
    }

    public function findById(int $id): ?array
    {
        $stmt = $this->pdo->prepare("SELECT * FROM users WHERE id = ?");
        $stmt->execute([$id]);
        $user = $stmt->fetch();
        
        return $user !== false ? $user : null;
    }

    public function findByEmail(string $email): ?array
    {
        $stmt = $this->pdo->prepare("SELECT * FROM users WHERE email = ?");
        $stmt->execute([$email]);
        $user = $stmt->fetch();
        
        return $user !== false ? $user : null;
    }

    public function update(int $id, array $data): bool
    {
        $fields = [];
        $values = [];
        
        foreach ($data as $key => $value) {
            $fields[] = "{$key} = ?";
            $values[] = $value;
        }
        
        $values[] = $id;
        $sql = "UPDATE users SET " . implode(', ', $fields) . " WHERE id = ?";
        
        $stmt = $this->pdo->prepare($sql);
        return $stmt->execute($values);
    }

    public function delete(int $id): bool
    {
        $stmt = $this->pdo->prepare("UPDATE users SET deleted_at = NOW() WHERE id = ?");
        return $stmt->execute([$id]);
    }

    public function findAll(int $limit = 10, int $offset = 0): array
    {
        $stmt = $this->pdo->prepare(
            "SELECT * FROM users WHERE deleted_at IS NULL ORDER BY created_at DESC LIMIT ? OFFSET ?"
        );
        $stmt->execute([$limit, $offset]);
        return $stmt->fetchAll();
    }
}
```

## 注意事项

1. **使用预处理语句**：所有操作都使用预处理语句
2. **软删除**：优先使用软删除，保留数据
3. **获取模式**：根据需求选择合适的获取模式
4. **错误处理**：使用异常处理捕获错误

## 练习

1. 编写一个通用的数据库操作类，提供 find、create、update、delete 方法。

2. 实现一个分页查询函数，使用预处理语句和 LIMIT/OFFSET。

3. 创建一个批量插入函数，使用事务确保数据一致性。

4. 实现一个查询构建器，支持链式调用和安全的参数绑定。
