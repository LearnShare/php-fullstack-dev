# 5.5 并发控制与 MVCC

## 目标

- 理解数据库并发控制的基本概念。
- 掌握快照读与当前读的区别。
- 了解 MVCC（多版本并发控制）的工作原理。
- 理解 Undo Log 的作用。
- 熟悉幻读（Phantom Read）问题及解决方案。

## 并发问题

### 脏读（Dirty Read）

- 读取到未提交的数据。

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

- 同一事务中多次读取同一数据，结果不一致。

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

- 同一事务中多次查询，结果集数量不一致。

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

## MVCC（多版本并发控制）

### 什么是 MVCC

- **MVCC**：Multi-Version Concurrency Control（多版本并发控制）。
- InnoDB 使用 MVCC 实现可重复读隔离级别。
- 通过保存数据的历史版本来实现并发控制，避免读写冲突。

### 为什么需要 MVCC

**问题**：传统的锁机制会导致读写冲突
- 读操作需要等待写操作完成
- 写操作需要等待读操作完成
- 降低并发性能

**解决方案**：MVCC 允许读操作读取历史版本，写操作更新当前版本
- 读操作不需要等待写操作
- 写操作不需要等待读操作（除非是同一行）
- 提高并发性能

### MVCC 工作原理（渐进式理解）

#### 第一步：理解版本链

每行数据都有一个隐藏的版本链，存储该行的所有历史版本：

```
行数据（id=1, balance=1000）的版本链：
┌─────────────────────────────────────┐
│ 当前版本 (trx_id=100, balance=1000) │ ← 最新版本
│         ↓                            │
│ 历史版本 (trx_id=50, balance=800)    │
│         ↓                            │
│ 历史版本 (trx_id=20, balance=500)    │ ← 最早版本
└─────────────────────────────────────┘
```

**说明**：
- `trx_id`：事务 ID，标识哪个事务创建了这个版本
- 每次更新都会创建新版本，旧版本保留在 Undo Log 中

#### 第二步：理解 ReadView

每个事务在开始时创建一个 ReadView，决定该事务能看到哪些版本：

```php
<?php
declare(strict_types=1);

/**
 * ReadView 示例（简化版）
 * 
 * ReadView 包含：
 * 1. m_ids：创建 ReadView 时活跃的事务 ID 列表
 * 2. min_trx_id：最小活跃事务 ID
 * 3. max_trx_id：最大事务 ID（下一个要分配的事务 ID）
 */
class ReadView
{
    public function __construct(
        private array $activeTransactionIds,  // 活跃事务列表
        private int $minTrxId,                // 最小活跃事务 ID
        private int $maxTrxId                 // 最大事务 ID
    ) {}
    
    /**
     * 判断版本是否对当前事务可见
     * 
     * 规则：
     * 1. 如果版本的 trx_id < min_trx_id：可见（已提交）
     * 2. 如果版本的 trx_id >= max_trx_id：不可见（未来事务）
     * 3. 如果版本的 trx_id 在活跃列表中：不可见（未提交）
     * 4. 否则：可见（已提交）
     */
    public function isVisible(int $versionTrxId): bool
    {
        // 规则 1：已提交的旧事务
        if ($versionTrxId < $this->minTrxId) {
            return true;
        }
        
        // 规则 2：未来事务
        if ($versionTrxId >= $this->maxTrxId) {
            return false;
        }
        
        // 规则 3：活跃事务（未提交）
        if (in_array($versionTrxId, $this->activeTransactionIds, true)) {
            return false;
        }
        
        // 规则 4：已提交的事务
        return true;
    }
}
```

#### 第三步：完整示例

```php
<?php
declare(strict_types=1);

/**
 * MVCC 工作流程示例
 * 
 * 场景：两个事务同时操作同一行数据
 */

// 初始数据：accounts 表，id=1, balance=1000

// 事务 A（trx_id=100）：读取余额
$pdoA = new PDO('mysql:host=localhost;dbname=test', 'user', 'password');
$pdoA->beginTransaction();
$pdoA->exec("SET TRANSACTION ISOLATION LEVEL REPEATABLE READ");

// 创建 ReadView A
// 假设此时活跃事务：[100]
// min_trx_id = 100, max_trx_id = 101
$readViewA = new ReadView([100], 100, 101);

// 读取数据：沿着版本链查找可见的版本
// 当前版本 trx_id=50（已提交），可见
$stmt = $pdoA->query("SELECT balance FROM accounts WHERE id = 1");
$balanceA1 = $stmt->fetchColumn();  // 1000

// 事务 B（trx_id=101）：更新余额
$pdoB = new PDO('mysql:host=localhost;dbname=test', 'user', 'password');
$pdoB->beginTransaction();
$pdoB->exec("UPDATE accounts SET balance = 1200 WHERE id = 1");
// 创建新版本：trx_id=101, balance=1200
// 旧版本（trx_id=50, balance=1000）保存到 Undo Log

// 事务 A 再次读取（使用相同的 ReadView）
$stmt = $pdoA->query("SELECT balance FROM accounts WHERE id = 1");
// 沿着版本链查找：
// 1. 当前版本 trx_id=101，在活跃列表中，不可见
// 2. 继续查找：历史版本 trx_id=50，可见
$balanceA2 = $stmt->fetchColumn();  // 仍然是 1000（快照读）

// 事务 B 提交
$pdoB->commit();

// 事务 A 再次读取（ReadView 不变）
$stmt = $pdoA->query("SELECT balance FROM accounts WHERE id = 1");
// 仍然读取历史版本（ReadView 在事务开始时创建，不会改变）
$balanceA3 = $stmt->fetchColumn();  // 仍然是 1000

// 事务 A 提交
$pdoA->commit();

// 新事务 C（trx_id=102）：读取数据
$pdoC = new PDO('mysql:host=localhost;dbname=test', 'user', 'password');
$pdoC->beginTransaction();

// 创建 ReadView C
// 此时活跃事务：[102]
// min_trx_id = 102, max_trx_id = 103
$readViewC = new ReadView([102], 102, 103);

// 读取数据：当前版本 trx_id=101（已提交），可见
$stmt = $pdoC->query("SELECT balance FROM accounts WHERE id = 1");
$balanceC = $stmt->fetchColumn();  // 1200（最新版本）
```

### MVCC 工作原理总结

1. **版本链**：每行数据维护一个版本链，存储所有历史版本
2. **ReadView**：事务创建时生成，决定可见的版本
3. **Undo Log**：存储历史版本数据，用于版本链查找
4. **可见性判断**：根据 ReadView 规则判断版本是否可见

### 快照读

- 读取数据的历史版本，不读取最新版本。
- 使用 MVCC 实现，无需加锁。
- 保证可重复读：同一事务中多次读取结果一致。

```php
<?php
declare(strict_types=1);

/**
 * 快照读示例
 * 
 * 在 REPEATABLE READ 隔离级别下，普通 SELECT 使用快照读
 */
$pdo->beginTransaction();
$pdo->exec("SET TRANSACTION ISOLATION LEVEL REPEATABLE READ");

// 第一次读取：创建 ReadView，读取可见版本
$stmt = $pdo->query("SELECT * FROM accounts WHERE id = 1");
$balance1 = $stmt->fetchColumn();  // 假设是 1000

// 其他事务更新数据（创建新版本）
// ...

// 第二次读取：使用相同的 ReadView，仍然读取历史版本
$stmt = $pdo->query("SELECT * FROM accounts WHERE id = 1");
$balance2 = $stmt->fetchColumn();  // 仍然是 1000（快照读）

// 保证可重复读：balance1 === balance2
```

### 当前读

- 读取数据的最新版本，需要加锁。

```php
<?php
declare(strict_types=1);

// 当前读（加锁的 SELECT）
$pdo->beginTransaction();

// SELECT ... FOR UPDATE（排他锁）
$stmt = $pdo->prepare("SELECT * FROM accounts WHERE id = ? FOR UPDATE");
$stmt->execute([1]);
$account = $stmt->fetch();

// SELECT ... LOCK IN SHARE MODE（共享锁）
$stmt = $pdo->prepare("SELECT * FROM accounts WHERE id = ? LOCK IN SHARE MODE");
$stmt->execute([1]);
$account = $stmt->fetch();
```

## Undo Log

### Undo Log 作用

- 存储数据的历史版本。
- 支持事务回滚。
- 支持 MVCC 快照读。

### Undo Log 结构

```
Undo Log 记录：
- 事务 ID
- 回滚指针
- 历史版本数据
```

### 回滚示例

```php
<?php
declare(strict_types=1);

$pdo->beginTransaction();

// 更新数据（生成 Undo Log）
$stmt = $pdo->prepare("UPDATE accounts SET balance = balance + 100 WHERE id = ?");
$stmt->execute([1]);

// 回滚（使用 Undo Log 恢复）
$pdo->rollBack();
```

## InnoDB 锁机制

### 行锁类型

- **共享锁（S Lock）**：允许读取，禁止写入。
- **排他锁（X Lock）**：禁止读取和写入。

### 锁的兼容性

|      | S Lock | X Lock |
| :--- | :----- | :----- |
| S Lock | 兼容   | 不兼容 |
| X Lock | 不兼容 | 不兼容 |

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

## 幻读解决方案

### Next-Key Lock

- InnoDB 使用 Next-Key Lock 防止幻读。
- 锁定记录和记录之间的间隙。

```php
<?php
declare(strict_types=1);

// 使用 Next-Key Lock 防止幻读
$pdo->beginTransaction();
$pdo->exec("SET TRANSACTION ISOLATION LEVEL REPEATABLE READ");

// 范围查询加锁
$stmt = $pdo->prepare("SELECT * FROM orders WHERE user_id = ? FOR UPDATE");
$stmt->execute([1]);
// 锁定 user_id=1 的所有记录和间隙，防止插入新记录
```

### 唯一索引

- 使用唯一索引可以避免部分幻读问题。

```sql
-- 唯一索引防止重复插入
CREATE UNIQUE INDEX uk_user_order ON orders(user_id, order_number);
```

## 锁等待与超时

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

### 死锁检测

```php
<?php
declare(strict_types=1);

/**
 * 安全更新函数：自动处理死锁重试
 * 
 * @param PDO $pdo 数据库连接
 * @param callable $callback 要执行的回调函数（包含数据库操作）
 * @param int $maxRetries 最大重试次数
 * @return mixed 回调函数的返回值
 * @throws PDOException 如果重试后仍然失败，抛出异常
 */
function safeUpdate(PDO $pdo, callable $callback, int $maxRetries = 3): mixed
{
    $attempts = 0;
    
    // 循环重试，直到成功或达到最大重试次数
    while ($attempts < $maxRetries) {
        try {
            // 开始事务
            $pdo->beginTransaction();
            
            // 执行回调函数（包含实际的数据库操作）
            $result = $callback();
            
            // 提交事务
            $pdo->commit();
            
            // 成功，返回结果
            return $result;
            
        } catch (PDOException $e) {
            // 回滚事务，确保数据一致性
            $pdo->rollBack();
            
            // 检查是否是死锁错误（错误码 1213）
            // 如果是死锁且还有重试机会，则重试
            if ($e->getCode() == 1213 && $attempts < $maxRetries - 1) {
                $attempts++;
                
                // 随机延迟 100-500 毫秒，避免多个事务同时重试造成新的死锁
                // 使用随机延迟可以分散重试时间，减少冲突
                usleep(mt_rand(100000, 500000));
                
                // 继续下一次重试
                continue;
            }
            
            // 非死锁错误，或已达到最大重试次数，抛出异常
            throw $e;
        }
    }
}
```

## 性能优化

### 减少锁持有时间

```php
<?php
// 不推荐：长时间持有锁
$pdo->beginTransaction();
$stmt = $pdo->prepare("SELECT * FROM accounts WHERE id = ? FOR UPDATE");
$stmt->execute([1]);
sleep(10);  // 长时间操作
$pdo->commit();

// 推荐：快速获取和释放锁
$pdo->beginTransaction();
$stmt = $pdo->prepare("SELECT * FROM accounts WHERE id = ? FOR UPDATE");
$stmt->execute([1]);
$balance = $stmt->fetchColumn();
$pdo->commit();

// 长时间操作（不持有锁）
sleep(10);

// 再次加锁更新
$pdo->beginTransaction();
$stmt = $pdo->prepare("UPDATE accounts SET balance = ? WHERE id = ?");
$stmt->execute([$balance + 100, 1]);
$pdo->commit();
```

### 使用合适的隔离级别

```php
<?php
// 读多写少：使用 REPEATABLE READ（默认）
$pdo->exec("SET TRANSACTION ISOLATION LEVEL REPEATABLE READ");

// 写多读少：使用 READ COMMITTED（性能更好）
$pdo->exec("SET TRANSACTION ISOLATION LEVEL READ COMMITTED");
```

## 最佳实践

### 1. 按固定顺序加锁

```php
// 避免死锁：始终按 ID 顺序加锁
$ids = [$id1, $id2];
sort($ids);

foreach ($ids as $id) {
    $stmt = $pdo->prepare("SELECT * FROM accounts WHERE id = ? FOR UPDATE");
    $stmt->execute([$id]);
}
```

### 2. 使用索引

- 索引可以减少锁的范围。
- 避免全表扫描导致的表锁。

### 3. 避免长事务

- 长事务会增加锁持有时间。
- 增加死锁和锁等待的概率。

## 练习

1. 实现一个并发安全的账户余额更新函数，使用行锁防止并发问题。

2. 编写一个测试程序，演示脏读、不可重复读、幻读问题。

3. 创建一个死锁检测和自动重试机制。

4. 实现一个乐观锁机制，使用版本号控制并发更新。

5. 设计一个锁监控系统，记录锁等待和死锁信息。

6. 编写性能测试，比较不同隔离级别和锁策略的性能差异。
