# 5.5.3 锁机制

## 概述

InnoDB 的锁机制是保证数据一致性的关键。本节详细介绍锁类型、行锁、表锁、间隙锁、Next-Key Lock，以及死锁处理。

## 锁类型

### 行锁类型

- **共享锁（S Lock）**：允许读取，禁止写入
- **排他锁（X Lock）**：禁止读取和写入

### 锁的兼容性

| | S Lock | X Lock |
| :--- | :--- | :--- |
| S Lock | 兼容 | 不兼容 |
| X Lock | 不兼容 | 不兼容 |

## 行锁

### 排他锁（FOR UPDATE）

```php
<?php
declare(strict_types=1);

$pdo->beginTransaction();
$stmt = $pdo->prepare("SELECT * FROM accounts WHERE id = ? FOR UPDATE");
$stmt->execute([1]);
// 其他事务无法读取或修改该行
```

### 共享锁（LOCK IN SHARE MODE）

```php
<?php
declare(strict_types=1);

$pdo->beginTransaction();
$stmt = $pdo->prepare("SELECT * FROM accounts WHERE id = ? LOCK IN SHARE MODE");
$stmt->execute([1]);
// 其他事务可以读取，但不能修改
```

## 间隙锁（Gap Lock）

### 什么是间隙锁

- 锁定索引记录之间的间隙
- 防止在间隙中插入新记录
- 只在 REPEATABLE READ 隔离级别下使用

### 间隙锁示例

```php
<?php
declare(strict_types=1);

$pdo->beginTransaction();
$pdo->exec("SET TRANSACTION ISOLATION LEVEL REPEATABLE READ");

// 范围查询加锁
$stmt = $pdo->prepare("SELECT * FROM orders WHERE user_id BETWEEN ? AND ? FOR UPDATE");
$stmt->execute([1, 10]);
// 锁定 user_id 在 1-10 之间的所有记录和间隙
```

## Next-Key Lock

### 什么是 Next-Key Lock

- Next-Key Lock = 行锁 + 间隙锁
- InnoDB 使用 Next-Key Lock 防止幻读
- 锁定记录和记录之间的间隙

### Next-Key Lock 示例

```php
<?php
declare(strict_types=1);

$pdo->beginTransaction();
$pdo->exec("SET TRANSACTION ISOLATION LEVEL REPEATABLE READ");

// 范围查询加锁
$stmt = $pdo->prepare("SELECT * FROM orders WHERE user_id = ? FOR UPDATE");
$stmt->execute([1]);
// 锁定 user_id=1 的所有记录和间隙，防止插入新记录
```

## 死锁处理

### 死锁检测

```php
<?php
declare(strict_types=1);

function safeTransfer(PDO $pdo, int $fromId, int $toId, float $amount, int $maxRetries = 3): void
{
    $retries = 0;
    
    while ($retries < $maxRetries) {
        try {
            $pdo->beginTransaction();
            
            // 按 ID 顺序锁定，避免死锁
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
            return;
            
        } catch (PDOException $e) {
            $pdo->rollBack();
            
            // 检查是否是死锁错误
            if ($e->getCode() == 40001 || strpos($e->getMessage(), 'Deadlock') !== false) {
                $retries++;
                usleep(100000 * $retries);  // 指数退避
                continue;
            }
            
            throw $e;
        }
    }
    
    throw new Exception('Transaction failed after retries');
}
```

### 锁等待超时

```php
<?php
declare(strict_types=1);

// 设置锁等待超时（秒）
$pdo->exec("SET innodb_lock_wait_timeout = 10");

try {
    $pdo->beginTransaction();
    $stmt = $pdo->prepare("SELECT * FROM accounts WHERE id = ? FOR UPDATE");
    $stmt->execute([1]);
    // 如果 10 秒内无法获取锁，抛出异常
} catch (PDOException $e) {
    if ($e->getCode() == 1205) {  // Lock wait timeout
        echo "Lock wait timeout\n";
    }
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

class LockManager
{
    private PDO $pdo;

    public function acquireLock(int $resourceId, int $timeout = 10): bool
    {
        $this->pdo->exec("SET innodb_lock_wait_timeout = {$timeout}");
        
        try {
            $this->pdo->beginTransaction();
            $stmt = $this->pdo->prepare("SELECT * FROM resources WHERE id = ? FOR UPDATE");
            $stmt->execute([$resourceId]);
            return true;
        } catch (PDOException $e) {
            if ($e->getCode() == 1205) {
                return false;  // 锁等待超时
            }
            throw $e;
        }
    }

    public function releaseLock(): void
    {
        $this->pdo->commit();
    }
}
```

## 注意事项

1. **锁顺序**：按固定顺序锁定资源，避免死锁
2. **锁超时**：设置合理的锁等待超时
3. **死锁处理**：实现死锁检测和重试机制
4. **性能考虑**：尽量减少锁的持有时间

## 练习

1. 实现一个死锁检测和处理机制。

2. 创建一个锁管理器，处理资源锁定。

3. 编写一个并发测试，验证锁机制的正确性。

4. 实现一个锁等待超时处理机制。
