# 7.6.2 实体与值对象

## 概述

实体和值对象是 DDD 中的核心概念。本节介绍实体和值对象的概念、设计方法、区别等，帮助零基础学员掌握 DDD 的基础建模。

**章节类型**：概念性章节

**主要内容**：
- 实体概念
- 值对象概念
- 实体设计（唯一标识、生命周期）
- 值对象设计（不可变性、值相等）
- 实体 vs 值对象
- 选择原则
- 完整示例

## 核心内容

### 实体概念

- 什么是实体
- 唯一标识
- 生命周期
- 可变性

### 值对象概念

- 什么是值对象
- 值相等
- 不可变性
- 无标识

### 实体设计

- 标识设计
- 属性设计
- 行为设计
- 实体方法

### 值对象设计

- 值对象属性
- 不可变设计
- 值对象方法
- 值对象验证

## 基本用法

### 实体和值对象示例

```php
<?php
declare(strict_types=1);

// 实体：有唯一标识
class User {
    public function __construct(
        private UserId $id, // 唯一标识
        private Email $email // 值对象
    ) {}
}

// 值对象：通过值相等
class Email {
    public function __construct(
        private readonly string $value
    ) {
        $this->validate();
    }
    
    public function equals(Email $other): bool {
        return $this->value === $other->value;
    }
}
```

## 使用场景

- 领域建模
- 业务对象设计
- 数据建模
- 业务逻辑封装

## 注意事项

- 实体和值对象的区别
- 选择原则
- 不可变性
- 性能考虑

## 常见问题

- 实体和值对象的区别？
- 如何设计实体？
- 如何设计值对象？
- 何时使用实体或值对象？

## 最佳实践

- 明确实体标识
- 值对象不可变
- 封装业务逻辑
- 保持对象简单
