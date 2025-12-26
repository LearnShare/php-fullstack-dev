# 6.5.4 高级特性

## 概述

Redis 提供了丰富的高级特性。本节介绍 Redis 的高级功能，包括发布订阅、事务、Lua 脚本等，帮助零基础学员掌握 Redis 的高级应用。

**章节类型**：工具性章节

**主要内容**：
- 发布订阅（Pub/Sub）
- 事务（Transaction）
- Lua 脚本
- 持久化（RDB、AOF）
- 集群（Cluster）
- 完整示例

## 核心内容

### 发布订阅

- Pub/Sub 概念
- 发布消息
- 订阅频道
- 模式订阅

### 事务

- 事务概念
- MULTI/EXEC
- 事务特性
- 事务限制

### Lua 脚本

- Lua 脚本概念
- 脚本执行
- 原子性保证
- 脚本缓存

### 持久化

- RDB 持久化
- AOF 持久化
- 持久化选择
- 数据恢复

## 基本用法

### 发布订阅示例

```php
<?php
declare(strict_types=1);

// 发布
$redis->publish('channel', 'message');

// 订阅
$redis->subscribe(['channel'], function($redis, $channel, $message) {
    echo "收到消息: {$message}\n";
});
```

## 使用场景

- 消息队列
- 实时通知
- 原子操作
- 复杂逻辑

## 注意事项

- 性能考虑
- 数据一致性
- 持久化策略
- 集群配置

## 常见问题

- 如何使用发布订阅？
- Redis 事务的特点？
- Lua 脚本的作用？
- 如何选择持久化方式？

## 最佳实践

- 合理使用高级特性
- 注意性能影响
- 实现数据持久化
- 考虑集群方案
