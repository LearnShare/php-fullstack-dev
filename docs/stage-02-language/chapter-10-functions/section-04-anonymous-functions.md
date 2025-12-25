# 2.10.4 匿名函数

## 概述

匿名函数是没有名称的函数，可以作为值传递。本节介绍匿名函数的定义、调用、类型声明、在数组函数中使用、立即执行函数（IIFE）、作为参数/返回值等。

**章节类型**：语法性章节

**主要内容**：
- 匿名函数的定义语法
- 匿名函数的调用
- 匿名函数的类型声明
- 在数组函数中使用匿名函数
- 立即执行函数（IIFE）
- 匿名函数作为参数和返回值
- 使用场景和最佳实践

## 特性

- **无名称**：函数没有名称
- **作为值**：可以作为变量值、参数、返回值
- **灵活性**：提供更大的灵活性

## 语法/定义

### 匿名函数定义

**语法**：`$func = function($param) { ... };`
**特点**：函数赋值给变量

### 立即执行函数（IIFE）

**语法**：`(function($param) { ... })($value);`
**特点**：定义后立即执行

### 类型声明

**语法**：`$func = function(Type $param): ReturnType { ... };`
**特点**：支持类型声明

## 基本用法

### 基础示例

```php
<?php
declare(strict_types=1);

$greet = function(string $name): string {
    return "Hello, {$name}!\n";
};

echo $greet("World");
```

### 在数组函数中使用示例

```php
<?php
declare(strict_types=1);

$numbers = [1, 2, 3, 4, 5];
$doubled = array_map(function($n) {
    return $n * 2;
}, $numbers);
```

### IIFE 示例

```php
<?php
declare(strict_types=1);

$result = (function($x, $y) {
    return $x + $y;
})(3, 4);  // 7
```

### 作为参数示例

```php
<?php
declare(strict_types=1);

function process(callable $callback, $value)
{
    return $callback($value);
}

$result = process(function($x) {
    return $x * 2;
}, 5);  // 10
```

## 使用场景

- **回调函数**：作为回调函数传递
- **数组操作**：在数组函数中使用
- **事件处理**：处理事件回调
- **配置函数**：动态配置函数行为

## 注意事项

- **作用域**：匿名函数有自己的作用域
- **变量捕获**：使用 `use` 子句捕获外部变量
- **类型声明**：支持类型声明

## 常见问题

### 问题 1：变量作用域
- **原因**：匿名函数无法直接访问外部变量
- **解决**：使用 `use` 子句捕获变量

### 问题 2：类型声明
- **要求**：PHP 7.0+ 支持类型声明
- **解决**：使用类型声明提高代码质量

## 最佳实践

- 使用匿名函数作为回调
- 在数组函数中使用匿名函数
- 使用类型声明提高代码质量
- 理解作用域和变量捕获
