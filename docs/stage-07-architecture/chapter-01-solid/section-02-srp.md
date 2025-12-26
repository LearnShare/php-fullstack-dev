# 7.1.2 单一职责原则（SRP）

## 概述

单一职责原则要求一个类只有一个改变的理由。本节介绍 SRP 的概念、职责识别、职责分离等，帮助零基础学员掌握单一职责原则的应用。

**章节类型**：概念性章节

**主要内容**：
- SRP 概念
- 职责识别
- 职责分离方法
- 违反 SRP 的示例
- 符合 SRP 的重构示例
- 完整示例

## 核心内容

### SRP 概念

- 什么是单一职责
- 职责的定义
- 改变的理由

### 职责识别

- 如何识别职责
- 职责的粒度
- 职责的划分

### 职责分离

- 类的拆分
- 方法的提取
- 服务的分离

### 违反示例

- 违反 SRP 的代码
- 问题分析
- 重构方向

## 基本用法

### SRP 应用示例

```php
<?php
declare(strict_types=1);

// 违反 SRP：User 类承担了多个职责
class User {
    public function save(): void { }
    public function sendEmail(): void { }
    public function generateReport(): void { }
}

// 符合 SRP：职责分离
class User {
    public function save(): void { }
}

class EmailService {
    public function send(User $user): void { }
}

class ReportGenerator {
    public function generate(User $user): void { }
}
```

## 使用场景

- 类设计
- 代码重构
- 服务拆分
- 模块设计

## 注意事项

- 职责的粒度
- 过度拆分
- 实际权衡
- 团队理解

## 常见问题

- 如何识别职责？
- 职责如何划分？
- 如何避免过度拆分？
- SRP 的边界在哪里？

## 最佳实践

- 识别类的职责
- 分离不同职责
- 保持合理粒度
- 考虑实际场景
