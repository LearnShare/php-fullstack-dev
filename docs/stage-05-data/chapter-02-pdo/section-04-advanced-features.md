# 5.2.4 高级特性

## 概述

PDO 提供了许多高级特性，包括事务处理、错误处理、批量操作、性能优化等。本节详细介绍这些高级特性的使用方法和最佳实践。

## 事务处理

### 基础事务

```php
<?php
declare(strict_types=1);

$pdo->beginTransaction();
try {
    // 多个操作
    $stmt = $pdo->prepare("INSERT INTO users (username, email) VALUES (?, ?)");
    $stmt->execute(['alice', 'alice@example.com']);
    
    $stmt = $pdo->prepare("INSERT INTO profiles (user_id, bio) VALUES (?, ?)");
    $stmt->execute([$pdo->lastInsertId(), 'Bio text']);
    
    $pdo->commit();
} catch (Exception $e) {
    $pdo->rollBack();
    throw $e;
}
```

### 事务封装

```php
<?php
declare(strict_types=1);

function transaction(PDO $pdo, callable $callback): mixed
{
    $pdo->beginTransaction();
    try {
        $result = $callback($pdo);
        $pdo->commit();
        return $result;
    } catch (Exception $e) {
        $pdo->rollBack();
        throw $e;
    }
}

// 使用
transaction($pdo, function (PDO $pdo) {
    // 事务操作
    $stmt = $pdo->prepare("INSERT INTO users (username) VALUES (?)");
    $stmt->execute(['alice']);
    return $pdo->lastInsertId();
});
```

## 错误处理

### 错误模式

```php
<?php
declare(strict_types=1);

// ERRMODE_EXCEPTION（推荐）
$pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
try {
    $stmt = $pdo->prepare("SELECT * FROM invalid_table");
    $stmt->execute();
} catch (PDOException $e) {
    echo "Error: {$e->getMessage()}\n";
}
```

## 批量操作

### 批量插入

```php
<?php
declare(strict_types=1);

$users = [
    ['alice', 'alice@example.com'],
    ['bob', 'bob@example.com'],
    ['charlie', 'charlie@example.com'],
];

$pdo->beginTransaction();
$stmt = $pdo->prepare("INSERT INTO users (username, email) VALUES (?, ?)");
foreach ($users as $user) {
    $stmt->execute($user);
}
$pdo->commit();
```

### 批量更新

```php
<?php
declare(strict_types=1);

$updates = [
    [1, 'newemail1@example.com'],
    [2, 'newemail2@example.com'],
];

$pdo->beginTransaction();
$stmt = $pdo->prepare("UPDATE users SET email = ? WHERE id = ?");
foreach ($updates as $update) {
    $stmt->execute($update);
}
$pdo->commit();
```

## 性能优化

### 连接池

```php
<?php
declare(strict_types=1);

class ConnectionPool
{
    private array $connections = [];
    private int $maxConnections = 10;

    public function getConnection(): PDO
    {
        if (count($this->connections) < $this->maxConnections) {
            $pdo = new PDO($dsn, $user, $pass, $options);
            $this->connections[] = $pdo;
            return $pdo;
        }
        
        return $this->connections[array_rand($this->connections)];
    }
}
```

### 预处理语句复用

```php
<?php
declare(strict_types=1);

class StatementCache
{
    private PDO $pdo;
    private array $statements = [];

    public function prepare(string $sql): PDOStatement
    {
        $key = md5($sql);
        
        if (!isset($this->statements[$key])) {
            $this->statements[$key] = $this->pdo->prepare($sql);
        }
        
        return $this->statements[$key];
    }
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

class DatabaseService
{
    private PDO $pdo;

    public function __construct(PDO $pdo)
    {
        $this->pdo = $pdo;
    }

    public function transaction(callable $callback): mixed
    {
        $this->pdo->beginTransaction();
        try {
            $result = $callback($this->pdo);
            $this->pdo->commit();
            return $result;
        } catch (Exception $e) {
            $this->pdo->rollBack();
            throw $e;
        }
    }

    public function batchInsert(string $table, array $data): int
    {
        if (empty($data)) {
            return 0;
        }

        $fields = array_keys($data[0]);
        $placeholders = '(' . implode(',', array_fill(0, count($fields), '?')) . ')';
        $values = array_fill(0, count($data), $placeholders);
        
        $sql = "INSERT INTO {$table} (" . implode(',', $fields) . ") VALUES " . implode(',', $values);
        
        $stmt = $this->pdo->prepare($sql);
        $params = [];
        foreach ($data as $row) {
            $params = array_merge($params, array_values($row));
        }
        
        $stmt->execute($params);
        return $stmt->rowCount();
    }
}
```

## 注意事项

1. **事务管理**：合理使用事务，避免长时间占用连接
2. **错误处理**：使用异常模式便于错误处理
3. **性能优化**：复用预处理语句，使用连接池
4. **批量操作**：使用事务提升批量操作性能

## 练习

1. 创建一个事务管理类，封装事务操作。

2. 实现一个批量操作类，支持批量插入和更新。

3. 创建一个连接池管理类，管理数据库连接。

4. 实现一个预处理语句缓存机制，提升性能。
