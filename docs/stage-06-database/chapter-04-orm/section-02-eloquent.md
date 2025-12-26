# 6.4.2 Eloquent 使用

## 概述

Eloquent 是 Laravel 的 ORM 框架。本节介绍 Eloquent 的使用方法，包括模型定义、查询构建、CRUD 操作等，帮助零基础学员掌握 Eloquent。

**章节类型**：工具性章节

**主要内容**：
- Eloquent 概述
- 模型定义
- 查询构建器
- CRUD 操作
- 查询方法（find、where、get、first）
- 批量操作
- 完整示例

## 核心内容

### Eloquent 概述

- Eloquent 的特点
- Active Record 模式
- 模型约定

### 模型定义

- 模型类创建
- 表名约定
- 主键设置
- 时间戳

### 查询构建器

- 链式查询
- 条件查询
- 排序和分页
- 聚合查询

### CRUD 操作

- 创建（create、save）
- 读取（find、get、first）
- 更新（update、save）
- 删除（delete、destroy）

## 基本用法

### Eloquent 示例

```php
<?php
declare(strict_types=1);

// 查询
$user = User::find(1);
$users = User::where('status', 'active')->get();

// 创建
$user = User::create([
    'name' => 'John',
    'email' => 'john@example.com'
]);

// 更新
$user->update(['name' => 'Jane']);

// 删除
$user->delete();
```

## 使用场景

- Laravel 应用
- 快速开发
- 关系处理
- 数据操作

## 注意事项

- 模型约定
- 查询性能
- N+1 问题
- 批量操作

## 常见问题

- 如何定义 Eloquent 模型？
- 如何进行查询？
- 如何处理 N+1 问题？
- Eloquent 的性能优化？

## 最佳实践

- 遵循模型约定
- 使用查询优化
- 处理 N+1 问题
- 使用批量操作
