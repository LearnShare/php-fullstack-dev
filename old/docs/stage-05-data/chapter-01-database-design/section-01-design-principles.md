# 5.1.1 数据库设计原则

## 概述

良好的数据库设计是构建可靠应用的基础。本节详细介绍为什么需要数据库设计、ER 图（实体关系图）的绘制方法、关系类型，以及数据库设计最佳实践。

## 为什么需要数据库设计

- **数据一致性**：良好的设计确保数据的一致性和完整性
- **性能优化**：合理的表结构和索引设计提升查询性能
- **可扩展性**：良好的设计便于后续扩展和维护
- **减少冗余**：规范化设计减少数据冗余，节省存储空间

## ER 图（实体关系图）

### 基本概念

- **实体（Entity）**：现实世界中的对象，如表
- **属性（Attribute）**：实体的特征，如字段
- **关系（Relationship）**：实体之间的联系

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

## 关系类型

### 一对一（1:1）

一个用户对应一个用户资料。

```sql
CREATE TABLE users (
    id INT PRIMARY KEY,
    username VARCHAR(255)
);

CREATE TABLE user_profiles (
    id INT PRIMARY KEY,
    user_id INT UNIQUE,
    bio TEXT,
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

### 一对多（1:N）

一个用户有多个订单。

```sql
CREATE TABLE users (
    id INT PRIMARY KEY,
    username VARCHAR(255)
);

CREATE TABLE orders (
    id INT PRIMARY KEY,
    user_id INT,
    total DECIMAL(10,2),
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

### 多对多（N:M）

一个用户有多个角色，一个角色有多个用户。

```sql
CREATE TABLE users (
    id INT PRIMARY KEY,
    username VARCHAR(255)
);

CREATE TABLE roles (
    id INT PRIMARY KEY,
    name VARCHAR(255)
);

CREATE TABLE user_roles (
    user_id INT,
    role_id INT,
    PRIMARY KEY (user_id, role_id),
    FOREIGN KEY (user_id) REFERENCES users(id),
    FOREIGN KEY (role_id) REFERENCES roles(id)
);
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

### 4. 字段类型选择

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

- **MySQL Workbench**：官方工具，支持 ER 图设计
- **Navicat**：商业工具，功能强大
- **dbdiagram.io**：在线 ER 图工具
- **PlantUML**：代码生成 ER 图

## 注意事项

1. **命名规范**：统一使用小写下划线命名
2. **主键设计**：优先使用自增整数，分布式系统考虑 UUID
3. **外键约束**：合理使用外键保证数据完整性
4. **字符集**：统一使用 utf8mb4

## 练习

1. 设计一个博客系统的数据库，包含用户、文章、分类、标签、评论等表。

2. 创建一个数据库设计文档，包含 ER 图、表结构、关系说明。

3. 设计一个多租户 SaaS 系统的数据库，考虑数据隔离和性能优化。

4. 分析一个现有的数据库设计，识别可以改进的地方。
