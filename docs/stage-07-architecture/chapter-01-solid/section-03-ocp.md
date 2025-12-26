# 7.1.3 开闭原则（OCP）

## 概述

开闭原则要求对扩展开放，对修改关闭。本节介绍 OCP 的概念、扩展性设计、策略模式应用等，帮助零基础学员掌握开闭原则的应用。

**章节类型**：概念性章节

**主要内容**：
- OCP 概念
- 扩展性设计
- 抽象和接口的使用
- 策略模式应用
- 违反 OCP 的示例
- 符合 OCP 的重构示例
- 完整示例

## 核心内容

### OCP 概念

- 什么是开闭原则
- 扩展的含义
- 修改的含义

### 扩展性设计

- 抽象设计
- 接口设计
- 多态应用

### 策略模式

- 策略模式概念
- OCP 中的应用
- 实现方法

### 违反示例

- 违反 OCP 的代码
- 问题分析
- 重构方向

## 基本用法

### OCP 应用示例

```php
<?php
declare(strict_types=1);

// 违反 OCP：需要修改代码添加新功能
class PaymentProcessor {
    public function process(string $type, float $amount): void {
        if ($type === 'credit') {
            // 处理信用卡
        } elseif ($type === 'paypal') {
            // 处理 PayPal
        }
    }
}

// 符合 OCP：通过扩展添加新功能
interface PaymentMethod {
    public function process(float $amount): void;
}

class CreditCardPayment implements PaymentMethod {
    public function process(float $amount): void { }
}

class PayPalPayment implements PaymentMethod {
    public function process(float $amount): void { }
}
```

## 使用场景

- 功能扩展
- 插件系统
- 策略选择
- 框架设计

## 注意事项

- 抽象层次
- 过度抽象
- 实际需求
- 性能考虑

## 常见问题

- 如何实现开闭原则？
- 何时使用抽象？
- 如何避免过度抽象？
- OCP 的性能影响？

## 最佳实践

- 使用抽象和接口
- 通过扩展添加功能
- 避免修改现有代码
- 合理使用设计模式
