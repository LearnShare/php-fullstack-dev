# 3.1.5 静态属性与方法、魔术方法

## 概述

静态成员和魔术方法是 PHP 面向对象编程的重要特性。本节介绍静态属性与方法、静态工厂方法、常用魔术方法（`__toString`、`__get`、`__set`、`__call`、`__invoke`）等内容。

**章节类型**：语法性章节

**主要内容**：
- 静态属性的定义和使用
- 静态方法的定义和使用
- 静态工厂方法
- 常用魔术方法
- 魔术方法的使用场景
- 静态成员的最佳实践

## 特性

- **静态成员**：属于类而不是对象
- **魔术方法**：在特定时机自动调用
- **工厂方法**：使用静态方法创建对象

## 语法/定义

### 静态属性

**语法**：`public static Type $property;`
**访问**：`ClassName::$property` 或 `self::$property`
**特点**：所有对象共享

### 静态方法

**语法**：`public static function method(): ReturnType { ... }`
**调用**：`ClassName::method()` 或 `self::method()`
**特点**：不能使用 `$this`

### 魔术方法

**命名规则**：以 `__` 开头
**调用时机**：特定操作时自动调用
**常用方法**：`__toString`、`__get`、`__set`、`__call`、`__invoke`

## 基本用法

### 静态属性示例

```php
<?php
declare(strict_types=1);

class Counter
{
    public static int $count = 0;
    
    public function __construct()
    {
        self::$count++;
    }
}
```

### 静态方法示例

```php
<?php
declare(strict_types=1);

class Math
{
    public static function add(int $a, int $b): int
    {
        return $a + $b;
    }
}

$result = Math::add(3, 4);
```

### 静态工厂方法示例

```php
<?php
declare(strict_types=1);

class User
{
    public static function create(string $name, int $age): self
    {
        return new self($name, $age);
    }
}
```

### 魔术方法示例

```php
<?php
declare(strict_types=1);

class User
{
    public function __toString(): string
    {
        return "User: {$this->name}";
    }
    
    public function __invoke(): void
    {
        echo "User object called as function";
    }
}
```

## 使用场景

- **静态成员**：工具类、计数器、配置
- **魔术方法**：自定义对象行为、动态属性、方法重载
- **工厂方法**：复杂对象创建

## 注意事项

- **静态限制**：静态方法不能使用 `$this`
- **共享状态**：静态属性在所有对象间共享
- **性能考虑**：过度使用静态成员可能影响测试

## 常见问题

### 问题 1：静态方法中使用 $this
- **原因**：不理解静态方法的限制
- **解决**：静态方法不能使用 `$this`

### 问题 2：静态属性共享问题
- **原因**：不理解静态属性的共享特性
- **解决**：注意静态属性的共享特性

## 最佳实践

- 合理使用静态成员
- 理解魔术方法的调用时机
- 使用静态工厂方法简化对象创建
- 避免过度使用静态成员
