# 5.2.2 预处理语句与防注入

## 概述

预处理语句是防止 SQL 注入的关键机制。本节详细介绍 SQL 注入原理、预处理语句的使用、`bindParam` vs `bindValue` 的区别，以及防注入最佳实践。

## SQL 注入原理

### 什么是 SQL 注入

- **SQL 注入**：通过构造恶意 SQL 语句，绕过应用程序的安全检查
- 攻击者可以读取、修改、删除数据库数据

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
- 准备 SQL 语句，使用占位符代替实际值

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
- 执行预处理语句，传入参数数组

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

## bindParam vs bindValue

### bindParam

- **按引用绑定**：绑定变量引用，执行时使用变量的当前值
- 可以多次执行，每次使用变量的最新值

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

### bindValue

- **按值绑定**：绑定变量的值，执行时使用绑定时的值
- 即使变量后续改变，也不会影响已绑定的值

```php
<?php
declare(strict_types=1);

$stmt = $pdo->prepare("SELECT * FROM users WHERE id = :id");

$id = 1;
$stmt->bindValue(':id', $id, PDO::PARAM_INT);

$id = 2;  // 改变变量值

$stmt->execute();  // 仍然查询 id = 1（使用绑定时的值）
```

### 对比总结

| 特性 | bindParam | bindValue |
| :--- | :--- | :--- |
| 绑定方式 | 按引用 | 按值 |
| 变量变化 | 影响执行结果 | 不影响执行结果 |
| 适用场景 | 循环执行，变量会变化 | 单次执行，值固定 |
| 性能 | 稍好（避免值复制） | 稍差（需要值复制） |

## 防注入最佳实践

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

## 完整示例

```php
<?php
declare(strict_types=1);

class SecureQuery
{
    private PDO $pdo;

    public function __construct(PDO $pdo)
    {
        $this->pdo = $pdo;
    }

    public function findUser(string $username): ?array
    {
        $stmt = $this->pdo->prepare("SELECT * FROM users WHERE username = ?");
        $stmt->execute([$username]);
        $user = $stmt->fetch();
        
        return $user !== false ? $user : null;
    }

    public function searchUsers(string $keyword, int $limit = 10): array
    {
        $stmt = $this->pdo->prepare(
            "SELECT * FROM users WHERE username LIKE ? OR email LIKE ? LIMIT ?"
        );
        $searchTerm = "%{$keyword}%";
        $stmt->execute([$searchTerm, $searchTerm, $limit]);
        return $stmt->fetchAll();
    }
}
```

## 注意事项

1. **永远不拼接**：不要直接拼接用户输入到 SQL
2. **使用占位符**：所有用户输入都使用占位符
3. **禁用模拟**：使用数据库原生预处理
4. **类型绑定**：明确指定参数类型

## 练习

1. 实现一个安全的用户认证函数，使用预处理语句验证用户名和密码。

2. 创建一个查询构建器，支持安全的参数绑定。

3. 实现一个批量查询函数，使用预处理语句和循环。

4. 编写一个 SQL 注入测试，验证预处理语句的防护效果。
