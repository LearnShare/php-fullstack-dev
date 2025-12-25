# 5.5.1 并发控制

## 概述

数据库并发控制是保证数据一致性的关键机制。本节详细介绍并发问题（脏读、不可重复读、幻读）、并发控制机制、锁机制，以及完整的并发控制示例。

## 并发问题

### 脏读（Dirty Read）

读取到未提交的数据。

```sql
-- 事务 A
BEGIN;
UPDATE accounts SET balance = 1000 WHERE id = 1;
-- 未提交

-- 事务 B（READ UNCOMMITTED）
BEGIN;
SELECT balance FROM accounts WHERE id = 1;  -- 读取到 1000（脏读）

-- 事务 A 回滚
ROLLBACK;

-- 事务 B 读取的数据已失效
```

### 不可重复读（Non-Repeatable Read）

同一事务中多次读取同一数据，结果不一致。

```sql
-- 事务 A
BEGIN;
SELECT balance FROM accounts WHERE id = 1;  -- 500

-- 事务 B
BEGIN;
UPDATE accounts SET balance = 1000 WHERE id = 1;
COMMIT;

-- 事务 A 再次读取
SELECT balance FROM accounts WHERE id = 1;  -- 1000（不一致）
```

### 幻读（Phantom Read）

同一事务中多次查询，结果集数量不一致。

```sql
-- 事务 A
BEGIN;
SELECT COUNT(*) FROM orders WHERE user_id = 1;  -- 5 条

-- 事务 B
BEGIN;
INSERT INTO orders (user_id, total) VALUES (1, 100);
COMMIT;

-- 事务 A 再次查询
SELECT COUNT(*) FROM orders WHERE user_id = 1;  -- 6 条（幻读）
```

## 并发控制机制

### 隔离级别

| 隔离级别 | 脏读 | 不可重复读 | 幻读 |
| :--- | :--- | :--- | :--- |
| READ UNCOMMITTED | 是 | 是 | 是 |
| READ COMMITTED | 否 | 是 | 是 |
| REPEATABLE READ | 否 | 否 | 是 |
| SERIALIZABLE | 否 | 否 | 否 |

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

## 锁机制

### 行锁类型

- **共享锁（S Lock）**：允许读取，禁止写入
- **排他锁（X Lock）**：禁止读取和写入

### 加锁示例

```php
<?php
declare(strict_types=1);

// 排他锁（FOR UPDATE）
$pdo->beginTransaction();
$stmt = $pdo->prepare("SELECT * FROM accounts WHERE id = ? FOR UPDATE");
$stmt->execute([1]);
// 其他事务无法读取或修改该行

// 共享锁（LOCK IN SHARE MODE）
$pdo->beginTransaction();
$stmt = $pdo->prepare("SELECT * FROM accounts WHERE id = ? LOCK IN SHARE MODE");
$stmt->execute([1]);
// 其他事务可以读取，但不能修改
```

## 完整示例

```php
<?php
declare(strict_types=1);

class ConcurrencyControl
{
    private PDO $pdo;

    public function safeTransfer(int $fromId, int $toId, float $amount): void
    {
        $this->pdo->beginTransaction();
        $this->pdo->exec("SET TRANSACTION ISOLATION LEVEL REPEATABLE READ");
        
        try {
            // 锁定源账户
            $stmt = $this->pdo->prepare("SELECT * FROM accounts WHERE id = ? FOR UPDATE");
            $stmt->execute([$fromId]);
            $fromAccount = $stmt->fetch();
            
            if ($fromAccount['balance'] < $amount) {
                throw new Exception('Insufficient balance');
            }
            
            // 锁定目标账户
            $stmt = $this->pdo->prepare("SELECT * FROM accounts WHERE id = ? FOR UPDATE");
            $stmt->execute([$toId]);
            
            // 执行转账
            $stmt = $this->pdo->prepare("UPDATE accounts SET balance = balance - ? WHERE id = ?");
            $stmt->execute([$amount, $fromId]);
            
            $stmt = $this->pdo->prepare("UPDATE accounts SET balance = balance + ? WHERE id = ?");
            $stmt->execute([$amount, $toId]);
            
            $this->pdo->commit();
        } catch (Exception $e) {
            $this->pdo->rollBack();
            throw $e;
        }
    }
}
```

## 注意事项

1. **隔离级别选择**：根据业务需求选择合适的隔离级别
2. **锁的使用**：合理使用锁，避免死锁
3. **性能权衡**：隔离级别越高，性能越差
4. **死锁处理**：实现死锁检测和重试机制

## 练习

1. 实现不同隔离级别的事务测试，验证隔离效果。

2. 创建一个并发控制类，处理并发转账场景。

3. 实现死锁检测和处理机制。

4. 编写一个并发测试，验证锁机制的正确性。
