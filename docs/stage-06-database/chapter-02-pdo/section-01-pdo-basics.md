# 6.2.1 PDO 基础与连接

## 概述

PDO（PHP Data Objects）是 PHP 访问数据库的标准接口，提供统一的数据库访问抽象层。PDO 支持多种数据库（MySQL、PostgreSQL、SQLite 等），使用统一的 API 进行操作，简化了数据库操作的代码，提高了代码的可移植性和安全性。

本节详细介绍 PDO 的基础知识、连接方法、配置选项、错误处理等，帮助零基础学员掌握 PDO 的使用。理解 PDO 的连接机制和配置选项对于后续的数据库操作至关重要。

**主要内容**：
- PDO 概述和优势
- PDO 连接（DSN、用户名、密码）
- 连接配置（错误模式、属性设置）
- 错误处理和异常捕获
- 连接管理和最佳实践
- 完整示例

---

## 特性

- **数据库抽象**：统一的 API 访问不同数据库
- **预处理语句**：支持预处理语句，防止 SQL 注入
- **异常处理**：支持异常模式，便于错误处理
- **事务支持**：支持数据库事务
- **多种获取模式**：支持多种数据获取模式
- **跨数据库兼容**：代码可以在不同数据库间移植

---

## PDO 概述

### 什么是 PDO

PDO（PHP Data Objects）是 PHP 5.1+ 引入的数据库访问抽象层，提供统一的接口访问不同的数据库系统。

**PDO 的特点**：

- **数据库无关**：使用统一的 API 访问不同数据库
- **面向对象**：提供面向对象的接口
- **安全性**：支持预处理语句，防止 SQL 注入
- **灵活性**：支持多种数据获取模式

### PDO 的优势

**1. 统一的 API**

PDO 提供统一的 API，无论使用 MySQL、PostgreSQL 还是 SQLite，代码结构基本相同。

**示例**：

```php
<?php
declare(strict_types=1);

// MySQL 连接
$pdo_mysql = new PDO('mysql:host=localhost;dbname=test', 'user', 'pass');

// PostgreSQL 连接
$pdo_pgsql = new PDO('pgsql:host=localhost;dbname=test', 'user', 'pass');

// SQLite 连接
$pdo_sqlite = new PDO('sqlite:test.db');

// 使用相同的 API 操作
$stmt = $pdo_mysql->query('SELECT * FROM users');
$stmt = $pdo_pgsql->query('SELECT * FROM users');
$stmt = $pdo_sqlite->query('SELECT * FROM users');
```

**2. 安全性**

PDO 支持预处理语句，可以有效防止 SQL 注入攻击。

**示例**：

```php
<?php
declare(strict_types=1);

// 使用预处理语句，防止 SQL 注入
$stmt = $pdo->prepare('SELECT * FROM users WHERE id = ?');
$stmt->execute([$user_id]);
$user = $stmt->fetch();
```

**3. 异常处理**

PDO 支持异常模式，便于错误处理和调试。

**示例**：

```php
<?php
declare(strict_types=1);

try {
    $pdo = new PDO($dsn, $user, $pass, [
        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION
    ]);
    $pdo->query('SELECT * FROM users');
} catch (PDOException $e) {
    echo '数据库错误: ' . $e->getMessage();
}
```

### PDO 与 MySQLi 的对比

| 特性 | PDO | MySQLi |
|:-----|:----|:-------|
| 数据库支持 | 多种数据库 | 仅 MySQL |
| API 风格 | 面向对象 | 面向对象/过程式 |
| 预处理语句 | 支持 | 支持 |
| 异常处理 | 支持 | 支持 |
| 事务支持 | 支持 | 支持 |
| 代码可移植性 | 高 | 低 |

**选择建议**：

- **PDO**：适合需要支持多种数据库或需要代码可移植性的项目
- **MySQLi**：适合仅使用 MySQL 且需要 MySQL 特定功能的项目

---

## PDO 连接

### DSN（数据源名称）

DSN（Data Source Name）是连接数据库的字符串，包含数据库类型、主机、数据库名等信息。

**语法**：`driver:key1=value1;key2=value2;...`

**MySQL DSN 格式**：

```php
$dsn = 'mysql:host=localhost;dbname=test;charset=utf8mb4';
```

**DSN 参数**：

| 参数 | 说明 | 示例 |
|:-----|:-----|:-----|
| `host` | 数据库主机地址 | `localhost`、`127.0.0.1` |
| `port` | 数据库端口 | `3306` |
| `dbname` | 数据库名称 | `test` |
| `charset` | 字符集 | `utf8mb4` |
| `unix_socket` | Unix socket 路径 | `/tmp/mysql.sock` |

**示例**：

```php
<?php
declare(strict_types=1);

// 基本连接
$dsn = 'mysql:host=localhost;dbname=test';

// 指定端口
$dsn = 'mysql:host=localhost;port=3306;dbname=test';

// 指定字符集
$dsn = 'mysql:host=localhost;dbname=test;charset=utf8mb4';

// 使用 Unix socket
$dsn = 'mysql:unix_socket=/tmp/mysql.sock;dbname=test';
```

### 连接参数

PDO 构造函数接受四个参数：

**语法**：`new PDO(string $dsn, ?string $username = null, ?string $password = null, ?array $options = null)`

**参数**：

- `$dsn`：数据源名称（必需）
- `$username`：数据库用户名（可选）
- `$password`：数据库密码（可选）
- `$options`：连接选项数组（可选）

**示例**：

```php
<?php
declare(strict_types=1);

// 基本连接
$pdo = new PDO('mysql:host=localhost;dbname=test', 'username', 'password');

// 无密码连接
$pdo = new PDO('mysql:host=localhost;dbname=test', 'username');

// 使用选项连接
$pdo = new PDO(
    'mysql:host=localhost;dbname=test',
    'username',
    'password',
    [
        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC
    ]
);
```

### 连接选项

连接选项用于配置 PDO 的行为，常用选项包括：

| 选项 | 说明 | 值 |
|:-----|:-----|:---|
| `PDO::ATTR_ERRMODE` | 错误模式 | `ERRMODE_SILENT`、`ERRMODE_WARNING`、`ERRMODE_EXCEPTION` |
| `PDO::ATTR_DEFAULT_FETCH_MODE` | 默认获取模式 | `FETCH_ASSOC`、`FETCH_OBJ`、`FETCH_NUM` |
| `PDO::ATTR_EMULATE_PREPARES` | 模拟预处理 | `true`、`false` |
| `PDO::ATTR_PERSISTENT` | 持久连接 | `true`、`false` |
| `PDO::ATTR_TIMEOUT` | 连接超时 | 整数（秒） |

**示例**：

```php
<?php
declare(strict_types=1);

$options = [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
    PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
    PDO::ATTR_EMULATE_PREPARES => false,
    PDO::ATTR_PERSISTENT => false,
    PDO::ATTR_TIMEOUT => 5
];

$pdo = new PDO('mysql:host=localhost;dbname=test', 'user', 'pass', $options);
```

---

## 连接配置

### 错误模式设置

PDO 支持三种错误模式：

**1. ERRMODE_SILENT（静默模式）**

默认模式，错误不会抛出异常，需要手动检查错误。

**示例**：

```php
<?php
declare(strict_types=1);

$pdo = new PDO('mysql:host=localhost;dbname=test', 'user', 'pass', [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_SILENT
]);

$result = $pdo->query('SELECT * FROM nonexistent_table');
if ($result === false) {
    $error = $pdo->errorInfo();
    echo '错误: ' . $error[2];
}
```

**2. ERRMODE_WARNING（警告模式）**

错误会触发 PHP 警告，但不会抛出异常。

**示例**：

```php
<?php
declare(strict_types=1);

$pdo = new PDO('mysql:host=localhost;dbname=test', 'user', 'pass', [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_WARNING
]);

// 会触发 PHP 警告
$pdo->query('SELECT * FROM nonexistent_table');
```

**3. ERRMODE_EXCEPTION（异常模式）**

错误会抛出 `PDOException` 异常，推荐使用。

**示例**：

```php
<?php
declare(strict_types=1);

try {
    $pdo = new PDO('mysql:host=localhost;dbname=test', 'user', 'pass', [
        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION
    ]);
    $pdo->query('SELECT * FROM nonexistent_table');
} catch (PDOException $e) {
    echo '数据库错误: ' . $e->getMessage();
}
```

### 属性设置

可以使用 `setAttribute()` 方法设置 PDO 属性。

**语法**：`PDO::setAttribute(int $attribute, mixed $value): bool`

**示例**：

```php
<?php
declare(strict_types=1);

$pdo = new PDO('mysql:host=localhost;dbname=test', 'user', 'pass');

// 设置错误模式
$pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);

// 设置默认获取模式
$pdo->setAttribute(PDO::ATTR_DEFAULT_FETCH_MODE, PDO::FETCH_ASSOC);

// 禁用模拟预处理
$pdo->setAttribute(PDO::ATTR_EMULATE_PREPARES, false);
```

### 字符集设置

**方法 1：在 DSN 中设置**

```php
<?php
declare(strict_types=1);

$dsn = 'mysql:host=localhost;dbname=test;charset=utf8mb4';
$pdo = new PDO($dsn, 'user', 'pass');
```

**方法 2：连接后设置**

```php
<?php
declare(strict_types=1);

$pdo = new PDO('mysql:host=localhost;dbname=test', 'user', 'pass');
$pdo->exec('SET NAMES utf8mb4');
```

**推荐使用 DSN 方式**，更简洁且不易遗漏。

### 时区设置

**方法 1：在 DSN 中设置**

```php
<?php
declare(strict_types=1);

$dsn = 'mysql:host=localhost;dbname=test;charset=utf8mb4';
$pdo = new PDO($dsn, 'user', 'pass');
$pdo->exec('SET time_zone = "+08:00"');
```

**方法 2：连接后设置**

```php
<?php
declare(strict_types=1);

$pdo = new PDO('mysql:host=localhost;dbname=test', 'user', 'pass');
$pdo->exec('SET time_zone = "+08:00"');
```

---

## 错误处理

### 异常捕获

使用异常模式时，可以使用 `try-catch` 捕获异常。

**示例**：

```php
<?php
declare(strict_types=1);

try {
    $pdo = new PDO('mysql:host=localhost;dbname=test', 'user', 'pass', [
        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION
    ]);
    
    $stmt = $pdo->query('SELECT * FROM users');
    $users = $stmt->fetchAll();
} catch (PDOException $e) {
    echo '数据库错误: ' . $e->getMessage();
    echo '错误码: ' . $e->getCode();
}
```

### 错误信息获取

**方法 1：使用异常**

```php
<?php
declare(strict_types=1);

try {
    $pdo->query('SELECT * FROM nonexistent_table');
} catch (PDOException $e) {
    echo '错误信息: ' . $e->getMessage();
    echo '错误码: ' . $e->getCode();
    echo 'SQL 状态: ' . $e->errorInfo[0];
}
```

**方法 2：使用 errorInfo()**

```php
<?php
declare(strict_types=1);

$result = $pdo->query('SELECT * FROM nonexistent_table');
if ($result === false) {
    $error = $pdo->errorInfo();
    echo 'SQL 状态: ' . $error[0];
    echo '错误码: ' . $error[1];
    echo '错误信息: ' . $error[2];
}
```

### 错误码获取

**PDOException 错误码**：

- `00000`：成功
- `HY000`：通用错误
- `42000`：语法错误或访问规则冲突
- `42S02`：表不存在
- `42S22`：列不存在

**示例**：

```php
<?php
declare(strict_types=1);

try {
    $pdo->query('SELECT * FROM nonexistent_table');
} catch (PDOException $e) {
    $code = $e->getCode();
    if ($code == '42S02') {
        echo '表不存在';
    } else {
        echo '其他错误: ' . $e->getMessage();
    }
}
```

### 错误处理策略

**推荐策略**：

1. **使用异常模式**：`PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION`
2. **统一错误处理**：创建统一的错误处理函数
3. **记录错误日志**：记录错误信息，便于调试
4. **用户友好提示**：不直接显示数据库错误信息给用户

**示例**：

```php
<?php
declare(strict_types=1);

function handleDatabaseError(PDOException $e): void
{
    // 记录错误日志
    error_log('数据库错误: ' . $e->getMessage());
    
    // 用户友好提示
    if ($_ENV['APP_DEBUG'] ?? false) {
        echo '数据库错误: ' . $e->getMessage();
    } else {
        echo '系统错误，请稍后重试';
    }
}

try {
    $pdo->query('SELECT * FROM users');
} catch (PDOException $e) {
    handleDatabaseError($e);
}
```

---

## 连接管理

### 单例模式

使用单例模式管理数据库连接，避免重复创建连接。

**示例**：

```php
<?php
declare(strict_types=1);

class Database
{
    private static ?PDO $instance = null;
    
    private function __construct()
    {
    }
    
    public static function getInstance(): PDO
    {
        if (self::$instance === null) {
            $dsn = 'mysql:host=localhost;dbname=test;charset=utf8mb4';
            self::$instance = new PDO($dsn, 'user', 'pass', [
                PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
                PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC
            ]);
        }
        
        return self::$instance;
    }
}

// 使用
$pdo = Database::getInstance();
```

### 配置管理

将数据库配置从代码中分离，使用配置文件管理。

**示例**：

```php
<?php
declare(strict_types=1);

// config/database.php
return [
    'host' => $_ENV['DB_HOST'] ?? 'localhost',
    'port' => $_ENV['DB_PORT'] ?? 3306,
    'database' => $_ENV['DB_DATABASE'] ?? 'test',
    'username' => $_ENV['DB_USERNAME'] ?? 'root',
    'password' => $_ENV['DB_PASSWORD'] ?? '',
    'charset' => $_ENV['DB_CHARSET'] ?? 'utf8mb4',
];

// 使用配置
$config = require 'config/database.php';
$dsn = sprintf(
    'mysql:host=%s;port=%d;dbname=%s;charset=%s',
    $config['host'],
    $config['port'],
    $config['database'],
    $config['charset']
);
$pdo = new PDO($dsn, $config['username'], $config['password'], [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
    PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC
]);
```

### 连接池

虽然 PDO 本身不支持连接池，但可以使用持久连接提高性能。

**示例**：

```php
<?php
declare(strict_types=1);

$pdo = new PDO('mysql:host=localhost;dbname=test', 'user', 'pass', [
    PDO::ATTR_PERSISTENT => true  // 持久连接
]);
```

**注意**：持久连接可能导致连接数过多，需要谨慎使用。

---

## 完整示例

### 数据库连接类

```php
<?php
declare(strict_types=1);

class DatabaseConnection
{
    private static ?PDO $instance = null;
    
    private function __construct()
    {
    }
    
    public static function getInstance(): PDO
    {
        if (self::$instance === null) {
            $config = self::getConfig();
            $dsn = sprintf(
                'mysql:host=%s;port=%d;dbname=%s;charset=%s',
                $config['host'],
                $config['port'],
                $config['database'],
                $config['charset']
            );
            
            try {
                self::$instance = new PDO(
                    $dsn,
                    $config['username'],
                    $config['password'],
                    [
                        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
                        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
                        PDO::ATTR_EMULATE_PREPARES => false
                    ]
                );
                
                // 设置时区
                self::$instance->exec('SET time_zone = "+08:00"');
            } catch (PDOException $e) {
                error_log('数据库连接失败: ' . $e->getMessage());
                throw new RuntimeException('数据库连接失败', 0, $e);
            }
        }
        
        return self::$instance;
    }
    
    private static function getConfig(): array
    {
        return [
            'host' => $_ENV['DB_HOST'] ?? 'localhost',
            'port' => (int)($_ENV['DB_PORT'] ?? 3306),
            'database' => $_ENV['DB_DATABASE'] ?? 'test',
            'username' => $_ENV['DB_USERNAME'] ?? 'root',
            'password' => $_ENV['DB_PASSWORD'] ?? '',
            'charset' => $_ENV['DB_CHARSET'] ?? 'utf8mb4',
        ];
    }
}

// 使用
try {
    $pdo = DatabaseConnection::getInstance();
    $stmt = $pdo->query('SELECT * FROM users LIMIT 10');
    $users = $stmt->fetchAll();
    foreach ($users as $user) {
        echo $user['username'] . "\n";
    }
} catch (RuntimeException $e) {
    echo '错误: ' . $e->getMessage();
}
```

---

## 使用场景

### 数据库连接

PDO 用于建立和管理数据库连接。

**示例**：

```php
<?php
declare(strict_types=1);

$pdo = new PDO('mysql:host=localhost;dbname=test', 'user', 'pass');
```

### 数据库操作

PDO 用于执行数据库操作（查询、插入、更新、删除）。

**示例**：

```php
<?php
declare(strict_types=1);

$pdo = new PDO('mysql:host=localhost;dbname=test', 'user', 'pass');
$stmt = $pdo->query('SELECT * FROM users');
$users = $stmt->fetchAll();
```

### 数据访问层

PDO 作为数据访问层的基础，封装数据库操作。

**示例**：

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
    
    public function findAll(): array
    {
        $stmt = $this->pdo->query('SELECT * FROM users');
        return $stmt->fetchAll();
    }
}
```

---

## 注意事项

### 连接安全性

- **密码保护**：不要将密码硬编码在代码中，使用环境变量或配置文件
- **连接加密**：生产环境使用 SSL 连接
- **权限控制**：数据库用户只授予必要的权限

### 错误处理

- **使用异常模式**：`PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION`
- **统一错误处理**：创建统一的错误处理机制
- **记录错误日志**：记录错误信息，便于调试

### 连接池管理

- **避免过多连接**：使用单例模式或连接池
- **及时关闭连接**：不再需要时关闭连接
- **持久连接谨慎使用**：持久连接可能导致连接数过多

### 字符集设置

- **统一字符集**：确保数据库、表、连接的字符集一致
- **使用 utf8mb4**：支持完整的 UTF-8 字符集（包括 emoji）
- **在 DSN 中设置**：在 DSN 中设置字符集，避免遗漏

---

## 常见问题

### 如何连接数据库？

**步骤**：

1. 准备 DSN 字符串
2. 创建 PDO 实例
3. 设置错误模式
4. 处理连接异常

**示例**：

```php
<?php
declare(strict_types=1);

try {
    $dsn = 'mysql:host=localhost;dbname=test;charset=utf8mb4';
    $pdo = new PDO($dsn, 'user', 'pass', [
        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION
    ]);
} catch (PDOException $e) {
    echo '连接失败: ' . $e->getMessage();
}
```

### PDO 和 MySQLi 的区别？

**主要区别**：

- **数据库支持**：PDO 支持多种数据库，MySQLi 仅支持 MySQL
- **代码可移植性**：PDO 代码可移植性更高
- **API 风格**：PDO 仅面向对象，MySQLi 支持面向对象和过程式

### 如何设置错误模式？

**方法**：

```php
<?php
declare(strict_types=1);

// 方法 1：在构造函数中设置
$pdo = new PDO($dsn, $user, $pass, [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION
]);

// 方法 2：使用 setAttribute
$pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
```

### 如何处理连接错误？

**推荐方式**：

```php
<?php
declare(strict_types=1);

try {
    $pdo = new PDO($dsn, $user, $pass, [
        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION
    ]);
} catch (PDOException $e) {
    error_log('数据库连接失败: ' . $e->getMessage());
    // 用户友好提示
    die('系统错误，请稍后重试');
}
```

---

## 最佳实践

### 使用异常模式

始终使用异常模式，便于错误处理和调试。

```php
<?php
declare(strict_types=1);

$pdo = new PDO($dsn, $user, $pass, [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION
]);
```

### 设置合适的字符集

使用 utf8mb4 字符集，支持完整的 UTF-8 字符。

```php
<?php
declare(strict_types=1);

$dsn = 'mysql:host=localhost;dbname=test;charset=utf8mb4';
```

### 实现连接管理

使用单例模式或依赖注入管理数据库连接。

```php
<?php
declare(strict_types=1);

class DatabaseConnection
{
    private static ?PDO $instance = null;
    
    public static function getInstance(): PDO
    {
        if (self::$instance === null) {
            self::$instance = new PDO($dsn, $user, $pass);
        }
        return self::$instance;
    }
}
```

### 处理连接错误

统一处理连接错误，记录日志并提供用户友好提示。

```php
<?php
declare(strict_types=1);

try {
    $pdo = new PDO($dsn, $user, $pass);
} catch (PDOException $e) {
    error_log('数据库连接失败: ' . $e->getMessage());
    throw new RuntimeException('数据库连接失败', 0, $e);
}
```

---

## 练习任务

1. **PDO 连接配置**
   - 创建 PDO 连接
   - 设置错误模式为异常模式
   - 设置默认获取模式为关联数组
   - 测试连接是否成功

2. **错误处理**
   - 使用异常模式捕获连接错误
   - 实现统一的错误处理函数
   - 记录错误日志
   - 提供用户友好提示

3. **连接管理**
   - 实现单例模式的数据库连接类
   - 从配置文件读取数据库配置
   - 实现连接池管理
   - 测试连接复用

4. **字符集配置**
   - 在 DSN 中设置字符集为 utf8mb4
   - 测试中文字符存储和读取
   - 测试 emoji 字符存储和读取
   - 验证字符集设置是否正确

5. **综合应用**
   - 创建一个完整的数据库连接类
   - 实现配置管理
   - 实现错误处理
   - 实现连接管理
   - 编写单元测试

---

**相关章节**：

- [6.2.2 预处理语句与防注入](section-02-prepared-statements.md)
- [6.2.3 CRUD 操作](section-03-crud-operations.md)
- [6.2.4 高级特性](section-04-advanced-features.md)
