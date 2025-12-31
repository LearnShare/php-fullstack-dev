# 6.2.2 预处理语句与防注入

## 概述

预处理语句（Prepared Statements）是防止 SQL 注入攻击的关键技术，也是 PDO 最重要的安全特性。预处理语句将 SQL 语句和参数分离，通过参数绑定传递数据，确保用户输入被正确处理，防止恶意 SQL 代码被执行。

本节详细介绍预处理语句的概念、使用方法、参数绑定方式、防注入原理等，帮助零基础学员掌握安全的数据库操作方法。理解预处理语句的工作原理对于编写安全的 PHP 应用至关重要。

**主要内容**：
- 预处理语句的概念和优势
- prepare() 方法的使用
- 参数绑定（命名参数、位置参数）
- execute() 方法执行
- SQL 注入攻击原理和防护
- 完整示例和最佳实践

---

## 特性

- **安全性**：有效防止 SQL 注入攻击
- **性能优化**：SQL 语句只需编译一次，可重复执行
- **类型安全**：支持参数类型绑定
- **代码清晰**：SQL 语句和参数分离，代码更易读
- **数据库支持**：所有主流数据库都支持预处理语句

---

## 预处理语句概念

### 什么是预处理语句

预处理语句是数据库服务器预先编译的 SQL 语句模板，其中包含占位符（如 `?` 或 `:name`），在执行时通过参数绑定传入实际值。

**工作流程**：

1. **准备阶段**：SQL 语句被发送到数据库服务器编译
2. **绑定阶段**：参数值绑定到占位符
3. **执行阶段**：数据库服务器执行已编译的语句

**示例**：

```php
<?php
declare(strict_types=1);

// 1. 准备阶段
$stmt = $pdo->prepare('SELECT * FROM users WHERE email = :email');

// 2. 绑定和执行阶段
$stmt->execute(['email' => 'user@example.com']);

// 3. 获取结果
$user = $stmt->fetch();
```

### 预处理语句的优势

**1. 安全性**

预处理语句将 SQL 语句和参数分离，参数值被转义处理，防止 SQL 注入攻击。

**示例**：

```php
<?php
declare(strict_types=1);

// 不安全的方式（容易受到 SQL 注入攻击）
$email = $_GET['email'];
$sql = "SELECT * FROM users WHERE email = '$email'";
$stmt = $pdo->query($sql);  // 危险！

// 安全的方式（使用预处理语句）
$stmt = $pdo->prepare('SELECT * FROM users WHERE email = :email');
$stmt->execute(['email' => $email]);  // 安全
```

**2. 性能优化**

SQL 语句只需编译一次，可以重复执行，提高性能。

**示例**：

```php
<?php
declare(strict_types=1);

// 准备一次，执行多次
$stmt = $pdo->prepare('INSERT INTO users (name, email) VALUES (:name, :email)');

foreach ($users as $user) {
    $stmt->execute([
        'name' => $user['name'],
        'email' => $user['email']
    ]);
}
```

**3. 类型安全**

预处理语句支持参数类型绑定，确保数据类型正确。

**示例**：

```php
<?php
declare(strict_types=1);

$stmt = $pdo->prepare('SELECT * FROM users WHERE id = :id');
$stmt->bindValue(':id', $id, PDO::PARAM_INT);  // 绑定为整数类型
$stmt->execute();
```

### 防注入原理

**SQL 注入攻击原理**：

SQL 注入攻击是通过在用户输入中插入恶意 SQL 代码，欺骗数据库执行非预期的 SQL 语句。

**示例攻击**：

```php
<?php
// 危险的代码
$email = "user@example.com' OR '1'='1";
$sql = "SELECT * FROM users WHERE email = '$email'";
// 实际执行的 SQL: SELECT * FROM users WHERE email = 'user@example.com' OR '1'='1'
// 这会返回所有用户！
```

**预处理语句防注入原理**：

1. **编译阶段**：SQL 语句在编译时就已经确定结构，占位符只是参数位置
2. **参数绑定**：参数值在绑定阶段被转义和类型检查
3. **执行阶段**：数据库服务器将参数值插入已编译的 SQL 结构中，不会改变 SQL 结构

**示例**：

```php
<?php
declare(strict_types=1);

// 即使传入恶意代码，也不会执行
$email = "user@example.com' OR '1'='1";
$stmt = $pdo->prepare('SELECT * FROM users WHERE email = :email');
$stmt->execute(['email' => $email]);
// 数据库会查找 email 值为 "user@example.com' OR '1'='1" 的用户
// 不会执行 OR 条件，因为 SQL 结构已经固定
```

---

## prepare() 方法

### 语法

**语法**：`PDO::prepare(string $statement, array $options = []): PDOStatement|false`

**参数**：

- `$statement`：SQL 语句，包含占位符（必需）
- `$options`：选项数组（可选）

**返回值**：`PDOStatement` 对象或 `false`（失败时）

### SQL 语句准备

使用 `prepare()` 方法准备 SQL 语句，SQL 语句可以包含占位符。

**示例**：

```php
<?php
declare(strict_types=1);

// 命名参数占位符
$stmt = $pdo->prepare('SELECT * FROM users WHERE email = :email AND status = :status');

// 位置参数占位符
$stmt = $pdo->prepare('SELECT * FROM users WHERE email = ? AND status = ?');
```

### 占位符使用

PDO 支持两种占位符：

**1. 命名参数（:name）**

使用冒号前缀的命名参数，推荐使用。

**示例**：

```php
<?php
declare(strict_types=1);

$stmt = $pdo->prepare('SELECT * FROM users WHERE email = :email AND age > :age');
$stmt->execute([
    'email' => 'user@example.com',
    'age' => 18
]);
```

**2. 位置参数（?）**

使用问号作为位置参数。

**示例**：

```php
<?php
declare(strict_types=1);

$stmt = $pdo->prepare('SELECT * FROM users WHERE email = ? AND age > ?');
$stmt->execute(['user@example.com', 18]);
```

**选择建议**：

- **命名参数**：推荐使用，代码更清晰，参数顺序不重要
- **位置参数**：参数较少时可以使用，但需要注意参数顺序

### 语句对象

`prepare()` 方法返回 `PDOStatement` 对象，用于执行和获取结果。

**示例**：

```php
<?php
declare(strict_types=1);

$stmt = $pdo->prepare('SELECT * FROM users WHERE id = :id');

// PDOStatement 对象的方法
$stmt->execute(['id' => 1]);
$user = $stmt->fetch();
$users = $stmt->fetchAll();
$rowCount = $stmt->rowCount();
```

---

## 参数绑定

### bindValue() 方法

`bindValue()` 方法将值绑定到占位符，值在绑定时就确定，不会改变。

**语法**：`PDOStatement::bindValue(string|int $param, mixed $value, int $type = PDO::PARAM_STR): bool`

**参数**：

- `$param`：参数名（命名参数）或位置（位置参数）
- `$value`：要绑定的值
- `$type`：参数类型（可选）

**示例**：

```php
<?php
declare(strict_types=1);

$stmt = $pdo->prepare('SELECT * FROM users WHERE email = :email AND age > :age');

// 绑定值
$stmt->bindValue(':email', 'user@example.com', PDO::PARAM_STR);
$stmt->bindValue(':age', 18, PDO::PARAM_INT);

// 执行
$stmt->execute();
```

**特点**：

- 值在绑定时就确定
- 即使变量后续改变，绑定的值不会改变
- 适合绑定常量或立即值

### bindParam() 方法

`bindParam()` 方法将变量绑定到占位符，绑定的是变量引用，执行时使用变量的当前值。

**语法**：`PDOStatement::bindParam(string|int $param, mixed &$var, int $type = PDO::PARAM_STR, ?int $maxLength = null, ?int $driverOptions = null): bool`

**参数**：

- `$param`：参数名（命名参数）或位置（位置参数）
- `$var`：要绑定的变量（引用传递）
- `$type`：参数类型（可选）
- `$maxLength`：最大长度（可选）
- `$driverOptions`：驱动选项（可选）

**示例**：

```php
<?php
declare(strict_types=1);

$stmt = $pdo->prepare('INSERT INTO users (name, email) VALUES (:name, :email)');

$name = 'John';
$email = 'john@example.com';

// 绑定变量引用
$stmt->bindParam(':name', $name, PDO::PARAM_STR);
$stmt->bindParam(':email', $email, PDO::PARAM_STR);

// 执行第一次
$stmt->execute();  // 插入 John

// 改变变量值
$name = 'Jane';
$email = 'jane@example.com';

// 执行第二次（使用新的变量值）
$stmt->execute();  // 插入 Jane
```

**特点**：

- 绑定的是变量引用
- 执行时使用变量的当前值
- 适合在循环中使用

### 命名参数绑定

使用命名参数时，参数名以冒号开头。

**示例**：

```php
<?php
declare(strict_types=1);

// 方法 1：使用 bindValue()
$stmt = $pdo->prepare('SELECT * FROM users WHERE email = :email');
$stmt->bindValue(':email', 'user@example.com');
$stmt->execute();

// 方法 2：在 execute() 中传递数组（推荐）
$stmt = $pdo->prepare('SELECT * FROM users WHERE email = :email');
$stmt->execute(['email' => 'user@example.com']);

// 多个参数
$stmt = $pdo->prepare('SELECT * FROM users WHERE email = :email AND age > :age');
$stmt->execute([
    'email' => 'user@example.com',
    'age' => 18
]);
```

### 位置参数绑定

使用位置参数时，使用问号作为占位符。

**示例**：

```php
<?php
declare(strict_types=1);

// 方法 1：使用 bindValue()
$stmt = $pdo->prepare('SELECT * FROM users WHERE email = ? AND age > ?');
$stmt->bindValue(1, 'user@example.com');
$stmt->bindValue(2, 18);
$stmt->execute();

// 方法 2：在 execute() 中传递数组（推荐）
$stmt = $pdo->prepare('SELECT * FROM users WHERE email = ? AND age > ?');
$stmt->execute(['user@example.com', 18]);
```

**注意**：位置参数从 1 开始编号，不是从 0 开始。

---

## execute() 方法

### 执行预处理语句

`execute()` 方法执行预处理语句，传入参数值。

**语法**：`PDOStatement::execute(?array $params = null): bool`

**参数**：

- `$params`：参数数组（可选，如果使用 bindValue() 或 bindParam() 绑定则不需要）

**返回值**：执行成功返回 `true`，失败返回 `false`

**示例**：

```php
<?php
declare(strict_types=1);

// 使用命名参数
$stmt = $pdo->prepare('SELECT * FROM users WHERE email = :email');
$result = $stmt->execute(['email' => 'user@example.com']);

// 使用位置参数
$stmt = $pdo->prepare('SELECT * FROM users WHERE email = ?');
$result = $stmt->execute(['user@example.com']);

// 已绑定参数，不需要传递
$stmt = $pdo->prepare('SELECT * FROM users WHERE email = :email');
$stmt->bindValue(':email', 'user@example.com');
$result = $stmt->execute();
```

### 参数传递

在 `execute()` 方法中传递参数是最常用的方式。

**示例**：

```php
<?php
declare(strict_types=1);

// 插入数据
$stmt = $pdo->prepare('INSERT INTO users (name, email, age) VALUES (:name, :email, :age)');
$stmt->execute([
    'name' => 'John Doe',
    'email' => 'john@example.com',
    'age' => 30
]);

// 更新数据
$stmt = $pdo->prepare('UPDATE users SET name = :name WHERE id = :id');
$stmt->execute([
    'id' => 1,
    'name' => 'Jane Doe'
]);

// 删除数据
$stmt = $pdo->prepare('DELETE FROM users WHERE id = :id');
$stmt->execute(['id' => 1]);
```

### 返回值

`execute()` 方法返回布尔值，表示执行是否成功。

**示例**：

```php
<?php
declare(strict_types=1);

$stmt = $pdo->prepare('INSERT INTO users (name, email) VALUES (:name, :email)');

if ($stmt->execute([
    'name' => 'John Doe',
    'email' => 'john@example.com'
])) {
    echo '插入成功';
} else {
    echo '插入失败';
}
```

**注意**：在异常模式下，执行失败会抛出异常，不会返回 `false`。

### 错误处理

在异常模式下，执行失败会抛出 `PDOException` 异常。

**示例**：

```php
<?php
declare(strict_types=1);

try {
    $stmt = $pdo->prepare('INSERT INTO users (name, email) VALUES (:name, :email)');
    $stmt->execute([
        'name' => 'John Doe',
        'email' => 'john@example.com'
    ]);
} catch (PDOException $e) {
    echo '执行失败: ' . $e->getMessage();
}
```

---

## 防注入原理

### SQL 注入攻击示例

**不安全的代码**：

```php
<?php
// 危险！容易受到 SQL 注入攻击
$email = $_POST['email'];
$sql = "SELECT * FROM users WHERE email = '$email'";
$stmt = $pdo->query($sql);
```

**攻击示例**：

如果用户输入 `' OR '1'='1`，实际执行的 SQL 为：

```sql
SELECT * FROM users WHERE email = '' OR '1'='1'
```

这会返回所有用户，绕过了身份验证。

**更危险的攻击**：

```php
// 用户输入: '; DROP TABLE users; --
$email = "'; DROP TABLE users; --";
$sql = "SELECT * FROM users WHERE email = '$email'";
// 实际执行的 SQL: SELECT * FROM users WHERE email = ''; DROP TABLE users; --'
// 这会删除 users 表！
```

### 预处理语句防护

预处理语句通过以下方式防止 SQL 注入：

**1. SQL 结构固定**

在准备阶段，SQL 语句结构就已经确定，占位符只是参数位置，不会改变 SQL 结构。

**示例**：

```php
<?php
declare(strict_types=1);

// 准备阶段：SQL 结构已确定
$stmt = $pdo->prepare('SELECT * FROM users WHERE email = :email');
// SQL 结构：SELECT * FROM users WHERE email = [参数位置]

// 执行阶段：参数值被安全处理
$stmt->execute(['email' => "'; DROP TABLE users; --"]);
// 数据库会查找 email 值为 "'; DROP TABLE users; --" 的用户
// 不会执行 DROP TABLE，因为 SQL 结构已经固定
```

**2. 参数转义**

参数值在绑定和执行时会被正确转义，特殊字符被处理为普通字符。

**示例**：

```php
<?php
declare(strict_types=1);

// 即使用户输入包含单引号，也会被正确处理
$stmt = $pdo->prepare('SELECT * FROM users WHERE email = :email');
$stmt->execute(['email' => "o'connor@example.com"]);
// 数据库会查找 email 值为 "o'connor@example.com" 的用户
// 单引号被转义，不会破坏 SQL 语句
```

**3. 类型检查**

预处理语句支持类型绑定，确保数据类型正确。

**示例**：

```php
<?php
declare(strict_types=1);

// 绑定为整数类型，防止类型混淆攻击
$stmt = $pdo->prepare('SELECT * FROM users WHERE id = :id');
$stmt->bindValue(':id', $id, PDO::PARAM_INT);
$stmt->execute();
```

### 最佳防护实践

**1. 始终使用预处理语句**

所有包含用户输入的 SQL 语句都应使用预处理语句。

**示例**：

```php
<?php
declare(strict_types=1);

// ✅ 正确：使用预处理语句
$stmt = $pdo->prepare('SELECT * FROM users WHERE email = :email');
$stmt->execute(['email' => $_POST['email']]);

// ❌ 错误：直接拼接 SQL
$sql = "SELECT * FROM users WHERE email = '{$_POST['email']}'";
$stmt = $pdo->query($sql);
```

**2. 验证输入数据**

即使使用预处理语句，也应验证输入数据。

**示例**：

```php
<?php
declare(strict_types=1);

$email = $_POST['email'] ?? '';

// 验证邮箱格式
if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
    throw new InvalidArgumentException('无效的邮箱地址');
}

// 使用预处理语句
$stmt = $pdo->prepare('SELECT * FROM users WHERE email = :email');
$stmt->execute(['email' => $email]);
```

**3. 使用白名单**

对于有限的选择项，使用白名单验证。

**示例**：

```php
<?php
declare(strict_types=1);

$status = $_GET['status'] ?? '';

// 白名单验证
$allowedStatuses = ['active', 'inactive', 'pending'];
if (!in_array($status, $allowedStatuses, true)) {
    throw new InvalidArgumentException('无效的状态值');
}

// 使用预处理语句
$stmt = $pdo->prepare('SELECT * FROM users WHERE status = :status');
$stmt->execute(['status' => $status]);
```

---

## 完整示例

### 用户登录示例

```php
<?php
declare(strict_types=1);

class UserLogin
{
    private PDO $pdo;
    
    public function __construct(PDO $pdo)
    {
        $this->pdo = $pdo;
    }
    
    public function authenticate(string $email, string $password): ?array
    {
        // 使用预处理语句防止 SQL 注入
        $stmt = $this->pdo->prepare('SELECT * FROM users WHERE email = :email LIMIT 1');
        $stmt->execute(['email' => $email]);
        
        $user = $stmt->fetch();
        
        if ($user && password_verify($password, $user['password_hash'])) {
            return $user;
        }
        
        return null;
    }
}

// 使用
try {
    $pdo = new PDO('mysql:host=localhost;dbname=test', 'user', 'pass', [
        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION
    ]);
    
    $login = new UserLogin($pdo);
    $user = $login->authenticate($_POST['email'], $_POST['password']);
    
    if ($user) {
        echo '登录成功';
    } else {
        echo '登录失败';
    }
} catch (PDOException $e) {
    error_log('登录错误: ' . $e->getMessage());
    echo '登录失败，请稍后重试';
}
```

### 批量插入示例

```php
<?php
declare(strict_types=1);

function insertUsers(PDO $pdo, array $users): void
{
    // 准备一次，执行多次
    $stmt = $pdo->prepare('INSERT INTO users (name, email, age) VALUES (:name, :email, :age)');
    
    foreach ($users as $user) {
        $stmt->execute([
            'name' => $user['name'],
            'email' => $user['email'],
            'age' => $user['age']
        ]);
    }
}

// 使用
try {
    $pdo = new PDO('mysql:host=localhost;dbname=test', 'user', 'pass', [
        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION
    ]);
    
    $users = [
        ['name' => 'John', 'email' => 'john@example.com', 'age' => 30],
        ['name' => 'Jane', 'email' => 'jane@example.com', 'age' => 25],
        ['name' => 'Bob', 'email' => 'bob@example.com', 'age' => 35]
    ];
    
    insertUsers($pdo, $users);
    echo '批量插入成功';
} catch (PDOException $e) {
    error_log('插入错误: ' . $e->getMessage());
    echo '插入失败';
}
```

### 动态查询示例

```php
<?php
declare(strict_types=1);

function searchUsers(PDO $pdo, array $filters): array
{
    $conditions = [];
    $params = [];
    
    // 构建动态查询条件
    if (isset($filters['name'])) {
        $conditions[] = 'name LIKE :name';
        $params['name'] = '%' . $filters['name'] . '%';
    }
    
    if (isset($filters['email'])) {
        $conditions[] = 'email = :email';
        $params['email'] = $filters['email'];
    }
    
    if (isset($filters['min_age'])) {
        $conditions[] = 'age >= :min_age';
        $params['min_age'] = $filters['min_age'];
    }
    
    // 构建 SQL 语句
    $sql = 'SELECT * FROM users';
    if (!empty($conditions)) {
        $sql .= ' WHERE ' . implode(' AND ', $conditions);
    }
    
    // 使用预处理语句
    $stmt = $pdo->prepare($sql);
    $stmt->execute($params);
    
    return $stmt->fetchAll();
}

// 使用
try {
    $pdo = new PDO('mysql:host=localhost;dbname=test', 'user', 'pass', [
        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION
    ]);
    
    $filters = [
        'name' => 'John',
        'min_age' => 18
    ];
    
    $users = searchUsers($pdo, $filters);
    foreach ($users as $user) {
        echo $user['name'] . "\n";
    }
} catch (PDOException $e) {
    error_log('查询错误: ' . $e->getMessage());
    echo '查询失败';
}
```

---

## 使用场景

### 所有数据库操作

预处理语句应该用于所有包含用户输入的数据库操作。

**示例**：

```php
<?php
declare(strict_types=1);

// 查询
$stmt = $pdo->prepare('SELECT * FROM users WHERE id = :id');
$stmt->execute(['id' => $_GET['id']]);

// 插入
$stmt = $pdo->prepare('INSERT INTO users (name, email) VALUES (:name, :email)');
$stmt->execute(['name' => $_POST['name'], 'email' => $_POST['email']]);

// 更新
$stmt = $pdo->prepare('UPDATE users SET name = :name WHERE id = :id');
$stmt->execute(['id' => $_POST['id'], 'name' => $_POST['name']]);

// 删除
$stmt = $pdo->prepare('DELETE FROM users WHERE id = :id');
$stmt->execute(['id' => $_POST['id']]);
```

### 用户输入处理

所有来自用户的输入都应使用预处理语句处理。

**示例**：

```php
<?php
declare(strict_types=1);

// 表单数据
$stmt = $pdo->prepare('INSERT INTO users (name, email) VALUES (:name, :email)');
$stmt->execute([
    'name' => $_POST['name'],
    'email' => $_POST['email']
]);

// URL 参数
$stmt = $pdo->prepare('SELECT * FROM users WHERE id = :id');
$stmt->execute(['id' => $_GET['id']]);

// Cookie 数据
$stmt = $pdo->prepare('SELECT * FROM users WHERE session_id = :session_id');
$stmt->execute(['session_id' => $_COOKIE['session_id']]);
```

### 动态查询

构建动态查询时，使用预处理语句确保安全。

**示例**：

```php
<?php
declare(strict_types=1);

$conditions = [];
$params = [];

if (!empty($_GET['name'])) {
    $conditions[] = 'name LIKE :name';
    $params['name'] = '%' . $_GET['name'] . '%';
}

if (!empty($_GET['status'])) {
    $conditions[] = 'status = :status';
    $params['status'] = $_GET['status'];
}

$sql = 'SELECT * FROM users';
if (!empty($conditions)) {
    $sql .= ' WHERE ' . implode(' AND ', $conditions);
}

$stmt = $pdo->prepare($sql);
$stmt->execute($params);
$users = $stmt->fetchAll();
```

---

## 注意事项

### 始终使用预处理语句

所有包含用户输入的 SQL 语句都应使用预处理语句，不要直接拼接 SQL。

**示例**：

```php
<?php
declare(strict_types=1);

// ❌ 错误：直接拼接 SQL
$sql = "SELECT * FROM users WHERE email = '{$_POST['email']}'";
$stmt = $pdo->query($sql);

// ✅ 正确：使用预处理语句
$stmt = $pdo->prepare('SELECT * FROM users WHERE email = :email');
$stmt->execute(['email' => $_POST['email']]);
```

### 参数绑定方式

优先使用 `execute()` 方法传递参数，代码更简洁。

**示例**：

```php
<?php
declare(strict_types=1);

// 推荐：使用 execute() 传递参数
$stmt = $pdo->prepare('SELECT * FROM users WHERE email = :email');
$stmt->execute(['email' => $email]);

// 也可以：使用 bindValue()
$stmt = $pdo->prepare('SELECT * FROM users WHERE email = :email');
$stmt->bindValue(':email', $email);
$stmt->execute();
```

### 类型处理

对于需要特定类型的参数，使用类型绑定。

**示例**：

```php
<?php
declare(strict_types=1);

// 整数类型
$stmt = $pdo->prepare('SELECT * FROM users WHERE id = :id');
$stmt->bindValue(':id', $id, PDO::PARAM_INT);

// 布尔类型
$stmt = $pdo->prepare('SELECT * FROM users WHERE is_active = :is_active');
$stmt->bindValue(':is_active', $isActive, PDO::PARAM_BOOL);

// 字符串类型（默认）
$stmt = $pdo->prepare('SELECT * FROM users WHERE email = :email');
$stmt->bindValue(':email', $email, PDO::PARAM_STR);
```

### 错误处理

在异常模式下，执行失败会抛出异常，需要捕获处理。

**示例**：

```php
<?php
declare(strict_types=1);

try {
    $stmt = $pdo->prepare('INSERT INTO users (name, email) VALUES (:name, :email)');
    $stmt->execute([
        'name' => $name,
        'email' => $email
    ]);
} catch (PDOException $e) {
    error_log('数据库错误: ' . $e->getMessage());
    throw new RuntimeException('操作失败', 0, $e);
}
```

---

## 常见问题

### 为什么使用预处理语句？

**原因**：

1. **安全性**：防止 SQL 注入攻击
2. **性能**：SQL 语句只需编译一次，可重复执行
3. **类型安全**：支持参数类型绑定
4. **代码清晰**：SQL 语句和参数分离

### 命名参数和位置参数的区别？

**命名参数（:name）**：

- 可读性更好
- 参数顺序不重要
- 适合参数较多的情况

**位置参数（?）**：

- 代码更简洁
- 参数顺序重要
- 适合参数较少的情况

**选择建议**：推荐使用命名参数，代码更清晰。

### 如何防止 SQL 注入？

**方法**：

1. **始终使用预处理语句**：所有包含用户输入的 SQL 都应使用预处理语句
2. **验证输入数据**：验证输入数据的格式和范围
3. **使用白名单**：对于有限的选择项，使用白名单验证
4. **最小权限原则**：数据库用户只授予必要的权限

### 预处理语句的性能影响？

**性能影响**：

- **首次执行**：预处理语句需要编译，可能稍慢
- **重复执行**：预处理语句只需编译一次，重复执行时性能更好
- **总体影响**：对于重复执行的查询，预处理语句性能更好

**建议**：对于执行一次的查询，性能差异可以忽略；对于重复执行的查询，预处理语句性能更好。

---

## 最佳实践

### 始终使用预处理语句

所有包含用户输入的 SQL 语句都应使用预处理语句。

```php
<?php
declare(strict_types=1);

// ✅ 正确
$stmt = $pdo->prepare('SELECT * FROM users WHERE email = :email');
$stmt->execute(['email' => $_POST['email']]);
```

### 使用命名参数提高可读性

命名参数比位置参数更清晰，推荐使用。

```php
<?php
declare(strict_types=1);

// ✅ 推荐：命名参数
$stmt = $pdo->prepare('SELECT * FROM users WHERE email = :email AND age > :age');
$stmt->execute(['email' => $email, 'age' => 18]);
```

### 验证输入数据

即使使用预处理语句，也应验证输入数据。

```php
<?php
declare(strict_types=1);

$email = $_POST['email'] ?? '';

// 验证邮箱格式
if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
    throw new InvalidArgumentException('无效的邮箱地址');
}

// 使用预处理语句
$stmt = $pdo->prepare('SELECT * FROM users WHERE email = :email');
$stmt->execute(['email' => $email]);
```

### 处理执行错误

在异常模式下，捕获并处理执行错误。

```php
<?php
declare(strict_types=1);

try {
    $stmt = $pdo->prepare('INSERT INTO users (name, email) VALUES (:name, :email)');
    $stmt->execute(['name' => $name, 'email' => $email]);
} catch (PDOException $e) {
    error_log('数据库错误: ' . $e->getMessage());
    throw new RuntimeException('操作失败', 0, $e);
}
```

---

## 练习任务

1. **预处理语句基础**
   - 使用预处理语句查询用户
   - 使用命名参数和位置参数
   - 测试参数绑定
   - 验证防注入效果

2. **用户认证**
   - 实现用户登录功能
   - 使用预处理语句防止 SQL 注入
   - 验证密码
   - 处理登录错误

3. **动态查询**
   - 实现动态查询功能
   - 根据条件构建 SQL 语句
   - 使用预处理语句执行查询
   - 测试各种查询组合

4. **批量操作**
   - 使用预处理语句批量插入数据
   - 使用预处理语句批量更新数据
   - 测试性能差异
   - 处理批量操作错误

5. **综合应用**
   - 创建一个用户管理类
   - 实现 CRUD 操作
   - 所有操作使用预处理语句
   - 实现错误处理和日志记录

---

**相关章节**：

- [6.2.1 PDO 基础与连接](section-01-pdo-basics.md)
- [6.2.3 CRUD 操作](section-03-crud-operations.md)
- [6.2.4 高级特性](section-04-advanced-features.md)
