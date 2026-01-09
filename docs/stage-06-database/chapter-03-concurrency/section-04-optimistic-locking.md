# 6.3.4 锁机制：乐观锁 (Locking Mechanism: Optimistic Locking)

## 概述

与悲观锁的“先锁后做”策略相反，**乐观锁 (Optimistic Locking)** 对数据冲突持“乐观”态度。它**假设并发冲突是稀有事件**，因此它在处理数据时并不会预先加锁。

乐观锁的策略是：
1.  自由地读取数据。
2.  在准备提交更新时，先**检查**一下，在此期间是否有其他事务已经修改了这份数据。
3.  如果数据未被修改，则成功提交更新。
4.  如果数据已被修改，则认为发生了冲突，放弃本次更新，并由应用程序决定如何处理冲突（例如，提示用户、自动重试或合并更改）。

这种策略可以概括为“先做后验”。因为它避免了数据库层面的加锁开销，所以在**读多写少**、**数据竞争不激烈**的场景下，可以极大地提升并发性能。绝大多数 Web 应用，如内容管理系统 (CMS)、用户资料编辑等，都非常适合使用乐观锁。

## 实现方法

乐观锁通常不是由数据库直接提供，而是由**应用程序层面**的逻辑来实现。最主流的实现方式有两种：**版本号机制**和**时间戳机制**。

### 1. 版本号机制 (Version Number)

这是实现乐观锁**最推荐、最常用**的方法。

**实现步骤**：
1.  **添加版本号字段**：在需要进行乐观锁控制的表中，增加一个整型字段，通常命名为 `version`，并给一个默认值（如 1）。
    ```sql
    ALTER TABLE products ADD COLUMN version INT UNSIGNED NOT NULL DEFAULT 1;
    ```
2.  **读取数据时获取版本号**：当应用程序读取一行数据时，必须同时将 `version` 字段的值也读取出来。
    ```php
    // ...
    $stmt = $pdo->prepare("SELECT id, stock, version FROM products WHERE id = :id");
    $stmt->execute([':id' => $productId]);
    $product = $stmt->fetch();
    $currentVersion = $product['version'];
    ```
3.  **更新数据时校验并递增版本号**：在提交 `UPDATE` 时，必须在 `WHERE` 子句中校验 `version` 是否与读取时的一致，并在 `SET` 子句中将 `version` 加 1。
    ```sql
    UPDATE products
    SET stock = :new_stock, version = version + 1
    WHERE id = :id AND version = :expected_version;
    ```
4.  **检查受影响的行数**：执行 `UPDATE`后，检查 `rowCount()` 的返回值。
    -   如果 `rowCount()` 为 `1`，说明 `WHERE` 子句（`id` 和 `version`）匹配成功，更新已完成。
    -   如果 `rowCount()` 为 `0`，说明在你读取数据和提交更新之间，已经有另一个事务修改了该行数据，导致 `version` 不匹配，更新失败。

**PHP + PDO 完整示例**:
```php
<?php
// ... $pdo 连接对象

$productId = 101;
$quantityToBuy = 1;

try {
    // 1. 读取数据和版本号
    $stmt = $pdo->prepare("SELECT stock, version FROM products WHERE id = :id");
    $stmt->execute([':id' => $productId]);
    $product = $stmt->fetch();

    if (!$product) {
        throw new \Exception("商品不存在！");
    }

    $currentStock = (int)$product['stock'];
    $currentVersion = (int)$product['version'];

    // 2. 业务逻辑判断
    if ($currentStock >= $quantityToBuy) {
        $newStock = $currentStock - $quantityToBuy;

        // 3. 更新时带上版本号校验
        $stmtUpdate = $pdo->prepare(
            "UPDATE products 
             SET stock = :stock, version = version + 1 
             WHERE id = :id AND version = :version"
        );
        
        $stmtUpdate->execute([
            ':stock' => $newStock,
            ':id' => $productId,
            ':version' => $currentVersion
        ]);

        // 4. 检查更新是否成功
        if ($stmtUpdate->rowCount() > 0) {
            echo "购买成功！商品版本已更新。";
        } else {
            // 如果 rowCount() 为 0，说明发生冲突
            throw new \Exception("操作冲突，请刷新后重试。可能其他用户已修改了库存。");
        }
    } else {
        throw new \Exception("库存不足！");
    }

} catch (\Exception $e) {
    echo "操作失败: " . $e->getMessage();
    // 在这里，应用可以决定是简单地显示错误，还是重新加载数据让用户再试一次
}
```

### 2. 时间戳机制 (Timestamp)

这种方法的原理与版本号机制类似，只是它使用一个时间戳字段（如 `updated_at`）来代替 `version` 字段。

-   **实现步骤**：
    1.  在表中增加一个 `TIMESTAMP` 或 `DATETIME` 类型的字段，如 `updated_at`，并设置为 `ON UPDATE CURRENT_TIMESTAMP`。
    2.  读取数据时，获取 `updated_at` 的值。
    3.  更新数据时，在 `WHERE` 子句中校验 `updated_at` 的值是否与读取时一致。

-   **缺点与风险**：
    -   **精度问题**：时间戳的精度可能不够高。在并发量极大的情况下，多个事务可能在同一个时间单位内完成，导致冲突检测失效。
    -   **分布式系统问题**：在分布式系统中，不同服务器的时钟可能存在微小的偏差，依赖时间戳会带来不确定性。
    -   **逻辑表达不清**：版本号的自增明确地表达了“更新次数”的含义，而时间戳则没有这个含义。

**结论**：由于存在上述风险，**强烈推荐使用版本号机制**来实现乐观锁。

## 悲观锁 vs. 乐观锁

| 特性         | 悲观锁 (Pessimistic Locking)                                | 乐观锁 (Optimistic Locking)                                    |
|:-------------|:------------------------------------------------------------|:---------------------------------------------------------------|
| **核心思想** | 假设冲突会发生，先加锁，再操作。                            | 假设冲突很少发生，先操作，提交时再校验。                       |
| **实现机制** | 依赖数据库的锁机制（如 `SELECT ... FOR UPDATE`）。          | 依赖应用程序的逻辑（通常是 `version` 字段）。                  |
| **数据一致性** | 强一致性。由数据库保证，操作期间数据被独占。              | 最终一致性。在提交时才保证，可能需要应用重试。               |
| **并发性能** | 较低。锁会阻塞其他事务，降低了并发度。                      | 较高。无锁操作，允许多个事务同时读取和处理数据。             |
| **开销**     | 数据库锁的开销较大，可能导致死锁。                          | 几乎没有额外开销，只需增加一个版本字段。                       |
| **冲突处理** | 事务被阻塞等待，直到锁被释放。                              | 操作失败，由应用程序决定如何处理冲突（重试、报错等）。         |
| **适用场景** | **写密集型**、**高竞争**、事务短小的场景（如库存、金融）。  | **读密集型**、**低竞争**的场景（如编辑文章、修改用户资料）。 |

## 总结

乐观锁和悲观锁是两种应对并发问题的不同哲学。它们没有绝对的优劣，只有在特定业务场景下的适用性。
-   **悲观锁**把并发控制的压力交给了数据库，简单直接，但牺牲了性能。
-   **乐观锁**则把并发控制的逻辑放在了应用程序中，避免了锁的开销，提升了并发能力，但需要开发者自己处理冲突。

在现代 Web 开发中，由于大部分操作都是读操作，数据竞争通常不激烈，因此**乐观锁的应用场景更为广泛**。

---

## 练习任务

1.  **实现用户资料更新**：
    假设 `users` 表中有一个 `bio` (个人简介) 字段。请为 `users` 表增加一个 `version` 字段，并编写一个 `update_profile.php` 脚本。该脚本需要：
    1.  读取指定用户的 `bio` 和 `version`。
    2.  允许用户提交新的 `bio` 内容。
    3.  使用乐观锁的方式更新 `bio` 和 `version`。
    4.  模拟冲突：打开两个浏览器窗口，同时加载同一个用户的编辑页面，在一个窗口提交更新后，再在另一个窗口提交，观察并处理冲突。

2.  **选择合适的锁**：
    请分析以下业务场景，并说明你应该选择悲观锁还是乐观锁，并阐述理由：
    -   一个多人在线文档编辑应用，允许多个用户同时编辑同一篇文档。
    -   一个内部后台系统，管理员正在编辑一篇待发布的公告。
    -   一个电子商务网站，用户正在修改自己的收货地址。
    -   一个高并发的抽奖活动，用户点击“抽奖”按钮会扣减其积分。

3.  **乐观锁的ABA问题**：
    查阅资料，了解什么是乐观锁的“ABA问题”，并思考为什么基于版本号（递增）的乐观锁可以避免此问题，而基于时间戳的机制在理论上可能无法完全避免。
