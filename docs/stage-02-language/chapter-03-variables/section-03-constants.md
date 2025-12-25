# 2.3.3 常量

## 概述

常量是不可改变的值，用于存储配置、魔法值等。本节介绍常量的定义、常量类型、类常量、魔术常量等，帮助理解常量的使用场景和最佳实践。

**章节类型**：语法性章节

**主要内容**：
- 常量的定义方法（`const`、`define()`）
- 常量类型和命名规则
- 类常量的定义和使用
- 魔术常量的使用
- 常量和变量的区别
- 常量最佳实践

## 特性

- **不可改变**：常量一旦定义就不能修改
- **全局作用域**：常量在全局作用域可用
- **命名规范**：通常使用大写字母和下划线

## 语法/定义

### const 定义常量

**语法**：`const CONSTANT_NAME = value;`
**特点**：编译时定义，性能更好
**限制**：只能在类或顶层作用域定义

### define() 定义常量

**语法**：`define(string $name, mixed $value, bool $case_insensitive = false): bool`
**特点**：运行时定义，更灵活
**限制**：可以在任何地方定义

### 类常量

**语法**：`class ClassName { const CONSTANT_NAME = value; }`
**访问**：`ClassName::CONSTANT_NAME` 或 `$obj::CONSTANT_NAME`

## 基本用法

### const 示例

```php
<?php
declare(strict_types=1);

const APP_NAME = "My App";
const MAX_USERS = 100;

echo APP_NAME . "\n";
echo MAX_USERS . "\n";
```

### define() 示例

```php
<?php
declare(strict_types=1);

define("APP_NAME", "My App");
define("MAX_USERS", 100);

echo APP_NAME . "\n";
echo MAX_USERS . "\n";
```

### 类常量示例

```php
<?php
declare(strict_types=1);

class Config
{
    public const APP_NAME = "My App";
    public const MAX_USERS = 100;
}

echo Config::APP_NAME . "\n";
echo Config::MAX_USERS . "\n";
```

## 使用场景

- **配置值**：存储应用配置
- **魔法值**：避免硬编码
- **类常量**：存储类相关的常量值

## 注意事项

- **命名规范**：使用大写字母和下划线
- **作用域**：常量在全局作用域可用
- **类型**：常量可以是标量类型或数组

## 常见问题

### 问题 1：常量未定义
- **原因**：常量未定义就使用
- **解决**：使用 `defined()` 检查

### 问题 2：常量命名冲突
- **原因**：常量名重复定义
- **解决**：使用命名空间或类常量

## 最佳实践

- 使用有意义的常量名
- 遵循命名规范
- 使用类常量组织相关常量
- 避免硬编码，使用常量
