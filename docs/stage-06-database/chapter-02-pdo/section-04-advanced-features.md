# 6.2.4 高级特性

## 概述

PDO 提供了许多高级特性来增强数据库操作能力，包括事务处理、存储过程调用、批量操作、结果集处理等。这些高级特性对于处理复杂的业务逻辑、保证数据一致性、提高操作性能至关重要。

本节详细介绍 PDO 的高级特性，包括事务的 ACID 特性、存储过程的调用方法、批量操作的优化技巧、结果集的高级处理方式等，帮助零基础学员掌握 PDO 的高级应用。

**主要内容**：
- 事务处理（ACID 特性、beginTransaction、commit、rollBack）
- 存储过程调用
- 批量操作优化
- 结果集高级处理（fetch、fetchAll、fetchColumn、获取模式）
- 获取元数据
- 完整示例和最佳实践

---

## 特性

- **事务支持**：支持 ACID 事务，保证数据一致性
- **存储过程**：支持调用数据库存储过程
- **批量操作**：支持批量插入、更新、删除，提高性能
- **结果集处理**：提供多种结果集获取方式
- **元数据获取**：支持获取表结构、列信息等元数据

---

## 事务处理

### 事务概念

事务（Transaction）是一组数据库操作，作为一个不可分割的工作单元执行。事务具有 ACID 特性：

- **原子性（Atomicity）**：事务中的所有操作要么全部成功，要么全部失败
- **一致性（Consistency）**：事务执行前后，数据库保持一致状态
- **隔离性（Isolation）**：并发事务之间相互隔离
- **持久性（Durability）**：事务提交后，数据永久保存

### beginTransaction() 方法

`beginTransaction()` 方法开始一个事务。

**语法**：`PDO::beginTransaction(): bool`

**返回值**：成功返回 `true`，失败返回 `false`

**示例**：

```php
<?php
declare(strict_types=1);

try {
    $pdo->beginTransaction();
    
    // 执行多个操作
    
    $pdo->commit();
} catch (PDOException $e) {
    $pdo->rollBack();
    throw $e;
}
```

### commit() 方法

`commit()` 方法提交事务，使所有操作永久生效。

**语法**：`PDO::commit(): bool`

**返回值**：成功返回 `true`，失败返回 `false`

**示例**：

```php
<?php
declare(strict_types=1);

try {
    $pdo->beginTransaction();
    
    $stmt1 = $pdo->prepare('INSERT INTO users (name) VALUES (:name)');
    $stmt1->execute(['name' => 'John']);
    
    $stmt2 = $pdo->prepare('INSERT INTO posts (user_id, title) VALUES (:user_id, :title)');
    $stmt2->execute([
        'user_id' => $pdo->lastInsertId(),
        'title' => 'First Post'
    ]);
    
    $pdo->commit();  // 提交事务
    echo "操作成功\n";
} catch (PDOException $e) {
    $pdo->rollBack();  // 回滚事务
    echo "操作失败: " . $e->getMessage() . "\n";
}
```

### rollBack() 方法

`rollBack()` 方法回滚事务，撤销所有未提交的操作。

**语法**：`PDO::rollBack(): bool`

**返回值**：成功返回 `true`，失败返回 `false`

**示例**：

```php
<?php
declare(strict_types=1);

try {
    $pdo->beginTransaction();
    
    $stmt = $pdo->prepare('INSERT INTO users (name, email) VALUES (:name, :email)');
    $stmt->execute([
        'name' => 'John',
        'email' => 'john@example.com'
    ]);
    
    // 发生错误，回滚事务
    if (someCondition()) {
        throw new Exception('操作失败');
    }
    
    $pdo->commit();
} catch (Exception $e) {
    $pdo->rollBack();  // 回滚，撤销插入操作
    echo "操作失败，已回滚\n";
}
```

### 事务嵌套

PDO 不支持真正的事务嵌套，但可以通过保存点（Savepoint）模拟。

**MySQL 保存点示例**：

```php
<?php
declare(strict_types=1);

try {
    $pdo->beginTransaction();
    
    // 第一个操作
    $stmt1 = $pdo->prepare('INSERT INTO users (name) VALUES (:name)');
    $stmt1->execute(['name' => 'John']);
    
    // 创建保存点
    $pdo->exec('SAVEPOINT sp1');
    
    try {
        // 第二个操作
        $stmt2 = $pdo->prepare('INSERT INTO posts (user_id, title) VALUES (:user_id, :title)');
        $stmt2->execute([
            'user_id' => $pdo->lastInsertId(),
            'title' => 'Post'
        ]);
        
        // 回滚到保存点
        $pdo->exec('ROLLBACK TO SAVEPOINT sp1');
    } catch (PDOException $e) {
        // 回滚到保存点，保留第一个操作
        $pdo->exec('ROLLBACK TO SAVEPOINT sp1');
    }
    
    $pdo->commit();
} catch (PDOException $e) {
    $pdo->rollBack();
    throw $e;
}
```

### 事务最佳实践

**1. 及时提交或回滚**

事务应尽快提交或回滚，避免长时间锁定资源。

```php
<?php
declare(strict_types=1);

try {
    $pdo->beginTransaction();
    
    // 执行操作
    
    $pdo->commit();  // 尽快提交
} catch (PDOException $e) {
    $pdo->rollBack();  // 及时回滚
    throw $e;
}
```

**2. 避免在事务中进行长时间操作**

避免在事务中进行文件操作、网络请求等耗时操作。

**3. 处理死锁**

使用重试机制处理死锁。

```php
<?php
declare(strict_types=1);

function executeWithRetry(callable $callback, int $maxRetries = 3): mixed
{
    $attempts = 0;
    
    while ($attempts < $maxRetries) {
        try {
            return $callback();
        } catch (PDOException $e) {
            if ($e->getCode() == 40001 || $e->getCode() == 1213) {  // 死锁错误码
                $attempts++;
                if ($attempts >= $maxRetries) {
                    throw $e;
                }
                usleep(100000 * $attempts);  // 指数退避
                continue;
            }
            throw $e;
        }
    }
}
```

---

## 存储过程

### 存储过程调用

PDO 支持调用数据库存储过程。

**语法**：`CALL procedure_name(param1, param2, ...)`

**示例**：

```php
<?php
declare(strict_types=1);

// 调用无参数的存储过程
$stmt = $pdo->prepare('CALL get_all_users()');
$stmt->execute();
$users = $stmt->fetchAll();

// 调用带参数的存储过程
$stmt = $pdo->prepare('CALL get_user_by_id(:id)');
$stmt->execute(['id' => 1]);
$user = $stmt->fetch();
```

### 参数传递

存储过程可以接受输入参数、输出参数和输入输出参数。

**示例**：

```php
<?php
declare(strict_types=1);

// 输入参数
$stmt = $pdo->prepare('CALL create_user(:name, :email)');
$stmt->execute([
    'name' => 'John',
    'email' => 'john@example.com'
]);

// 输出参数（MySQL）
$stmt = $pdo->prepare('CALL get_user_count(@count)');
$stmt->execute();
$stmt->closeCursor();

$stmt = $pdo->query('SELECT @count AS count');
$result = $stmt->fetch();
echo "用户数量: {$result['count']}\n";
```

### 结果获取

存储过程可能返回多个结果集。

**示例**：

```php
<?php
declare(strict_types=1);

$stmt = $pdo->prepare('CALL get_users_and_posts()');
$stmt->execute();

// 获取第一个结果集
$users = $stmt->fetchAll();

// 移动到下一个结果集
$stmt->nextRowset();

// 获取第二个结果集
$posts = $stmt->fetchAll();
```

### 输出参数

处理存储过程的输出参数。

**示例**：

```php
<?php
declare(strict_types=1);

// MySQL 存储过程示例
// CREATE PROCEDURE get_user_info(IN user_id INT, OUT user_name VARCHAR(100), OUT user_email VARCHAR(100))

$stmt = $pdo->prepare('CALL get_user_info(:user_id, @user_name, @user_email)');
$stmt->execute(['user_id' => 1]);
$stmt->closeCursor();

// 获取输出参数
$stmt = $pdo->query('SELECT @user_name AS name, @user_email AS email');
$result = $stmt->fetch();
echo "用户名: {$result['name']}, 邮箱: {$result['email']}\n";
```

---

## 批量操作

### 批量插入

批量插入可以提高性能，特别适合需要插入大量数据的场景。

**方法 1：循环执行（推荐）**

```php
<?php
declare(strict_types=1);

try {
    $pdo->beginTransaction();
    
    $stmt = $pdo->prepare('INSERT INTO users (name, email, age) VALUES (:name, :email, :age)');
    
    $users = [
        ['name' => 'John', 'email' => 'john@example.com', 'age' => 30],
        ['name' => 'Jane', 'email' => 'jane@example.com', 'age' => 25],
        ['name' => 'Bob', 'email' => 'bob@example.com', 'age' => 35]
    ];
    
    foreach ($users as $user) {
        $stmt->execute($user);
    }
    
    $pdo->commit();
    echo "批量插入成功\n";
} catch (PDOException $e) {
    $pdo->rollBack();
    echo "批量插入失败: " . $e->getMessage() . "\n";
}
```

**方法 2：使用 bindParam()**

```php
<?php
declare(strict_types=1);

try {
    $pdo->beginTransaction();
    
    $stmt = $pdo->prepare('INSERT INTO users (name, email, age) VALUES (:name, :email, :age)');
    $stmt->bindParam(':name', $name, PDO::PARAM_STR);
    $stmt->bindParam(':email', $email, PDO::PARAM_STR);
    $stmt->bindParam(':age', $age, PDO::PARAM_INT);
    
    $users = [
        ['name' => 'John', 'email' => 'john@example.com', 'age' => 30],
        ['name' => 'Jane', 'email' => 'jane@example.com', 'age' => 25]
    ];
    
    foreach ($users as $userData) {
        $name = $userData['name'];
        $email = $userData['email'];
        $age = $userData['age'];
        $stmt->execute();
    }
    
    $pdo->commit();
} catch (PDOException $e) {
    $pdo->rollBack();
    throw $e;
}
```

### 批量更新

批量更新多条记录。

**示例**：

```php
<?php
declare(strict_types=1);

try {
    $pdo->beginTransaction();
    
    $stmt = $pdo->prepare('UPDATE users SET status = :status WHERE id = :id');
    
    $updates = [
        ['id' => 1, 'status' => 'active'],
        ['id' => 2, 'status' => 'inactive'],
        ['id' => 3, 'status' => 'active']
    ];
    
    foreach ($updates as $update) {
        $stmt->execute($update);
    }
    
    $pdo->commit();
    echo "批量更新成功\n";
} catch (PDOException $e) {
    $pdo->rollBack();
    echo "批量更新失败: " . $e->getMessage() . "\n";
}
```

### 批量删除

批量删除多条记录。

**示例**：

```php
<?php
declare(strict_types=1);

try {
    $pdo->beginTransaction();
    
    $stmt = $pdo->prepare('DELETE FROM users WHERE id = :id');
    
    $ids = [1, 2, 3, 4, 5];
    
    foreach ($ids as $id) {
        $stmt->execute(['id' => $id]);
    }
    
    $pdo->commit();
    echo "批量删除成功\n";
} catch (PDOException $e) {
    $pdo->rollBack();
    echo "批量删除失败: " . $e->getMessage() . "\n";
}
```

### 性能优化

**1. 使用事务**

批量操作时使用事务可以提高性能。

```php
<?php
declare(strict_types=1);

// 不使用事务：每条操作都需要网络往返
foreach ($users as $user) {
    $stmt->execute($user);
}

// 使用事务：减少网络往返
$pdo->beginTransaction();
foreach ($users as $user) {
    $stmt->execute($user);
}
$pdo->commit();
```

**2. 批量大小控制**

对于大量数据，分批处理，避免单次事务过大。

```php
<?php
declare(strict_types=1);

function batchInsert(PDO $pdo, array $users, int $batchSize = 1000): void
{
    $stmt = $pdo->prepare('INSERT INTO users (name, email) VALUES (:name, :email)');
    
    $total = count($users);
    $processed = 0;
    
    while ($processed < $total) {
        $batch = array_slice($users, $processed, $batchSize);
        
        try {
            $pdo->beginTransaction();
            
            foreach ($batch as $user) {
                $stmt->execute($user);
            }
            
            $pdo->commit();
            $processed += count($batch);
            
            echo "已处理 $processed / $total\n";
        } catch (PDOException $e) {
            $pdo->rollBack();
            throw $e;
        }
    }
}
```

---

## 结果集处理

### fetch() 方法详解

`fetch()` 方法支持多种获取模式。

**获取模式**：

| 模式 | 说明 | 示例 |
|:-----|:-----|:-----|
| `PDO::FETCH_ASSOC` | 关联数组 | `['id' => 1, 'name' => 'John']` |
| `PDO::FETCH_NUM` | 数字索引数组 | `[0 => 1, 1 => 'John']` |
| `PDO::FETCH_BOTH` | 关联数组和数字索引数组 | 两者结合 |
| `PDO::FETCH_OBJ` | 对象 | `stdClass { id: 1, name: 'John' }` |
| `PDO::FETCH_CLASS` | 指定类的对象 | 自定义类的实例 |
| `PDO::FETCH_COLUMN` | 单列值 | `'John'` |
| `PDO::FETCH_KEY_PAIR` | 键值对数组 | `[1 => 'John', 2 => 'Jane']` |

**示例**：

```php
<?php
declare(strict_types=1);

$stmt = $pdo->prepare('SELECT * FROM users WHERE id = :id');
$stmt->execute(['id' => 1]);

// 关联数组（默认）
$user = $stmt->fetch(PDO::FETCH_ASSOC);
echo $user['name'];

// 对象
$user = $stmt->fetch(PDO::FETCH_OBJ);
echo $user->name;

// 自定义类
class User
{
    public int $id;
    public string $name;
    public string $email;
}

$stmt = $pdo->prepare('SELECT * FROM users WHERE id = :id');
$stmt->execute(['id' => 1]);
$user = $stmt->fetch(PDO::FETCH_CLASS, User::class);
echo $user->name;
```

### fetchAll() 方法详解

`fetchAll()` 方法获取所有结果，支持多种获取模式。

**示例**：

```php
<?php
declare(strict_types=1);

$stmt = $pdo->prepare('SELECT * FROM users');

// 关联数组
$users = $stmt->fetchAll(PDO::FETCH_ASSOC);

// 对象数组
$users = $stmt->fetchAll(PDO::FETCH_OBJ);

// 自定义类数组
$users = $stmt->fetchAll(PDO::FETCH_CLASS, User::class);

// 键值对数组（第一列作为键，第二列作为值）
$stmt = $pdo->prepare('SELECT id, name FROM users');
$stmt->execute();
$users = $stmt->fetchAll(PDO::FETCH_KEY_PAIR);
// 结果: [1 => 'John', 2 => 'Jane']
```

### fetchColumn() 方法

`fetchColumn()` 方法获取单列值。

**示例**：

```php
<?php
declare(strict_types=1);

// 获取第一列
$stmt = $pdo->prepare('SELECT name FROM users');
$stmt->execute();

while ($name = $stmt->fetchColumn()) {
    echo $name . "\n";
}

// 获取指定列（索引从0开始）
$stmt = $pdo->prepare('SELECT name, email FROM users');
$stmt->execute();

while ($email = $stmt->fetchColumn(1)) {
    echo $email . "\n";
}
```

### 获取模式设置

可以设置默认的获取模式。

**示例**：

```php
<?php
declare(strict_types=1);

// 在连接时设置
$pdo = new PDO($dsn, $user, $pass, [
    PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC
]);

// 使用 setAttribute 设置
$pdo->setAttribute(PDO::ATTR_DEFAULT_FETCH_MODE, PDO::FETCH_OBJ);

// 在语句级别设置
$stmt = $pdo->prepare('SELECT * FROM users');
$stmt->setFetchMode(PDO::FETCH_CLASS, User::class);
$stmt->execute();
$users = $stmt->fetchAll();
```

---

## 获取元数据

### 获取列信息

使用 `getColumnMeta()` 方法获取列信息。

**语法**：`PDOStatement::getColumnMeta(int $column): array|false`

**示例**：

```php
<?php
declare(strict_types=1);

$stmt = $pdo->prepare('SELECT id, name, email FROM users WHERE id = :id');
$stmt->execute(['id' => 1]);
$user = $stmt->fetch();

// 获取列信息
$columnCount = $stmt->columnCount();
for ($i = 0; $i < $columnCount; $i++) {
    $meta = $stmt->getColumnMeta($i);
    echo "列名: {$meta['name']}, 类型: {$meta['native_type']}\n";
}
```

### 获取表信息

使用 `query()` 方法查询表信息。

**示例**：

```php
<?php
declare(strict_types=1);

// 获取表结构
$stmt = $pdo->query("DESCRIBE users");
$columns = $stmt->fetchAll(PDO::FETCH_ASSOC);

foreach ($columns as $column) {
    echo "字段: {$column['Field']}, 类型: {$column['Type']}\n";
}

// 获取所有表
$stmt = $pdo->query("SHOW TABLES");
$tables = $stmt->fetchAll(PDO::FETCH_COLUMN);

foreach ($tables as $table) {
    echo "表名: $table\n";
}
```

---

## 完整示例

### 转账操作示例（事务）

```php
<?php
declare(strict_types=1);

class TransferService
{
    private PDO $pdo;
    
    public function __construct(PDO $pdo)
    {
        $this->pdo = $pdo;
    }
    
    public function transfer(int $fromUserId, int $toUserId, float $amount): bool
    {
        try {
            $this->pdo->beginTransaction();
            
            // 检查转出账户余额
            $stmt = $this->pdo->prepare('SELECT balance FROM accounts WHERE user_id = :user_id FOR UPDATE');
            $stmt->execute(['user_id' => $fromUserId]);
            $fromAccount = $stmt->fetch();
            
            if (!$fromAccount || $fromAccount['balance'] < $amount) {
                throw new RuntimeException('余额不足');
            }
            
            // 扣除转出账户余额
            $stmt = $this->pdo->prepare('UPDATE accounts SET balance = balance - :amount WHERE user_id = :user_id');
            $stmt->execute([
                'amount' => $amount,
                'user_id' => $fromUserId
            ]);
            
            // 增加转入账户余额
            $stmt = $this->pdo->prepare('UPDATE accounts SET balance = balance + :amount WHERE user_id = :user_id');
            $stmt->execute([
                'amount' => $amount,
                'user_id' => $toUserId
            ]);
            
            // 记录转账日志
            $stmt = $this->pdo->prepare('INSERT INTO transfers (from_user_id, to_user_id, amount) VALUES (:from_user_id, :to_user_id, :amount)');
            $stmt->execute([
                'from_user_id' => $fromUserId,
                'to_user_id' => $toUserId,
                'amount' => $amount
            ]);
            
            $this->pdo->commit();
            return true;
        } catch (Exception $e) {
            $this->pdo->rollBack();
            error_log('转账失败: ' . $e->getMessage());
            return false;
        }
    }
}

// 使用
try {
    $pdo = new PDO('mysql:host=localhost;dbname=test', 'user', 'pass', [
        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION
    ]);
    
    $service = new TransferService($pdo);
    $result = $service->transfer(1, 2, 100.00);
    
    if ($result) {
        echo "转账成功\n";
    } else {
        echo "转账失败\n";
    }
} catch (PDOException $e) {
    echo '数据库错误: ' . $e->getMessage() . "\n";
}
```

---

## 使用场景

### 复杂业务逻辑

事务用于处理需要保证一致性的复杂业务逻辑。

**示例**：

```php
<?php
declare(strict_types=1);

// 订单创建：创建订单、扣减库存、记录日志
try {
    $pdo->beginTransaction();
    
    // 创建订单
    $stmt = $pdo->prepare('INSERT INTO orders (user_id, total) VALUES (:user_id, :total)');
    $stmt->execute(['user_id' => 1, 'total' => 100.00]);
    $orderId = $pdo->lastInsertId();
    
    // 扣减库存
    $stmt = $pdo->prepare('UPDATE products SET stock = stock - :quantity WHERE id = :id');
    $stmt->execute(['id' => 1, 'quantity' => 2]);
    
    // 记录日志
    $stmt = $pdo->prepare('INSERT INTO order_logs (order_id, action) VALUES (:order_id, :action)');
    $stmt->execute(['order_id' => $orderId, 'action' => 'created']);
    
    $pdo->commit();
} catch (PDOException $e) {
    $pdo->rollBack();
    throw $e;
}
```

### 数据一致性保证

使用事务保证数据一致性。

**示例**：

```php
<?php
declare(strict_types=1);

// 确保用户和用户资料同时创建
try {
    $pdo->beginTransaction();
    
    $stmt = $pdo->prepare('INSERT INTO users (name, email) VALUES (:name, :email)');
    $stmt->execute(['name' => 'John', 'email' => 'john@example.com']);
    $userId = $pdo->lastInsertId();
    
    $stmt = $pdo->prepare('INSERT INTO user_profiles (user_id, bio) VALUES (:user_id, :bio)');
    $stmt->execute(['user_id' => $userId, 'bio' => 'Bio']);
    
    $pdo->commit();
} catch (PDOException $e) {
    $pdo->rollBack();
    throw $e;
}
```

---

## 注意事项

### 事务范围

事务应尽可能小，只包含必要的操作。

```php
<?php
declare(strict_types=1);

// ✅ 好的做法：事务范围小
try {
    $pdo->beginTransaction();
    // 只包含相关的数据库操作
    $pdo->commit();
} catch (PDOException $e) {
    $pdo->rollBack();
}

// ❌ 不好的做法：事务范围过大
try {
    $pdo->beginTransaction();
    // 数据库操作
    // 文件操作（不应该在事务中）
    // 网络请求（不应该在事务中）
    $pdo->commit();
} catch (PDOException $e) {
    $pdo->rollBack();
}
```

### 错误处理

事务中的错误必须正确处理，确保回滚。

```php
<?php
declare(strict_types=1);

try {
    $pdo->beginTransaction();
    
    // 操作
    
    $pdo->commit();
} catch (PDOException $e) {
    // 必须回滚
    $pdo->rollBack();
    throw $e;
} catch (Exception $e) {
    // 非数据库异常也要回滚
    $pdo->rollBack();
    throw $e;
}
```

### 性能影响

事务会锁定资源，影响并发性能。

**建议**：

- 事务范围尽可能小
- 避免在事务中进行耗时操作
- 合理设置事务隔离级别

### 死锁处理

使用重试机制处理死锁。

```php
<?php
declare(strict_types=1);

function executeWithRetry(callable $callback, int $maxRetries = 3): mixed
{
    $attempts = 0;
    
    while ($attempts < $maxRetries) {
        try {
            return $callback();
        } catch (PDOException $e) {
            // MySQL 死锁错误码: 40001, 1213
            if ($e->getCode() == 40001 || $e->getCode() == 1213) {
                $attempts++;
                if ($attempts >= $maxRetries) {
                    throw $e;
                }
                usleep(100000 * $attempts);  // 指数退避
                continue;
            }
            throw $e;
        }
    }
}
```

---

## 常见问题

### 如何使用事务？

使用 `beginTransaction()`、`commit()`、`rollBack()` 方法。

```php
<?php
declare(strict_types=1);

try {
    $pdo->beginTransaction();
    // 操作
    $pdo->commit();
} catch (PDOException $e) {
    $pdo->rollBack();
    throw $e;
}
```

### 如何调用存储过程？

使用 `CALL` 语句调用存储过程。

```php
<?php
declare(strict_types=1);

$stmt = $pdo->prepare('CALL procedure_name(:param1, :param2)');
$stmt->execute(['param1' => $value1, 'param2' => $value2]);
$result = $stmt->fetchAll();
```

### 如何进行批量操作？

使用事务和循环执行预处理语句。

```php
<?php
declare(strict_types=1);

try {
    $pdo->beginTransaction();
    $stmt = $pdo->prepare('INSERT INTO users (name) VALUES (:name)');
    foreach ($users as $user) {
        $stmt->execute($user);
    }
    $pdo->commit();
} catch (PDOException $e) {
    $pdo->rollBack();
    throw $e;
}
```

### 如何处理结果集？

使用 `fetch()`、`fetchAll()`、`fetchColumn()` 等方法。

```php
<?php
declare(strict_types=1);

// 单条
$user = $stmt->fetch(PDO::FETCH_ASSOC);

// 多条
$users = $stmt->fetchAll(PDO::FETCH_ASSOC);

// 单列
$name = $stmt->fetchColumn();
```

---

## 最佳实践

### 合理使用事务

- 事务范围尽可能小
- 只包含相关的数据库操作
- 避免在事务中进行耗时操作

### 实现错误回滚

所有异常都应触发回滚，确保数据一致性。

```php
<?php
declare(strict_types=1);

try {
    $pdo->beginTransaction();
    // 操作
    $pdo->commit();
} catch (Exception $e) {
    $pdo->rollBack();
    throw $e;
}
```

### 优化批量操作

- 使用事务减少网络往返
- 控制批量大小，避免事务过大
- 分批处理大量数据

### 选择合适的结果获取方式

- 单条数据使用 `fetch()`
- 多条数据使用 `fetchAll()`
- 单列数据使用 `fetchColumn()`
- 根据需求选择合适的获取模式

---

## 练习任务

1. **事务处理**
   - 实现转账功能
   - 使用事务保证一致性
   - 处理事务错误
   - 测试并发情况

2. **批量操作**
   - 实现批量插入功能
   - 使用事务优化性能
   - 实现分批处理
   - 测试性能差异

3. **结果集处理**
   - 使用不同的获取模式
   - 实现自定义类映射
   - 处理多个结果集
   - 优化结果集处理

4. **存储过程**
   - 创建存储过程
   - 调用存储过程
   - 处理输出参数
   - 处理多个结果集

5. **综合应用**
   - 创建一个完整的业务类
   - 使用事务处理复杂逻辑
   - 实现批量操作
   - 优化性能

---

**相关章节**：

- [6.2.1 PDO 基础与连接](section-01-pdo-basics.md)
- [6.2.2 预处理语句与防注入](section-02-prepared-statements.md)
- [6.2.3 CRUD 操作](section-03-crud-operations.md)
- [6.3 并发控制与 MVCC](../chapter-03-concurrency/readme.md)
