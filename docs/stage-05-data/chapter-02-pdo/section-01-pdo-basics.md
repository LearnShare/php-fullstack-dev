# 5.2.1 PDO 基础与连接

## 概述

PDO（PHP Data Objects）是 PHP 统一的数据库访问接口。本节详细介绍什么是 PDO、DSN 格式、连接选项配置、错误处理模式，以及完整的连接示例。

## 什么是 PDO

- **PDO**：PHP Data Objects，PHP 数据对象
- 统一的数据库访问接口，支持多种数据库
- 提供预处理语句，有效防止 SQL 注入

## DSN 格式

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

## 连接选项详解

| 选项 | 说明 | 推荐值 |
| :--- | :--- | :--- |
| `PDO::ATTR_ERRMODE` | 错误处理模式 | `PDO::ERRMODE_EXCEPTION` |
| `PDO::ATTR_DEFAULT_FETCH_MODE` | 默认获取模式 | `PDO::FETCH_ASSOC` |
| `PDO::ATTR_EMULATE_PREPARES` | 是否模拟预处理 | `false`（使用原生预处理） |
| `PDO::ATTR_PERSISTENT` | 持久连接 | `false`（生产环境谨慎使用） |
| `PDO::ATTR_TIMEOUT` | 连接超时（秒） | `5` |

## 错误处理模式

```php
<?php
declare(strict_types=1);

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

```php
<?php
declare(strict_types=1);

class DatabaseConnection
{
    private PDO $pdo;

    public function __construct(string $dsn, string $user, string $pass)
    {
        $options = [
            PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
            PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
            PDO::ATTR_EMULATE_PREPARES => false,
        ];

        $this->pdo = new PDO($dsn, $user, $pass, $options);
    }

    public function getPdo(): PDO
    {
        return $this->pdo;
    }
}
```

## 注意事项

1. **安全配置**：使用 `PDO::ATTR_EMULATE_PREPARES => false`
2. **错误处理**：使用异常模式便于错误处理
3. **连接管理**：合理使用持久连接
4. **字符集**：明确指定字符集为 utf8mb4

## 练习

1. 创建一个 PDO 连接类，封装连接配置和错误处理。

2. 实现不同数据库的连接配置（MySQL、PostgreSQL、SQLite）。

3. 创建一个数据库连接池管理类。
