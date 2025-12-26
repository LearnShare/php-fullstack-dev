# 7.2.2 创建型模式

## 概述

创建型模式关注对象的创建过程。本节介绍常见的创建型模式，包括工厂模式、单例模式、建造者模式等，帮助零基础学员掌握对象创建的模式。

**章节类型**：语法性章节

**主要内容**：
- 创建型模式概述
- 工厂模式（Factory Pattern）
- 抽象工厂模式（Abstract Factory）
- 单例模式（Singleton Pattern）
- 建造者模式（Builder Pattern）
- 原型模式（Prototype Pattern）
- 完整示例

## 核心内容

### 工厂模式

- 简单工厂
- 工厂方法
- 应用场景
- 实现方法

### 抽象工厂

- 抽象工厂概念
- 产品族创建
- 应用场景
- 实现方法

### 单例模式

- 单例概念
- 线程安全
- 应用场景
- 实现方法

### 建造者模式

- 建造者概念
- 复杂对象构建
- 链式调用
- 应用场景

## 基本用法

### 创建型模式示例

```php
<?php
declare(strict_types=1);

// 工厂模式
class ProductFactory {
    public static function create(string $type): Product {
        return match($type) {
            'A' => new ProductA(),
            'B' => new ProductB()
        };
    }
}

// 单例模式
class Database {
    private static ?self $instance = null;
    
    public static function getInstance(): self {
        if (self::$instance === null) {
            self::$instance = new self();
        }
        return self::$instance;
    }
}
```

## 使用场景

- 对象创建
- 复杂对象构建
- 资源管理
- 配置管理

## 注意事项

- 模式选择
- 性能考虑
- 测试友好
- 过度使用

## 常见问题

- 工厂模式和抽象工厂的区别？
- 单例模式的线程安全？
- 何时使用建造者模式？
- 创建型模式如何选择？

## 最佳实践

- 根据场景选择模式
- 注意线程安全
- 考虑测试友好
- 避免过度使用
