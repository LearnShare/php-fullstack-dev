# 3.1.2 可见性修饰符

## 概述

可见性修饰符控制类成员的访问权限，是封装的核心机制。本节介绍 `public`、`protected`、`private` 的区别、可见性最佳实践、属性封装、方法可见性等。

**章节类型**：语法性章节

**主要内容**：
- `public` 修饰符的用法
- `protected` 修饰符的用法
- `private` 修饰符的用法
- 三种修饰符的区别和适用场景
- 可见性最佳实践
- 属性封装的重要性
- 方法可见性的设计

## 特性

- **public**：公开访问，任何地方都可以访问
- **protected**：受保护，类内部和子类可以访问
- **private**：私有，只有类内部可以访问

## 语法/定义

### public

**语法**：`public $property;` 或 `public function method()`
**访问范围**：任何地方都可以访问

### protected

**语法**：`protected $property;` 或 `protected function method()`
**访问范围**：类内部和子类可以访问

### private

**语法**：`private $property;` 或 `private function method()`
**访问范围**：只有类内部可以访问

## 基本用法

### public 示例

```php
<?php
declare(strict_types=1);

class User
{
    public string $name;  // 公开属性
    
    public function getName(): string
    {
        return $this->name;
    }
}
```

### protected 示例

```php
<?php
declare(strict_types=1);

class User
{
    protected string $email;  // 受保护属性
    
    protected function validateEmail(): bool
    {
        // 验证逻辑
        return true;
    }
}
```

### private 示例

```php
<?php
declare(strict_types=1);

class User
{
    private string $password;  // 私有属性
    
    private function hashPassword(): string
    {
        // 密码哈希逻辑
        return password_hash($this->password, PASSWORD_DEFAULT);
    }
}
```

## 使用场景

- **public**：需要外部访问的接口
- **protected**：需要子类访问的成员
- **private**：内部实现细节

## 注意事项

- **最小权限原则**：使用最小必要的可见性
- **属性封装**：属性通常使用 `private`，通过方法访问
- **方法设计**：根据使用场景选择合适的可见性

## 常见问题

### 问题 1：过度使用 public
- **原因**：不理解封装的重要性
- **解决**：遵循最小权限原则

### 问题 2：可见性设计不当
- **原因**：不理解三种修饰符的区别
- **解决**：理解访问范围和使用场景

## 最佳实践

- 属性通常使用 `private`，通过 getter/setter 访问
- 方法根据使用场景选择合适的可见性
- 遵循最小权限原则
- 理解封装的重要性
