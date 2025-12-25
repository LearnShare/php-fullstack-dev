# 3.2.4 属性钩子与静态方法

## 概述

属性钩子是 PHP 8.4 引入的特性，允许自定义属性的访问逻辑。本节介绍属性钩子基础语法、不对称可见性、静态属性滥用风险、静态方法合理使用、静态工厂方法等内容。

**章节类型**：语法性章节

**主要内容**：
- 属性钩子的基础语法
- getter 和 setter 钩子
- 不对称可见性
- 静态属性的滥用风险
- 静态方法的合理使用
- 静态工厂方法
- 使用场景和最佳实践

## 特性

- **属性钩子**：自定义属性访问逻辑（PHP 8.4+）
- **不对称可见性**：读写可见性不同
- **静态方法**：属于类的方法

## 语法/定义

### 属性钩子

**语法**：`private string $name { get { ... } set(string $value) { ... } }`
**要求**：PHP 8.4+
**特点**：自定义属性的 getter 和 setter

### 不对称可见性

**语法**：`public string $name { get; private set; }`
**要求**：PHP 8.4+
**特点**：读写可见性不同

## 基本用法

### 属性钩子示例

```php
<?php
declare(strict_types=1);

class User
{
    private string $name {
        get {
            return $this->name;
        }
        set(string $value) {
            $this->name = trim($value);
        }
    }
}
```

### 不对称可见性示例

```php
<?php
declare(strict_types=1);

class User
{
    public string $name { get; private set; }
    
    public function setName(string $name): void
    {
        $this->name = $name;  // 可以设置
    }
}
```

### 静态方法示例

```php
<?php
declare(strict_types=1);

class User
{
    public static function create(string $name): self
    {
        return new self($name);
    }
}
```

## 使用场景

- **属性钩子**：需要自定义属性访问逻辑
- **不对称可见性**：只读属性，通过方法设置
- **静态方法**：工具方法、工厂方法

## 注意事项

- **版本要求**：属性钩子需要 PHP 8.4+
- **性能考虑**：属性钩子有性能开销
- **静态滥用**：避免过度使用静态成员

## 常见问题

### 问题 1：属性钩子版本要求
- **原因**：使用了不支持的 PHP 版本
- **解决**：确保 PHP 版本 >= 8.4

### 问题 2：静态属性滥用
- **问题**：过度使用静态属性
- **解决**：合理使用静态成员

## 最佳实践

- 使用属性钩子自定义属性访问（PHP 8.4+）
- 使用不对称可见性控制属性访问
- 合理使用静态方法
- 避免静态属性滥用
