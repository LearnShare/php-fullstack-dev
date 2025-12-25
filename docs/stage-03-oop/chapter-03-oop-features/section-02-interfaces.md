# 3.3.2 接口（Interface）

## 概述

接口定义类的契约，是实现多态的重要机制。本节介绍基础语法、接口方法可见性、实现多个接口、接口继承、接口作为类型提示、多态实现等内容。

**章节类型**：语法性章节

**主要内容**：
- 接口的定义语法
- 接口方法的可见性
- 实现多个接口
- 接口继承
- 接口作为类型提示
- 多态实现
- 接口与抽象类的区别

## 特性

- **契约定义**：定义类必须实现的方法
- **多实现**：一个类可以实现多个接口
- **类型提示**：接口可以作为类型提示
- **多态基础**：实现多态的重要机制

## 语法/定义

### 接口定义

**语法**：`interface InterfaceName { public function method(): ReturnType; }`
**特点**：只定义方法签名，不实现

### 接口实现

**语法**：`class ClassName implements InterfaceName { ... }`
**要求**：必须实现接口的所有方法

### 多接口实现

**语法**：`class ClassName implements Interface1, Interface2 { ... }`
**特点**：一个类可以实现多个接口

## 基本用法

### 接口定义示例

```php
<?php
declare(strict_types=1);

interface Flyable
{
    public function fly(): void;
}
```

### 接口实现示例

```php
<?php
declare(strict_types=1);

class Bird implements Flyable
{
    public function fly(): void
    {
        echo "Bird is flying\n";
    }
}
```

### 多接口实现示例

```php
<?php
declare(strict_types=1);

interface Flyable { public function fly(): void; }
interface Swimmable { public function swim(): void; }

class Duck implements Flyable, Swimmable
{
    public function fly(): void { echo "Flying\n"; }
    public function swim(): void { echo "Swimming\n"; }
}
```

### 接口作为类型提示

```php
<?php
declare(strict_types=1);

function makeFly(Flyable $animal): void
{
    $animal->fly();
}
```

## 使用场景

- **契约定义**：定义类必须实现的契约
- **多态实现**：实现多态设计
- **依赖注入**：接口作为依赖类型
- **测试**：使用接口进行 Mock

## 注意事项

- **方法可见性**：接口方法必须是 `public`
- **必须实现**：必须实现接口的所有方法
- **类型安全**：接口提供类型安全

## 常见问题

### 问题 1：未实现所有方法
- **原因**：未实现接口的所有方法
- **解决**：实现接口的所有方法

### 问题 2：方法可见性错误
- **原因**：接口方法必须是 `public`
- **解决**：确保方法可见性是 `public`

## 最佳实践

- 使用接口定义契约
- 接口方法命名清晰
- 合理设计接口粒度
- 理解接口与抽象类的区别
