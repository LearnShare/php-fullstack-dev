# 5.1.3 索引设计

## 概述

索引是提升数据库查询性能的关键。本节详细介绍索引基础、索引类型、索引设计原则、索引优化，以及字符集与排序规则的选择。

## 索引基础

### 什么是索引

索引是数据库中用于快速查找数据的数据结构，类似于书籍的目录。

### 索引的作用

- **提升查询速度**：快速定位数据
- **加速排序**：ORDER BY 操作更快
- **加速连接**：JOIN 操作更快
- **保证唯一性**：唯一索引保证数据唯一

## 索引类型

### 主键索引（PRIMARY KEY）

```sql
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,  -- 自动创建主键索引
    username VARCHAR(255)
);
```

### 唯一索引（UNIQUE）

```sql
CREATE TABLE users (
    id INT PRIMARY KEY,
    email VARCHAR(255) UNIQUE,  -- 唯一索引
    username VARCHAR(255)
);

-- 或单独创建
CREATE UNIQUE INDEX uk_email ON users(email);
```

### 普通索引（INDEX）

```sql
CREATE TABLE users (
    id INT PRIMARY KEY,
    username VARCHAR(255),
    INDEX idx_username (username)  -- 普通索引
);

-- 或单独创建
CREATE INDEX idx_username ON users(username);
```

### 复合索引

```sql
CREATE TABLE orders (
    id INT PRIMARY KEY,
    user_id INT,
    status VARCHAR(50),
    created_at TIMESTAMP,
    INDEX idx_user_status (user_id, status)  -- 复合索引
);
```

### 全文索引（FULLTEXT）

```sql
CREATE TABLE articles (
    id INT PRIMARY KEY,
    title VARCHAR(255),
    content TEXT,
    FULLTEXT INDEX ft_content (title, content)  -- 全文索引
);
```

## 索引设计原则

### 1. 选择合适的字段

- **WHERE 子句**：经常用于查询条件的字段
- **JOIN 子句**：用于连接的字段
- **ORDER BY 子句**：用于排序的字段

### 2. 避免过多索引

- 索引会占用存储空间
- 索引会降低写入性能
- 只创建必要的索引

### 3. 复合索引顺序

```sql
-- 复合索引遵循最左前缀原则
CREATE INDEX idx_user_status_created ON orders(user_id, status, created_at);

-- 可以使用索引的查询：
-- WHERE user_id = 1
-- WHERE user_id = 1 AND status = 'active'
-- WHERE user_id = 1 AND status = 'active' AND created_at > '2024-01-01'

-- 不能使用索引的查询：
-- WHERE status = 'active'  -- 没有 user_id
-- WHERE created_at > '2024-01-01'  -- 没有 user_id 和 status
```

## 字符集与排序规则

### utf8 vs utf8mb4

- **utf8**：MySQL 的 utf8 实际上是 utf8mb3，只支持最多 3 字节的字符
- **utf8mb4**：完整的 UTF-8 实现，支持 4 字节字符（如 Emoji）

```sql
-- 使用 utf8mb4（推荐）
CREATE TABLE users (
    id INT PRIMARY KEY,
    name VARCHAR(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci,
    bio TEXT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci
) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- 或设置数据库默认字符集
CREATE DATABASE myapp CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

### 排序规则（Collation）

| 排序规则 | 说明 | 使用场景 |
| :--- | :--- | :--- |
| `utf8mb4_unicode_ci` | 基于 Unicode 标准，支持多语言 | 国际化应用（推荐） |
| `utf8mb4_general_ci` | 简化排序，性能稍好 | 性能要求高的场景 |
| `utf8mb4_bin` | 二进制排序，区分大小写 | 需要精确匹配 |
| `utf8mb4_0900_ai_ci` | MySQL 8.0+ 默认，Unicode 9.0 | MySQL 8.0+（推荐） |

## 索引优化

### 查看索引使用情况

```sql
-- 查看表的所有索引
SHOW INDEX FROM users;

-- 分析查询执行计划
EXPLAIN SELECT * FROM users WHERE email = 'alice@example.com';
```

### 索引维护

```sql
-- 重建索引
ALTER TABLE users DROP INDEX idx_email;
ALTER TABLE users ADD INDEX idx_email (email);

-- 优化表（重建索引和统计信息）
OPTIMIZE TABLE users;
```

## 完整示例

```sql
-- 用户表索引设计
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    status ENUM('active', 'inactive') DEFAULT 'active',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    INDEX idx_email (email),
    INDEX idx_status (status),
    INDEX idx_created_at (created_at)
) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

## 注意事项

1. **索引选择**：根据查询模式选择合适的索引
2. **复合索引**：遵循最左前缀原则
3. **索引维护**：定期优化表和索引
4. **字符集**：统一使用 utf8mb4

## 练习

1. 为一个现有的表设计合适的索引，提升查询性能。

2. 分析查询执行计划，优化索引设计。

3. 创建一个包含多种索引类型的表结构。

4. 实现索引的创建、删除和维护操作。
