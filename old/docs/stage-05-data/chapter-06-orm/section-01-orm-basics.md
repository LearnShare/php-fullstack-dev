# 5.6.1 ORM 基础

## 概述

ORM（Object-Relational Mapping，对象关系映射）将数据库表映射为对象，简化数据库操作。本节详细介绍什么是 ORM、ORM 优势、ORM vs 原生 SQL，以及 ORM 的选择。

## 什么是 ORM

- **ORM**：Object-Relational Mapping（对象关系映射）
- 将数据库表映射为对象，简化数据库操作
- 提供类型安全、自动关联、查询构建等功能

## ORM 优势

- **类型安全**：使用对象而非 SQL 字符串
- **代码复用**：模型可在多处使用
- **自动关联**：处理表之间的关系
- **迁移管理**：版本化数据库结构

## ORM vs 原生 SQL

| 特性 | ORM | 原生 SQL |
| :--- | :--- | :--- |
| 学习曲线 | 需要学习 ORM API | 需要学习 SQL |
| 类型安全 | 是 | 否 |
| 性能 | 可能稍慢 | 最快 |
| 可维护性 | 高 | 中等 |
| 复杂查询 | 可能受限 | 完全支持 |

## ORM 选择

### Eloquent（Laravel）

- 简单易用，功能强大
- 适合 Laravel 项目
- 文档完善，社区活跃

### Doctrine（Symfony）

- 功能完整，性能优秀
- 适合 Symfony 项目
- 支持复杂查询和高级特性

## 完整示例

```php
<?php
declare(strict_types=1);

// ORM 示例：Eloquent
use App\Models\User;

$user = User::find(1);
$user->email = 'newemail@example.com';
$user->save();

// 原生 SQL 示例
$stmt = $pdo->prepare("SELECT * FROM users WHERE id = ?");
$stmt->execute([1]);
$user = $stmt->fetch();
$stmt = $pdo->prepare("UPDATE users SET email = ? WHERE id = ?");
$stmt->execute(['newemail@example.com', 1]);
```

## 注意事项

1. **性能考虑**：ORM 可能比原生 SQL 慢
2. **复杂查询**：复杂查询可能需要使用原生 SQL
3. **学习成本**：需要学习 ORM API
4. **选择建议**：根据项目需求选择合适的 ORM

## 练习

1. 对比 ORM 和原生 SQL 的性能差异。

2. 实现一个简单的 ORM 框架。

3. 分析不同 ORM 框架的特点和适用场景。

4. 编写一个 ORM 使用指南。
