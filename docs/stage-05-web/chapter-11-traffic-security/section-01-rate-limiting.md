# 5.11.1 Rate Limiting

## 概述

Rate Limiting 用于限制请求频率，防止滥用和攻击。本节介绍 Rate Limiting 的概念和实现方法，包括基于 IP、基于用户的限制，以及使用 Redis 的实现，帮助零基础学员掌握流量控制技术。

**章节类型**：实践性章节

**主要内容**：
- Rate Limiting 概念
- 实现方法
- 基于 IP 的限制
- 基于用户的限制
- Redis 实现
- 滑动窗口算法
- 完整示例

## 核心内容

### Rate Limiting 概念

- 什么是 Rate Limiting
- Rate Limiting 的作用
- 限制策略

### 实现方法

- 内存计数
- 文件存储
- 数据库存储
- Redis 存储

### 基于 IP 的限制

- IP 识别
- 计数管理
- 限制规则
- 实现示例

### Redis 实现

- Redis 计数器
- 过期时间设置
- 原子操作
- 分布式限制

## 基本用法

### Rate Limiting 示例

```php
<?php
declare(strict_types=1);

function checkRateLimit(string $key, int $limit, int $window): bool {
    $redis = new Redis();
    $current = $redis->incr($key);
    
    if ($current === 1) {
        $redis->expire($key, $window);
    }
    
    return $current <= $limit;
}
```

## 使用场景

- API 保护
- 防止暴力破解
- 防止 DDoS 攻击
- 资源保护

## 注意事项

- 限制策略设计
- 误杀处理
- 分布式环境
- 性能影响

## 常见问题

- 如何实现 Rate Limiting？
- 如何选择限制策略？
- 如何处理分布式环境？
- Rate Limiting 的性能影响？

## 最佳实践

- 使用 Redis 实现
- 设置合理的限制
- 提供清晰的错误信息
- 考虑白名单机制
