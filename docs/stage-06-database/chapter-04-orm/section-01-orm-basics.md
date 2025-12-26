# 6.4.1 ORM 基础

## 概述

ORM（对象关系映射）简化了数据库操作。本节介绍 ORM 的概念、优势、框架对比等，帮助零基础学员理解 ORM 的价值。

**章节类型**：概念性章节

**主要内容**：
- ORM 概念
- ORM 的优势
- ORM 的劣势
- ORM 框架对比（Eloquent、Doctrine、Propel）
- Active Record 模式
- Data Mapper 模式
- 完整示例

## 核心内容

### ORM 概念

- 什么是 ORM
- ORM 的作用
- ORM 的工作原理

### ORM 的优势

- 开发效率
- 代码可读性
- 数据库抽象
- 关系处理

### ORM 的劣势

- 性能开销
- 学习曲线
- 灵活性限制
- 复杂查询

### ORM 框架

- Eloquent（Laravel）
- Doctrine（Symfony）
- Propel
- 框架选择

## 基本用法

### ORM 示例

```php
<?php
declare(strict_types=1);

// 使用 ORM 查询
$user = User::find(1);

// 使用 ORM 创建
$user = new User();
$user->name = 'John';
$user->email = 'john@example.com';
$user->save();
```

## 使用场景

- 快速开发
- 关系处理
- 代码维护
- 团队协作

## 注意事项

- 性能考虑
- 复杂查询
- 学习成本
- 框架选择

## 常见问题

- 什么是 ORM？
- ORM 的优势和劣势？
- 如何选择 ORM 框架？
- ORM 的性能影响？

## 最佳实践

- 理解 ORM 的适用场景
- 选择合适的 ORM 框架
- 注意性能优化
- 结合原生 SQL 使用
