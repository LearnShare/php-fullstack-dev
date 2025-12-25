# 3.4.2 Attributes（注解）

## 概述

Attributes（注解）是 PHP 8.0 引入的特性，用于添加元数据。本节介绍 Attributes 基础语法、读取 Attributes、内置 Attributes、自定义 Attributes、Attributes 应用场景（路由、验证、ORM）等内容。

**章节类型**：语法性章节

**主要内容**：
- Attributes 的基础语法
- 读取 Attributes（反射 API）
- 内置 Attributes
- 自定义 Attributes
- Attributes 应用场景
- 路由注解
- 验证注解
- ORM 注解

## 特性

- **元数据**：为代码添加元数据
- **类型安全**：编译时检查
- **反射支持**：通过反射 API 读取

## 语法/定义

### Attributes 语法

**语法**：`#[AttributeName]` 或 `#[AttributeName(arguments)]`
**要求**：PHP 8.0+
**位置**：类、方法、属性、参数前

### 定义 Attributes

**语法**：`#[Attribute] class AttributeName { ... }`
**特点**：使用 `#[Attribute]` 标记

### 读取 Attributes

**方法**：使用反射 API（`ReflectionClass`、`ReflectionMethod` 等）

## 基本用法

### 基础示例

```php
<?php
declare(strict_types=1);

#[Route('/users', methods: ['GET'])]
class UserController
{
    #[Authorize]
    public function index(): array
    {
        return [];
    }
}
```

### 定义 Attributes 示例

```php
<?php
declare(strict_types=1);

#[Attribute]
class Route
{
    public function __construct(
        public string $path,
        public array $methods = ['GET']
    ) {}
}
```

### 读取 Attributes 示例

```php
<?php
declare(strict_types=1);

$reflection = new ReflectionClass(UserController::class);
$attributes = $reflection->getAttributes(Route::class);
```

## 使用场景

- **路由定义**：框架路由注解
- **验证规则**：数据验证注解
- **ORM 映射**：数据库映射注解
- **依赖注入**：DI 容器注解

## 注意事项

- **版本要求**：需要 PHP 8.0+
- **反射性能**：反射有性能开销
- **缓存考虑**：考虑缓存反射结果

## 常见问题

### 问题 1：版本要求
- **原因**：使用了不支持的 PHP 版本
- **解决**：确保 PHP 版本 >= 8.0

### 问题 2：反射性能
- **问题**：反射有性能开销
- **解决**：缓存反射结果

## 最佳实践

- 使用 Attributes 添加元数据
- 合理使用内置 Attributes
- 自定义 Attributes 时遵循命名规范
- 考虑反射性能，适当缓存
