# 6.2.3 CRUD 操作

## 概述

CRUD（Create、Read、Update、Delete）操作是数据库操作的基础，涵盖了数据的创建、读取、更新和删除。本节详细介绍使用 PDO 进行 CRUD 操作的方法，包括单条和批量操作、结果集处理、错误处理等，帮助零基础学员掌握完整的数据库操作技能。

理解 CRUD 操作对于进行 Web 开发至关重要，几乎所有 Web 应用都需要进行数据的增删改查操作。本节将从基础操作开始，逐步介绍高级用法和最佳实践。

**主要内容**：
- CRUD 操作概述
- 创建操作（INSERT）
- 读取操作（SELECT）
- 更新操作（UPDATE）
- 删除操作（DELETE）
- 结果集处理
- 完整示例和最佳实践

---

## 特性

- **完整性**：涵盖数据的创建、读取、更新、删除
- **安全性**：使用预处理语句防止 SQL 注入
- **灵活性**：支持单条和批量操作
- **易用性**：提供简洁的 API 进行操作
- **性能**：支持批量操作，提高性能

---

## CRUD 操作概述

### 什么是 CRUD

CRUD 是四个基本数据库操作的缩写：

- **Create（创建）**：插入新数据
- **Read（读取）**：查询数据
- **Update（更新）**：修改现有数据
- **Delete（删除）**：删除数据

### CRUD 操作的重要性

CRUD 操作是数据库操作的基础，几乎所有应用都需要：

- **数据管理**：管理应用的核心数据
- **用户交互**：响应用户的增删改查请求
- **业务逻辑**：实现业务功能的基础

### 使用预处理语句

所有 CRUD 操作都应使用预处理语句，确保安全性。

**示例**：

```php
<?php
declare(strict_types=1);

// ✅ 正确：使用预处理语句
$stmt = $pdo->prepare('INSERT INTO users (name, email) VALUES (:name, :email)');
$stmt->execute(['name' => $name, 'email' => $email]);

// ❌ 错误：直接拼接 SQL
$sql = "INSERT INTO users (name, email) VALUES ('$name', '$email')";
$pdo->exec($sql);
```

---

## 创建操作（INSERT）

### INSERT 语句

INSERT 语句用于向表中插入新数据。

**语法**：`INSERT INTO table_name (column1, column2, ...) VALUES (value1, value2, ...)`

### 单条插入

插入单条数据是最基本的操作。

**示例**：

```php
<?php
declare(strict_types=1);

try {
    $stmt = $pdo->prepare('INSERT INTO users (name, email, age) VALUES (:name, :email, :age)');
    $stmt->execute([
        'name' => 'John Doe',
        'email' => 'john@example.com',
        'age' => 30
    ]);
    
    $id = $pdo->lastInsertId();
    echo "插入成功，ID: $id\n";
} catch (PDOException $e) {
    echo '插入失败: ' . $e->getMessage() . "\n";
}
```

### 批量插入

批量插入可以提高性能，特别适合需要插入多条数据的场景。

**方法 1：循环执行**

```php
<?php
declare(strict_types=1);

$stmt = $pdo->prepare('INSERT INTO users (name, email, age) VALUES (:name, :email, :age)');

$users = [
    ['name' => 'John', 'email' => 'john@example.com', 'age' => 30],
    ['name' => 'Jane', 'email' => 'jane@example.com', 'age' => 25],
    ['name' => 'Bob', 'email' => 'bob@example.com', 'age' => 35]
];

foreach ($users as $user) {
    $stmt->execute($user);
}
```

**方法 2：使用事务批量插入**

```php
<?php
declare(strict_types=1);

try {
    $pdo->beginTransaction();
    
    $stmt = $pdo->prepare('INSERT INTO users (name, email, age) VALUES (:name, :email, :age)');
    
    $users = [
        ['name' => 'John', 'email' => 'john@example.com', 'age' => 30],
        ['name' => 'Jane', 'email' => 'jane@example.com', 'age' => 25],
        ['name' => 'Bob', 'email' => 'bob@example.com', 'age' => 35]
    ];
    
    foreach ($users as $user) {
        $stmt->execute($user);
    }
    
    $pdo->commit();
    echo "批量插入成功\n";
} catch (PDOException $e) {
    $pdo->rollBack();
    echo '批量插入失败: ' . $e->getMessage() . "\n";
}
```

**方法 3：使用 VALUES 子句批量插入**

```php
<?php
declare(strict_types=1);

// 注意：MySQL 5.7+ 支持，但需要注意参数绑定
$stmt = $pdo->prepare('
    INSERT INTO users (name, email, age) 
    VALUES 
        (:name1, :email1, :age1),
        (:name2, :email2, :age2),
        (:name3, :email3, :age3)
');

$stmt->execute([
    'name1' => 'John', 'email1' => 'john@example.com', 'age1' => 30,
    'name2' => 'Jane', 'email2' => 'jane@example.com', 'age2' => 25,
    'name3' => 'Bob', 'email3' => 'bob@example.com', 'age3' => 35
]);
```

### 获取插入 ID

使用 `lastInsertId()` 方法获取最后插入的自增 ID。

**语法**：`PDO::lastInsertId(?string $name = null): string|false`

**示例**：

```php
<?php
declare(strict_types=1);

$stmt = $pdo->prepare('INSERT INTO users (name, email) VALUES (:name, :email)');
$stmt->execute([
    'name' => 'John Doe',
    'email' => 'john@example.com'
]);

$id = $pdo->lastInsertId();
echo "插入的用户 ID: $id\n";
```

**注意**：

- `lastInsertId()` 返回字符串类型
- 只适用于自增主键
- 需要在插入后立即调用
- 如果表名包含特殊字符，需要传递表名参数

---

## 读取操作（SELECT）

### SELECT 语句

SELECT 语句用于从表中查询数据。

**语法**：`SELECT column1, column2, ... FROM table_name WHERE condition`

### 单条查询

查询单条数据使用 `fetch()` 方法。

**示例**：

```php
<?php
declare(strict_types=1);

// 查询单条数据
$stmt = $pdo->prepare('SELECT * FROM users WHERE id = :id');
$stmt->execute(['id' => 1]);
$user = $stmt->fetch();

if ($user) {
    echo "用户名: {$user['name']}\n";
    echo "邮箱: {$user['email']}\n";
} else {
    echo "用户不存在\n";
}
```

### 多条查询

查询多条数据使用 `fetchAll()` 方法。

**示例**：

```php
<?php
declare(strict_types=1);

// 查询多条数据
$stmt = $pdo->prepare('SELECT * FROM users WHERE age > :age');
$stmt->execute(['age' => 18]);
$users = $stmt->fetchAll();

foreach ($users as $user) {
    echo "{$user['name']} - {$user['email']}\n";
}
```

### 条件查询

使用 WHERE 子句进行条件查询。

**示例**：

```php
<?php
declare(strict_types=1);

// 单个条件
$stmt = $pdo->prepare('SELECT * FROM users WHERE email = :email');
$stmt->execute(['email' => 'john@example.com']);
$user = $stmt->fetch();

// 多个条件（AND）
$stmt = $pdo->prepare('SELECT * FROM users WHERE email = :email AND age > :age');
$stmt->execute([
    'email' => 'john@example.com',
    'age' => 18
]);
$users = $stmt->fetchAll();

// LIKE 查询
$stmt = $pdo->prepare('SELECT * FROM users WHERE name LIKE :name');
$stmt->execute(['name' => '%John%']);
$users = $stmt->fetchAll();

// IN 查询
$stmt = $pdo->prepare('SELECT * FROM users WHERE id IN (:id1, :id2, :id3)');
$stmt->execute([
    'id1' => 1,
    'id2' => 2,
    'id3' => 3
]);
$users = $stmt->fetchAll();
```

### 分页查询

使用 LIMIT 和 OFFSET 实现分页。

**示例**：

```php
<?php
declare(strict_types=1);

function getUsers(PDO $pdo, int $page = 1, int $perPage = 10): array
{
    $offset = ($page - 1) * $perPage;
    
    $stmt = $pdo->prepare('SELECT * FROM users ORDER BY id DESC LIMIT :limit OFFSET :offset');
    $stmt->bindValue(':limit', $perPage, PDO::PARAM_INT);
    $stmt->bindValue(':offset', $offset, PDO::PARAM_INT);
    $stmt->execute();
    
    return $stmt->fetchAll();
}

// 使用
$users = getUsers($pdo, 1, 10);
```

### 排序查询

使用 ORDER BY 子句进行排序。

**示例**：

```php
<?php
declare(strict_types=1);

// 升序排序
$stmt = $pdo->prepare('SELECT * FROM users ORDER BY name ASC');
$stmt->execute();
$users = $stmt->fetchAll();

// 降序排序
$stmt = $pdo->prepare('SELECT * FROM users ORDER BY created_at DESC');
$stmt->execute();
$users = $stmt->fetchAll();

// 多字段排序
$stmt = $pdo->prepare('SELECT * FROM users ORDER BY status ASC, created_at DESC');
$stmt->execute();
$users = $stmt->fetchAll();
```

---

## 更新操作（UPDATE）

### UPDATE 语句

UPDATE 语句用于更新表中的现有数据。

**语法**：`UPDATE table_name SET column1 = value1, column2 = value2, ... WHERE condition`

### 单条更新

更新单条数据。

**示例**：

```php
<?php
declare(strict_types=1);

try {
    $stmt = $pdo->prepare('UPDATE users SET name = :name, email = :email WHERE id = :id');
    $stmt->execute([
        'id' => 1,
        'name' => 'Jane Doe',
        'email' => 'jane@example.com'
    ]);
    
    if ($stmt->rowCount() > 0) {
        echo "更新成功\n";
    } else {
        echo "没有数据被更新\n";
    }
} catch (PDOException $e) {
    echo '更新失败: ' . $e->getMessage() . "\n";
}
```

### 批量更新

更新多条数据。

**示例**：

```php
<?php
declare(strict_types=1);

try {
    $pdo->beginTransaction();
    
    $stmt = $pdo->prepare('UPDATE users SET status = :status WHERE id = :id');
    
    $updates = [
        ['id' => 1, 'status' => 'active'],
        ['id' => 2, 'status' => 'inactive'],
        ['id' => 3, 'status' => 'active']
    ];
    
    foreach ($updates as $update) {
        $stmt->execute($update);
    }
    
    $pdo->commit();
    echo "批量更新成功\n";
} catch (PDOException $e) {
    $pdo->rollBack();
    echo '批量更新失败: ' . $e->getMessage() . "\n";
}
```

### 条件更新

使用 WHERE 子句进行条件更新。

**示例**：

```php
<?php
declare(strict_types=1);

// 更新所有符合条件的记录
$stmt = $pdo->prepare('UPDATE users SET status = :status WHERE age < :age');
$stmt->execute([
    'status' => 'inactive',
    'age' => 18
]);

echo "更新了 {$stmt->rowCount()} 条记录\n";
```

### 影响行数

使用 `rowCount()` 方法获取受影响的行数。

**示例**：

```php
<?php
declare(strict_types=1);

$stmt = $pdo->prepare('UPDATE users SET status = :status WHERE id = :id');
$stmt->execute([
    'id' => 1,
    'status' => 'active'
]);

$affectedRows = $stmt->rowCount();

if ($affectedRows > 0) {
    echo "更新了 $affectedRows 条记录\n";
} else {
    echo "没有记录被更新（可能记录不存在或值未改变）\n";
}
```

**注意**：

- `rowCount()` 返回受影响的行数
- 对于 SELECT 语句，行为未定义
- 如果更新的值与原值相同，可能返回 0

---

## 删除操作（DELETE）

### DELETE 语句

DELETE 语句用于从表中删除数据。

**语法**：`DELETE FROM table_name WHERE condition`

### 单条删除

删除单条数据。

**示例**：

```php
<?php
declare(strict_types=1);

try {
    $stmt = $pdo->prepare('DELETE FROM users WHERE id = :id');
    $stmt->execute(['id' => 1]);
    
    if ($stmt->rowCount() > 0) {
        echo "删除成功\n";
    } else {
        echo "记录不存在\n";
    }
} catch (PDOException $e) {
    echo '删除失败: ' . $e->getMessage() . "\n";
}
```

### 批量删除

删除多条数据。

**示例**：

```php
<?php
declare(strict_types=1);

try {
    $pdo->beginTransaction();
    
    $stmt = $pdo->prepare('DELETE FROM users WHERE id = :id');
    
    $ids = [1, 2, 3];
    
    foreach ($ids as $id) {
        $stmt->execute(['id' => $id]);
    }
    
    $pdo->commit();
    echo "批量删除成功\n";
} catch (PDOException $e) {
    $pdo->rollBack();
    echo '批量删除失败: ' . $e->getMessage() . "\n";
}
```

### 条件删除

使用 WHERE 子句进行条件删除。

**示例**：

```php
<?php
declare(strict_types=1);

// 删除所有符合条件的记录
$stmt = $pdo->prepare('DELETE FROM users WHERE status = :status');
$stmt->execute(['status' => 'inactive']);

echo "删除了 {$stmt->rowCount()} 条记录\n";
```

### 软删除

软删除（Soft Delete）是指不真正删除数据，而是通过更新标志字段标记为已删除。

**示例**：

```php
<?php
declare(strict_types=1);

// 软删除：更新 deleted_at 字段
$stmt = $pdo->prepare('UPDATE users SET deleted_at = NOW() WHERE id = :id');
$stmt->execute(['id' => 1]);

// 查询时排除已删除的记录
$stmt = $pdo->prepare('SELECT * FROM users WHERE deleted_at IS NULL');
$stmt->execute();
$users = $stmt->fetchAll();
```

**软删除的优势**：

- 数据可以恢复
- 保留历史记录
- 不影响外键约束

---

## 结果集处理

### fetch() 方法

`fetch()` 方法获取结果集中的下一行。

**语法**：`PDOStatement::fetch(int $mode = PDO::FETCH_DEFAULT, int $cursorOrientation = PDO::FETCH_ORI_NEXT, int $cursorOffset = 0): mixed`

**获取模式**：

| 模式 | 说明 | 示例 |
|:-----|:-----|:-----|
| `PDO::FETCH_ASSOC` | 关联数组 | `['id' => 1, 'name' => 'John']` |
| `PDO::FETCH_NUM` | 数字索引数组 | `[0 => 1, 1 => 'John']` |
| `PDO::FETCH_BOTH` | 关联数组和数字索引数组 | 两者结合 |
| `PDO::FETCH_OBJ` | 对象 | `stdClass { id: 1, name: 'John' }` |
| `PDO::FETCH_CLASS` | 指定类的对象 | 自定义类的实例 |

**示例**：

```php
<?php
declare(strict_types=1);

$stmt = $pdo->prepare('SELECT * FROM users WHERE id = :id');
$stmt->execute(['id' => 1]);

// 关联数组
$user = $stmt->fetch(PDO::FETCH_ASSOC);
echo $user['name'];

// 对象
$user = $stmt->fetch(PDO::FETCH_OBJ);
echo $user->name;
```

### fetchAll() 方法

`fetchAll()` 方法获取结果集中的所有行。

**语法**：`PDOStatement::fetchAll(int $mode = PDO::FETCH_DEFAULT): array`

**示例**：

```php
<?php
declare(strict_types=1);

$stmt = $pdo->prepare('SELECT * FROM users');
$stmt->execute();

// 获取所有行（关联数组）
$users = $stmt->fetchAll(PDO::FETCH_ASSOC);

foreach ($users as $user) {
    echo $user['name'] . "\n";
}

// 获取所有行（对象）
$users = $stmt->fetchAll(PDO::FETCH_OBJ);

foreach ($users as $user) {
    echo $user->name . "\n";
}
```

### fetchColumn() 方法

`fetchColumn()` 方法获取结果集中下一行的指定列。

**语法**：`PDOStatement::fetchColumn(int $column = 0): mixed`

**示例**：

```php
<?php
declare(strict_types=1);

// 获取第一列
$stmt = $pdo->prepare('SELECT name FROM users');
$stmt->execute();

while ($name = $stmt->fetchColumn()) {
    echo $name . "\n";
}

// 获取指定列
$stmt = $pdo->prepare('SELECT name, email, age FROM users');
$stmt->execute();

while ($email = $stmt->fetchColumn(1)) {  // 第二列（索引从0开始）
    echo $email . "\n";
}
```

### rowCount() 方法

`rowCount()` 方法返回受影响的行数。

**语法**：`PDOStatement::rowCount(): int`

**示例**：

```php
<?php
declare(strict_types=1);

// UPDATE 操作
$stmt = $pdo->prepare('UPDATE users SET status = :status WHERE id = :id');
$stmt->execute(['id' => 1, 'status' => 'active']);
echo "更新了 {$stmt->rowCount()} 条记录\n";

// DELETE 操作
$stmt = $pdo->prepare('DELETE FROM users WHERE id = :id');
$stmt->execute(['id' => 1]);
echo "删除了 {$stmt->rowCount()} 条记录\n";
```

**注意**：对于 SELECT 语句，`rowCount()` 的行为未定义，不应依赖其返回值。

---

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
    
    // 创建用户
    public function create(string $name, string $email, int $age): int
    {
        $stmt = $this->pdo->prepare('INSERT INTO users (name, email, age) VALUES (:name, :email, :age)');
        $stmt->execute([
            'name' => $name,
            'email' => $email,
            'age' => $age
        ]);
        
        return (int)$this->pdo->lastInsertId();
    }
    
    // 根据 ID 查询用户
    public function findById(int $id): ?array
    {
        $stmt = $this->pdo->prepare('SELECT * FROM users WHERE id = :id');
        $stmt->execute(['id' => $id]);
        
        $user = $stmt->fetch(PDO::FETCH_ASSOC);
        
        return $user ?: null;
    }
    
    // 根据邮箱查询用户
    public function findByEmail(string $email): ?array
    {
        $stmt = $this->pdo->prepare('SELECT * FROM users WHERE email = :email');
        $stmt->execute(['email' => $email]);
        
        $user = $stmt->fetch(PDO::FETCH_ASSOC);
        
        return $user ?: null;
    }
    
    // 查询所有用户
    public function findAll(): array
    {
        $stmt = $this->pdo->prepare('SELECT * FROM users ORDER BY id DESC');
        $stmt->execute();
        
        return $stmt->fetchAll(PDO::FETCH_ASSOC);
    }
    
    // 分页查询
    public function findPaginated(int $page = 1, int $perPage = 10): array
    {
        $offset = ($page - 1) * $perPage;
        
        $stmt = $this->pdo->prepare('SELECT * FROM users ORDER BY id DESC LIMIT :limit OFFSET :offset');
        $stmt->bindValue(':limit', $perPage, PDO::PARAM_INT);
        $stmt->bindValue(':offset', $offset, PDO::PARAM_INT);
        $stmt->execute();
        
        return $stmt->fetchAll(PDO::FETCH_ASSOC);
    }
    
    // 更新用户
    public function update(int $id, array $data): bool
    {
        $fields = [];
        $params = ['id' => $id];
        
        foreach ($data as $key => $value) {
            $fields[] = "$key = :$key";
            $params[$key] = $value;
        }
        
        $sql = 'UPDATE users SET ' . implode(', ', $fields) . ' WHERE id = :id';
        $stmt = $this->pdo->prepare($sql);
        $stmt->execute($params);
        
        return $stmt->rowCount() > 0;
    }
    
    // 删除用户
    public function delete(int $id): bool
    {
        $stmt = $this->pdo->prepare('DELETE FROM users WHERE id = :id');
        $stmt->execute(['id' => $id]);
        
        return $stmt->rowCount() > 0;
    }
    
    // 软删除用户
    public function softDelete(int $id): bool
    {
        $stmt = $this->pdo->prepare('UPDATE users SET deleted_at = NOW() WHERE id = :id');
        $stmt->execute(['id' => $id]);
        
        return $stmt->rowCount() > 0;
    }
}

// 使用
try {
    $pdo = new PDO('mysql:host=localhost;dbname=test', 'user', 'pass', [
        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC
    ]);
    
    $repo = new UserRepository($pdo);
    
    // 创建用户
    $id = $repo->create('John Doe', 'john@example.com', 30);
    echo "创建用户，ID: $id\n";
    
    // 查询用户
    $user = $repo->findById($id);
    if ($user) {
        echo "用户: {$user['name']}\n";
    }
    
    // 更新用户
    $repo->update($id, ['name' => 'Jane Doe', 'age' => 25]);
    echo "更新成功\n";
    
    // 查询所有用户
    $users = $repo->findAll();
    echo "共有 " . count($users) . " 个用户\n";
    
    // 删除用户
    $repo->delete($id);
    echo "删除成功\n";
} catch (PDOException $e) {
    echo '数据库错误: ' . $e->getMessage() . "\n";
}
```

---

## 使用场景

### 数据管理

CRUD 操作用于管理应用的核心数据。

**示例**：

```php
<?php
declare(strict_types=1);

// 创建文章
$stmt = $pdo->prepare('INSERT INTO articles (title, content, user_id) VALUES (:title, :content, :user_id)');
$stmt->execute([
    'title' => 'PHP 教程',
    'content' => '这是 PHP 教程的内容...',
    'user_id' => 1
]);

// 查询文章
$stmt = $pdo->prepare('SELECT * FROM articles WHERE user_id = :user_id');
$stmt->execute(['user_id' => 1]);
$articles = $stmt->fetchAll();

// 更新文章
$stmt = $pdo->prepare('UPDATE articles SET title = :title WHERE id = :id');
$stmt->execute(['id' => 1, 'title' => '更新的标题']);

// 删除文章
$stmt = $pdo->prepare('DELETE FROM articles WHERE id = :id');
$stmt->execute(['id' => 1]);
```

### 用户管理

CRUD 操作用于管理用户数据。

**示例**：

```php
<?php
declare(strict_types=1);

// 用户注册
$stmt = $pdo->prepare('INSERT INTO users (username, email, password_hash) VALUES (:username, :email, :password_hash)');
$stmt->execute([
    'username' => $_POST['username'],
    'email' => $_POST['email'],
    'password_hash' => password_hash($_POST['password'], PASSWORD_DEFAULT)
]);

// 用户登录
$stmt = $pdo->prepare('SELECT * FROM users WHERE email = :email');
$stmt->execute(['email' => $_POST['email']]);
$user = $stmt->fetch();

// 更新用户信息
$stmt = $pdo->prepare('UPDATE users SET name = :name WHERE id = :id');
$stmt->execute(['id' => $_SESSION['user_id'], 'name' => $_POST['name']]);

// 删除用户
$stmt = $pdo->prepare('DELETE FROM users WHERE id = :id');
$stmt->execute(['id' => $_POST['user_id']]);
```

---

## 注意事项

### 使用预处理语句

所有 CRUD 操作都应使用预处理语句，确保安全性。

```php
<?php
declare(strict_types=1);

// ✅ 正确
$stmt = $pdo->prepare('INSERT INTO users (name, email) VALUES (:name, :email)');
$stmt->execute(['name' => $name, 'email' => $email]);
```

### 错误处理

在异常模式下，操作失败会抛出异常，需要捕获处理。

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

### 事务管理

对于需要保证一致性的操作，使用事务管理。

```php
<?php
declare(strict_types=1);

try {
    $pdo->beginTransaction();
    
    // 多个操作
    $stmt1 = $pdo->prepare('INSERT INTO users (name) VALUES (:name)');
    $stmt1->execute(['name' => 'John']);
    
    $stmt2 = $pdo->prepare('INSERT INTO profiles (user_id, bio) VALUES (:user_id, :bio)');
    $stmt2->execute(['user_id' => $pdo->lastInsertId(), 'bio' => 'Bio']);
    
    $pdo->commit();
} catch (PDOException $e) {
    $pdo->rollBack();
    throw $e;
}
```

### 性能优化

对于批量操作，使用事务可以提高性能。

```php
<?php
declare(strict_types=1);

try {
    $pdo->beginTransaction();
    
    $stmt = $pdo->prepare('INSERT INTO users (name, email) VALUES (:name, :email)');
    
    foreach ($users as $user) {
        $stmt->execute($user);
    }
    
    $pdo->commit();
} catch (PDOException $e) {
    $pdo->rollBack();
    throw $e;
}
```

---

## 常见问题

### 如何插入数据？

使用 INSERT 语句和预处理语句插入数据。

```php
<?php
declare(strict_types=1);

$stmt = $pdo->prepare('INSERT INTO users (name, email) VALUES (:name, :email)');
$stmt->execute(['name' => $name, 'email' => $email]);
$id = $pdo->lastInsertId();
```

### 如何查询数据？

使用 SELECT 语句和预处理语句查询数据。

```php
<?php
declare(strict_types=1);

// 单条查询
$stmt = $pdo->prepare('SELECT * FROM users WHERE id = :id');
$stmt->execute(['id' => $id]);
$user = $stmt->fetch();

// 多条查询
$stmt = $pdo->prepare('SELECT * FROM users');
$stmt->execute();
$users = $stmt->fetchAll();
```

### 如何更新数据？

使用 UPDATE 语句和预处理语句更新数据。

```php
<?php
declare(strict_types=1);

$stmt = $pdo->prepare('UPDATE users SET name = :name WHERE id = :id');
$stmt->execute(['id' => $id, 'name' => $name]);
$affectedRows = $stmt->rowCount();
```

### 如何删除数据？

使用 DELETE 语句和预处理语句删除数据。

```php
<?php
declare(strict_types=1);

$stmt = $pdo->prepare('DELETE FROM users WHERE id = :id');
$stmt->execute(['id' => $id]);
$affectedRows = $stmt->rowCount();
```

---

## 最佳实践

### 使用预处理语句

所有 CRUD 操作都应使用预处理语句，确保安全性。

```php
<?php
declare(strict_types=1);

$stmt = $pdo->prepare('INSERT INTO users (name, email) VALUES (:name, :email)');
$stmt->execute(['name' => $name, 'email' => $email]);
```

### 实现错误处理

在异常模式下，捕获并处理数据库错误。

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

### 使用事务管理

对于需要保证一致性的操作，使用事务管理。

```php
<?php
declare(strict_types=1);

try {
    $pdo->beginTransaction();
    
    // 多个操作
    
    $pdo->commit();
} catch (PDOException $e) {
    $pdo->rollBack();
    throw $e;
}
```

### 优化查询性能

- 使用索引优化查询
- 使用 LIMIT 限制结果集大小
- 使用批量操作提高性能
- 使用事务减少网络往返

---

## 练习任务

1. **基础 CRUD 操作**
   - 实现用户的创建、读取、更新、删除操作
   - 使用预处理语句
   - 实现错误处理
   - 测试各种操作

2. **批量操作**
   - 实现批量插入用户
   - 实现批量更新用户状态
   - 使用事务保证一致性
   - 测试性能差异

3. **条件查询**
   - 实现多条件查询
   - 实现分页查询
   - 实现排序查询
   - 实现搜索功能

4. **软删除**
   - 实现软删除功能
   - 查询时排除已删除记录
   - 实现数据恢复功能
   - 测试软删除效果

5. **综合应用**
   - 创建一个完整的用户管理类
   - 实现所有 CRUD 操作
   - 实现错误处理和日志记录
   - 编写单元测试

---

**相关章节**：

- [6.2.1 PDO 基础与连接](section-01-pdo-basics.md)
- [6.2.2 预处理语句与防注入](section-02-prepared-statements.md)
- [6.2.4 高级特性](section-04-advanced-features.md)
