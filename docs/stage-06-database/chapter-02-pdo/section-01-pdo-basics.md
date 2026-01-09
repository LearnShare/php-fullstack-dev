# 6.2.1 PDO 基础与连接管理

## 概述

**PDO (PHP Data Objects)** 是 PHP 中用于访问数据库的一个现代、对象化的标准接口。它并非一个数据库抽象库，而是一个**数据访问抽象层**。这意味着 PDO 提供了一套统一的 API，让开发者可以使用同样的方法和函数来与多种不同的数据库（如 MySQL, PostgreSQL, SQLite 等）进行交互，而无需修改大量的业务逻辑代码。

本节将介绍 PDO 的核心优势，并详细讲解如何配置和建立一个安全的数据库连接。这是后续所有数据库操作的基础。

## PDO 的核心优势

相比于旧的 `mysql_*` 函数（已在 PHP 7 中被废弃）和 `mysqli` 扩展，PDO 提供了几个关键优势：

1.  **数据库无关性 (Database Agnostic)**
    -   PDO 的 API 是统一的。如果你需要将项目从 MySQL 迁移到 PostgreSQL，理论上你只需要修改数据库连接字符串（DSN），而大部分的数据操作代码（如查询、绑定参数）都可以保持不变。

2.  **更高的安全性**
    -   PDO 的核心特性是**预处理语句 (Prepared Statements)**，它将 SQL 指令和用户数据分离发送到数据库服务器。这从根本上杜绝了 **SQL 注入**的风险，是现代数据库编程的安全基石。

3.  **一致的接口**
    -   PDO 完全是面向对象的，提供了清晰、一致的类和方法（如 `PDO`, `PDOStatement`），使代码更易于编写、阅读和维护。

4.  **更强的错误处理**
    -   PDO 支持三种错误处理模式，特别是 `PDO::ERRMODE_EXCEPTION` 模式，它允许你使用现代 PHP 的 `try...catch` 块来捕获和处理数据库错误，使错误处理逻辑更清晰、更健壮。

## 启用 PDO 扩展

在使用 PDO 之前，你需要确保 PHP 的 PDO 扩展和对应的数据库驱动已经启用。对于 MySQL，你需要 `pdo` 和 `pdo_mysql`。

你可以通过 `phpinfo()` 或在命令行运行 `php -m` 来检查。如果未启用，需要在你的 `php.ini` 文件中，取消以下两行的注释（去掉行首的分号 `;`）：

```ini
extension=pdo
extension=pdo_mysql
```
修改后，需要重启你的 Web 服务器或 PHP-FPM 服务。

## 建立数据库连接

### 1. DSN (Data Source Name)

DSN 是一个格式化的字符串，用于定义连接数据库所需的信息。它告诉 PDO 要使用哪个驱动，以及连接到哪个主机和数据库。

**MySQL DSN 的格式**：
`mysql:host=HOSTNAME;port=PORT;dbname=DATABASE_NAME;charset=CHARSET`

-   `mysql:`: 指定使用 MySQL 驱动。
-   `host`: 数据库服务器地址（如 `localhost` 或 `127.0.0.1`）。
-   `port`: 数据库服务器端口（MySQL 默认为 `3306`，如果使用默认端口，此项可省略）。
-   `dbname`: 要连接的数据库名称。
-   `charset`: **强烈推荐设置**。用于指定客户端与服务器通信时使用的字符集，设置为 `utf8mb4` 可以支持包括 Emoji 在内的所有 Unicode 字符。

**示例 DSN**:
`"mysql:host=localhost;port=3306;dbname=my_blog;charset=utf8mb4"`

### 2. PDO 构造函数

通过实例化 `PDO` 类来创建一个数据库连接：
`new PDO(string $dsn, ?string $username, ?string $password, ?array $options = null)`

-   `$dsn`: 数据源名称字符串。
-   `$username`: 数据库用户名。
-   `$password`: 数据库密码。
-   `$options`: 一个可选的、包含驱动特定连接选项的键值对数组。**这是进行安全和高级配置的关键**。

### 3. 配置高安全和便利性的连接选项

通过 `$options` 数组，我们可以配置 PDO 的行为，使其更安全、更易用。以下是推荐的最佳实践配置：

```php
<?php
declare(strict_types=1);

$options = [
    // 1. 错误处理模式：抛出异常
    // 这是最推荐的模式，当发生数据库错误时，PDO会抛出一个 PDOException 异常。
    PDO::ATTR_ERRMODE            => PDO::ERRMODE_EXCEPTION,

    // 2. 默认提取模式：关联数组
    // 设置 fetchAll() 和 fetch() 的默认返回格式为键值对数组，更直观。
    PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,

    // 3. 关闭模拟预处理
    // 强制 PDO 使用真正的预处理语句。这是提升安全性的重要一步，
    // 可以防止某些边缘情况下的 SQL 注入。
    PDO::ATTR_EMULATE_PREPARES   => false,
];
```

**对 `PDO::ATTR_EMULATE_PREPARES => false` 的解释**：
-   当为 `true` (默认值) 时，PDO 客户端会“模拟”预处理。它在 PHP 端将用户数据插入到 SQL 字符串中，然后再发送给数据库，这在某些特定情况下可能存在安全风险。
-   当为 `false` 时，PDO 会将 SQL 模板和用户数据分两次发送给数据库服务器，由服务器进行参数绑定。这是**真正的预处理**，安全性最高。

### 4. 完整的连接示例

将以上所有部分组合起来，一个健壮的数据库连接脚本如下所示。将敏感信息（如密码）存储在环境变量或配置文件中是更好的实践。

**db_connection.php**
```php
<?php
declare(strict_types=1);

// 数据库连接参数
$host = 'localhost';
$port = 3306;
$db   = 'my_blog';
$user = 'root';
$pass = 'your_password'; // 在实际项目中，应使用更安全的方式管理密码
$charset = 'utf8mb4';

// PDO 连接选项
$options = [
    PDO::ATTR_ERRMODE            => PDO::ERRMODE_EXCEPTION,
    PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
    PDO::ATTR_EMULATE_PREPARES   => false,
];

// 构建 DSN
$dsn = "mysql:host=$host;port=$port;dbname=$db;charset=$charset";

// 尝试建立连接
try {
    $pdo = new PDO($dsn, $user, $pass, $options);
    echo "数据库连接成功！";
} catch (\PDOException $e) {
    // 如果连接失败，抛出异常并终止脚本
    // 在生产环境中，应记录错误日志而不是直接输出错误信息
    throw new \PDOException($e->getMessage(), (int)$e->getCode());
}

// 在其他文件中可以通过 include 'db_connection.php'; 来获取 $pdo 对象
// return $pdo; // 在函数或类的方法中，可以返回这个连接对象
```

## 关闭连接

在 PHP 中，数据库连接通常无需手动关闭。当脚本执行结束时，所有对象（包括 PDO 对象）都会被销毁，其拥有的数据库连接也会被自动释放。

如果你在一个长时运行的脚本中，确实需要在某个点显式地关闭连接以释放资源，可以将 PDO 对象设置为 `null`：

```php
$pdo = null;
```
这会触发 PDO 对象的析构函数，从而关闭数据库连接。但在典型的 Web 请求-响应生命周期中，这完全没有必要。

---

## 练习任务

1.  **检查环境**：
    创建一个 PHP 文件，使用 `phpinfo()` 函数检查你的 PHP环境中 `pdo` 和 `pdo_mysql` 扩展是否已经启用。

2.  **创建连接脚本**：
    仿照本节的“完整连接示例”，创建一个名为 `config.php` 的文件，在其中定义你的本地数据库连接参数。然后创建另一个文件 `connect.php`，引入 `config.php`，并尝试建立一个 PDO 连接。如果连接成功，输出“连接成功”；如果失败，捕获异常并输出错误信息。

3.  **探索 DSN 和 Options**：
    -   尝试在 DSN 中故意写错数据库名称 (`dbname`)，看看 `try...catch` 块是否能捕获到预期的 `PDOException` 异常。
    -   查阅 PHP 官方文档，了解 `PDO::ATTR_CASE` 这个连接选项的作用是什么，并尝试在你的连接选项中设置它，看看对查询结果有什么影响（例如，查询 `SELECT userName FROM users;` 时，返回的数组键是 `userName` 还是 `USERNAME`）。