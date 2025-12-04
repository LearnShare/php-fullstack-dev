# 5.3.2 ACID 特性

## 概述

ACID 是事务的四个核心特性，保证数据库操作的可靠性。本节详细介绍原子性（Atomicity）、一致性（Consistency）、隔离性（Isolation）、持久性（Durability）的含义和实现。

## ACID 特性

### 什么是 ACID

- **Atomicity（原子性）**：事务中的所有操作要么全部成功，要么全部失败
- **Consistency（一致性）**：事务执行前后数据库保持一致状态
- **Isolation（隔离性）**：并发事务之间相互隔离，互不干扰
- **Durability（持久性）**：事务提交后，数据永久保存

## 原子性（Atomicity）

### 含义

事务中的所有操作作为一个不可分割的整体执行。

### 示例

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

## 一致性（Consistency）

### 含义

事务执行前后数据库保持一致状态，不违反任何约束。

### 示例

```php
<?php
declare(strict_types=1);

function createOrder(PDO $pdo, int $userId, array $items): int
{
    try {
        $pdo->beginTransaction();
        
        // 创建订单
        $stmt = $pdo->prepare("INSERT INTO orders (user_id, total) VALUES (?, 0)");
        $stmt->execute([$userId]);
        $orderId = (int) $pdo->lastInsertId();
        
        $total = 0;
        foreach ($items as $item) {
            // 检查库存
            $stmt = $pdo->prepare("SELECT stock, price FROM products WHERE id = ?");
            $stmt->execute([$item['product_id']]);
            $product = $stmt->fetch();
            
            if ($product['stock'] < $item['quantity']) {
                throw new Exception('Insufficient stock');
            }
            
            $total += $product['price'] * $item['quantity'];
            
            // 创建订单项
            $stmt = $pdo->prepare(
                "INSERT INTO order_items (order_id, product_id, quantity, price) VALUES (?, ?, ?, ?)"
            );
            $stmt->execute([$orderId, $item['product_id'], $item['quantity'], $product['price']]);
            
            // 更新库存
            $stmt = $pdo->prepare("UPDATE products SET stock = stock - ? WHERE id = ?");
            $stmt->execute([$item['quantity'], $item['product_id']]);
        }
        
        // 更新订单总价
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

## 隔离性（Isolation）

### 含义

并发事务之间相互隔离，互不干扰。

### 隔离级别

| 隔离级别 | 脏读 | 不可重复读 | 幻读 |
| :--- | :--- | :--- | :--- |
| READ UNCOMMITTED | 是 | 是 | 是 |
| READ COMMITTED | 否 | 是 | 是 |
| REPEATABLE READ | 否 | 否 | 是 |
| SERIALIZABLE | 否 | 否 | 否 |

## 持久性（Durability）

### 含义

事务提交后，数据永久保存，即使系统崩溃也不会丢失。

### 实现

- 使用 WAL（Write-Ahead Logging）
- 数据刷盘到磁盘
- 事务日志持久化

## 完整示例

```php
<?php
declare(strict_types=1);

class TransactionManager
{
    private PDO $pdo;

    public function __construct(PDO $pdo)
    {
        $this->pdo = $pdo;
    }

    public function execute(callable $callback): mixed
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
}
```

## 注意事项

1. **原子性**：确保所有操作要么全部成功，要么全部失败
2. **一致性**：保证数据约束和业务规则
3. **隔离性**：选择合适的隔离级别
4. **持久性**：确保数据持久化到磁盘

## 练习

1. 实现一个转账功能，保证原子性和一致性。

2. 创建一个事务管理类，封装 ACID 特性。

3. 编写一个订单创建函数，保证数据一致性。

4. 实现一个并发测试，验证隔离性。
