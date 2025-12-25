# 3.1.4 对象克隆与引用

## 概述

理解对象克隆和引用是掌握面向对象编程的重要部分。本节介绍对象引用传递、`clone` 关键字、`__clone` 方法、深拷贝、`clone with` 语法（PHP 8.5+）、对象比较等内容。

**章节类型**：概念性章节

**主要内容**：
- 对象引用的概念
- 对象赋值是引用赋值
- `clone` 关键字的使用
- `__clone` 方法的使用
- 浅拷贝和深拷贝
- `clone with` 语法（PHP 8.5+）
- 对象比较（`==` 和 `===`）

## 核心内容

### 对象引用

**概念**：对象赋值是引用赋值，多个变量指向同一个对象
**特点**：修改一个变量会影响其他变量

### 对象克隆

**语法**：`$clone = clone $object;`
**特点**：创建对象的副本
**方法**：`__clone()` 在克隆时调用

### clone with

**语法**：`$clone = clone $object with (property: value);`
**要求**：PHP 8.5+
**特点**：克隆时修改属性

## 基本用法

### 对象引用示例

```php
<?php
declare(strict_types=1);

$user1 = new User("John", 25);
$user2 = $user1;  // 引用赋值
$user2->name = "Jane";  // 修改 $user2 也会影响 $user1
echo $user1->name;  // "Jane"
```

### 对象克隆示例

```php
<?php
declare(strict_types=1);

$user1 = new User("John", 25);
$user2 = clone $user1;  // 克隆对象
$user2->name = "Jane";  // 修改 $user2 不影响 $user1
echo $user1->name;  // "John"
```

### __clone 方法示例

```php
<?php
declare(strict_types=1);

class User
{
    public function __clone()
    {
        // 克隆时的自定义逻辑
        $this->id = uniqid();
    }
}
```

### clone with 示例

```php
<?php
declare(strict_types=1);

$user1 = new User("John", 25);
$user2 = clone $user1 with (age: 26);  // 克隆并修改属性
```

## 使用场景

- **对象复制**：需要对象的独立副本
- **不可变对象**：创建修改后的新对象
- **资源管理**：克隆时重新分配资源

## 注意事项

- **浅拷贝**：默认克隆是浅拷贝
- **深拷贝**：需要手动实现深拷贝
- **性能考虑**：克隆对象有性能开销

## 常见问题

### 问题 1：意外修改原对象
- **原因**：不理解对象引用
- **解决**：需要独立副本时使用 `clone`

### 问题 2：浅拷贝问题
- **原因**：嵌套对象未深度克隆
- **解决**：在 `__clone()` 中手动克隆嵌套对象

## 最佳实践

- 理解对象引用和克隆的区别
- 需要独立副本时使用 `clone`
- 在 `__clone()` 中处理深拷贝
- 使用 `clone with` 简化克隆代码（PHP 8.5+）
