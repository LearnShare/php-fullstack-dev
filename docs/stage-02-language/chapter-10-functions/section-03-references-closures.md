# 2.10.3 引用、箭头函数与闭包

## 概述

引用、箭头函数和闭包是 PHP 的高级特性。本节介绍引用传参、箭头函数（PHP 7.4+）、闭包、变量捕获（按值、按引用）、闭包与类、`Closure::bindTo()`、变量生命周期、与 JavaScript 对比等。

**章节类型**：语法性章节

**主要内容**：
- 引用传参的使用
- 箭头函数的语法和用法
- 闭包的定义和使用
- 变量捕获机制（按值、按引用）
- 闭包与类的关系
- `Closure::bindTo()` 的使用
- 变量生命周期
- 与 JavaScript 的对比

## 特性

- **引用传参**：可以修改函数参数
- **箭头函数**：简洁的函数语法（PHP 7.4+）
- **闭包**：可以捕获外部变量的函数

## 语法/定义

### 引用传参

**语法**：`function func(&$param)`
**特点**：修改参数会影响原变量

### 箭头函数

**语法**：`fn($param) => expression`
**要求**：PHP 7.4+
**特点**：自动捕获外部变量，简洁语法

### 闭包

**语法**：`function($param) use ($var) { ... }`
**特点**：可以捕获外部变量

## 基本用法

### 引用传参示例

```php
<?php
declare(strict_types=1);

function swap(&$a, &$b): void
{
    $temp = $a;
    $a = $b;
    $b = $temp;
}

$x = 1;
$y = 2;
swap($x, $y);  // $x = 2, $y = 1
```

### 箭头函数示例

```php
<?php
declare(strict_types=1);

$multiply = fn($x, $y) => $x * $y;
echo $multiply(3, 4);  // 12

// 自动捕获外部变量
$factor = 2;
$multiply = fn($x) => $x * $factor;
echo $multiply(5);  // 10
```

### 闭包示例

```php
<?php
declare(strict_types=1);

$factor = 2;
$multiply = function($x) use ($factor) {
    return $x * $factor;
};

echo $multiply(5);  // 10
```

### 按引用捕获示例

```php
<?php
declare(strict_types=1);

$count = 0;
$increment = function() use (&$count) {
    $count++;
};

$increment();
$increment();
echo $count;  // 2
```

## 使用场景

- **引用传参**：需要修改函数参数
- **箭头函数**：简单的回调函数
- **闭包**：需要捕获外部变量的函数

## 注意事项

- **变量捕获**：理解按值和按引用的区别
- **生命周期**：理解变量的生命周期
- **性能**：闭包有性能开销

## 常见问题

### 问题 1：变量捕获方式混淆
- **原因**：不理解按值和按引用的区别
- **解决**：理解 `use` 子句的作用

### 问题 2：箭头函数限制
- **限制**：只能包含单个表达式
- **解决**：复杂逻辑使用闭包

## 最佳实践

- 简单回调使用箭头函数
- 需要捕获变量使用闭包
- 理解变量捕获机制
- 注意变量的生命周期
