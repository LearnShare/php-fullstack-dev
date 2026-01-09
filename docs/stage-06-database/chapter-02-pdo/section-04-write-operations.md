# 6.2.4 执行写操作与获取ID (Write Operations & Getting ID)

## 概述

除了查询数据，应用程序的核心功能还包括对数据的修改：插入 (INSERT)、更新 (UPDATE) 和删除 (DELETE)。在 PDO 中，执行这些“写”操作同样遵循 `prepare()` -> `execute()` 的安全模式，以防止 SQL 注入。

本节将分别演示如何执行这三种操作，并介绍一个非常重要的功能：如何在插入新纪录后获取其自动生成的 ID。

## 插入数据 (INSERT)

执行 `INSERT` 语句与执行 `SELECT` 语句非常相似，只是它不返回结果集。

**示例**：向 `users` 表中插入一个新用户。

```php
<?php
// ... $pdo 连接对象

$username = 'johndoe';
$email = 'john.doe@example.com';
// 密码应该总是被哈希后存储
$password_hash = password_hash('password123', PASSWORD_DEFAULT);

try {
    // 1. 准备 SQL 模板
    $sql = "INSERT INTO users (username, email, password_hash) VALUES (:username, :email, :password)";
    $stmt = $pdo->prepare($sql);

    // 2. 准备要插入的数据数组
    $data = [
        ':username' => $username,
        ':email' => $email,
        ':password' => $password_hash,
    ];

    // 3. 执行
    $stmt->execute($data);

    echo "新用户已成功插入！";

} catch (\PDOException $e) {
    // 处理可能发生的错误，例如'username'或'email'已存在（如果设置了唯一约束）
    if ($e->getCode() == 23000) {
        echo "错误：用户名或邮箱已存在。";
    } else {
        throw $e; // 重新抛出其他类型的错误
    }
}
```

## 获取最后插入的 ID

当向一个有 `AUTO_INCREMENT` 主键的表中插入新记录后，我们经常需要立即知道这个新纪录的 ID，以便进行后续操作（例如，为新注册的用户创建个人资料记录）。

PDO 提供了 `PDO::lastInsertId()` 方法来实现这一功能。

**重要规则**：
1.  必须在执行 `INSERT` 语句的 `execute()` 之后**立即**调用。
2.  必须在**同一个数据库连接**上调用。
3.  它返回的是**此连接**上最后一次 `INSERT` 操作产生的 ID，不受其他并发连接的影响。
4.  该方法是调用在 **PDO 对象**上，而不是 `PDOStatement` 对象上。

**示例**：插入一个新用户并立即获取其 ID。

```php
<?php
// ... $pdo 连接对象

$username = 'janedoe';
$email = 'jane.doe@example.com';
$password_hash = password_hash('password456', PASSWORD_DEFAULT);

try {
    $sql = "INSERT INTO users (username, email, password_hash) VALUES (:username, :email, :password)";
    $stmt = $pdo->prepare($sql);
    
    $stmt->execute([
        ':username' => $username,
        ':email' => $email,
        ':password' => $password_hash,
    ]);

    // 立即获取新用户的 ID
    $newUserId = $pdo->lastInsertId();

    echo "用户 '{$username}' 已成功创建，其 ID 为: {$newUserId}";

    // 现在你可以使用 $newUserId 进行后续操作，例如创建用户资料
    // $pdo->prepare("INSERT INTO user_profiles (user_id, ...) ...")->execute([':user_id' => $newUserId, ...]);

} catch (\PDOException $e) {
    // ... 错误处理
    throw $e;
}
```

## 更新数据 (UPDATE)

执行 `UPDATE` 操作时，预处理语句对于 `SET` 子句中的新值和 `WHERE` 子句中的条件值都至关重要。

**示例**：根据用户 ID 更新其邮箱地址。

```php
<?php
// ... $pdo 连接对象

$userId = 1;
$newEmail = 'admin.updated@example.com';

try {
    // 准备 SQL 模板，注意 SET 和 WHERE 子句都使用了占位符
    $sql = "UPDATE users SET email = :email WHERE id = :id";
    $stmt = $pdo->prepare($sql);

    // 执行更新
    $stmt->execute([
        ':email' => $newEmail,
        ':id' => $userId,
    ]);
    
    // 检查受影响的行数
    $affectedRows = $stmt->rowCount();
    
    if ($affectedRows > 0) {
        echo "用户 {$userId} 的邮箱已成功更新。";
    } else {
        echo "没有行被更新。可能原因：用户不存在，或新邮箱与旧邮箱相同。";
    }

} catch (\PDOException $e) {
    // ... 错误处理
    throw $e;
}
```
`PDOStatement::rowCount()` 在这里非常有用。它可以告诉你 `UPDATE` 语句实际影响了多少行。如果返回 `0`，并不一定意味着错误，可能只是 `WHERE` 条件没有匹配到任何行，或者你提供的新值与数据库中已有的值完全相同。

## 删除数据 (DELETE)

执行 `DELETE` 操作与 `UPDATE` 非常相似，同样需要对 `WHERE` 子句中的条件使用预处理语句。

**示例**：根据用户 ID 删除用户。

```php
<?php
// ... $pdo 连接对象

$userIdToDelete = 2;

try {
    // 准备 SQL 模板
    $sql = "DELETE FROM users WHERE id = :id";
    $stmt = $pdo->prepare($sql);

    // 执行删除
    $stmt->execute([':id' => $userIdToDelete]);

    // 检查受影响的行数
    $affectedRows = $stmt->rowCount();

    if ($affectedRows > 0) {
        echo "成功删除了 {$affectedRows} 行。";
    } else {
        echo "没有行被删除。可能该用户不存在。";
    }

} catch (\PDOException $e) {
    // ... 错误处理
    throw $e;
}
```
对于写操作，`rowCount()` 的返回值是判断操作是否按预期执行的重要依据。

## 总结

-   所有 `INSERT`, `UPDATE`, `DELETE` 操作都应遵循 `prepare()` -> `execute()` 的模式来防止 SQL 注入。
-   `INSERT` 后，使用 `$pdo->lastInsertId()` 来获取新记录的自增 ID。
-   `UPDATE` 和 `DELETE` 后，使用 `$stmt->rowCount()` 来检查有多少行受到了影响。

---

## 练习任务

1.  **产品管理**：
    -   创建一个 `products` 表，包含 `id` (AUTO_INCREMENT), `name` (VARCHAR), `price` (DECIMAL(10,2)), `stock` (INT)。
    -   编写一个脚本 `add_product.php`，使用预处理语句向该表插入一个新产品，并在成功后打印出新产品的 ID。

2.  **更新库存**：
    -   编写一个脚本 `update_stock.php`，接收 `product_id` 和 `quantity` 两个参数。
    -   使用预处理语句，将对应产品的 `stock` 增加指定的 `quantity`。
    -   执行后，打印出受影响的行数。

3.  **删除绝版商品**：
    -   假设 `products` 表中 `stock` 为 0 的商品为绝版商品。
    -   编写一个脚本 `delete_discontinued.php`，使用预处理语句删除所有库存为 0 的商品。
    -   执行后，打印出总共删除了多少件绝版商品。
