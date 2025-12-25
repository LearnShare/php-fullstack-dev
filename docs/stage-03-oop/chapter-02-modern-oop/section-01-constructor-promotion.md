# 3.2.1 构造器属性提升

## 概述

构造器属性提升是 PHP 8.0 引入的特性，可以简化类的定义。本节介绍基础语法、可见性修饰符、混合使用、默认值、类型声明等内容。

**章节类型**：语法性章节

**主要内容**：
- 构造器属性提升的基础语法
- 可见性修饰符的使用
- 类型声明的使用
- 默认值的设置
- 混合使用（提升属性和普通属性）
- 与传统写法的对比
- 使用场景和最佳实践

## 特性

- **简化代码**：减少重复的属性定义
- **类型安全**：支持类型声明
- **灵活性**：支持可见性修饰符和默认值

## 语法/定义

### 基础语法

**语法**：`public function __construct(public Type $property) { ... }`
**要求**：PHP 8.0+
**作用**：同时定义和初始化属性

### 可见性修饰符

**支持**：`public`、`protected`、`private`
**示例**：`public function __construct(private string $name) { ... }`

### 类型声明

**支持**：所有 PHP 类型
**示例**：`public function __construct(public string $name, public int $age) { ... }`

### 默认值

**语法**：`public function __construct(public string $name = "Guest") { ... }`
**特点**：支持默认值

## 基本用法

### 基础示例

```php
<?php
declare(strict_types=1);

class User
{
    public function __construct(
        public string $name,
        public int $age
    ) {}
}
```

### 可见性修饰符示例

```php
<?php
declare(strict_types=1);

class User
{
    public function __construct(
        public string $name,
        protected int $age,
        private string $email
    ) {}
}
```

### 默认值示例

```php
<?php
declare(strict_types=1);

class User
{
    public function __construct(
        public string $name,
        public int $age = 0
    ) {}
}
```

### 混合使用示例

```php
<?php
declare(strict_types=1);

class User
{
    private string $id;
    
    public function __construct(
        public string $name,
        public int $age
    ) {
        $this->id = uniqid();
    }
}
```

## 使用场景

- **DTO**：数据传输对象
- **值对象**：不可变值对象
- **配置类**：配置对象
- **简化类定义**：减少样板代码

## 注意事项

- **版本要求**：需要 PHP 8.0+
- **类型声明**：建议使用类型声明
- **可见性**：根据需求选择合适的可见性

## 常见问题

### 问题 1：版本要求
- **原因**：使用了不支持的 PHP 版本
- **解决**：确保 PHP 版本 >= 8.0

### 问题 2：类型声明
- **建议**：始终使用类型声明
- **好处**：提高代码质量和可读性

## 最佳实践

- 使用构造器属性提升简化代码
- 始终使用类型声明
- 根据需求选择合适的可见性
- 理解与传统写法的区别
