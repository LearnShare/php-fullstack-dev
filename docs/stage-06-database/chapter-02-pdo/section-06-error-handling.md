# 6.2.6 错误处理模式 (Error Handling Modes)

## 概述

当数据库操作失败时（例如，SQL 语法错误、违反约束、连接失败），PDO 需要一种方式来通知应用程序。PDO 提供了三种不同的**错误处理模式**，让开发者可以根据自己的需求选择最合适的错误报告策略。

选择正确的错误处理模式对于构建健壮、可维护的应用程序至关重要。一个好的错误处理策略可以帮助我们快速定位问题、防止数据损坏，并向用户提供友好的反馈。

本节将详细介绍 PDO 的三种错误处理模式，并解释为什么其中一种是现代 PHP 开发的最佳选择。

## 三种错误处理模式

可以通过在 PDO 连接选项中设置 `PDO::ATTR_ERRMODE` 的值来选择错误处理模式。这三个值是：
1.  `PDO::ERRMODE_SILENT`
2.  `PDO::ERRMODE_WARNING`
3.  `PDO::ERRMODE_EXCEPTION`

---

### 1. `PDO::ERRMODE_SILENT` (静默模式)

-   **描述**：这是 PDO 的**默认**模式。当发生错误时，它**不会**产生任何可见的 PHP 错误或警告。它只是在内部设置错误码和错误信息。
-   **如何获取错误**：你必须在每次执行数据库操作后，手动调用 `PDO::errorCode()` 或 `PDOStatement::errorCode()` 来检查是否发生了错误，然后使用 `PDO::errorInfo()` 或 `PDOStatement::errorInfo()` 来获取详细信息。
-   **问题所在**：
    -   **极易被忽略**：如果开发者忘记在每次调用后都进行检查，错误就会悄无声-息地发生，导致所谓的“静默失败”。
    -   **代码冗长**：在每个数据库操作后都加上 `if ($stmt->errorCode() !== '00000') { ... }` 会让代码变得极其冗长和混乱。
-   **示例**：
    ```php
    <?php
    // 假设 $pdo 使用的是默认的静默模式
    
    // 故意制造一个语法错误
    $stmt = $pdo->query("SELECT oops_id, name FROM users");
    
    // 脚本不会停止，但 $stmt 是 false
    if ($stmt === false) {
        echo "发生了一个错误。\n";
        print_r($pdo->errorInfo());
    }
    
    echo "\n脚本继续执行...";
    // 输出:
    // 发生了一个错误。
    // Array
    // (
    //     [0] => 42S22
    //     [1] => 1054
    //     [2] => Unknown column 'oops_id' in 'field list'
    // )
    // 
    // 脚本继续继续执行...
    ```
-   **结论**：由于其容易导致未被发现的错误，**强烈不推荐**在任何现代应用程序中使用此模式。

### 2. `PDO::ERRMODE_WARNING` (警告模式)

-   **描述**：除了像静默模式一样设置错误码外，此模式还会额外发出一个标准的 PHP `E_WARNING` 级别的错误。
-   **问题所在**：默认情况下，`E_WARNING` **不会中止脚本的执行**。这意味着虽然你可能会在错误日志中看到一条警告，但你的应用程序逻辑会继续向下执行，就好像数据库操作成功了一样。这同样可能导致不可预料的行为和数据不一致。
-   **示例**：
    ```php
    <?php
    // $pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_WARNING);
    
    // 同样执行一个错误的查询
    $stmt = $pdo->query("SELECT oops_id, name FROM users");
    
    // PHP 会输出一条警告: 
    // Warning: PDO::query(): SQLSTATE[42S22]: Column not found: 1054 Unknown column 'oops_id' in 'field list' ...
    
    // 但脚本依旧继续
    echo "脚本继续执行...";
    ```
-   **结论**：比静默模式好一点，因为它至少会产生日志，但因为它不中断程序流程，所以仍然不是处理数据库错误的安全方式。

### 3. `PDO::ERRMODE_EXCEPTION` (异常模式)

-   **描述**：当发生数据库错误时，PDO 会**抛出一个 `PDOException` 异常**。
-   **这是现代 PHP 开发中唯一推荐的模式。**
-   **优点**：
    1.  **中断程序流程**：一个未被捕获的异常会立即终止脚本的执行，这可以防止在错误发生后对数据造成进一步的破坏。
    2.  **优雅的错误处理**：它允许你使用 `try...catch` 块来集中处理数据库错误。你可以将一系列数据库操作放在 `try` 块中，并在 `catch` 块中定义统一的错误处理逻辑（如回滚事务、记录日志、显示友好错误页面等）。
    3.  **清晰的逻辑分离**：将正常的业务逻辑（在 `try` 块中）与异常处理逻辑（在 `catch` 块中）完全分离开来，使代码更清晰、更易于维护。
    4.  **丰富的错误信息**：`PDOException` 对象包含了详细的错误信息（通过 `$e->getMessage()`）和 SQLSTATE 码（通过 `$e->getCode()`），便于调试和记录。
-   **示例**：
    ```php
    <?php
    // $pdo 在连接时已设置为 PDO::ERRMODE_EXCEPTION
    
    try {
        // 尝试执行一个错误的查询
        $stmt = $pdo->query("SELECT oops_id, name FROM users");
    
        // 如果上面一行抛出异常，这里的代码将永远不会被执行
        echo "查询成功！";
    
    } catch (PDOException $e) {
        // 捕获到异常
        echo "数据库操作失败！\n";
        echo "错误信息: " . $e->getMessage() . "\n";
        echo "错误码: " . $e->getCode() . "\n";
    
        // 在生产环境中，这里应该是记录日志，并向用户显示一个通用错误信息
        // error_log($e->getMessage());
        // header("Location: /error-page.html");
        // exit();
    }
    
    echo "脚本执行结束。"; // 这行代码在捕获异常后仍然会执行
    ```

## 如何设置错误模式

最佳实践是在创建 PDO 连接时，通过构造函数的第四个参数 `$options` 数组来立即设置错误模式。

```php
<?php
$options = [
    PDO::ATTR_ERRMODE            => PDO::ERRMODE_EXCEPTION,
    PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
    PDO::ATTR_EMULATE_PREPARES   => false,
];

try {
    $pdo = new PDO($dsn, $user, $pass, $options);
} catch (PDOException $e) {
    throw new PDOException($e->getMessage(), (int)$e->getCode());
}
```

当然，你也可以在连接之后使用 `setAttribute()` 方法来修改它，但不推荐这样做，因为在调用 `setAttribute` 之前发生的连接错误将不会遵循此模式。

`$pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);`

## 总结

| 模式                  | 描述                                     | 优点                       | 缺点                                       | 推荐度 |
|:----------------------|:-----------------------------------------|:---------------------------|:-------------------------------------------|:-------|
| `PDO::ERRMODE_SILENT` | 静默，只设置内部错误码                   | -                          | 极易忽略错误，导致“静默失败”               | 不推荐 |
| `PDO::ERRMODE_WARNING`| 产生 `E_WARNING` 警告                    | 至少会留下日志             | 不中断脚本执行，可能导致后续逻辑错误       | 不推荐 |
| `PDO::ERRMODE_EXCEPTION`| 抛出 `PDOException` 异常               | 中断流程，可用 `try/catch` | 需要编写 `try/catch` 块（但这本身是优点）  | **强烈推荐** |

为了构建可靠、专业的 PHP 应用程序，请**始终**将 PDO 的错误处理模式设置为 `PDO::ERRMODE_EXCEPTION`。

---

## 练习任务

1.  **体验不同模式**：
    修改你的数据库连接脚本。
    -   首先，不设置任何 `ATTR_ERRMODE`，体验默认的 `SILENT` 模式。尝试执行一个错误的 SQL 查询，看看会发生什么。
    -   然后，使用 `$pdo->setAttribute()` 将其改为 `WARNING` 模式，再执行一次，观察警告信息。
    -   最后，改回 `EXCEPTION` 模式，并用 `try...catch` 捕获它。

2.  **自定义异常处理**：
    创建一个函数 `function executeQuery($pdo, $sql)`。在这个函数内部，使用 `try...catch` 来执行传入的 `$sql`。如果捕获到 `PDOException`，函数不应该直接输出错误，而是应该将错误信息记录到一个名为 `db_errors.log` 的文本文件中，然后向上层调用者 `throw` 一个新的、更通用的 `Exception`，消息为“数据库查询失败，请联系管理员”。

3.  **分析错误码**：
    在 `EXCEPTION` 模式的 `catch` 块中，通过 `$e->getCode()` 获取 SQLSTATE 错误码。查阅 MySQL 的官方文档，找出当试图向一个有唯一约束的列插入重复值时，返回的 SQLSTATE 码是什么（提示：通常是 `23000`）。编写一个脚本来验证这一点。
