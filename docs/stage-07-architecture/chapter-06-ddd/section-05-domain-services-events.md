# 7.6.5 领域服务与领域事件

## 概述

领域服务和领域事件是 DDD 中的重要模式。本节介绍领域服务和领域事件的概念、设计方法、应用场景等，帮助零基础学员掌握这些模式。

**章节类型**：概念性章节

**主要内容**：
- 领域服务概念
- 领域服务设计
- 领域事件概念
- 领域事件发布
- 领域事件处理
- 完整示例

## 核心内容

### 领域服务

- 什么是领域服务
- 服务的作用
- 服务设计原则
- 服务 vs 实体方法

### 领域事件

- 什么是领域事件
- 事件的作用
- 事件设计
- 事件命名

### 事件发布

- 事件发布机制
- 事件存储
- 事件总线
- 发布时机

### 事件处理

- 事件处理器
- 事件订阅
- 异步处理
- 事件溯源

## 基本用法

### 领域服务和事件示例

```php
<?php
declare(strict_types=1);

// 领域服务
class TransferService {
    public function transfer(
        Account $from,
        Account $to,
        Money $amount
    ): void {
        $from->withdraw($amount);
        $to->deposit($amount);
        
        // 发布领域事件
        DomainEventPublisher::publish(
            new MoneyTransferredEvent($from->id(), $to->id(), $amount)
        );
    }
}

// 领域事件
class MoneyTransferredEvent {
    public function __construct(
        private AccountId $from,
        private AccountId $to,
        private Money $amount
    ) {}
}
```

## 使用场景

- 跨聚合操作
- 业务规则
- 事件驱动
- 系统集成

## 注意事项

- 服务设计
- 事件设计
- 事件处理
- 性能考虑

## 常见问题

- 什么是领域服务？
- 何时使用领域服务？
- 什么是领域事件？
- 如何发布和处理事件？

## 最佳实践

- 谨慎使用领域服务
- 设计有意义的领域事件
- 实现事件处理机制
- 考虑事件溯源
