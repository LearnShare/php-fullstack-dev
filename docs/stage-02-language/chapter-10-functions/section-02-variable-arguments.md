# 2.10.2 可变参数

## 概述

可变参数允许函数接受不定数量的参数。本节介绍可变参数语法（`...` 运算符）、`func_get_args()`、`func_num_args()`、`func_get_arg()`、展开运算符的使用及完整示例。

**章节类型**：语法性章节

**主要内容**：
- 可变参数语法（`...` 运算符）
- `func_get_args()` 函数的使用
- `func_num_args()` 函数的使用
- `func_get_arg()` 函数的使用
- 展开运算符的使用
- 可变参数的类型声明
- 使用场景和最佳实践

## 特性

- **灵活参数**：可以接受不定数量的参数
- **类型声明**：支持类型声明（PHP 8.0+）
- **展开运算符**：可以将数组展开为参数

## 语法/定义

### 可变参数语法

**语法**：`function func(...$args)`
**特点**：`$args` 是一个数组，包含所有传递的参数

### 类型声明

**语法**：`function func(string ...$names)`
**要求**：PHP 8.0+
**特点**：所有参数必须是同一类型

### 展开运算符

**语法**：`func(...$array)`
**功能**：将数组展开为函数参数

## 基本用法

### 可变参数示例

```php
<?php
declare(strict_types=1);

function sum(...$numbers): int
{
    return array_sum($numbers);
}

echo sum(1, 2, 3, 4, 5);  // 15
```

### 类型声明示例

```php
<?php
declare(strict_types=1);

function greet(string ...$names): void
{
    foreach ($names as $name) {
        echo "Hello, {$name}!\n";
    }
}

greet("John", "Jane", "Bob");
```

### 展开运算符示例

```php
<?php
declare(strict_types=1);

function add(int $a, int $b, int $c): int
{
    return $a + $b + $c;
}

$numbers = [1, 2, 3];
echo add(...$numbers);  // 6
```

### 传统方法示例

```php
<?php
declare(strict_types=1);

function sum(): int
{
    $total = 0;
    $args = func_get_args();
    foreach ($args as $arg) {
        $total += $arg;
    }
    return $total;
}
```

## 使用场景

- **不定参数**：函数需要接受不定数量的参数
- **参数传递**：将数组展开为函数参数
- **函数包装**：包装其他函数

## 注意事项

- **类型声明**：PHP 8.0+ 支持类型声明
- **性能**：可变参数有性能开销
- **可读性**：过度使用可能降低可读性

## 常见问题

### 问题 1：类型声明要求
- **原因**：PHP 8.0 以下不支持类型声明
- **解决**：升级 PHP 版本或使用传统方法

### 问题 2：展开运算符误用
- **原因**：不理解展开运算符的作用
- **解决**：理解展开运算符将数组展开为参数

## 最佳实践

- 使用可变参数语法（`...`）
- PHP 8.0+ 使用类型声明
- 理解展开运算符的使用
- 注意性能和可读性
