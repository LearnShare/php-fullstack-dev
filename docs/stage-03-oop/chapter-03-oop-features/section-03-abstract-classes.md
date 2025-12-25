# 3.3.3 抽象类（Abstract Class）

## 概述

抽象类提供部分实现的模板，是介于接口和具体类之间的概念。本节介绍基础语法、抽象方法、抽象类 vs 接口、模板方法模式、使用场景等内容。

**章节类型**：语法性章节

**主要内容**：
- 抽象类的定义语法
- 抽象方法的定义
- 抽象类与接口的区别
- 模板方法模式
- 使用场景和选择建议
- 抽象类的最佳实践

## 特性

- **部分实现**：可以提供部分实现
- **抽象方法**：必须由子类实现
- **模板方法**：定义算法骨架

## 语法/定义

### 抽象类定义

**语法**：`abstract class AbstractClass { ... }`
**特点**：不能直接实例化

### 抽象方法

**语法**：`abstract public function method(): ReturnType;`
**特点**：只有方法签名，没有实现

### 模板方法模式

**概念**：定义算法骨架，具体步骤由子类实现

## 基本用法

### 抽象类示例

```php
<?php
declare(strict_types=1);

abstract class Animal
{
    protected string $name;
    
    public function __construct(string $name)
    {
        $this->name = $name;
    }
    
    abstract public function speak(): string;
    
    public function introduce(): string
    {
        return "I am {$this->name}, {$this->speak()}";
    }
}
```

### 抽象方法实现示例

```php
<?php
declare(strict_types=1);

class Dog extends Animal
{
    public function speak(): string
    {
        return "Woof!";
    }
}
```

### 模板方法模式示例

```php
<?php
declare(strict_types=1);

abstract class DataProcessor
{
    public function process(): void
    {
        $this->load();
        $this->transform();
        $this->save();
    }
    
    abstract protected function load(): void;
    abstract protected function transform(): void;
    abstract protected function save(): void;
}
```

## 使用场景

- **模板方法**：定义算法骨架
- **部分实现**：提供部分通用实现
- **代码复用**：复用抽象类的实现

## 注意事项

- **不能实例化**：抽象类不能直接实例化
- **必须实现**：子类必须实现所有抽象方法
- **单继承**：仍然受单继承限制

## 常见问题

### 问题 1：抽象类 vs 接口
- **区别**：抽象类可以有实现，接口不能
- **选择**：需要部分实现时使用抽象类

### 问题 2：抽象方法未实现
- **原因**：子类未实现抽象方法
- **解决**：实现所有抽象方法

## 最佳实践

- 需要部分实现时使用抽象类
- 使用模板方法模式定义算法骨架
- 理解抽象类与接口的区别
- 合理设计抽象类的粒度
