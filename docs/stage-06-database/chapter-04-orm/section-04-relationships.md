# 6.4.4 关系映射

## 概述

关系映射是 ORM 处理数据库关系的核心功能。本节介绍关系类型、关系定义、关系查询等，帮助零基础学员掌握关系映射技术。

**章节类型**：语法性章节

**主要内容**：
- 关系映射概述
- 一对一关系（hasOne、belongsTo）
- 一对多关系（hasMany、belongsTo）
- 多对多关系（belongsToMany）
- 关系查询（Eager Loading）
- 关系操作
- 完整示例

## 核心内容

### 关系映射概述

- 关系类型
- 关系定义
- 关系查询
- 关系操作

### 一对一关系

- hasOne 关系
- belongsTo 关系
- 关系定义
- 关系查询

### 一对多关系

- hasMany 关系
- belongsTo 关系
- 关系定义
- 关系查询

### 多对多关系

- belongsToMany 关系
- 中间表
- 关系定义
- 关系操作

## 基本用法

### 关系映射示例

```php
<?php
declare(strict_types=1);

// 模型定义
class User extends Model {
    public function posts(): HasMany {
        return $this->hasMany(Post::class);
    }
}

class Post extends Model {
    public function user(): BelongsTo {
        return $this->belongsTo(User::class);
    }
}

// 关系查询
$user = User::with('posts')->find(1);
$posts = $user->posts;
```

## 使用场景

- 复杂数据关系
- 关联查询
- 数据操作
- 关系维护

## 注意事项

- N+1 问题
- 预加载（Eager Loading）
- 关系性能
- 关系维护

## 常见问题

- 如何定义关系？
- 一对一和一对多的区别？
- 如何处理多对多关系？
- 如何避免 N+1 问题？

## 最佳实践

- 使用预加载避免 N+1
- 合理定义关系
- 优化关系查询
- 维护关系一致性
