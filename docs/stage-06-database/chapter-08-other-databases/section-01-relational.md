# 6.8.1 其他关系型数据库

## 概述

除了 MySQL，还有其他关系型数据库系统，如 PostgreSQL、SQLite、MariaDB 等。了解这些数据库的特点和使用场景，有助于选择合适的数据库系统。

本节详细介绍其他关系型数据库的概念、特点、使用场景、PHP 连接方式等，帮助零基础学员了解不同数据库系统的特点和选择方法。

**主要内容**：
- 关系型数据库概述
- PostgreSQL 介绍和使用
- SQLite 介绍和使用
- MariaDB 介绍和使用
- 数据库选择建议
- 完整示例和最佳实践

---

## 特性

- **多样性**：了解不同数据库系统
- **选择灵活性**：根据需求选择合适数据库
- **兼容性**：了解数据库兼容性
- **性能对比**：了解性能差异
- **适用场景**：了解适用场景

---

## 关系型数据库概述

### 什么是关系型数据库

关系型数据库（RDBMS）是基于关系模型的数据库系统，使用 SQL 语言进行数据操作。

**主要特点**：

- **ACID 特性**：保证数据一致性
- **SQL 标准**：使用标准 SQL 语言
- **表结构**：数据存储在表中
- **关系**：表之间可以建立关系

### 常见关系型数据库

常见的关系型数据库系统：

- **MySQL**：最流行的开源数据库
- **PostgreSQL**：功能强大的开源数据库
- **SQLite**：轻量级嵌入式数据库
- **MariaDB**：MySQL 的分支
- **Oracle**：商业数据库
- **SQL Server**：微软的数据库

---

## PostgreSQL

### PostgreSQL 简介

PostgreSQL 是一个功能强大的开源关系型数据库系统，具有高度的 SQL 标准兼容性和丰富的功能。

**主要特点**：

- **功能丰富**：支持复杂数据类型和操作
- **标准兼容**：高度符合 SQL 标准
- **扩展性强**：支持自定义函数和扩展
- **性能优秀**：性能表现优秀

### PostgreSQL 安装

**Ubuntu/Debian**：

```bash
# 安装 PostgreSQL
sudo apt-get update
sudo apt-get install postgresql postgresql-contrib

# 启动服务
sudo systemctl start postgresql
sudo systemctl enable postgresql
```

**CentOS/RHEL**：

```bash
# 安装 PostgreSQL
sudo yum install postgresql-server postgresql-contrib

# 初始化数据库
sudo postgresql-setup initdb

# 启动服务
sudo systemctl start postgresql
sudo systemctl enable postgresql
```

### PHP 连接 PostgreSQL

使用 PDO 连接 PostgreSQL。

**连接示例**：

```php
<?php
declare(strict_types=1);

try {
    $dsn = 'pgsql:host=localhost;port=5432;dbname=mydb';
    $pdo = new PDO($dsn, 'username', 'password', [
        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC
    ]);
    
    echo "连接成功\n";
} catch (PDOException $e) {
    die('连接失败: ' . $e->getMessage());
}
```

### PostgreSQL 基本操作

**创建表**：

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**插入数据**：

```sql
INSERT INTO users (username, email) VALUES ('john', 'john@example.com');
```

**查询数据**：

```sql
SELECT * FROM users WHERE username = 'john';
```

### PostgreSQL 与 MySQL 的区别

**主要区别**：

- **数据类型**：PostgreSQL 支持更多数据类型
- **功能**：PostgreSQL 功能更丰富
- **性能**：各有优势
- **扩展性**：PostgreSQL 扩展性更强

---

## SQLite

### SQLite 简介

SQLite 是一个轻量级的嵌入式数据库，不需要独立的服务器进程，数据存储在单个文件中。

**主要特点**：

- **轻量级**：体积小，资源占用少
- **嵌入式**：不需要独立服务器
- **零配置**：无需配置
- **跨平台**：支持多种平台

### SQLite 使用场景

**适用场景**：

- **小型应用**：小型 Web 应用
- **移动应用**：移动应用数据存储
- **开发测试**：开发和测试环境
- **嵌入式系统**：嵌入式系统

### PHP 连接 SQLite

使用 PDO 连接 SQLite。

**连接示例**：

```php
<?php
declare(strict_types=1);

try {
    $dsn = 'sqlite:/path/to/database.db';
    $pdo = new PDO($dsn, null, null, [
        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC
    ]);
    
    echo "连接成功\n";
} catch (PDOException $e) {
    die('连接失败: ' . $e->getMessage());
}
```

### SQLite 基本操作

**创建表**：

```sql
CREATE TABLE users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    username TEXT UNIQUE NOT NULL,
    email TEXT UNIQUE NOT NULL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

**插入数据**：

```sql
INSERT INTO users (username, email) VALUES ('john', 'john@example.com');
```

**查询数据**：

```sql
SELECT * FROM users WHERE username = 'john';
```

### SQLite 限制

**主要限制**：

- **并发写入**：并发写入性能较差
- **数据类型**：数据类型支持有限
- **功能**：功能相对简单
- **规模**：不适合大规模应用

---

## MariaDB

### MariaDB 简介

MariaDB 是 MySQL 的一个分支，由 MySQL 的原始开发者创建，目标是保持与 MySQL 的兼容性。

**主要特点**：

- **MySQL 兼容**：高度兼容 MySQL
- **开源**：完全开源
- **性能优化**：性能优化
- **功能增强**：功能增强

### MariaDB 与 MySQL 的关系

**关系**：

- **分支**：MariaDB 是 MySQL 的分支
- **兼容性**：高度兼容 MySQL
- **语法**：SQL 语法基本相同
- **迁移**：可以轻松迁移

### PHP 连接 MariaDB

使用 PDO 连接 MariaDB，连接方式与 MySQL 相同。

**连接示例**：

```php
<?php
declare(strict_types=1);

try {
    $dsn = 'mysql:host=localhost;dbname=mydb;charset=utf8mb4';
    $pdo = new PDO($dsn, 'username', 'password', [
        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC
    ]);
    
    echo "连接成功\n";
} catch (PDOException $e) {
    die('连接失败: ' . $e->getMessage());
}
```

---

## 数据库选择建议

### 选择因素

选择数据库时需要考虑的因素：

- **应用规模**：应用规模和用户量
- **性能要求**：性能要求
- **功能需求**：功能需求
- **成本**：成本和预算
- **团队经验**：团队技术栈和经验

### 选择建议

**MySQL**：

- **适用场景**：Web 应用、中小型应用
- **优势**：流行、易用、社区支持好
- **劣势**：功能相对简单

**PostgreSQL**：

- **适用场景**：复杂应用、数据分析
- **优势**：功能丰富、标准兼容
- **劣势**：学习曲线较陡

**SQLite**：

- **适用场景**：小型应用、移动应用
- **优势**：轻量级、零配置
- **劣势**：并发性能差

**MariaDB**：

- **适用场景**：MySQL 替代方案
- **优势**：MySQL 兼容、开源
- **劣势**：生态相对较小

---

## 完整示例

### 数据库抽象层

创建一个数据库抽象层，支持多种数据库。

```php
<?php
declare(strict_types=1);

interface DatabaseInterface
{
    public function connect(): PDO;
    public function query(string $sql, array $params = []): array;
    public function execute(string $sql, array $params = []): int;
}

class MySQLDatabase implements DatabaseInterface
{
    private string $dsn;
    private string $username;
    private string $password;
    private array $options;
    
    public function __construct(string $host, string $dbname, string $username, string $password, array $options = [])
    {
        $this->dsn = "mysql:host={$host};dbname={$dbname};charset=utf8mb4";
        $this->username = $username;
        $this->password = $password;
        $this->options = array_merge([
            PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
            PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC
        ], $options);
    }
    
    public function connect(): PDO
    {
        return new PDO($this->dsn, $this->username, $this->password, $this->options);
    }
    
    public function query(string $sql, array $params = []): array
    {
        $pdo = $this->connect();
        $stmt = $pdo->prepare($sql);
        $stmt->execute($params);
        return $stmt->fetchAll();
    }
    
    public function execute(string $sql, array $params = []): int
    {
        $pdo = $this->connect();
        $stmt = $pdo->prepare($sql);
        $stmt->execute($params);
        return $stmt->rowCount();
    }
}

class PostgreSQLDatabase implements DatabaseInterface
{
    private string $dsn;
    private string $username;
    private string $password;
    private array $options;
    
    public function __construct(string $host, int $port, string $dbname, string $username, string $password, array $options = [])
    {
        $this->dsn = "pgsql:host={$host};port={$port};dbname={$dbname}";
        $this->username = $username;
        $this->password = $password;
        $this->options = array_merge([
            PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
            PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC
        ], $options);
    }
    
    public function connect(): PDO
    {
        return new PDO($this->dsn, $this->username, $this->password, $this->options);
    }
    
    public function query(string $sql, array $params = []): array
    {
        $pdo = $this->connect();
        $stmt = $pdo->prepare($sql);
        $stmt->execute($params);
        return $stmt->fetchAll();
    }
    
    public function execute(string $sql, array $params = []): int
    {
        $pdo = $this->connect();
        $stmt = $pdo->prepare($sql);
        $stmt->execute($params);
        return $stmt->rowCount();
    }
}

class SQLiteDatabase implements DatabaseInterface
{
    private string $dsn;
    private array $options;
    
    public function __construct(string $path, array $options = [])
    {
        $this->dsn = "sqlite:{$path}";
        $this->options = array_merge([
            PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
            PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC
        ], $options);
    }
    
    public function connect(): PDO
    {
        return new PDO($this->dsn, null, null, $this->options);
    }
    
    public function query(string $sql, array $params = []): array
    {
        $pdo = $this->connect();
        $stmt = $pdo->prepare($sql);
        $stmt->execute($params);
        return $stmt->fetchAll();
    }
    
    public function execute(string $sql, array $params = []): int
    {
        $pdo = $this->connect();
        $stmt = $pdo->prepare($sql);
        $stmt->execute($params);
        return $stmt->rowCount();
    }
}

// 使用
$mysql = new MySQLDatabase('localhost', 'mydb', 'user', 'pass');
$users = $mysql->query('SELECT * FROM users WHERE id = ?', [1]);

$pgsql = new PostgreSQLDatabase('localhost', 5432, 'mydb', 'user', 'pass');
$users = $pgsql->query('SELECT * FROM users WHERE id = ?', [1]);

$sqlite = new SQLiteDatabase('/path/to/database.db');
$users = $sqlite->query('SELECT * FROM users WHERE id = ?', [1]);
```

---

## 使用场景

### 数据库选择

根据应用需求选择合适的数据库。

### 数据库迁移

在不同数据库之间迁移数据。

### 多数据库支持

支持多种数据库的应用。

---

## 注意事项

### 兼容性

注意不同数据库的 SQL 语法差异。

### 性能

不同数据库的性能特点不同。

### 功能

不同数据库的功能支持不同。

---

## 常见问题

### 如何选择数据库？

根据应用规模、性能要求、功能需求等因素选择。

### PostgreSQL 和 MySQL 有什么区别？

PostgreSQL 功能更丰富，MySQL 更流行易用。

### SQLite 适合什么场景？

适合小型应用、移动应用、开发测试环境。

---

## 最佳实践

### 了解特点

了解不同数据库的特点和适用场景。

### 统一接口

使用统一的接口访问不同数据库。

### 测试兼容性

在不同数据库上测试应用兼容性。

---

## 练习任务

1. **PostgreSQL 使用**
   - 安装 PostgreSQL
   - 创建数据库和表
   - 使用 PHP 连接 PostgreSQL
   - 执行基本 CRUD 操作

2. **SQLite 使用**
   - 创建 SQLite 数据库
   - 使用 PHP 连接 SQLite
   - 执行基本 CRUD 操作
   - 测试 SQLite 功能

3. **数据库抽象层**
   - 创建数据库接口
   - 实现 MySQL 适配器
   - 实现 PostgreSQL 适配器
   - 实现 SQLite 适配器

4. **数据库对比**
   - 对比不同数据库的特点
   - 测试性能差异
   - 分析适用场景
   - 编写对比报告

5. **综合应用**
   - 创建一个支持多数据库的应用
   - 实现数据库抽象层
   - 测试各种场景
   - 编写最佳实践文档

---

**相关章节**：

- [6.8.2 NoSQL 数据库](section-02-nosql.md)
- [6.2 PDO 入门与高安全模式](../chapter-02-pdo/readme.md)
- [6.1 数据库设计基础](../chapter-01-database-design/readme.md)
