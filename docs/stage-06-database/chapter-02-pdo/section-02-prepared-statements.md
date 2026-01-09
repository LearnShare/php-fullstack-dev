# 6.2.2 预处理语句与防注入 (Prepared Statements & Injection Prevention)

## 概述

在所有数据库安全问题中，**SQL 注入 (SQL Injection)** 是最常见、危害最大的漏洞之一。如果应用程序直接将用户输入拼接到 SQL 查询字符串中，攻击者就可以通过构造恶意的输入来篡改原始的 SQL 逻辑，从而窃取数据、绕过验证，甚至破坏整个数据库。

**预处理语句 (Prepared Statements)** 是 PDO 提供的一种强大的机制，它从根本上杜绝了 SQL 注入的风险。本节将首先展示 SQL 注入的危害，然后深入讲解预处理语句的工作原理和使用方法。

**核心原则：永远不要信任用户输入，始终使用预处理语句处理动态数据。**

---

## SQL 注入的危害：一个具体的例子

假设你有一个简单的登录脚本，它根据用户提交的 `username` 和 `password` 来查询数据库。

**一个极其危险的、错误的代码示例（千万不要在实际项目中使用）：**
```php
<?php
// ... 包含 $pdo 连接对象

// 直接从 POST 获取用户输入
$username = $_POST['username'];
$password = $_POST['password'];

// 灾难性的代码：直接将用户输入拼接到 SQL 语句中
$sql = "SELECT * FROM users WHERE username = '$username' AND password = '$password'";

echo "执行的 SQL: " . $sql; // 仅用于演示

// 执行查询
$stmt = $pdo->query($sql);
$user = $stmt->fetch();

if ($user) {
    echo "\n登录成功! 欢迎, " . $user['username'];
} else {
    echo "\n登录失败!";
}
```

### 正常登录

如果一个普通用户输入 `admin` 和 `password123`，那么拼接出的 SQL 是：
`SELECT * FROM users WHERE username = 'admin' AND password = 'password123'`
这看起来没有问题。

### 攻击者如何利用漏洞

现在，一个攻击者在用户名字段输入 `admin' -- ` (注意最后的空格)，密码字段随便输入。
拼接出的 SQL 将变成：
`SELECT * FROM users WHERE username = 'admin' -- ' AND password = 'whatever'`

在 SQL 中，`-- ` (两个短横线加一个空格) 是行注释符。这意味着 `AND password = 'whatever'` 这部分被数据库忽略了！实际执行的 SQL 变成了：
`SELECT * FROM users WHERE username = 'admin'`

这条 SQL 会选中用户名为 `admin` 的用户并直接返回其信息，攻击者无需密码就成功登录了 `admin` 账户。这只是 SQL 注入攻击的冰山一角。

---

## 解决方案：预处理语句的工作原理

预处理语句将 SQL 查询的**结构（模板）**和**数据**分离开来处理，遵循“先准备，后执行”的两步流程。

1.  **准备 (Prepare)**
    -   **做什么**：应用程序将一个包含**占位符 (Placeholders)** 的 SQL 模板发送给数据库服务器。
        -   `SELECT * FROM users WHERE username = :username AND password = :password`
    -   **服务器响应**：数据库服务器接收到这个模板后，对其进行解析、编译和查询优化，然后生成一个执行计划，并存储起来等待数据。

2.  **执行 (Execute)**
    -   **做什么**：应用程序将用户的实际输入（数据）作为独立的参数发送给数据库服务器。
        -   `['username' => "admin' -- ", 'password' => 'whatever']`
    -   **服务器响应**：服务器接收到数据后，将这些数据应用到之前已经准备好的执行计划中来完成查询。关键在于，此时服务器**只会将这些数据作为纯文本数据来处理**，而不会将其作为 SQL 命令的一部分来解析。

**为什么能防注入？**
因为攻击者输入的恶意字符串（如 `admin' -- `）在第二步才被发送给数据库。数据库此时已经完成了 SQL 语法的解析，它“知道”这个字符串只是 `username` 字段要匹配的值，而不是 SQL 命令的一部分。因此，它只会去数据库里忠实地查找一个用户名正好是 `admin' -- ` 的用户，而不会执行注释逻辑。

---

## 如何使用预处理语句

PDO 中使用预处理语句主要涉及三个方法：
-   `PDO::prepare()`: 创建一个 `PDOStatement` 对象，准备要执行的 SQL 模板。
-   `PDOStatement::execute()`: 执行预处理语句。
-   `PDOStatement::bindValue()` 或 `PDOStatement::bindParam()`: (可选) 将一个值或变量绑定到 SQL 模板中的占位符。

占位符主要有两种风格：**命名占位符**和**位置占位符**。

### 1. 命名占位符 (Named Placeholders)

使用一个冒号 `:` 后跟一个名称（如 `:id`, `:email`）作为占位符。这是**推荐**的方式，因为它更具可读性，且不关心参数的顺序。

**完整示例**：
```php
<?php
// ... $pdo 连接对象

$username = "admin";
$email = "admin@example.com";

// 1. 准备 (Prepare)
$sql = "SELECT * FROM users WHERE username = :user AND email = :email";
$stmt = $pdo->prepare($sql);

// 2. 执行 (Execute)，将一个关联数组传递给 execute()
// 数组的键必须与占位符名称匹配（包括冒号）
$stmt->execute([
    ':user' => $username,
    ':email' => $email,
]);

// 3. 获取结果
$user = $stmt->fetch();
```

你也可以使用 `bindValue()` 方法来分别绑定值：
```php
// ...
$stmt = $pdo->prepare($sql);

// 2. 绑定值 (Bind)
$stmt->bindValue(':user', $username);
$stmt->bindValue(':email', $email);

// 3. 执行 (Execute)
$stmt->execute();

// 4. 获取结果
// ...
```
**注意**：向 `execute()` 传递数组的方式更简洁，是目前最主流的用法。

### 2. 位置占位符 (Positional Placeholders)

使用一个问号 `?` 作为占位符。参数的绑定顺序必须与 `?` 出现的顺序严格一致。

**完整示例**：
```php
<?php
// ... $pdo 连接对象

$username = "admin";
$email = "admin@example.com";

// 1. 准备 (Prepare)
$sql = "SELECT * FROM users WHERE username = ? AND email = ?";
$stmt = $pdo->prepare($sql);

// 2. 执行 (Execute)，将一个索引数组传递给 execute()
// 数组元素的顺序必须与 `?` 的顺序一致
$stmt->execute([$username, $email]);

// 3. 获取结果
$user = $stmt->fetch();
```
同样，也可以使用 `bindValue()`，但需要指定参数的位置（从 1 开始）：
`$stmt->bindValue(1, $username);`
`$stmt->bindValue(2, $email);`

### `bindValue()` vs. `bindParam()`

PDO 提供了两个相似的绑定方法，但它们有本质区别：

-   `PDOStatement::bindValue($placeholder, $value, $dataType = PDO::PARAM_STR)`
    -   **绑定值**：它绑定的是变量在**调用 `bindValue` 时**的**值**。
-   `PDOStatement::bindParam($placeholder, &$variable, $dataType = PDO::PARAM_STR)`
    -   **绑定变量**：它通过**引用**的方式绑定一个 PHP **变量**。该变量的值在**调用 `execute()` 时**才会被读取。

**示例**：
```php
$name = 'Alice';
$stmt = $pdo->prepare("SELECT * FROM users WHERE name = :name");

// 使用 bindValue
$stmt->bindValue(':name', $name);
$name = 'Bob'; // 在 execute() 之前修改变量
$stmt->execute(); // 数据库会查找 'Alice'，因为绑定的是当时的值

// 使用 bindParam
$stmt->bindParam(':name', $name);
$name = 'Carol'; // 在 execute() 之前修改变量
$stmt->execute(); // 数据库会查找 'Carol'，因为在执行时才读取变量的值
```
**建议**：在绝大多数情况下，行为更可预测的 `bindValue()` 或直接向 `execute()` 传递数组是更简单、更安全的选择。仅在你需要利用 `bindParam` 的引用特性（例如在循环中多次执行同一语句）时才使用它。

---

## 练习任务

1.  **修复漏洞**：
    将本节开头的那个有 SQL 注入漏洞的登录脚本，使用**命名占位符**的预处理语句进行重写，使其变得安全。

2.  **实现用户注册**：
    创建一个 `register.php` 脚本。接收 `username`, `email`, `password` 三个 POST 参数。使用**位置占位符 (`?`)** 的预处理语句，将新用户信息插入到 `users` 表中。

3.  **理解 `bindValue` 与 `bindParam`**：
    编写一个脚本，使用 `bindParam` 在一个 `foreach` 循环中向数据库批量插入多条数据。思考一下，如果在这里用 `bindValue` 会发生什么？（提示：思考循环变量的值在何时被评估）。