# 3.3.4 多态（Polymorphism）

## 概述

多态是面向对象编程的核心特性，允许同一接口有不同的实现。本节介绍多态概念、在依赖注入中的应用、完整示例（支付系统）、多态优势、注意事项等内容。

**章节类型**：概念性章节

**主要内容**：
- 多态的概念
- 多态的实现方式
- 在依赖注入中的应用
- 完整示例（支付系统）
- 多态的优势
- 注意事项和最佳实践

## 核心内容

### 多态概念

**定义**：同一接口，不同实现
**实现**：通过继承和接口实现
**优势**：提高代码灵活性和可扩展性

### 多态实现

**方式**：接口、抽象类、继承
**特点**：运行时决定调用哪个实现

## 基本用法

### 接口多态示例

```php
<?php
declare(strict_types=1);

interface PaymentMethod
{
    public function pay(float $amount): bool;
}

class CreditCard implements PaymentMethod
{
    public function pay(float $amount): bool
    {
        // 信用卡支付逻辑
        return true;
    }
}

class PayPal implements PaymentMethod
{
    public function pay(float $amount): bool
    {
        // PayPal 支付逻辑
        return true;
    }
}

function processPayment(PaymentMethod $method, float $amount): void
{
    $method->pay($amount);  // 多态调用
}
```

### 依赖注入示例

```php
<?php
declare(strict_types=1);

class PaymentProcessor
{
    public function __construct(
        private PaymentMethod $paymentMethod
    ) {}
    
    public function process(float $amount): bool
    {
        return $this->paymentMethod->pay($amount);
    }
}
```

## 使用场景

- **支付系统**：不同的支付方式
- **数据存储**：不同的存储后端
- **通知系统**：不同的通知渠道
- **策略模式**：不同的算法实现

## 注意事项

- **接口设计**：合理设计接口
- **实现一致性**：确保实现符合接口契约
- **性能考虑**：多态有轻微性能开销

## 常见问题

### 问题 1：接口设计不当
- **原因**：接口设计过于复杂或过于简单
- **解决**：合理设计接口粒度

### 问题 2：实现不一致
- **原因**：实现不符合接口契约
- **解决**：确保实现符合接口定义

## 最佳实践

- 使用接口定义多态契约
- 合理设计接口粒度
- 确保实现的一致性
- 理解多态的优势和应用场景
