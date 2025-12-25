# 2.4.5 类型声明与联合类型

## 概述

类型声明是 PHP 7.0+ 引入的重要特性，可以提高代码质量和可维护性。本节介绍函数参数和返回值类型声明、可选参数、可空类型、联合类型、属性类型声明、只读属性、严格模式等现代 PHP 特性。

**章节类型**：语法性章节

**主要内容**：
- 函数参数类型声明
- 函数返回值类型声明
- 可选参数和默认值
- 可空类型（`?Type`）
- 联合类型（`Type1|Type2`）
- 属性类型声明（PHP 7.4+）
- 只读属性（PHP 8.1+）
- 严格模式（`declare(strict_types=1)`）

## 特性

- **类型安全**：类型声明提供编译时类型检查
- **代码提示**：IDE 可以提供更好的代码补全
- **文档作用**：类型声明本身就是文档
- **性能优化**：类型声明有助于 JIT 优化

## 语法/定义

### 函数参数类型声明

**语法**：`function func(Type $param)`
**支持类型**：标量类型、数组、对象、可调用类型、联合类型

### 函数返回值类型声明

**语法**：`function func(): ReturnType`
**支持类型**：所有类型，包括 `void`

### 可空类型

**语法**：`?Type` 或 `Type|null`
**作用**：允许参数或返回值为指定类型或 `null`

### 联合类型

**语法**：`Type1|Type2|Type3`
**作用**：允许参数或返回值为多个类型之一

### 属性类型声明

**语法**：`public Type $property;`
**要求**：PHP 7.4+，必须提供默认值或使用构造器属性提升

### 只读属性

**语法**：`public readonly Type $property;`
**要求**：PHP 8.1+，只能在初始化时赋值一次

## 基本用法

### 函数类型声明示例

```php
<?php
declare(strict_types=1);

function greet(string $name): string
{
    return "Hello, {$name}!\n";
}

function add(int $a, int $b): int
{
    return $a + $b;
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

### 属性类型声明示例

```php
<?php
declare(strict_types=1);

class User
{
    public string $name;
    public int $age;
    
    public function __construct(string $name, int $age)
    {
        $this->name = $name;
        $this->age = $age;
    }
}
```

### 只读属性示例

```php
<?php
declare(strict_types=1);

class User
{
    public readonly string $name;
    
    public function __construct(string $name)
    {
        $this->name = $name;  // 只能在构造器中赋值
    }
}
```

## 使用场景

- **函数参数**：提高函数调用的类型安全
- **函数返回值**：明确函数的返回类型
- **类属性**：明确属性的类型
- **代码文档**：类型声明本身就是文档

## 注意事项

- **严格模式**：必须使用 `declare(strict_types=1);` 启用严格类型检查
- **类型转换**：严格模式下不会自动转换类型
- **向后兼容**：类型声明不会影响现有代码

## 常见问题

### 问题 1：类型不匹配错误
- **原因**：传递的类型与声明不匹配
- **解决**：检查类型声明和实际值

### 问题 2：严格模式未启用
- **原因**：未使用 `declare(strict_types=1);`
- **解决**：在文件开头添加严格类型声明

## 最佳实践

- 始终使用类型声明
- 启用严格模式
- 使用联合类型提高灵活性
- 使用只读属性保护不可变数据
