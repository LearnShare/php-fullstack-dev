# 5.5.2 MVCC 机制

## 概述

MVCC（Multi-Version Concurrency Control，多版本并发控制）是 InnoDB 实现可重复读隔离级别的核心机制。本节详细介绍 MVCC 原理、版本链、ReadView、快照读与当前读，以及 Undo Log 的作用。

## MVCC 原理

### 什么是 MVCC

- **MVCC**：Multi-Version Concurrency Control（多版本并发控制）
- InnoDB 使用 MVCC 实现可重复读隔离级别
- 通过保存数据的历史版本来实现并发控制，避免读写冲突

### 为什么需要 MVCC

**问题**：传统的锁机制会导致读写冲突
- 读操作需要等待写操作完成
- 写操作需要等待读操作完成
- 降低并发性能

**解决方案**：MVCC 允许读操作读取历史版本，写操作更新当前版本
- 读操作不需要等待写操作
- 写操作不需要等待读操作（除非是同一行）
- 提高并发性能

## 版本链

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

## ReadView

每个事务在开始时创建一个 ReadView，决定该事务能看到哪些版本：

```php
<?php
declare(strict_types=1);

class ReadView
{
    public function __construct(
        private array $activeTransactionIds,  // 活跃事务列表
        private int $minTrxId,                // 最小活跃事务 ID
        private int $maxTrxId                 // 最大事务 ID
    ) {}
    
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

## 快照读与当前读

### 快照读

- 读取数据的历史版本，不读取最新版本
- 使用 MVCC 实现，无需加锁
- 保证可重复读：同一事务中多次读取结果一致

```php
<?php
declare(strict_types=1);

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
```

### 当前读

- 读取数据的最新版本，需要加锁

```php
<?php
declare(strict_types=1);

// 当前读（加锁的 SELECT）
$pdo->beginTransaction();

// SELECT ... FOR UPDATE（排他锁）
$stmt = $pdo->prepare("SELECT * FROM accounts WHERE id = ? FOR UPDATE");
$stmt->execute([1]);
$account = $stmt->fetch();
```

## Undo Log

### Undo Log 作用

- 存储数据的历史版本
- 支持事务回滚
- 支持 MVCC 快照读

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

## 完整示例

```php
<?php
declare(strict_types=1);

class MVCCDemo
{
    private PDO $pdo;

    public function demonstrateSnapshotRead(): void
    {
        // 事务 A：快照读
        $this->pdo->beginTransaction();
        $this->pdo->exec("SET TRANSACTION ISOLATION LEVEL REPEATABLE READ");
        
        // 创建 ReadView，读取可见版本
        $stmt = $this->pdo->query("SELECT balance FROM accounts WHERE id = 1");
        $balance1 = $stmt->fetchColumn();
        
        // 其他事务更新数据...
        
        // 再次读取，使用相同的 ReadView
        $stmt = $this->pdo->query("SELECT balance FROM accounts WHERE id = 1");
        $balance2 = $stmt->fetchColumn();
        
        // 保证可重复读：balance1 === balance2
        assert($balance1 === $balance2);
        
        $this->pdo->commit();
    }
}
```

## 注意事项

1. **版本链**：每行数据维护版本链
2. **ReadView**：事务创建时生成，决定可见版本
3. **Undo Log**：存储历史版本数据
4. **性能考虑**：MVCC 提升并发性能

## 练习

1. 实现一个 MVCC 演示程序，展示快照读的工作原理。

2. 创建一个 ReadView 模拟器，验证可见性判断规则。

3. 编写一个测试，验证 MVCC 的可重复读特性。

4. 实现一个 Undo Log 查看工具，分析版本链。
