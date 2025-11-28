# 5.2 PDO 入门与高安全模式

## 目标

- 掌握 PDO（PHP Data Objects）的基础用法。
- 理解 DSN 格式与连接选项配置。
- 深入理解预处理语句防注入的原理。
- 掌握 `bindParam` 与 `bindValue` 的区别。
- 能够进行安全的 CRUD 操作。

## PDO 基础

### 什么是 PDO

- **PDO**：PHP Data Objects，PHP 数据对象。
- 统一的数据库访问接口，支持多种数据库。
- 提供预处理语句，有效防止 SQL 注入。

### DSN 格式

**语法**：`driver:host=hostname;dbname=database;charset=utf8mb4`

```php
<?php
declare(strict_types=1);

// MySQL DSN
$dsn = 'mysql:host=localhost;dbname=myapp;charset=utf8mb4';

// 连接选项
$options = [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
    PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
    PDO::ATTR_EMULATE_PREPARES => false,  // 禁用预处理模拟
];

// 创建连接
try {
    $pdo = new PDO($dsn, 'username', 'password', $options);
} catch (PDOException $e) {
    die('Connection failed: ' . $e->getMessage());
}
```

### 连接选项详解

| 选项                          | 说明                           | 推荐值                    |
| :---------------------------- | :----------------------------- | :------------------------ |
| `PDO::ATTR_ERRMODE`           | 错误处理模式                   | `PDO::ERRMODE_EXCEPTION`  |
| `PDO::ATTR_DEFAULT_FETCH_MODE` | 默认获取模式                   | `PDO::FETCH_ASSOC`        |
| `PDO::ATTR_EMULATE_PREPARES`  | 是否模拟预处理                 | `false`（使用原生预处理） |
| `PDO::ATTR_PERSISTENT`        | 持久连接                       | `false`（生产环境谨慎使用） |
| `PDO::ATTR_TIMEOUT`           | 连接超时（秒）                 | `5`                       |

```php
<?php
declare(strict_types=1);

$options = [
    // 错误模式：抛出异常
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
    
    // 默认获取模式：关联数组
    PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
    
    // 禁用预处理模拟（使用数据库原生预处理）
    PDO::ATTR_EMULATE_PREPARES => false,
    
    // 连接超时
    PDO::ATTR_TIMEOUT => 5,
];

$pdo = new PDO($dsn, $user, $pass, $options);
```

## SQL 注入原理

### 什么是 SQL 注入

- **SQL 注入**：通过构造恶意 SQL 语句，绕过应用程序的安全检查。
- 攻击者可以读取、修改、删除数据库数据。

### 危险示例

```php
<?php
// 危险：直接拼接用户输入
$username = $_GET['username'];
$sql = "SELECT * FROM users WHERE username = '{$username}'";
$result = $pdo->query($sql);

// 攻击示例：
// username = "admin' OR '1'='1"
// SQL: SELECT * FROM users WHERE username = 'admin' OR '1'='1'
// 结果：返回所有用户
```

### 防护方法

**使用预处理语句**：将 SQL 语句与数据分离。

```php
<?php
declare(strict_types=1);

// 安全：使用预处理语句
$username = $_GET['username'];
$stmt = $pdo->prepare("SELECT * FROM users WHERE username = ?");
$stmt->execute([$username]);
$user = $stmt->fetch();
```

## 预处理语句

### prepare 方法

- **语法**：`prepare(string $statement, array $options = []): PDOStatement|false`
- **参数说明**：
  - `$statement`：要准备的 SQL 语句，可以使用 `?` 或命名占位符 `:name`。
  - `$options`：可选参数数组，如 `[PDO::ATTR_CURSOR => PDO::CURSOR_FWDONLY]`。
- **返回值**：成功返回 `PDOStatement` 对象，失败返回 `false`。
- **异常**：可能抛出 `PDOException`。
- 准备 SQL 语句，使用占位符代替实际值。

```php
<?php
declare(strict_types=1);

// 使用 ? 占位符
$stmt = $pdo->prepare("SELECT * FROM users WHERE id = ? AND status = ?");

// 使用命名占位符
$stmt = $pdo->prepare("SELECT * FROM users WHERE id = :id AND status = :status");
```

### execute 方法

- **语法**：`execute(?array $params = null): bool`
- **参数说明**：
  - `$params`：可选参数数组，如果占位符是 `?`，使用索引数组；如果是命名占位符，使用关联数组。
- **返回值**：成功返回 `true`，失败返回 `false`。
- **异常**：可能抛出 `PDOException`。
- 执行预处理语句，传入参数数组。

```php
<?php
declare(strict_types=1);

// 使用 ? 占位符
$stmt = $pdo->prepare("SELECT * FROM users WHERE id = ? AND email = ?");
$stmt->execute([1, 'alice@example.com']);

// 使用命名占位符
$stmt = $pdo->prepare("SELECT * FROM users WHERE id = :id AND email = :email");
$stmt->execute([
    'id' => 1,
    'email' => 'alice@example.com',
]);
```

### bindParam vs bindValue

#### bindParam

- **按引用绑定**：绑定变量引用，执行时使用变量的当前值。
- 可以多次执行，每次使用变量的最新值。

```php
<?php
declare(strict_types=1);

$stmt = $pdo->prepare("SELECT * FROM users WHERE id = :id");

$id = 1;
$stmt->bindParam(':id', $id, PDO::PARAM_INT);

$stmt->execute();  // 查询 id = 1

$id = 2;
$stmt->execute();  // 查询 id = 2（使用变量的新值）
```

#### bindValue

- **按值绑定**：绑定变量的值，执行时使用绑定时的值。
- 即使变量后续改变，也不会影响已绑定的值。

```php
<?php
declare(strict_types=1);

$stmt = $pdo->prepare("SELECT * FROM users WHERE id = :id");

$id = 1;
$stmt->bindValue(':id', $id, PDO::PARAM_INT);

$id = 2;  // 改变变量值

$stmt->execute();  // 仍然查询 id = 1（使用绑定时的值）
```

#### 对比总结

| 特性       | bindParam              | bindValue              |
| :--------- | :--------------------- | :--------------------- |
| 绑定方式   | 按引用                 | 按值                   |
| 变量变化   | 影响执行结果           | 不影响执行结果         |
| 适用场景   | 循环执行，变量会变化   | 单次执行，值固定       |
| 性能       | 稍好（避免值复制）     | 稍差（需要值复制）     |

## CRUD 操作

### Create（创建）

```php
<?php
declare(strict_types=1);

// 插入单条记录
$stmt = $pdo->prepare("INSERT INTO users (username, email, password_hash) VALUES (?, ?, ?)");
$stmt->execute(['alice', 'alice@example.com', password_hash('password', PASSWORD_DEFAULT)]);

$userId = $pdo->lastInsertId();
echo "New user ID: {$userId}\n";

// 插入多条记录（批量插入）
$users = [
    ['bob', 'bob@example.com', password_hash('password', PASSWORD_DEFAULT)],
    ['charlie', 'charlie@example.com', password_hash('password', PASSWORD_DEFAULT)],
];

$stmt = $pdo->prepare("INSERT INTO users (username, email, password_hash) VALUES (?, ?, ?)");
foreach ($users as $user) {
    $stmt->execute($user);
}
```

### Read（读取）

```php
<?php
declare(strict_types=1);

// 查询单条记录
$stmt = $pdo->prepare("SELECT * FROM users WHERE id = ?");
$stmt->execute([1]);
$user = $stmt->fetch();

if ($user === false) {
    echo "User not found\n";
} else {
    echo "User: {$user['username']}\n";
}

// 查询多条记录
$stmt = $pdo->prepare("SELECT * FROM users WHERE status = ? ORDER BY created_at DESC");
$stmt->execute(['active']);
$users = $stmt->fetchAll();

foreach ($users as $user) {
    echo "User: {$user['username']}\n";
}

// 查询单列
$stmt = $pdo->prepare("SELECT username FROM users");
$stmt->execute();
$usernames = $stmt->fetchAll(PDO::FETCH_COLUMN);

// 查询单个值
$stmt = $pdo->prepare("SELECT COUNT(*) FROM users WHERE status = ?");
$stmt->execute(['active']);
$count = $stmt->fetchColumn();
```

### Update（更新）

```php
<?php
declare(strict_types=1);

// 更新记录
$stmt = $pdo->prepare("UPDATE users SET email = ?, updated_at = NOW() WHERE id = ?");
$stmt->execute(['newemail@example.com', 1]);

$affectedRows = $stmt->rowCount();
echo "Updated {$affectedRows} row(s)\n";

// 更新多个字段
$stmt = $pdo->prepare("
    UPDATE users 
    SET username = ?, email = ?, updated_at = NOW() 
    WHERE id = ?
");
$stmt->execute(['newname', 'newemail@example.com', 1]);
```

### Delete（删除）

```php
<?php
declare(strict_types=1);

// 删除记录
$stmt = $pdo->prepare("DELETE FROM users WHERE id = ?");
$stmt->execute([1]);

$affectedRows = $stmt->rowCount();
echo "Deleted {$affectedRows} row(s)\n";

// 软删除（推荐）
$stmt = $pdo->prepare("UPDATE users SET deleted_at = NOW() WHERE id = ?");
$stmt->execute([1]);
```

## 获取模式

### 常用获取模式

| 模式                | 说明                     | 示例                           |
| :------------------ | :----------------------- | :----------------------------- |
| `PDO::FETCH_ASSOC`  | 关联数组                 | `['id' => 1, 'name' => 'Alice']` |
| `PDO::FETCH_NUM`    | 数字索引数组             | `[0 => 1, 1 => 'Alice']`      |
| `PDO::FETCH_BOTH`   | 同时返回关联和数字索引   | 两者结合                       |
| `PDO::FETCH_OBJ`    | 对象                     | `$user->id, $user->name`       |
| `PDO::FETCH_CLASS`  | 映射到类                 | `new User($data)`              |
| `PDO::FETCH_COLUMN` | 单列                     | `['Alice', 'Bob']`             |

```php
<?php
declare(strict_types=1);

// FETCH_ASSOC（默认，推荐）
$stmt = $pdo->prepare("SELECT * FROM users WHERE id = ?");
$stmt->execute([1]);
$user = $stmt->fetch(PDO::FETCH_ASSOC);
echo $user['username'];

// FETCH_OBJ
$stmt = $pdo->prepare("SELECT * FROM users WHERE id = ?");
$stmt->execute([1]);
$user = $stmt->fetch(PDO::FETCH_OBJ);
echo $user->username;

// FETCH_CLASS
class User
{
    public int $id;
    public string $username;
    public string $email;
}

$stmt = $pdo->prepare("SELECT * FROM users WHERE id = ?");
$stmt->setFetchMode(PDO::FETCH_CLASS, User::class);
$stmt->execute([1]);
$user = $stmt->fetch();
echo $user->username;

// FETCH_COLUMN
$stmt = $pdo->prepare("SELECT username FROM users");
$stmt->execute();
$usernames = $stmt->fetchAll(PDO::FETCH_COLUMN);
```

## 错误处理

### 错误模式

```php
<?php
declare(strict_types=1);

// ERRMODE_SILENT（默认，不推荐）
$pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_SILENT);
$stmt = $pdo->prepare("SELECT * FROM invalid_table");
$stmt->execute();
if ($stmt->errorCode() !== '00000') {
    $error = $stmt->errorInfo();
    echo "Error: {$error[2]}\n";
}

// ERRMODE_WARNING（不推荐）
$pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_WARNING);
$stmt = $pdo->prepare("SELECT * FROM invalid_table");
$stmt->execute();  // 产生警告

// ERRMODE_EXCEPTION（推荐）
$pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
try {
    $stmt = $pdo->prepare("SELECT * FROM invalid_table");
    $stmt->execute();
} catch (PDOException $e) {
    echo "Error: {$e->getMessage()}\n";
}
```

## 完整示例

### 用户管理类

```php
<?php
declare(strict_types=1);

class UserRepository
{
    private PDO $pdo;

    public function __construct(PDO $pdo)
    {
        $this->pdo = $pdo;
    }

    public function create(string $username, string $email, string $password): int
    {
        $stmt = $this->pdo->prepare(
            "INSERT INTO users (username, email, password_hash) VALUES (?, ?, ?)"
        );
        $stmt->execute([
            $username,
            $email,
            password_hash($password, PASSWORD_DEFAULT),
        ]);
        
        return (int) $this->pdo->lastInsertId();
    }

    public function findById(int $id): ?array
    {
        $stmt = $this->pdo->prepare("SELECT * FROM users WHERE id = ?");
        $stmt->execute([$id]);
        $user = $stmt->fetch();
        
        return $user !== false ? $user : null;
    }

    public function findByEmail(string $email): ?array
    {
        $stmt = $this->pdo->prepare("SELECT * FROM users WHERE email = ?");
        $stmt->execute([$email]);
        $user = $stmt->fetch();
        
        return $user !== false ? $user : null;
    }

    public function update(int $id, array $data): bool
    {
        $fields = [];
        $values = [];
        
        foreach ($data as $key => $value) {
            $fields[] = "{$key} = ?";
            $values[] = $value;
        }
        
        $values[] = $id;
        $sql = "UPDATE users SET " . implode(', ', $fields) . " WHERE id = ?";
        
        $stmt = $this->pdo->prepare($sql);
        return $stmt->execute($values);
    }

    public function delete(int $id): bool
    {
        $stmt = $this->pdo->prepare("UPDATE users SET deleted_at = NOW() WHERE id = ?");
        return $stmt->execute([$id]);
    }

    public function findAll(int $limit = 10, int $offset = 0): array
    {
        $stmt = $this->pdo->prepare(
            "SELECT * FROM users WHERE deleted_at IS NULL ORDER BY created_at DESC LIMIT ? OFFSET ?"
        );
        $stmt->execute([$limit, $offset]);
        return $stmt->fetchAll();
    }
}

// 使用
$pdo = new PDO($dsn, $user, $pass, $options);
$userRepo = new UserRepository($pdo);

$userId = $userRepo->create('alice', 'alice@example.com', 'password123');
$user = $userRepo->findById($userId);
```

## 最佳实践

### 1. 始终使用预处理语句

```php
// 不推荐
$sql = "SELECT * FROM users WHERE id = {$id}";

// 推荐
$stmt = $pdo->prepare("SELECT * FROM users WHERE id = ?");
$stmt->execute([$id]);
```

### 2. 禁用预处理模拟

```php
$options = [
    PDO::ATTR_EMULATE_PREPARES => false,  // 使用数据库原生预处理
];
```

### 3. 使用异常处理

```php
$pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
```

### 4. 合理使用事务

```php
$pdo->beginTransaction();
try {
    // 多个操作
    $pdo->commit();
} catch (Exception $e) {
    $pdo->rollBack();
    throw $e;
}
```

## 练习

1. 创建一个 PDO 连接类，封装连接配置和错误处理。

2. 实现一个安全的用户认证函数，使用预处理语句验证用户名和密码。

3. 编写一个通用的数据库操作类，提供 find、create、update、delete 方法。

4. 实现一个分页查询函数，使用预处理语句和 LIMIT/OFFSET。

5. 创建一个批量插入函数，使用事务确保数据一致性。

6. 设计一个查询构建器，支持链式调用和安全的参数绑定。
