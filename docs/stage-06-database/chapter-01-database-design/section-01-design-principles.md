# 6.1.1 数据库设计原则

## 概述

数据库设计原则是设计高质量数据库的基础。本节介绍数据库设计的基本原则，包括实体关系、完整性约束等，帮助零基础学员掌握数据库设计方法。

**章节类型**：概念性章节

**主要内容**：
- 数据库设计概述
- 设计原则（规范化、完整性、性能、可扩展性）
- 实体关系设计（ER 图）
- 完整性约束（主键、外键、唯一约束、检查约束）
- 命名规范
- 完整示例

## 核心内容

### 数据库设计概述

- 什么是数据库设计
- 设计的目标
- 设计的重要性

### 设计原则

- 规范化原则
- 完整性原则
- 性能原则
- 可扩展性原则

### 实体关系设计

- 实体识别
- 关系识别
- ER 图绘制
- 关系类型（一对一、一对多、多对多）

### 完整性约束

- 主键约束
- 外键约束
- 唯一约束
- 检查约束
- 非空约束

## 基本用法

### 数据库设计示例

```sql
CREATE TABLE users (
    id INT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE posts (
    id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT NOT NULL,
    title VARCHAR(200) NOT NULL,
    content TEXT,
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

## 使用场景

- 新项目数据库设计
- 数据库重构
- 数据库优化
- 系统设计

## 注意事项

- 设计的前瞻性
- 性能考虑
- 扩展性考虑
- 命名规范

## 常见问题

- 如何开始数据库设计？
- 如何识别实体和关系？
- 如何设计完整性约束？
- 设计原则如何平衡？

## 最佳实践

- 遵循设计原则
- 使用 ER 图辅助设计
- 建立完整的约束
- 遵循命名规范
