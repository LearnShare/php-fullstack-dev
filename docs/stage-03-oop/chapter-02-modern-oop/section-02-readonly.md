# 3.2.2 Readonly 属性

## 概述

Readonly 属性是 PHP 8.1 引入的特性，用于创建不可变对象。本节介绍 Readonly 属性基础、Readonly 类、与克隆的关系、最佳实践（值对象、DTO、配置对象）等内容。

**章节类型**：语法性章节

**主要内容**：
- Readonly 属性的定义和使用
- Readonly 类的定义和使用
- 与克隆的关系
- 初始化规则
- 使用场景和最佳实践
- 与传统写法的对比

## 特性

- **不可变性**：属性只能初始化一次
- **类型安全**：提高代码安全性
- **简化代码**：减少样板代码

## 语法/定义

### Readonly 属性

**语法**：`public readonly Type $property;`
**要求**：PHP 8.1+
**特点**：只能在初始化时赋值

### Readonly 类

**语法**：`readonly class ClassName { ... }`
**要求**：PHP 8.2+
**特点**：所有属性默认 readonly

## 基本用法

### Readonly 属性示例

```php
<?php
declare(strict_types=1);

class User
{
    public function __construct(
        public readonly string $name,
        public readonly int $age
    ) {}
}
```

### Readonly 类示例

```php
<?php
declare(strict_types=1);

readonly class User
{
    public function __construct(
        public string $name,
        public int $age
    ) {}
}
```

### 与克隆的关系

```php
<?php
declare(strict_types=1);

$user1 = new User("John", 25);
$user2 = clone $user1;  // 可以克隆
// $user2->name = "Jane";  // 错误：不能修改 readonly 属性
```

## 使用场景

- **值对象**：不可变的值对象
- **DTO**：数据传输对象
- **配置对象**：配置信息对象
- **不可变对象**：需要不可变性的场景

## 注意事项

- **初始化**：必须在构造器中初始化
- **克隆**：可以克隆，但克隆后仍不可修改
- **类型要求**：属性必须有类型声明

## 常见问题

### 问题 1：修改 readonly 属性
- **原因**：尝试修改 readonly 属性
- **解决**：创建新对象而不是修改现有对象

### 问题 2：初始化错误
- **原因**：未在构造器中初始化
- **解决**：在构造器中初始化所有 readonly 属性

## 最佳实践

- 使用 readonly 创建不可变对象
- 值对象使用 readonly 类
- 理解不可变性的优势
- 注意初始化和克隆的规则
