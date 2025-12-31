# 6.3.3 锁机制

## 概述

锁机制是数据库并发控制的基础，通过对数据资源加锁来协调多个事务的并发访问，防止数据冲突和不一致。理解锁机制对于开发高并发应用、处理并发更新、避免死锁至关重要。

本节详细介绍锁的概念、类型（共享锁、排他锁）、粒度（行锁、表锁）、使用方法（FOR UPDATE、LOCK IN SHARE MODE）、死锁处理等，帮助零基础学员掌握锁的使用方法。

**主要内容**：
- 锁的概念和作用
- 锁的类型（共享锁、排他锁、意向锁）
- 锁的粒度（行锁、表锁、页锁）
- 锁的使用（SELECT ... FOR UPDATE、SELECT ... LOCK IN SHARE MODE）
- 死锁处理和避免
- 完整示例和最佳实践

---

## 特性

- **并发控制**：通过加锁控制并发访问
- **数据一致性**：保证数据在并发更新时的一致性
- **冲突预防**：防止并发更新冲突
- **灵活控制**：支持不同粒度和类型的锁

---

## 锁的概念

### 什么是锁

锁（Lock）是数据库用于控制并发访问的机制，通过对数据资源加锁，确保同一时间只有一个或特定的事务可以访问该资源。

**锁的基本原理**：

- 事务在访问数据前需要先获取锁
- 持有锁的事务可以访问数据
- 其他事务需要等待锁释放
- 事务提交或回滚后释放锁

### 锁的作用

锁的主要作用包括：

1. **防止并发冲突**：避免多个事务同时修改同一数据
2. **保证数据一致性**：确保数据在并发访问时保持一致
3. **实现事务隔离**：配合隔离级别实现事务隔离
4. **协调并发访问**：协调多个事务的访问顺序

**示例**：

```php
<?php
declare(strict_types=1);

// 转账操作需要加锁防止并发冲突
$pdo->beginTransaction();

// 加锁查询余额
$stmt = $pdo->prepare('SELECT balance FROM accounts WHERE user_id = :user_id FOR UPDATE');
$stmt->execute(['user_id' => 1]);
$account = $stmt->fetch();

if ($account['balance'] >= 100) {
    // 扣除余额
    $stmt = $pdo->prepare('UPDATE accounts SET balance = balance - 100 WHERE user_id = :user_id');
    $stmt->execute(['user_id' => 1]);
}

$pdo->commit();
// 提交后释放锁
```

### 锁的分类

锁可以按不同维度分类：

**按操作类型**：

- 共享锁（S Lock）：读锁
- 排他锁（X Lock）：写锁

**按锁粒度**：

- 行锁（Row Lock）：锁定单行
- 表锁（Table Lock）：锁定整个表
- 页锁（Page Lock）：锁定数据页

**按功能**：

- 意向锁（Intent Lock）：表级锁，表示事务意图
- Gap Lock：间隙锁
- Next-Key Lock：行锁 + 间隙锁

---

## 锁的类型

### 共享锁（S Lock）

共享锁（Shared Lock），也称为读锁，允许多个事务同时读取同一数据。

**特点**：

- 多个事务可以同时持有同一数据的共享锁
- 持有共享锁的事务只能读取数据，不能修改
- 共享锁与共享锁兼容，与排他锁不兼容

**示例**：

```php
<?php
declare(strict_types=1);

// 使用共享锁（MySQL 8.0+）
$stmt = $pdo->prepare('SELECT * FROM users WHERE id = :id LOCK IN SHARE MODE');
$stmt->execute(['id' => 1]);
$user = $stmt->fetch();

// 或者使用 FOR SHARE（MySQL 8.0+）
$stmt = $pdo->prepare('SELECT * FROM users WHERE id = :id FOR SHARE');
$stmt->execute(['id' => 1]);
$user = $stmt->fetch();
```

### 排他锁（X Lock）

排他锁（Exclusive Lock），也称为写锁，只允许一个事务持有，其他事务无法读取或修改。

**特点**：

- 同一时间只有一个事务可以持有排他锁
- 持有排他锁的事务可以读取和修改数据
- 排他锁与所有锁（共享锁、排他锁）都不兼容

**示例**：

```php
<?php
declare(strict_types=1);

// 使用排他锁
$stmt = $pdo->prepare('SELECT * FROM users WHERE id = :id FOR UPDATE');
$stmt->execute(['id' => 1]);
$user = $stmt->fetch();

// 可以进行更新操作
$stmt = $pdo->prepare('UPDATE users SET balance = balance - 100 WHERE id = :id');
$stmt->execute(['id' => 1]);
```

### 意向锁

意向锁（Intent Lock）是表级锁，表示事务意图对表中的行加锁。

**类型**：

- **意向共享锁（IS）**：事务意图对表中的行加共享锁
- **意向排他锁（IX）**：事务意图对表中的行加排他锁

**作用**：

- 提高表级锁的效率
- 避免全表扫描检查行锁

### 锁的兼容性

不同类型的锁的兼容性：

| 当前锁 \ 请求锁 | 共享锁（S） | 排他锁（X） |
|:---------------|:-----------|:-----------|
| 共享锁（S） | 兼容 | 不兼容 |
| 排他锁（X） | 不兼容 | 不兼容 |

**说明**：

- **兼容**：可以同时持有
- **不兼容**：需要等待锁释放

---

## 锁的粒度

### 行锁（Row Lock）

行锁锁定单行数据，粒度最小，并发度最高。

**特点**：

- 锁定单行数据
- 并发度最高
- 开销较大
- MySQL InnoDB 默认使用行锁

**示例**：

```php
<?php
declare(strict_types=1);

// 锁定单行
$stmt = $pdo->prepare('SELECT * FROM users WHERE id = :id FOR UPDATE');
$stmt->execute(['id' => 1]);
// 只锁定 id = 1 的行，其他行不受影响
```

### 表锁（Table Lock）

表锁锁定整个表，粒度最大，并发度最低。

**特点**：

- 锁定整个表
- 并发度最低
- 开销较小
- MyISAM 使用表锁

**示例**：

```php
<?php
declare(strict_types=1);

// 显式加表锁
$pdo->exec('LOCK TABLES users READ');
// 执行读操作
$pdo->exec('UNLOCK TABLES');

// 写锁
$pdo->exec('LOCK TABLES users WRITE');
// 执行写操作
$pdo->exec('UNLOCK TABLES');
```

### 页锁（Page Lock）

页锁锁定数据页，粒度介于行锁和表锁之间。

**特点**：

- 锁定数据页（通常 8KB 或 16KB）
- 并发度中等
- 开销中等
- 较少使用

### 粒度选择

选择锁粒度的原则：

**行锁适用场景**：

- 高并发场景
- 更新操作分散
- 需要高并发度

**表锁适用场景**：

- 低并发场景
- 批量操作
- 整表更新

---

## 锁的使用

### SELECT ... FOR UPDATE

`FOR UPDATE` 加排他锁，用于更新操作前锁定数据。

**语法**：`SELECT ... FOR UPDATE`

**示例**：

```php
<?php
declare(strict_types=1);

$pdo->beginTransaction();

// 加排他锁
$stmt = $pdo->prepare('SELECT balance FROM accounts WHERE user_id = :user_id FOR UPDATE');
$stmt->execute(['user_id' => 1]);
$account = $stmt->fetch();

// 检查余额
if ($account['balance'] >= 100) {
    // 扣除余额
    $stmt = $pdo->prepare('UPDATE accounts SET balance = balance - 100 WHERE user_id = :user_id');
    $stmt->execute(['user_id' => 1]);
}

$pdo->commit();
```

### SELECT ... LOCK IN SHARE MODE / FOR SHARE

`LOCK IN SHARE MODE`（MySQL 5.7）或 `FOR SHARE`（MySQL 8.0+）加共享锁，用于读取时防止修改。

**语法**：
- MySQL 5.7: `SELECT ... LOCK IN SHARE MODE`
- MySQL 8.0+: `SELECT ... FOR SHARE`

**示例**：

```php
<?php
declare(strict_types=1);

$pdo->beginTransaction();

// 加共享锁（MySQL 8.0+）
$stmt = $pdo->prepare('SELECT * FROM users WHERE id = :id FOR SHARE');
$stmt->execute(['id' => 1]);
$user = $stmt->fetch();

// 其他事务可以读取，但不能修改

$pdo->commit();
```

### UPDATE/DELETE 自动加锁

UPDATE 和 DELETE 语句会自动加排他锁。

**示例**：

```php
<?php
declare(strict_types=1);

$pdo->beginTransaction();

// UPDATE 自动加排他锁
$stmt = $pdo->prepare('UPDATE users SET balance = balance - 100 WHERE id = :id');
$stmt->execute(['id' => 1]);
// 自动对 id = 1 的行加排他锁

$pdo->commit();
```

---

## 死锁处理

### 死锁概念

死锁（Deadlock）是指两个或多个事务相互等待对方释放锁，导致所有事务都无法继续执行。

**死锁示例**：

```php
<?php
declare(strict_types=1);

// 事务 A
$pdo1->beginTransaction();
$stmt = $pdo1->prepare('UPDATE accounts SET balance = balance - 100 WHERE user_id = 1');
$stmt->execute();  // 锁定 user_id = 1

// 事务 B
$pdo2->beginTransaction();
$stmt = $pdo2->prepare('UPDATE accounts SET balance = balance - 100 WHERE user_id = 2');
$stmt->execute();  // 锁定 user_id = 2

// 事务 A 尝试锁定 user_id = 2
$stmt = $pdo1->prepare('UPDATE accounts SET balance = balance + 100 WHERE user_id = 2');
$stmt->execute();  // 等待事务 B 释放锁

// 事务 B 尝试锁定 user_id = 1
$stmt = $pdo2->prepare('UPDATE accounts SET balance = balance + 100 WHERE user_id = 1');
$stmt->execute();  // 等待事务 A 释放锁

// 死锁！
```

### 死锁检测

MySQL InnoDB 自动检测死锁，并回滚其中一个事务。

**检测机制**：

- InnoDB 有专门的死锁检测机制
- 检测到死锁后，选择代价最小的事务回滚
- 被回滚的事务会抛出异常

**示例**：

```php
<?php
declare(strict_types=1);

try {
    $pdo->beginTransaction();
    
    // 可能导致死锁的操作
    
    $pdo->commit();
} catch (PDOException $e) {
    if ($e->getCode() == 40001 || $e->getCode() == 1213) {
        // 死锁错误
        echo "检测到死锁，事务已回滚\n";
    }
    $pdo->rollBack();
}
```

### 死锁避免

避免死锁的策略：

**1. 统一加锁顺序**

```php
<?php
declare(strict_types=1);

// 始终按 user_id 升序加锁
function transfer(PDO $pdo, int $fromUserId, int $toUserId, float $amount): bool
{
    // 确保加锁顺序一致
    $firstId = min($fromUserId, $toUserId);
    $secondId = max($fromUserId, $toUserId);
    
    $pdo->beginTransaction();
    
    // 按顺序加锁
    $stmt = $pdo->prepare('SELECT balance FROM accounts WHERE user_id = :user_id FOR UPDATE');
    $stmt->execute(['user_id' => $firstId]);
    
    $stmt->execute(['user_id' => $secondId]);
    
    // 执行转账逻辑
    
    $pdo->commit();
    return true;
}
```

**2. 减少事务持有锁的时间**

```php
<?php
declare(strict_types=1);

// 事务范围尽可能小
$pdo->beginTransaction();

// 只包含必要的数据库操作
$stmt = $pdo->prepare('UPDATE users SET balance = balance - 100 WHERE id = :id');
$stmt->execute(['id' => 1]);

$pdo->commit();
// 尽快提交，释放锁
```

**3. 使用重试机制**

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
            // 检测死锁错误码
            if ($e->getCode() == 40001 || $e->getCode() == 1213) {
                $attempts++;
                if ($attempts >= $maxRetries) {
                    throw $e;
                }
                // 指数退避
                usleep(100000 * $attempts);
                continue;
            }
            throw $e;
        }
    }
}

// 使用
executeWithRetry(function() use ($pdo) {
    $pdo->beginTransaction();
    // 操作
    $pdo->commit();
});
```

### 死锁解决

当发生死锁时的处理方法：

**1. 捕获异常并重试**

```php
<?php
declare(strict_types=1);

try {
    $pdo->beginTransaction();
    // 操作
    $pdo->commit();
} catch (PDOException $e) {
    if ($e->getCode() == 40001 || $e->getCode() == 1213) {
        // 死锁，重试
        $pdo->rollBack();
        // 实现重试逻辑
    } else {
        throw $e;
    }
}
```

**2. 查看死锁日志**

```sql
-- 查看最近一次死锁信息
SHOW ENGINE INNODB STATUS;
```

---

## 锁超时

### 设置锁超时

可以设置锁等待超时时间，避免长时间等待。

**示例**：

```php
<?php
declare(strict_types=1);

// 设置锁等待超时时间（秒）
$pdo->exec('SET innodb_lock_wait_timeout = 5');

try {
    $pdo->beginTransaction();
    
    $stmt = $pdo->prepare('SELECT * FROM users WHERE id = :id FOR UPDATE');
    $stmt->execute(['id' => 1]);
    // 如果锁等待超过 5 秒，抛出异常
    
    $pdo->commit();
} catch (PDOException $e) {
    if ($e->getCode() == 'HY000' && strpos($e->getMessage(), 'Lock wait timeout') !== false) {
        echo "锁等待超时\n";
    }
    $pdo->rollBack();
}
```

---

## 完整示例

### 转账操作（使用锁）

```php
<?php
declare(strict_types=1);

class AccountService
{
    private PDO $pdo;
    
    public function __construct(PDO $pdo)
    {
        $this->pdo = $pdo;
        // 设置锁超时
        $this->pdo->exec('SET innodb_lock_wait_timeout = 10');
    }
    
    public function transfer(int $fromUserId, int $toUserId, float $amount): bool
    {
        // 统一加锁顺序，避免死锁
        $firstId = min($fromUserId, $toUserId);
        $secondId = max($fromUserId, $toUserId);
        
        try {
            $this->pdo->beginTransaction();
            
            // 按顺序加锁
            $stmt = $this->pdo->prepare('SELECT balance FROM accounts WHERE user_id = :user_id FOR UPDATE');
            $stmt->execute(['user_id' => $firstId]);
            $firstAccount = $stmt->fetch();
            
            $stmt->execute(['user_id' => $secondId]);
            $secondAccount = $stmt->fetch();
            
            // 确定转出和转入账户
            if ($firstId == $fromUserId) {
                $fromAccount = $firstAccount;
                $toAccount = $secondAccount;
            } else {
                $fromAccount = $secondAccount;
                $toAccount = $firstAccount;
            }
            
            // 检查余额
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
            
            $this->pdo->commit();
            return true;
        } catch (PDOException $e) {
            $this->pdo->rollBack();
            
            // 处理死锁
            if ($e->getCode() == 40001 || $e->getCode() == 1213) {
                error_log('死锁发生: ' . $e->getMessage());
                // 可以实现重试逻辑
                return false;
            }
            
            throw $e;
        }
    }
}

// 使用
try {
    $pdo = new PDO('mysql:host=localhost;dbname=test', 'user', 'pass', [
        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION
    ]);
    
    $service = new AccountService($pdo);
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

### 并发更新

需要防止并发更新冲突时使用锁。

**示例**：

```php
<?php
declare(strict_types=1);

// 库存扣减
$pdo->beginTransaction();

$stmt = $pdo->prepare('SELECT stock FROM products WHERE id = :id FOR UPDATE');
$stmt->execute(['id' => 1]);
$product = $stmt->fetch();

if ($product['stock'] > 0) {
    $stmt = $pdo->prepare('UPDATE products SET stock = stock - 1 WHERE id = :id');
    $stmt->execute(['id' => 1]);
}

$pdo->commit();
```

### 数据一致性

需要保证数据一致性时使用锁。

**示例**：

```php
<?php
declare(strict_types=1);

// 订单创建和库存扣减
$pdo->beginTransaction();

// 锁定商品
$stmt = $pdo->prepare('SELECT stock FROM products WHERE id = :id FOR UPDATE');
$stmt->execute(['id' => 1]);
$product = $stmt->fetch();

if ($product['stock'] >= $quantity) {
    // 创建订单
    $stmt = $pdo->prepare('INSERT INTO orders (user_id, product_id, quantity) VALUES (:user_id, :product_id, :quantity)');
    $stmt->execute(['user_id' => 1, 'product_id' => 1, 'quantity' => $quantity]);
    
    // 扣减库存
    $stmt = $pdo->prepare('UPDATE products SET stock = stock - :quantity WHERE id = :id');
    $stmt->execute(['quantity' => $quantity, 'id' => 1]);
}

$pdo->commit();
```

---

## 注意事项

### 锁的范围

锁的范围应尽可能小，减少锁竞争。

```php
<?php
declare(strict_types=1);

// ✅ 好的做法：锁的范围小
$pdo->beginTransaction();
$stmt = $pdo->prepare('SELECT * FROM users WHERE id = :id FOR UPDATE');
$stmt->execute(['id' => 1]);
// 只锁定 id = 1 的行
$pdo->commit();

// ❌ 不好的做法：锁的范围大
$pdo->beginTransaction();
$pdo->exec('LOCK TABLES users WRITE');
// 锁定整个表
$pdo->exec('UNLOCK TABLES');
$pdo->commit();
```

### 死锁风险

注意避免死锁：

- 统一加锁顺序
- 减少事务持有锁的时间
- 实现重试机制

### 性能影响

锁会影响并发性能：

- 锁粒度越大，并发度越低
- 锁持有时间越长，性能越差
- 使用行锁提高并发度

### 锁超时设置

合理设置锁超时时间：

```php
<?php
declare(strict_types=1);

// 设置锁等待超时（秒）
$pdo->exec('SET innodb_lock_wait_timeout = 10');
```

---

## 常见问题

### 什么是锁？

锁是数据库用于控制并发访问的机制，通过对数据资源加锁，确保同一时间只有特定的事务可以访问该资源。

### 共享锁和排他锁的区别？

- **共享锁**：允许多个事务同时读取，不允许修改
- **排他锁**：只允许一个事务持有，可以读取和修改

### 如何避免死锁？

避免死锁的方法：

1. 统一加锁顺序
2. 减少事务持有锁的时间
3. 使用重试机制
4. 设置锁超时

### 锁对性能的影响？

锁的影响：

- **并发度**：锁粒度越大，并发度越低
- **等待时间**：锁持有时间越长，等待时间越长
- **死锁风险**：加锁越多，死锁风险越高

---

## 最佳实践

### 最小化锁的范围

使用行锁而不是表锁，减少锁的范围。

```php
<?php
declare(strict_types=1);

// 使用行锁
$stmt = $pdo->prepare('SELECT * FROM users WHERE id = :id FOR UPDATE');
$stmt->execute(['id' => 1]);
```

### 避免长时间持锁

事务范围尽可能小，尽快提交释放锁。

```php
<?php
declare(strict_types=1);

$pdo->beginTransaction();
// 只包含必要的数据库操作
$pdo->commit();  // 尽快提交
```

### 实现死锁检测

捕获死锁异常并实现重试。

```php
<?php
declare(strict_types=1);

try {
    $pdo->beginTransaction();
    // 操作
    $pdo->commit();
} catch (PDOException $e) {
    if ($e->getCode() == 40001 || $e->getCode() == 1213) {
        // 死锁，重试
    }
    $pdo->rollBack();
}
```

### 设置锁超时

合理设置锁等待超时时间。

```php
<?php
declare(strict_types=1);

$pdo->exec('SET innodb_lock_wait_timeout = 10');
```

---

## 练习任务

1. **锁的使用**
   - 使用 FOR UPDATE 实现库存扣减
   - 使用 FOR SHARE 实现读取保护
   - 测试不同类型的锁
   - 验证锁的兼容性

2. **死锁演示**
   - 模拟死锁场景
   - 观察死锁检测
   - 实现重试机制
   - 分析死锁原因

3. **死锁避免**
   - 实现统一加锁顺序
   - 优化事务范围
   - 测试死锁避免效果
   - 编写最佳实践文档

4. **性能测试**
   - 测试不同锁粒度的性能
   - 对比行锁和表锁
   - 分析锁竞争情况
   - 优化锁使用

5. **综合应用**
   - 创建一个高并发业务场景
   - 使用锁保证数据一致性
   - 实现死锁处理
   - 优化并发性能

---

**相关章节**：

- [6.3.1 并发控制](section-01-concurrency-control.md)
- [6.3.2 MVCC 机制](section-02-mvcc.md)
- [6.2.4 高级特性](../chapter-02-pdo/section-04-advanced-features.md)
