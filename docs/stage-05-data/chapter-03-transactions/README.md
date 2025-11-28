# 5.3 MySQL 事务处理

## 目标

- 理解 ACID 四特性的含义与重要性。
- 掌握事务的基本操作：`beginTransaction()`、`commit()`、`rollBack()`。
- 熟悉 MySQL 事务隔离级别及其影响。
- 了解死锁的产生原因与重试策略。

## ACID 特性

### 什么是 ACID

- **Atomicity（原子性）**：事务中的所有操作要么全部成功，要么全部失败。
- **Consistency（一致性）**：事务执行前后数据库保持一致状态。
- **Isolation（隔离性）**：并发事务之间相互隔离，互不干扰。
- **Durability（持久性）**：事务提交后，数据永久保存。

### ACID 示例

```php
<?php
declare(strict_types=1);

// 转账操作：必须保证原子性
function transfer(PDO $pdo, int $fromId, int $toId, float $amount): void
{
    try {
        $pdo->beginTransaction();
        
        // 扣款
        $stmt = $pdo->prepare("UPDATE accounts SET balance = balance - ? WHERE id = ?");
        $stmt->execute([$amount, $fromId]);
        
        // 检查余额
        $stmt = $pdo->prepare("SELECT balance FROM accounts WHERE id = ?");
        $stmt->execute([$fromId]);
        $balance = $stmt->fetchColumn();
        
        if ($balance < 0) {
            throw new Exception('Insufficient balance');
        }
        
        // 存款
        $stmt = $pdo->prepare("UPDATE accounts SET balance = balance + ? WHERE id = ?");
        $stmt->execute([$amount, $toId]);
        
        $pdo->commit();
    } catch (Exception $e) {
        $pdo->rollBack();
        throw $e;
    }
}
```

## 事务基本操作

### beginTransaction

- **语法**：`beginTransaction(): bool`
- 开始一个新事务，关闭自动提交。

```php
<?php
declare(strict_types=1);

$pdo->beginTransaction();
```

### commit

- **语法**：`commit(): bool`
- 提交事务，使所有更改永久生效。

```php
<?php
declare(strict_types=1);

$pdo->commit();
```

### rollBack

- **语法**：`rollBack(): bool`
- 回滚事务，撤销所有未提交的更改。

```php
<?php
declare(strict_types=1);

$pdo->rollBack();
```

### 完整示例

```php
<?php
declare(strict_types=1);

function createOrder(PDO $pdo, int $userId, array $items): int
{
    try {
        $pdo->beginTransaction();
        
        // 1. 创建订单
        $stmt = $pdo->prepare("INSERT INTO orders (user_id, total, status) VALUES (?, 0, 'pending')");
        $stmt->execute([$userId]);
        $orderId = (int) $pdo->lastInsertId();
        
        $total = 0;
        
        // 2. 创建订单项并计算总价
        foreach ($items as $item) {
            $stmt = $pdo->prepare("SELECT price FROM products WHERE id = ?");
            $stmt->execute([$item['product_id']]);
            $price = $stmt->fetchColumn();
            
            if ($price === false) {
                throw new Exception("Product {$item['product_id']} not found");
            }
            
            $itemTotal = $price * $item['quantity'];
            $total += $itemTotal;
            
            $stmt = $pdo->prepare(
                "INSERT INTO order_items (order_id, product_id, quantity, price) VALUES (?, ?, ?, ?)"
            );
            $stmt->execute([$orderId, $item['product_id'], $item['quantity'], $price]);
            
            // 3. 更新库存
            $stmt = $pdo->prepare("UPDATE products SET stock = stock - ? WHERE id = ?");
            $stmt->execute([$item['quantity'], $item['product_id']]);
            
            // 检查库存
            $stmt = $pdo->prepare("SELECT stock FROM products WHERE id = ?");
            $stmt->execute([$item['product_id']]);
            $stock = $stmt->fetchColumn();
            
            if ($stock < 0) {
                throw new Exception("Insufficient stock for product {$item['product_id']}");
            }
        }
        
        // 4. 更新订单总价
        $stmt = $pdo->prepare("UPDATE orders SET total = ? WHERE id = ?");
        $stmt->execute([$total, $orderId]);
        
        $pdo->commit();
        return $orderId;
        
    } catch (Exception $e) {
        $pdo->rollBack();
        throw $e;
    }
}
```

## 事务隔离级别

### 隔离级别说明

| 隔离级别           | 脏读 | 不可重复读 | 幻读 | 说明                           |
| :----------------- | :--- | :--------- | :--- | :----------------------------- |
| READ UNCOMMITTED  | 是   | 是         | 是   | 最低隔离级别，性能最好         |
| READ COMMITTED    | 否   | 是         | 是   | 只能读取已提交的数据           |
| REPEATABLE READ   | 否   | 否         | 是   | MySQL InnoDB 默认级别          |
| SERIALIZABLE      | 否   | 否         | 否   | 最高隔离级别，性能最差         |

### 设置隔离级别

```php
<?php
declare(strict_types=1);

// 设置隔离级别
$pdo->exec("SET TRANSACTION ISOLATION LEVEL READ COMMITTED");

// 或在事务中设置
$pdo->beginTransaction();
$pdo->exec("SET TRANSACTION ISOLATION LEVEL SERIALIZABLE");
// ... 事务操作
$pdo->commit();
```

### 隔离级别示例

#### READ UNCOMMITTED（脏读）

```php
<?php
// 事务 A
$pdo->beginTransaction();
$pdo->exec("SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED");
$pdo->exec("UPDATE accounts SET balance = 1000 WHERE id = 1");
// 未提交

// 事务 B（可以读取到未提交的数据）
$pdo->beginTransaction();
$pdo->exec("SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED");
$stmt = $pdo->query("SELECT balance FROM accounts WHERE id = 1");
$balance = $stmt->fetchColumn();  // 可能读取到 1000（脏读）
```

#### READ COMMITTED（不可重复读）

```php
<?php
// 事务 A
$pdo->beginTransaction();
$pdo->exec("SET TRANSACTION ISOLATION LEVEL READ COMMITTED");
$stmt = $pdo->query("SELECT balance FROM accounts WHERE id = 1");
$balance1 = $stmt->fetchColumn();  // 假设是 500

// 事务 B 更新并提交
$pdo2->beginTransaction();
$pdo2->exec("UPDATE accounts SET balance = 1000 WHERE id = 1");
$pdo2->commit();

// 事务 A 再次读取
$stmt = $pdo->query("SELECT balance FROM accounts WHERE id = 1");
$balance2 = $stmt->fetchColumn();  // 现在是 1000（不可重复读）
```

#### REPEATABLE READ（可重复读）

```php
<?php
// 事务 A
$pdo->beginTransaction();
$pdo->exec("SET TRANSACTION ISOLATION LEVEL REPEATABLE READ");
$stmt = $pdo->query("SELECT balance FROM accounts WHERE id = 1");
$balance1 = $stmt->fetchColumn();  // 假设是 500

// 事务 B 更新并提交
$pdo2->beginTransaction();
$pdo2->exec("UPDATE accounts SET balance = 1000 WHERE id = 1");
$pdo2->commit();

// 事务 A 再次读取（使用快照读）
$stmt = $pdo->query("SELECT balance FROM accounts WHERE id = 1");
$balance2 = $stmt->fetchColumn();  // 仍然是 500（可重复读）
```

## 死锁

### 死锁产生原因

- 两个或多个事务相互等待对方释放锁。
- 通常发生在多个事务同时访问多个资源时。

### 死锁示例

```php
<?php
// 事务 A
$pdo1->beginTransaction();
$pdo1->exec("UPDATE accounts SET balance = balance - 100 WHERE id = 1");
// 持有 id=1 的锁

// 事务 B
$pdo2->beginTransaction();
$pdo2->exec("UPDATE accounts SET balance = balance - 100 WHERE id = 2");
// 持有 id=2 的锁

// 事务 A 尝试获取 id=2 的锁（等待）
$pdo1->exec("UPDATE accounts SET balance = balance + 100 WHERE id = 2");

// 事务 B 尝试获取 id=1 的锁（等待）
$pdo2->exec("UPDATE accounts SET balance = balance + 100 WHERE id = 1");

// 死锁！
```

### 死锁检测与重试

```php
<?php
declare(strict_types=1);

function transferWithRetry(PDO $pdo, int $fromId, int $toId, float $amount, int $maxRetries = 3): void
{
    $attempts = 0;
    
    while ($attempts < $maxRetries) {
        try {
            $pdo->beginTransaction();
            
            // 按 ID 顺序加锁，避免死锁
            $ids = [$fromId, $toId];
            sort($ids);
            
            foreach ($ids as $id) {
                $stmt = $pdo->prepare("SELECT * FROM accounts WHERE id = ? FOR UPDATE");
                $stmt->execute([$id]);
            }
            
            // 执行转账
            $stmt = $pdo->prepare("UPDATE accounts SET balance = balance - ? WHERE id = ?");
            $stmt->execute([$amount, $fromId]);
            
            $stmt = $pdo->prepare("UPDATE accounts SET balance = balance + ? WHERE id = ?");
            $stmt->execute([$amount, $toId]);
            
            $pdo->commit();
            return;  // 成功
            
        } catch (PDOException $e) {
            $pdo->rollBack();
            
            // 检查是否是死锁错误
            if ($e->getCode() == 1213 || strpos($e->getMessage(), 'Deadlock') !== false) {
                $attempts++;
                if ($attempts >= $maxRetries) {
                    throw new Exception("Transaction failed after {$maxRetries} attempts", 0, $e);
                }
                // 随机延迟后重试
                usleep(mt_rand(100000, 500000));  // 100-500ms
                continue;
            }
            
            throw $e;
        }
    }
}
```

## 保存点（Savepoint）

### 嵌套事务模拟

```php
<?php
declare(strict_types=1);

function nestedTransaction(PDO $pdo): void
{
    $pdo->beginTransaction();
    
    try {
        // 第一个操作
        $pdo->exec("INSERT INTO users (username) VALUES ('user1')");
        
        // 创建保存点
        $pdo->exec("SAVEPOINT sp1");
        
        try {
            // 第二个操作
            $pdo->exec("INSERT INTO users (username) VALUES ('user2')");
            
            // 回滚到保存点（只撤销第二个操作）
            $pdo->exec("ROLLBACK TO SAVEPOINT sp1");
            
        } catch (Exception $e) {
            $pdo->exec("ROLLBACK TO SAVEPOINT sp1");
            throw $e;
        }
        
        // 提交事务（第一个操作已提交）
        $pdo->commit();
        
    } catch (Exception $e) {
        $pdo->rollBack();
        throw $e;
    }
}
```

## 事务最佳实践

### 1. 保持事务简短

```php
// 不推荐：事务中包含长时间操作
$pdo->beginTransaction();
// ... 数据库操作
sleep(10);  // 长时间操作
// ... 更多数据库操作
$pdo->commit();

// 推荐：将长时间操作移到事务外
$pdo->beginTransaction();
// ... 数据库操作
$pdo->commit();

// 长时间操作
sleep(10);
```

### 2. 按固定顺序访问资源

```php
// 避免死锁：始终按 ID 顺序访问
$ids = [$id1, $id2];
sort($ids);

foreach ($ids as $id) {
    $stmt = $pdo->prepare("SELECT * FROM accounts WHERE id = ? FOR UPDATE");
    $stmt->execute([$id]);
}
```

### 3. 使用适当的隔离级别

```php
// 根据业务需求选择隔离级别
// 读多写少：REPEATABLE READ
// 写多读少：READ COMMITTED
```

### 4. 实现重试机制

```php
function withRetry(callable $callback, int $maxRetries = 3): mixed
{
    $attempts = 0;
    
    while ($attempts < $maxRetries) {
        try {
            return $callback();
        } catch (PDOException $e) {
            if ($e->getCode() == 1213 && $attempts < $maxRetries - 1) {
                $attempts++;
                usleep(mt_rand(100000, 500000));
                continue;
            }
            throw $e;
        }
    }
}
```

## 练习

1. 实现一个转账函数，使用事务确保原子性，包含余额检查。

2. 创建一个订单处理函数，使用事务确保订单、订单项、库存更新的一致性。

3. 编写一个死锁检测和重试机制，处理并发事务冲突。

4. 实现一个嵌套事务模拟，使用保存点实现部分回滚。

5. 设计一个事务管理器类，封装事务的开始、提交、回滚和重试逻辑。

6. 创建一个性能测试，比较不同隔离级别对并发性能的影响。
