# 6.3.3 锁机制

## 概述

锁机制是数据库并发控制的基础。本节介绍锁的概念、类型、使用场景等，帮助零基础学员掌握锁的使用方法。

**章节类型**：概念性章节

**主要内容**：
- 锁的概念
- 锁的类型（共享锁、排他锁）
- 锁的粒度（行锁、表锁）
- 锁的使用（SELECT ... FOR UPDATE、SELECT ... LOCK IN SHARE MODE）
- 死锁处理
- 锁超时
- 完整示例

## 核心内容

### 锁的概念

- 什么是锁
- 锁的作用
- 锁的分类

### 锁的类型

- 共享锁（S Lock）
- 排他锁（X Lock）
- 意向锁
- 锁的兼容性

### 锁的粒度

- 行锁（Row Lock）
- 表锁（Table Lock）
- 页锁（Page Lock）
- 粒度选择

### 死锁处理

- 死锁概念
- 死锁检测
- 死锁避免
- 死锁解决

## 基本用法

### 锁使用示例

```php
<?php
declare(strict_types=1);

// 排他锁
$stmt = $pdo->prepare('SELECT * FROM users WHERE id = ? FOR UPDATE');
$stmt->execute([$id]);
$user = $stmt->fetch();

// 更新操作
$stmt = $pdo->prepare('UPDATE users SET balance = balance - ? WHERE id = ?');
$stmt->execute([$amount, $id]);
```

## 使用场景

- 并发更新
- 数据一致性
- 防止并发冲突
- 关键操作保护

## 注意事项

- 锁的范围
- 死锁风险
- 性能影响
- 锁超时设置

## 常见问题

- 什么是锁？
- 共享锁和排他锁的区别？
- 如何避免死锁？
- 锁对性能的影响？

## 最佳实践

- 最小化锁的范围
- 避免长时间持锁
- 实现死锁检测
- 设置锁超时
