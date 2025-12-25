# 5.3.1 事务基础

## 概述

事务是保证数据库操作一致性的关键机制。本节详细介绍什么是事务、事务的基本操作（beginTransaction、commit、rollBack），以及完整的事务处理示例。

## 什么是事务

- **事务**：一组数据库操作，要么全部成功，要么全部失败
- **原子性**：事务中的所有操作作为一个整体执行
- **一致性**：事务执行前后数据库保持一致状态

## 事务基本操作

### beginTransaction

- **语法**：`beginTransaction(): bool`
- 开始一个新事务，关闭自动提交

```php
<?php
declare(strict_types=1);

$pdo->beginTransaction();
```

### commit

- **语法**：`commit(): bool`
- 提交事务，使所有更改永久生效

```php
<?php
declare(strict_types=1);

$pdo->commit();
```

### rollBack

- **语法**：`rollBack(): bool`
- 回滚事务，撤销所有未提交的更改

```php
<?php
declare(strict_types=1);

$pdo->rollBack();
```

## 事务示例

### 基础示例

```php
<?php
declare(strict_types=1);

try {
    $pdo->beginTransaction();
    
    // 操作 1
    $stmt = $pdo->prepare("INSERT INTO users (username, email) VALUES (?, ?)");
    $stmt->execute(['alice', 'alice@example.com']);
    
    // 操作 2
    $stmt = $pdo->prepare("INSERT INTO profiles (user_id, bio) VALUES (?, ?)");
    $stmt->execute([$pdo->lastInsertId(), 'Bio text']);
    
    $pdo->commit();
} catch (Exception $e) {
    $pdo->rollBack();
    throw $e;
}
```

### 转账示例

```php
<?php
declare(strict_types=1);

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

## 完整示例

### 订单创建

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

## 注意事项

1. **异常处理**：使用 try-catch 确保事务回滚
2. **嵌套事务**：MySQL 不支持嵌套事务
3. **自动提交**：beginTransaction 会关闭自动提交
4. **连接管理**：事务期间保持连接

## 练习

1. 创建一个事务管理类，封装事务操作。

2. 实现一个转账功能，使用事务保证原子性。

3. 编写一个订单创建函数，包含库存检查和更新。

4. 实现一个批量操作函数，使用事务提升性能。
