# 6.3.1 并发控制

## 概述

并发控制是数据库管理并发访问的重要机制，用于处理多个事务同时访问同一数据时可能出现的冲突和问题。理解并发控制对于开发高并发应用、保证数据一致性至关重要。

本节详细介绍并发控制的概念、并发问题的类型（脏读、不可重复读、幻读）、事务隔离级别及其选择原则等，帮助零基础学员理解并发控制的重要性和实现方式。

**主要内容**：
- 并发控制的概念和重要性
- 并发问题类型（脏读、不可重复读、幻读）
- 事务隔离级别（READ UNCOMMITTED、READ COMMITTED、REPEATABLE READ、SERIALIZABLE）
- 隔离级别的选择原则
- 完整示例和最佳实践

---

## 特性

- **并发安全**：保证多事务并发访问时的数据一致性
- **隔离控制**：通过隔离级别控制事务间的可见性
- **性能平衡**：在一致性和性能之间找到平衡
- **问题预防**：预防脏读、不可重复读、幻读等问题

---

## 并发控制概念

### 什么是并发控制

并发控制（Concurrency Control）是数据库管理系统用于协调多个事务同时访问同一数据资源的机制，确保数据的一致性和完整性。

**并发控制的必要性**：

- **多用户系统**：多个用户同时访问数据库
- **高并发应用**：Web 应用需要处理大量并发请求
- **数据一致性**：确保数据在并发访问时保持一致
- **避免冲突**：避免数据更新冲突和数据丢失

### 为什么需要并发控制

**没有并发控制的问题**：

1. **数据不一致**：多个事务同时修改数据，导致数据不一致
2. **丢失更新**：一个事务的更新被另一个事务覆盖
3. **脏读**：读取到未提交的数据
4. **不可重复读**：同一事务中多次读取结果不一致

**示例场景**：

```php
<?php
declare(strict_types=1);

// 场景：两个用户同时购买同一件商品
// 商品库存：1

// 用户 A 的事务
$pdo->beginTransaction();
$stmt = $pdo->prepare('SELECT stock FROM products WHERE id = :id');
$stmt->execute(['id' => 1]);
$product = $stmt->fetch();
// 读取到 stock = 1

// 用户 B 的事务（同时执行）
$pdo2->beginTransaction();
$stmt2 = $pdo2->prepare('SELECT stock FROM products WHERE id = :id');
$stmt2->execute(['id' => 1]);
$product2 = $stmt2->fetch();
// 也读取到 stock = 1

// 用户 A 购买
$stmt = $pdo->prepare('UPDATE products SET stock = stock - 1 WHERE id = :id');
$stmt->execute(['id' => 1]);
$pdo->commit();
// stock 变为 0

// 用户 B 也购买
$stmt2 = $pdo2->prepare('UPDATE products SET stock = stock - 1 WHERE id = :id');
$stmt2->execute(['id' => 1]);
$pdo2->commit();
// stock 变为 -1（错误！）
```

### 并发控制的目标

并发控制的目标包括：

1. **数据一致性**：保证数据在并发访问时保持一致
2. **隔离性**：事务之间相互隔离，互不干扰
3. **正确性**：确保操作结果的正确性
4. **性能**：在保证正确性的前提下，尽可能提高性能

---

## 并发问题

### 脏读（Dirty Read）

脏读是指一个事务读取了另一个未提交事务修改的数据。

**问题场景**：

```php
<?php
declare(strict_types=1);

// 事务 A：更新用户余额
$pdo1->beginTransaction();
$stmt = $pdo1->prepare('UPDATE accounts SET balance = balance - 100 WHERE user_id = :user_id');
$stmt->execute(['user_id' => 1]);
// 余额从 1000 变为 900（未提交）

// 事务 B：读取用户余额（脏读）
$pdo2->beginTransaction();
$stmt = $pdo2->prepare('SELECT balance FROM accounts WHERE user_id = :user_id');
$stmt->execute(['user_id' => 1]);
$balance = $stmt->fetchColumn();
// 读取到 900（未提交的数据）

// 事务 A：回滚
$pdo1->rollBack();
// 余额恢复为 1000

// 事务 B：基于错误的数据（900）进行操作
// 问题：事务 B 读取到了不存在的数据
```

**影响**：

- 读取到未提交的数据，可能被回滚
- 基于错误数据做出错误决策
- 数据不一致

### 不可重复读（Non-repeatable Read）

不可重复读是指同一事务中，多次读取同一数据得到不同的结果。

**问题场景**：

```php
<?php
declare(strict_types=1);

// 事务 A：第一次读取
$pdo1->beginTransaction();
$stmt = $pdo1->prepare('SELECT balance FROM accounts WHERE user_id = :user_id');
$stmt->execute(['user_id' => 1]);
$balance1 = $stmt->fetchColumn();
// 读取到 1000

// 事务 B：更新余额
$pdo2->beginTransaction();
$stmt = $pdo2->prepare('UPDATE accounts SET balance = balance - 100 WHERE user_id = :user_id');
$stmt->execute(['user_id' => 1]);
$pdo2->commit();
// 余额变为 900

// 事务 A：第二次读取（不可重复读）
$stmt = $pdo1->prepare('SELECT balance FROM accounts WHERE user_id = :user_id');
$stmt->execute(['user_id' => 1]);
$balance2 = $stmt->fetchColumn();
// 读取到 900（与第一次不同）

// 问题：同一事务中两次读取结果不一致
```

**影响**：

- 同一事务中数据不一致
- 可能导致业务逻辑错误
- 影响数据统计的准确性

### 幻读（Phantom Read）

幻读是指同一事务中，多次执行同一查询，得到不同的结果集（新增或删除的行）。

**问题场景**：

```php
<?php
declare(strict_types=1);

// 事务 A：第一次查询
$pdo1->beginTransaction();
$stmt = $pdo1->prepare('SELECT COUNT(*) FROM orders WHERE user_id = :user_id');
$stmt->execute(['user_id' => 1]);
$count1 = $stmt->fetchColumn();
// 查询到 5 个订单

// 事务 B：插入新订单
$pdo2->beginTransaction();
$stmt = $pdo2->prepare('INSERT INTO orders (user_id, total) VALUES (:user_id, :total)');
$stmt->execute(['user_id' => 1, 'total' => 100]);
$pdo2->commit();
// 新增 1 个订单

// 事务 A：第二次查询（幻读）
$stmt = $pdo1->prepare('SELECT COUNT(*) FROM orders WHERE user_id = :user_id');
$stmt->execute(['user_id' => 1]);
$count2 = $stmt->fetchColumn();
// 查询到 6 个订单（与第一次不同）

// 问题：同一事务中两次查询结果集不同
```

**影响**：

- 结果集不一致
- 可能导致统计错误
- 影响业务逻辑判断

---

## 事务隔离级别

### READ UNCOMMITTED（读未提交）

最低隔离级别，允许读取未提交的数据。

**特点**：

- 允许脏读
- 允许不可重复读
- 允许幻读
- 性能最高，但数据一致性最差

**示例**：

```php
<?php
declare(strict_types=1);

$pdo->exec('SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED');
$pdo->beginTransaction();

// 可能读取到未提交的数据
$stmt = $pdo->prepare('SELECT balance FROM accounts WHERE user_id = :user_id');
$stmt->execute(['user_id' => 1]);
$balance = $stmt->fetchColumn();

$pdo->commit();
```

### READ COMMITTED（读已提交）

只允许读取已提交的数据，MySQL 默认隔离级别（InnoDB 实际使用 REPEATABLE READ）。

**特点**：

- 防止脏读
- 允许不可重复读
- 允许幻读
- 性能较好，数据一致性较好

**示例**：

```php
<?php
declare(strict_types=1);

$pdo->exec('SET TRANSACTION ISOLATION LEVEL READ COMMITTED');
$pdo->beginTransaction();

// 只能读取已提交的数据
$stmt = $pdo->prepare('SELECT balance FROM accounts WHERE user_id = :user_id');
$stmt->execute(['user_id' => 1]);
$balance = $stmt->fetchColumn();

$pdo->commit();
```

### REPEATABLE READ（可重复读）

MySQL InnoDB 的默认隔离级别，保证同一事务中多次读取结果一致。

**特点**：

- 防止脏读
- 防止不可重复读
- 允许幻读（InnoDB 通过 MVCC 和 Next-Key Lock 基本防止）
- 性能良好，数据一致性好

**示例**：

```php
<?php
declare(strict_types=1);

$pdo->exec('SET TRANSACTION ISOLATION LEVEL REPEATABLE READ');
$pdo->beginTransaction();

// 第一次读取
$stmt = $pdo->prepare('SELECT balance FROM accounts WHERE user_id = :user_id');
$stmt->execute(['user_id' => 1]);
$balance1 = $stmt->fetchColumn();

// 第二次读取（结果一致）
$stmt->execute(['user_id' => 1]);
$balance2 = $stmt->fetchColumn();
// $balance1 和 $balance2 相同

$pdo->commit();
```

### SERIALIZABLE（串行化）

最高隔离级别，完全串行执行，性能最低。

**特点**：

- 防止脏读
- 防止不可重复读
- 防止幻读
- 性能最低，但数据一致性最好

**示例**：

```php
<?php
declare(strict_types=1);

$pdo->exec('SET TRANSACTION ISOLATION LEVEL SERIALIZABLE');
$pdo->beginTransaction();

// 完全串行执行，不会有并发问题
$stmt = $pdo->prepare('SELECT balance FROM accounts WHERE user_id = :user_id');
$stmt->execute(['user_id' => 1]);
$balance = $stmt->fetchColumn();

$pdo->commit();
```

### 级别对比

| 隔离级别 | 脏读 | 不可重复读 | 幻读 | 性能 |
|:---------|:-----|:-----------|:-----|:-----|
| READ UNCOMMITTED | 允许 | 允许 | 允许 | 最高 |
| READ COMMITTED | 防止 | 允许 | 允许 | 较高 |
| REPEATABLE READ | 防止 | 防止 | 允许（InnoDB 基本防止） | 良好 |
| SERIALIZABLE | 防止 | 防止 | 防止 | 最低 |

---

## 隔离级别选择

### 性能考虑

隔离级别越高，性能越低。

**选择原则**：

- **读多写少**：可以使用较低的隔离级别
- **写多读少**：需要较高的隔离级别
- **高并发**：优先考虑性能，使用较低隔离级别
- **数据一致性要求高**：使用较高隔离级别

### 一致性要求

根据业务对数据一致性的要求选择隔离级别。

**选择原则**：

- **一般业务**：READ COMMITTED 或 REPEATABLE READ
- **财务系统**：SERIALIZABLE
- **统计报表**：READ COMMITTED
- **高并发场景**：REPEATABLE READ（MySQL InnoDB 默认）

### 应用场景

**READ COMMITTED 适用场景**：

- 读多写少的应用
- 对数据一致性要求不高的场景
- 统计分析系统

**REPEATABLE READ 适用场景**：

- 大多数业务应用（MySQL InnoDB 默认）
- 需要保证同一事务中数据一致性的场景
- 平衡性能和一致性的场景

**SERIALIZABLE 适用场景**：

- 财务系统
- 对数据一致性要求极高的场景
- 可以接受性能损失的场景

### 选择原则

**一般原则**：

1. **默认使用 REPEATABLE READ**（MySQL InnoDB 默认）
2. **根据业务需求调整**：如果业务允许不可重复读，可以使用 READ COMMITTED
3. **性能优先时**：使用较低隔离级别
4. **一致性优先时**：使用较高隔离级别

**示例**：

```php
<?php
declare(strict_types=1);

// 财务系统：使用 SERIALIZABLE
$pdo->exec('SET TRANSACTION ISOLATION LEVEL SERIALIZABLE');

// 一般业务：使用 REPEATABLE READ（默认）
$pdo->exec('SET TRANSACTION ISOLATION LEVEL REPEATABLE READ');

// 统计分析：使用 READ COMMITTED
$pdo->exec('SET TRANSACTION ISOLATION LEVEL READ COMMITTED');
```

---

## 完整示例

### 并发控制示例

```php
<?php
declare(strict_types=1);

class AccountService
{
    private PDO $pdo;
    
    public function __construct(PDO $pdo)
    {
        $this->pdo = $pdo;
        // 设置隔离级别
        $this->pdo->exec('SET TRANSACTION ISOLATION LEVEL REPEATABLE READ');
    }
    
    public function transfer(int $fromUserId, int $toUserId, float $amount): bool
    {
        try {
            $this->pdo->beginTransaction();
            
            // 使用 FOR UPDATE 锁定行，防止并发问题
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
            $stmt = $this->pdo->prepare('SELECT balance FROM accounts WHERE user_id = :user_id FOR UPDATE');
            $stmt->execute(['user_id' => $toUserId]);
            
            $stmt = $this->pdo->prepare('UPDATE accounts SET balance = balance + :amount WHERE user_id = :user_id');
            $stmt->execute([
                'amount' => $amount,
                'user_id' => $toUserId
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

### 多用户系统

多用户系统中，多个用户可能同时访问和修改数据。

**示例**：

```php
<?php
declare(strict_types=1);

// 设置隔离级别
$pdo->exec('SET TRANSACTION ISOLATION LEVEL REPEATABLE READ');

$pdo->beginTransaction();

// 使用 FOR UPDATE 锁定
$stmt = $pdo->prepare('SELECT * FROM products WHERE id = :id FOR UPDATE');
$stmt->execute(['id' => 1]);
$product = $stmt->fetch();

// 更新库存
$stmt = $pdo->prepare('UPDATE products SET stock = stock - 1 WHERE id = :id');
$stmt->execute(['id' => 1]);

$pdo->commit();
```

### 高并发应用

高并发应用中，需要平衡性能和一致性。

**示例**：

```php
<?php
declare(strict_types=1);

// 根据业务需求选择隔离级别
if ($isFinancialOperation) {
    // 财务操作：使用 SERIALIZABLE
    $pdo->exec('SET TRANSACTION ISOLATION LEVEL SERIALIZABLE');
} else {
    // 一般操作：使用 REPEATABLE READ
    $pdo->exec('SET TRANSACTION ISOLATION LEVEL REPEATABLE READ');
}
```

---

## 注意事项

### 隔离级别与性能

隔离级别越高，性能越低。

**影响**：

- **锁的粒度**：隔离级别越高，锁的粒度越大
- **并发度**：隔离级别越高，并发度越低
- **响应时间**：隔离级别越高，响应时间越长

**建议**：在保证数据一致性的前提下，使用尽可能低的隔离级别。

### 隔离级别与一致性

隔离级别越高，数据一致性越好。

**选择原则**：

- 根据业务对一致性的要求选择
- 财务系统需要最高一致性
- 一般业务可以使用中等隔离级别

### 应用场景选择

根据应用场景选择合适的隔离级别。

**选择建议**：

- **Web 应用**：REPEATABLE READ（MySQL InnoDB 默认）
- **财务系统**：SERIALIZABLE
- **统计分析**：READ COMMITTED
- **高并发场景**：REPEATABLE READ

### 默认隔离级别

MySQL InnoDB 默认隔离级别是 REPEATABLE READ。

**验证**：

```php
<?php
declare(strict_types=1);

$stmt = $pdo->query('SELECT @@transaction_isolation');
$level = $stmt->fetchColumn();
echo "当前隔离级别: $level\n";
```

---

## 常见问题

### 什么是并发控制？

并发控制是数据库管理系统用于协调多个事务同时访问同一数据资源的机制，确保数据的一致性和完整性。

### 并发问题有哪些？

主要的并发问题包括：

- **脏读**：读取未提交的数据
- **不可重复读**：同一事务中多次读取结果不一致
- **幻读**：同一事务中多次查询结果集不同

### 如何选择隔离级别？

选择原则：

1. 默认使用 REPEATABLE READ（MySQL InnoDB 默认）
2. 根据业务对一致性的要求调整
3. 在性能和一致性之间找到平衡
4. 高并发场景优先考虑性能

### 隔离级别对性能的影响？

隔离级别越高，性能越低：

- **READ UNCOMMITTED**：性能最高
- **READ COMMITTED**：性能较高
- **REPEATABLE READ**：性能良好
- **SERIALIZABLE**：性能最低

---

## 最佳实践

### 理解并发问题

深入理解脏读、不可重复读、幻读等并发问题，有助于选择合适的隔离级别。

### 选择合适的隔离级别

根据业务需求选择合适的隔离级别：

- 一般业务：REPEATABLE READ
- 财务系统：SERIALIZABLE
- 统计分析：READ COMMITTED

### 平衡性能和数据一致性

在性能和数据一致性之间找到平衡：

- 性能优先：使用较低隔离级别
- 一致性优先：使用较高隔离级别
- 大多数场景：REPEATABLE READ

### 测试并发场景

在开发过程中测试并发场景，确保数据一致性。

```php
<?php
declare(strict_types=1);

// 模拟并发场景
$pdo1 = new PDO($dsn, $user, $pass);
$pdo2 = new PDO($dsn, $user, $pass);

// 事务 1
$pdo1->beginTransaction();
$stmt = $pdo1->prepare('SELECT balance FROM accounts WHERE user_id = :user_id');
$stmt->execute(['user_id' => 1]);

// 事务 2（并发）
$pdo2->beginTransaction();
$stmt = $pdo2->prepare('UPDATE accounts SET balance = balance - 100 WHERE user_id = :user_id');
$stmt->execute(['user_id' => 1]);
$pdo2->commit();

// 验证结果
```

---

## 练习任务

1. **并发问题演示**
   - 演示脏读问题
   - 演示不可重复读问题
   - 演示幻读问题
   - 使用不同隔离级别测试

2. **隔离级别测试**
   - 测试不同隔离级别的效果
   - 对比性能和一致性
   - 选择适合的隔离级别
   - 验证隔离级别设置

3. **并发控制应用**
   - 实现转账功能
   - 使用 FOR UPDATE 锁定
   - 测试并发场景
   - 验证数据一致性

4. **性能测试**
   - 测试不同隔离级别的性能
   - 分析性能差异
   - 优化隔离级别选择
   - 编写性能测试报告

5. **综合应用**
   - 创建一个高并发业务场景
   - 实现并发控制
   - 测试各种并发情况
   - 优化性能和一致性

---

**相关章节**：

- [6.2.4 高级特性](../chapter-02-pdo/section-04-advanced-features.md)
- [6.3.2 MVCC 机制](section-02-mvcc.md)
- [6.3.3 锁机制](section-03-locking.md)
