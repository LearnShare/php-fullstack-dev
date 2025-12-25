# 3.2.3 枚举（Enums）

## 概述

枚举是 PHP 8.1 引入的特性，提供类型安全的常量集合。本节介绍基础枚举、Backed Enums、枚举方法、接口实现、与 match 表达式配合、数据库存储等内容。

**章节类型**：语法性章节

**主要内容**：
- 基础枚举的定义和使用
- Backed Enums（有值的枚举）
- 枚举方法
- 枚举实现接口
- 与 match 表达式配合
- 数据库存储枚举值
- 使用场景和最佳实践

## 特性

- **类型安全**：编译时类型检查
- **方法支持**：枚举可以有方法
- **接口实现**：枚举可以实现接口
- **数据库友好**：可以存储为数据库值

## 语法/定义

### 基础枚举

**语法**：`enum EnumName { case Value1; case Value2; }`
**要求**：PHP 8.1+
**特点**：纯枚举值

### Backed Enums

**语法**：`enum EnumName: Type { case Value1 = 'value1'; }`
**特点**：枚举值有对应的标量值

### 枚举方法

**语法**：`enum EnumName { case Value; public function method(): void { ... } }`
**特点**：枚举可以有方法

## 基本用法

### 基础枚举示例

```php
<?php
declare(strict_types=1);

enum Status
{
    case PENDING;
    case ACTIVE;
    case INACTIVE;
}
```

### Backed Enums 示例

```php
<?php
declare(strict_types=1);

enum Status: string
{
    case PENDING = 'pending';
    case ACTIVE = 'active';
    case INACTIVE = 'inactive';
}
```

### 枚举方法示例

```php
<?php
declare(strict_types=1);

enum Status: string
{
    case PENDING = 'pending';
    case ACTIVE = 'active';
    
    public function isActive(): bool
    {
        return $this === self::ACTIVE;
    }
}
```

### 与 match 表达式配合

```php
<?php
declare(strict_types=1);

$message = match($status) {
    Status::PENDING => "等待中",
    Status::ACTIVE => "已激活",
    Status::INACTIVE => "已停用"
};
```

## 使用场景

- **状态管理**：对象状态
- **配置选项**：配置选项值
- **类型常量**：替代传统常量类

## 注意事项

- **类型安全**：枚举是类型安全的
- **数据库存储**：Backed Enums 可以存储为数据库值
- **序列化**：枚举可以序列化

## 常见问题

### 问题 1：枚举值比较
- **方法**：使用 `===` 比较
- **注意**：枚举值比较是严格比较

### 问题 2：数据库存储
- **方法**：使用 Backed Enums
- **存储**：存储标量值，读取时转换为枚举

## 最佳实践

- 使用枚举替代常量类
- 状态管理使用枚举
- 使用 Backed Enums 存储到数据库
- 与 match 表达式配合使用
