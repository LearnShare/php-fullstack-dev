# 6.2.3 CRUD 操作

## 概述

CRUD 操作是数据库操作的基础。本节介绍使用 PDO 进行创建（Create）、读取（Read）、更新（Update）、删除（Delete）操作的方法，帮助零基础学员掌握完整的数据库操作。

**章节类型**：实践性章节

**主要内容**：
- CRUD 操作概述
- 创建（INSERT）
- 读取（SELECT）
- 更新（UPDATE）
- 删除（DELETE）
- 结果集处理
- 完整示例

## 核心内容

### 创建操作（INSERT）

- INSERT 语句
- 单条插入
- 批量插入
- 获取插入 ID

### 读取操作（SELECT）

- SELECT 语句
- 单条查询
- 多条查询
- 条件查询

### 更新操作（UPDATE）

- UPDATE 语句
- 单条更新
- 批量更新
- 影响行数

### 删除操作（DELETE）

- DELETE 语句
- 单条删除
- 批量删除
- 软删除

## 基本用法

### CRUD 操作示例

```php
<?php
declare(strict_types=1);

// 创建
$stmt = $pdo->prepare('INSERT INTO users (name, email) VALUES (:name, :email)');
$stmt->execute(['name' => 'John', 'email' => 'john@example.com']);
$id = $pdo->lastInsertId();

// 读取
$stmt = $pdo->prepare('SELECT * FROM users WHERE id = :id');
$stmt->execute(['id' => $id]);
$user = $stmt->fetch();

// 更新
$stmt = $pdo->prepare('UPDATE users SET name = :name WHERE id = :id');
$stmt->execute(['name' => 'Jane', 'id' => $id]);

// 删除
$stmt = $pdo->prepare('DELETE FROM users WHERE id = :id');
$stmt->execute(['id' => $id]);
```

## 使用场景

- 数据管理
- 用户管理
- 内容管理
- 所有数据库操作

## 注意事项

- 使用预处理语句
- 错误处理
- 事务管理
- 性能优化

## 常见问题

- 如何插入数据？
- 如何查询数据？
- 如何更新数据？
- 如何删除数据？

## 最佳实践

- 使用预处理语句
- 实现错误处理
- 使用事务管理
- 优化查询性能
