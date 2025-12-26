# 6.1.2 范式与反范式

## 概述

范式理论是数据库规范化的基础。本节介绍数据库范式的概念、各级范式的特点，以及反范式设计的应用，帮助零基础学员理解范式理论。

**章节类型**：概念性章节

**主要内容**：
- 范式概念
- 第一范式（1NF）
- 第二范式（2NF）
- 第三范式（3NF）
- 反范式设计
- 范式与性能的平衡
- 完整示例

## 核心内容

### 范式概念

- 什么是范式
- 规范化的目的
- 范式级别

### 第一范式（1NF）

- 1NF 的要求
- 原子性
- 消除重复组

### 第二范式（2NF）

- 2NF 的要求
- 部分函数依赖
- 消除部分依赖

### 第三范式（3NF）

- 3NF 的要求
- 传递依赖
- 消除传递依赖

### 反范式设计

- 什么是反范式
- 反范式的场景
- 性能优化
- 数据冗余

## 基本用法

### 范式化示例

```sql
-- 非范式化
CREATE TABLE orders (
    id INT PRIMARY KEY,
    customer_name VARCHAR(100),
    customer_email VARCHAR(100),
    product_name VARCHAR(100),
    product_price DECIMAL(10,2)
);

-- 范式化
CREATE TABLE customers (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100)
);

CREATE TABLE products (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    price DECIMAL(10,2)
);

CREATE TABLE orders (
    id INT PRIMARY KEY,
    customer_id INT,
    product_id INT,
    FOREIGN KEY (customer_id) REFERENCES customers(id),
    FOREIGN KEY (product_id) REFERENCES products(id)
);
```

## 使用场景

- 数据库设计
- 数据规范化
- 性能优化
- 数据一致性

## 注意事项

- 范式与性能的平衡
- 过度规范化
- 反范式的风险
- 数据一致性

## 常见问题

- 什么是范式？
- 如何判断范式级别？
- 何时使用反范式？
- 范式与性能如何平衡？

## 最佳实践

- 理解范式理论
- 根据场景选择范式级别
- 在范式和性能间平衡
- 谨慎使用反范式
