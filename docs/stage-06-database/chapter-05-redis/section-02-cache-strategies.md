# 6.5.2 缓存策略

## 概述

缓存策略决定了缓存的使用方式。本节介绍常见的缓存策略，包括 Cache-Aside、Write-Through、Write-Behind 等，帮助零基础学员选择合适的缓存方案。

**章节类型**：最佳实践章节

**主要内容**：
- 缓存策略概述
- Cache-Aside（旁路缓存）
- Write-Through（写透）
- Write-Behind（写回）
- 缓存更新策略
- 策略选择
- 完整示例

## 核心内容

### 缓存策略概述

- 为什么需要策略
- 策略类型
- 策略选择原则

### Cache-Aside

- 策略流程
- 读取流程
- 写入流程
- 适用场景

### Write-Through

- 策略流程
- 同步写入
- 一致性保证
- 适用场景

### Write-Behind

- 策略流程
- 异步写入
- 性能优势
- 适用场景

## 基本用法

### Cache-Aside 示例

```php
<?php
declare(strict_types=1);

function getUser(int $id): ?User {
    // 先查缓存
    $key = "user:{$id}";
    $cached = $redis->get($key);
    
    if ($cached !== false) {
        return unserialize($cached);
    }
    
    // 缓存未命中，查数据库
    $user = User::find($id);
    
    if ($user) {
        // 写入缓存
        $redis->setex($key, 3600, serialize($user));
    }
    
    return $user;
}
```

## 使用场景

- 读多写少场景
- 数据一致性要求
- 性能优化
- 缓存管理

## 注意事项

- 缓存一致性
- 缓存更新
- 缓存失效
- 性能平衡

## 常见问题

- 如何选择缓存策略？
- Cache-Aside 的流程？
- Write-Through 的优势？
- 如何保证缓存一致性？

## 最佳实践

- 根据场景选择策略
- 实现缓存更新机制
- 处理缓存失效
- 监控缓存命中率
