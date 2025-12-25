# 5.1.2 范式与反范式

## 概述

数据库范式是规范化设计的理论基础。本节详细介绍第一范式（1NF）、第二范式（2NF）、第三范式（3NF）、反范式设计，以及范式与反范式的权衡。

## 第一范式（1NF）

### 规则

每个字段都是原子值，不可再分。

### 示例

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

## 第二范式（2NF）

### 规则

在 1NF 基础上，非主键字段完全依赖于主键。

### 示例

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

## 第三范式（3NF）

### 规则

在 2NF 基础上，非主键字段不依赖于其他非主键字段。

### 示例

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

- **性能优化**：减少 JOIN 操作，提升查询速度
- **读多写少**：频繁读取但很少更新的场景
- **数据冗余可接受**：用空间换时间

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

## 范式与反范式权衡

| 场景 | 范式设计 | 反范式设计 |
| :--- | :--- | :--- |
| 查询性能 | 需要 JOIN，较慢 | 直接查询，较快 |
| 存储空间 | 节省空间 | 占用更多空间 |
| 数据一致性 | 容易保证 | 需要同步更新 |
| 更新复杂度 | 简单 | 复杂（需要更新多处） |
| 适用场景 | 写多读少、数据一致性要求高 | 读多写少、性能要求高 |

## 注意事项

1. **优先范式**：默认使用范式设计，保证数据一致性
2. **性能优化**：在性能瓶颈时考虑反范式
3. **同步更新**：反范式设计需要同步更新机制
4. **权衡取舍**：根据实际场景选择合适的设计

## 练习

1. 分析一个现有的数据库设计，识别不符合范式的地方。

2. 比较范式设计和反范式设计在查询性能上的差异。

3. 设计一个支持反范式的数据库结构，并实现同步更新机制。

4. 创建一个数据库设计，在范式与反范式之间找到平衡。
