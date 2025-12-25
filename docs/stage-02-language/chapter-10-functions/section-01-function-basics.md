# 2.10.1 函数基础

## 概述

函数是代码复用的基本单元。本节介绍函数定义、参数类型、默认值、可选参数、可空类型、联合类型、返回值类型、早返回、命名参数（PHP 8.0+）等函数基础知识。

**章节类型**：语法性章节

**主要内容**：
- 函数定义的语法
- 参数类型声明
- 默认值和可选参数
- 可空类型和联合类型
- 返回值类型声明
- 早返回（early return）
- 命名参数（PHP 8.0+）
- 函数最佳实践

## 特性

- **类型声明**：支持参数和返回值类型声明
- **默认值**：参数可以有默认值
- **命名参数**：可以按名称传递参数（PHP 8.0+）

## 语法/定义

### 函数定义

**语法**：`function functionName(Type $param = default): ReturnType { ... }`
**特点**：支持类型声明、默认值、返回值类型

### 参数类型

**标量类型**：`int`、`float`、`string`、`bool`
**复合类型**：`array`、`object`、`callable`
**特殊类型**：`?Type`（可空）、`Type1|Type2`（联合类型）

### 返回值类型

**类型声明**：`: ReturnType`
**void**：无返回值
**可空类型**：`?ReturnType`

## 基本用法

### 基础函数示例

```php
<?php
declare(strict_types=1);

function greet(string $name): string
{
    return "Hello, {$name}!\n";
}
```

### 默认值示例

```php
<?php
declare(strict_types=1);

function greet(string $name, string $title = "Mr."): string
{
    return "Hello, {$title} {$name}!\n";
}
```

### 可空类型示例

```php
<?php
declare(strict_types=1);

function findUser(?int $id): ?User
{
    if ($id === null) {
        return null;
    }
    // 查找用户
    return new User();
}
```

### 联合类型示例

```php
<?php
declare(strict_types=1);

function process(string|int $value): string|int
{
    if (is_string($value)) {
        return strtoupper($value);
    }
    return $value * 2;
}
```

### 命名参数示例

```php
<?php
declare(strict_types=1);

function createUser(string $name, int $age, string $email): User
{
    // ...
}

// PHP 8.0+ 命名参数
createUser(age: 25, email: "user@example.com", name: "John");
```

## 使用场景

- **代码复用**：将重复代码提取为函数
- **模块化**：将功能模块化
- **类型安全**：使用类型声明提高代码质量

## 注意事项

- **类型声明**：启用严格模式后类型必须匹配
- **默认值**：有默认值的参数必须在无默认值参数之后
- **命名参数**：命名参数必须在位置参数之后

## 常见问题

### 问题 1：类型不匹配错误
- **原因**：传递的类型与声明不匹配
- **解决**：检查类型声明和实际值

### 问题 2：默认值参数位置
- **原因**：有默认值的参数在无默认值参数之前
- **解决**：调整参数顺序

## 最佳实践

- 始终使用类型声明
- 使用有意义的函数名
- 函数职责单一
- 使用命名参数提高可读性
