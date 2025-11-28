# 5.1 数据库设计基础

## 目标

- 理解数据库设计的基本原则和范式理论。
- 掌握 ER 图（实体关系图）的绘制方法。
- 熟悉一、二、三范式的应用场景。
- 了解反范式的适用场景与权衡。
- 掌握 MySQL 字符集与排序规则的选择。

## 为什么需要数据库设计

- **数据一致性**：良好的设计确保数据的一致性和完整性。
- **性能优化**：合理的表结构和索引设计提升查询性能。
- **可扩展性**：良好的设计便于后续扩展和维护。
- **减少冗余**：规范化设计减少数据冗余，节省存储空间。

## ER 图（实体关系图）

### 基本概念

- **实体（Entity）**：现实世界中的对象，如表。
- **属性（Attribute）**：实体的特征，如字段。
- **关系（Relationship）**：实体之间的联系。

### ER 图示例

```
┌─────────────┐         ┌─────────────┐
│   用户      │         │   订单      │
├─────────────┤         ├─────────────┤
│ id (PK)     │         │ id (PK)     │
│ username    │   1    │ user_id (FK)│
│ email       │◄───N───│ total       │
│ password    │         │ status      │
└─────────────┘         └─────────────┘
                              │
                              │ 1
                              │
                              ▼
                        ┌─────────────┐
                        │  订单项     │
                        ├─────────────┤
                        │ id (PK)     │
                        │ order_id(FK)│
                        │ product_id  │
                        │ quantity    │
                        │ price       │
                        └─────────────┘
```

### 关系类型

- **一对一（1:1）**：一个用户对应一个用户资料。
- **一对多（1:N）**：一个用户有多个订单。
- **多对多（N:M）**：一个用户有多个角色，一个角色有多个用户。

## 数据库范式

### 第一范式（1NF）

**规则**：每个字段都是原子值，不可再分。

**示例：**

```sql
-- 不符合 1NF
CREATE TABLE users (
    id INT PRIMARY KEY,
    name VARCHAR(255),
    address VARCHAR(255)  -- 包含省、市、区，不是原子值
);

-- 符合 1NF
CREATE TABLE users (
    id INT PRIMARY KEY,
    name VARCHAR(255),
    province VARCHAR(50),
    city VARCHAR(50),
    district VARCHAR(50)
);
```

### 第二范式（2NF）

**规则**：在 1NF 基础上，非主键字段完全依赖于主键。

**示例：**

```sql
-- 不符合 2NF
CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    product_name VARCHAR(255),  -- 依赖于 product_id，而非主键
    quantity INT,
    price DECIMAL(10,2),
    PRIMARY KEY (order_id, product_id)
);

-- 符合 2NF
CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    quantity INT,
    price DECIMAL(10,2),
    PRIMARY KEY (order_id, product_id),
    FOREIGN KEY (product_id) REFERENCES products(id)
);

CREATE TABLE products (
    id INT PRIMARY KEY,
    name VARCHAR(255)
);
```

### 第三范式（3NF）

**规则**：在 2NF 基础上，非主键字段不依赖于其他非主键字段。

**示例：**

```sql
-- 不符合 3NF
CREATE TABLE orders (
    id INT PRIMARY KEY,
    user_id INT,
    user_name VARCHAR(255),  -- 依赖于 user_id，而非主键
    user_email VARCHAR(255),  -- 依赖于 user_id，而非主键
    total DECIMAL(10,2)
);

-- 符合 3NF
CREATE TABLE orders (
    id INT PRIMARY KEY,
    user_id INT,
    total DECIMAL(10,2),
    FOREIGN KEY (user_id) REFERENCES users(id)
);

CREATE TABLE users (
    id INT PRIMARY KEY,
    name VARCHAR(255),
    email VARCHAR(255)
);
```

## 反范式设计

### 何时使用反范式

- **性能优化**：减少 JOIN 操作，提升查询速度。
- **读多写少**：频繁读取但很少更新的场景。
- **数据冗余可接受**：用空间换时间。

### 反范式示例

```sql
-- 反范式：在订单表中冗余用户信息
CREATE TABLE orders (
    id INT PRIMARY KEY,
    user_id INT,
    user_name VARCHAR(255),  -- 冗余字段
    user_email VARCHAR(255),  -- 冗余字段
    total DECIMAL(10,2),
    created_at TIMESTAMP,
    INDEX idx_user_id (user_id)
);

-- 优点：查询订单时无需 JOIN users 表
-- 缺点：用户信息更新时需要同步更新所有订单
```

### 反范式权衡

| 场景           | 范式设计                     | 反范式设计                   |
| :------------- | :--------------------------- | :--------------------------- |
| 查询性能       | 需要 JOIN，较慢              | 直接查询，较快               |
| 存储空间       | 节省空间                     | 占用更多空间                 |
| 数据一致性     | 容易保证                     | 需要同步更新                 |
| 更新复杂度     | 简单                         | 复杂（需要更新多处）         |
| 适用场景       | 写多读少、数据一致性要求高   | 读多写少、性能要求高         |

## 字符集与排序规则

### utf8 vs utf8mb4

- **utf8**：MySQL 的 utf8 实际上是 utf8mb3，只支持最多 3 字节的字符。
- **utf8mb4**：完整的 UTF-8 实现，支持 4 字节字符（如 Emoji）。

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

| 排序规则                | 说明                           | 使用场景           |
| :---------------------- | :----------------------------- | :----------------- |
| `utf8mb4_unicode_ci`    | 基于 Unicode 标准，支持多语言  | 国际化应用（推荐） |
| `utf8mb4_general_ci`    | 简化排序，性能稍好             | 性能要求高的场景   |
| `utf8mb4_bin`           | 二进制排序，区分大小写         | 需要精确匹配       |
| `utf8mb4_0900_ai_ci`    | MySQL 8.0+ 默认，Unicode 9.0   | MySQL 8.0+（推荐） |

```sql
-- 示例：不同排序规则的影响
CREATE TABLE test_unicode (
    name VARCHAR(255) COLLATE utf8mb4_unicode_ci
);

CREATE TABLE test_bin (
    name VARCHAR(255) COLLATE utf8mb4_bin
);

-- utf8mb4_unicode_ci: 'A' = 'a' (不区分大小写)
-- utf8mb4_bin: 'A' ≠ 'a' (区分大小写)
```

## 数据库设计最佳实践

### 1. 命名规范

```sql
-- 表名：复数形式，小写，下划线分隔
CREATE TABLE users (...);
CREATE TABLE order_items (...);

-- 字段名：小写，下划线分隔
CREATE TABLE users (
    id INT PRIMARY KEY,
    user_name VARCHAR(255),
    created_at TIMESTAMP
);

-- 索引名：表名_字段名_idx
CREATE INDEX users_email_idx ON users(email);
```

### 2. 主键设计

```sql
-- 推荐：自增整数主键
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    ...
);

-- 或使用 UUID（分布式系统）
CREATE TABLE users (
    id CHAR(36) PRIMARY KEY DEFAULT (UUID()),
    ...
);
```

### 3. 外键约束

```sql
CREATE TABLE orders (
    id INT PRIMARY KEY,
    user_id INT,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);

-- 外键选项：
-- ON DELETE CASCADE: 删除用户时自动删除订单
-- ON DELETE SET NULL: 删除用户时将 user_id 设为 NULL
-- ON DELETE RESTRICT: 有订单时禁止删除用户（默认）
```

### 4. 索引设计

```sql
-- 主键索引（自动创建）
PRIMARY KEY (id)

-- 唯一索引
UNIQUE KEY uk_email (email)

-- 普通索引
INDEX idx_user_id (user_id)

-- 复合索引
INDEX idx_user_status (user_id, status)

-- 全文索引
FULLTEXT INDEX ft_content (content)
```

### 5. 字段类型选择

```sql
-- 整数类型
TINYINT    -- -128 到 127
SMALLINT   -- -32768 到 32767
INT        -- -2147483648 到 2147483647
BIGINT     -- 更大的整数

-- 字符串类型
VARCHAR(255)  -- 可变长度字符串
CHAR(10)      -- 固定长度字符串
TEXT          -- 长文本

-- 日期时间类型
DATE          -- 日期
TIME          -- 时间
DATETIME      -- 日期时间
TIMESTAMP     -- 时间戳（自动更新）

-- 数值类型
DECIMAL(10,2) -- 精确小数
FLOAT         -- 单精度浮点数
DOUBLE        -- 双精度浮点数
```

## 完整设计示例

### 电商系统数据库设计

```sql
-- 用户表
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    INDEX idx_email (email)
) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- 商品表
CREATE TABLE products (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    price DECIMAL(10,2) NOT NULL,
    stock INT DEFAULT 0,
    category_id INT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (category_id) REFERENCES categories(id),
    INDEX idx_category (category_id),
    INDEX idx_price (price)
) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- 订单表
CREATE TABLE orders (
    id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT NOT NULL,
    total DECIMAL(10,2) NOT NULL,
    status ENUM('pending', 'paid', 'shipped', 'completed', 'cancelled') DEFAULT 'pending',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE RESTRICT,
    INDEX idx_user_id (user_id),
    INDEX idx_status (status),
    INDEX idx_created_at (created_at)
) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- 订单项表
CREATE TABLE order_items (
    id INT AUTO_INCREMENT PRIMARY KEY,
    order_id INT NOT NULL,
    product_id INT NOT NULL,
    quantity INT NOT NULL,
    price DECIMAL(10,2) NOT NULL,
    FOREIGN KEY (order_id) REFERENCES orders(id) ON DELETE CASCADE,
    FOREIGN KEY (product_id) REFERENCES products(id),
    UNIQUE KEY uk_order_product (order_id, product_id)
) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

## 设计工具

### 推荐工具

- **MySQL Workbench**：官方工具，支持 ER 图设计。
- **Navicat**：商业工具，功能强大。
- **dbdiagram.io**：在线 ER 图工具。
- **PlantUML**：代码生成 ER 图。

## 练习

1. 设计一个博客系统的数据库，包含用户、文章、分类、标签、评论等表。

2. 分析一个现有的数据库设计，识别不符合范式的地方，并提出改进方案。

3. 设计一个多租户 SaaS 系统的数据库，考虑数据隔离和性能优化。

4. 创建一个数据库设计文档，包含 ER 图、表结构、索引设计等。

5. 比较范式设计和反范式设计在查询性能上的差异，编写测试用例。

6. 设计一个支持软删除的数据库结构，考虑查询性能和数据恢复需求。
