# 5.3.3 隔离级别

## 概述

事务隔离级别控制并发事务之间的隔离程度。本节详细介绍事务隔离级别、脏读、不可重复读、幻读，以及隔离级别的设置和死锁处理。

## 事务隔离级别

### 隔离级别说明

| 隔离级别 | 脏读 | 不可重复读 | 幻读 | 说明 |
| :--- | :--- | :--- | :--- | :--- |
| READ UNCOMMITTED | 是 | 是 | 是 | 最低隔离级别，性能最好 |
| READ COMMITTED | 否 | 是 | 是 | 只能读取已提交的数据 |
| REPEATABLE READ | 否 | 否 | 是 | MySQL InnoDB 默认级别 |
| SERIALIZABLE | 否 | 否 | 否 | 最高隔离级别，性能最差 |

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

## 脏读（Dirty Read）

### 什么是脏读

读取到未提交的数据。

### 示例

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
$stmt = $pdo->prepare("SELECT balance FROM accounts WHERE id = 1");
$stmt->execute();
$balance = $stmt->fetchColumn();  // 可能读取到 1000（未提交的数据）

// 如果事务 A 回滚，事务 B 读取到的就是脏数据
```

## 不可重复读（Non-Repeatable Read）

### 什么是不可重复读

同一事务中多次读取同一数据，结果不一致。

### 示例

```php
<?php
// 事务 A
$pdo->beginTransaction();
$pdo->exec("SET TRANSACTION ISOLATION LEVEL READ COMMITTED");
$stmt = $pdo->prepare("SELECT balance FROM accounts WHERE id = 1");
$stmt->execute();
$balance1 = $stmt->fetchColumn();  // 读取到 100

// 事务 B 更新数据并提交
$pdo->beginTransaction();
$pdo->exec("UPDATE accounts SET balance = 200 WHERE id = 1");
$pdo->commit();

// 事务 A 再次读取
$stmt->execute();
$balance2 = $stmt->fetchColumn();  // 读取到 200（与第一次不同）
```

## 幻读（Phantom Read）

### 什么是幻读

同一事务中多次查询，结果集不一致。

### 示例

```php
<?php
// 事务 A
$pdo->beginTransaction();
$pdo->exec("SET TRANSACTION ISOLATION LEVEL REPEATABLE READ");
$stmt = $pdo->prepare("SELECT COUNT(*) FROM orders WHERE user_id = 1");
$stmt->execute();
$count1 = $stmt->fetchColumn();  // 读取到 5

// 事务 B 插入新数据并提交
$pdo->beginTransaction();
$pdo->exec("INSERT INTO orders (user_id, total) VALUES (1, 100)");
$pdo->commit();

// 事务 A 再次查询
$stmt->execute();
$count2 = $stmt->fetchColumn();  // 读取到 6（与第一次不同）
```

## 死锁处理

### 什么是死锁

多个事务相互等待资源，导致无法继续执行。

### 死锁示例

```php
<?php
// 事务 A
$pdo->beginTransaction();
$pdo->exec("UPDATE accounts SET balance = balance - 100 WHERE id = 1");
// 等待事务 B 释放 id = 2 的锁

// 事务 B
$pdo->beginTransaction();
$pdo->exec("UPDATE accounts SET balance = balance - 100 WHERE id = 2");
// 等待事务 A 释放 id = 1 的锁

// 死锁发生
```

### 死锁处理

```php
<?php
declare(strict_types=1);

function transferWithRetry(PDO $pdo, int $fromId, int $toId, float $amount, int $maxRetries = 3): void
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

## 完整示例

```php
<?php
declare(strict_types=1);

class TransactionService
{
    private PDO $pdo;

    public function __construct(PDO $pdo)
    {
        $this->pdo = $pdo;
    }

    public function setIsolationLevel(string $level): void
    {
        $this->pdo->exec("SET TRANSACTION ISOLATION LEVEL {$level}");
    }

    public function executeWithIsolation(string $level, callable $callback): mixed
    {
        $this->setIsolationLevel($level);
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
}
```

## 注意事项

1. **隔离级别选择**：根据业务需求选择合适的隔离级别
2. **性能权衡**：隔离级别越高，性能越差
3. **死锁预防**：按固定顺序锁定资源
4. **死锁处理**：实现重试机制

## 练习

1. 实现不同隔离级别的事务测试，验证隔离效果。

2. 创建一个死锁检测和处理机制。

3. 编写一个并发测试，验证隔离级别的正确性。

4. 实现一个事务管理器，支持隔离级别配置。
