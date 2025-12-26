# 7.1.4 里氏替换原则（LSP）

## 概述

里氏替换原则要求子类可以替换父类而不影响程序正确性。本节介绍 LSP 的概念、子类替换、契约遵守等，帮助零基础学员掌握里氏替换原则的应用。

**章节类型**：概念性章节

**主要内容**：
- LSP 概念
- 子类替换规则
- 契约遵守
- 前置条件和后置条件
- 违反 LSP 的示例
- 符合 LSP 的重构示例
- 完整示例

## 核心内容

### LSP 概念

- 什么是里氏替换
- 替换的含义
- 正确性的保证

### 子类替换规则

- 行为一致性
- 契约遵守
- 异常处理

### 契约遵守

- 前置条件
- 后置条件
- 不变量

### 违反示例

- 违反 LSP 的代码
- 问题分析
- 重构方向

## 基本用法

### LSP 应用示例

```php
<?php
declare(strict_types=1);

// 违反 LSP：子类改变了父类的行为
class Rectangle {
    public function setWidth(int $width): void { }
    public function setHeight(int $height): void { }
}

class Square extends Rectangle {
    public function setWidth(int $width): void {
        $this->width = $width;
        $this->height = $width; // 违反 LSP
    }
}

// 符合 LSP：保持行为一致性
abstract class Shape {
    abstract public function getArea(): float;
}

class Rectangle extends Shape {
    public function getArea(): float { }
}

class Square extends Shape {
    public function getArea(): float { }
}
```

## 使用场景

- 继承设计
- 多态应用
- 接口实现
- 框架设计

## 注意事项

- 行为一致性
- 契约遵守
- 异常处理
- 设计约束

## 常见问题

- 什么是里氏替换？
- 如何保证子类可替换？
- 如何设计继承层次？
- LSP 与继承的关系？

## 最佳实践

- 保持行为一致性
- 遵守父类契约
- 合理使用继承
- 考虑多态应用
