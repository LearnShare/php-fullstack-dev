# 3.3.1 继承（Inheritance）

## 概述

继承是面向对象编程的核心特性之一，用于代码复用和层次化设计。本节介绍基础语法、`extends` 关键字、访问父类方法、`parent` 关键字、构造函数继承、`final` 关键字、继承层次示例等内容。

**章节类型**：语法性章节

**主要内容**：
- 继承的基础语法
- `extends` 关键字的使用
- 访问父类方法和属性
- `parent` 关键字的使用
- 构造函数的继承
- `final` 关键字的使用
- 继承层次设计
- 继承的最佳实践

## 特性

- **代码复用**：子类继承父类的属性和方法
- **层次化设计**：建立类层次结构
- **多态基础**：为多态提供基础

## 语法/定义

### 继承语法

**语法**：`class ChildClass extends ParentClass { ... }`
**关键字**：`extends`
**特点**：单继承（PHP 只支持单继承）

### parent 关键字

**语法**：`parent::method()`
**作用**：调用父类方法

### final 关键字

**语法**：`final class ClassName { ... }` 或 `final public function method() { ... }`
**作用**：防止类被继承或方法被重写

## 基本用法

### 基础继承示例

```php
<?php
declare(strict_types=1);

class Animal
{
    public string $name;
    
    public function speak(): string
    {
        return "Some sound";
    }
}

class Dog extends Animal
{
    public function speak(): string
    {
        return "Woof!";
    }
}
```

### parent 关键字示例

```php
<?php
declare(strict_types=1);

class Animal
{
    public function speak(): string
    {
        return "Some sound";
    }
}

class Dog extends Animal
{
    public function speak(): string
    {
        return parent::speak() . " Woof!";
    }
}
```

### 构造函数继承示例

```php
<?php
declare(strict_types=1);

class Animal
{
    public function __construct(public string $name) {}
}

class Dog extends Animal
{
    public function __construct(string $name, public string $breed)
    {
        parent::__construct($name);
    }
}
```

## 使用场景

- **代码复用**：复用父类的代码
- **层次化设计**：建立类层次结构
- **多态实现**：为多态提供基础

## 注意事项

- **单继承**：PHP 只支持单继承
- **可见性**：子类可以访问 `public` 和 `protected` 成员
- **方法重写**：子类可以重写父类方法

## 常见问题

### 问题 1：单继承限制
- **限制**：PHP 只支持单继承
- **解决**：使用 Traits 或接口

### 问题 2：构造函数继承
- **规则**：子类构造函数需要调用父类构造函数
- **解决**：使用 `parent::__construct()`

## 最佳实践

- 合理使用继承，避免过度继承
- 使用 `final` 防止不必要的继承
- 理解继承的可见性规则
- 设计清晰的类层次结构
