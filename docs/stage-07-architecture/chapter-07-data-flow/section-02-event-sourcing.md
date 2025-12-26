# 7.7.2 事件溯源（Event Sourcing）

## 概述

事件溯源通过存储事件来重建状态。本节介绍事件溯源的概念、事件存储、事件重放等，帮助零基础学员理解事件溯源。

**章节类型**：概念性章节

**主要内容**：
- 事件溯源概念
- 事件存储
- 事件重放
- 快照机制
- 事件版本
- 完整示例

## 核心内容

### 事件溯源概念

- 什么是事件溯源
- 事件溯源的优势
- 事件溯源的应用场景

### 事件存储

- 事件存储设计
- 事件格式
- 事件序列化
- 事件持久化

### 事件重放

- 重放机制
- 状态重建
- 性能优化
- 快照使用

### 快照机制

- 快照概念
- 快照创建
- 快照使用
- 快照策略

## 基本用法

### 事件溯源示例

```php
<?php
declare(strict_types=1);

// 事件
class UserCreatedEvent {
    public function __construct(
        public readonly UserId $id,
        public readonly string $name
    ) {}
}

// 事件存储
class EventStore {
    public function append(string $streamId, DomainEvent $event): void {
        // 存储事件
    }
    
    public function getEvents(string $streamId): array {
        // 获取事件
    }
}

// 事件重放
class User {
    public static function fromEvents(array $events): self {
        $user = new self();
        foreach ($events as $event) {
            $user->apply($event);
        }
        return $user;
    }
}
```

## 使用场景

- 审计需求
- 时间旅行
- 复杂状态
- 事件驱动

## 注意事项

- 事件版本
- 性能考虑
- 存储空间
- 复杂度

## 常见问题

- 什么是事件溯源？
- 如何存储事件？
- 如何重放事件？
- 事件溯源的性能？

## 最佳实践

- 设计清晰的事件
- 实现快照机制
- 处理事件版本
- 优化重放性能
