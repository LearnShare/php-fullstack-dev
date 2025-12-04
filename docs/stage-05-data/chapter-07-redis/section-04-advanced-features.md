# 5.7.4 高级特性

## 概述

Redis 提供了许多高级特性，包括分布式锁、Pub/Sub、Stream、Pipeline 等。本节详细介绍这些高级特性的使用方法和最佳实践。

## 分布式锁

### SetNX 实现

```php
<?php
declare(strict_types=1);

class DistributedLock
{
    private Redis $redis;
    
    public function __construct(Redis $redis)
    {
        $this->redis = $redis;
    }
    
    public function acquire(string $key, int $timeout = 10): ?string
    {
        $lockKey = "lock:{$key}";
        $lockValue = uniqid('', true);
        
        if ($this->redis->set($lockKey, $lockValue, ['nx', 'ex' => $timeout])) {
            return $lockValue;
        }
        
        return null;
    }
    
    public function release(string $key, string $lockValue): bool
    {
        $lockKey = "lock:{$key}";
        
        $script = "
            if redis.call('get', KEYS[1]) == ARGV[1] then
                return redis.call('del', KEYS[1])
            else
                return 0
            end
        ";
        
        return $this->redis->eval($script, [$lockKey, $lockValue], 1) > 0;
    }
}
```

## Pub/Sub

### 发布订阅

```php
<?php
declare(strict_types=1);

// 发布
$redis->publish('channel', json_encode(['message' => 'Hello']));

// 订阅
$redis->subscribe(['channel'], function ($redis, $channel, $message) {
    $data = json_decode($message, true);
    // 处理消息
});
```

## Stream

### 消息队列

```php
<?php
declare(strict_types=1);

// 生产消息
$redis->xAdd('orders', '*', [
    'order_id' => 123,
    'user_id' => 1,
    'amount' => 99.99,
]);

// 消费消息
$messages = $redis->xRead(['orders' => '$'], 10, 1000);
foreach ($messages as $stream => $streamMessages) {
    foreach ($streamMessages as $id => $message) {
        processOrder($message);
        $redis->xAck('orders', 'consumer-group', $id);
    }
}
```

## Pipeline

### 批量操作

```php
<?php
declare(strict_types=1);

$pipe = $redis->pipeline();
$pipe->set('key1', 'value1');
$pipe->set('key2', 'value2');
$pipe->get('key1');
$results = $pipe->execute();
```

## 完整示例

```php
<?php
declare(strict_types=1);

class RedisAdvanced
{
    private Redis $redis;

    public function __construct(Redis $redis)
    {
        $this->redis = $redis;
    }

    public function batchSet(array $data): void
    {
        $pipe = $this->redis->pipeline();
        foreach ($data as $key => $value) {
            $pipe->set($key, json_encode($value));
        }
        $pipe->execute();
    }

    public function publishEvent(string $channel, array $data): void
    {
        $this->redis->publish($channel, json_encode($data));
    }
}
```

## 注意事项

1. **分布式锁**：确保锁的正确释放
2. **Pub/Sub**：注意消息丢失问题
3. **Stream**：使用消费者组保证消息处理
4. **Pipeline**：批量操作提升性能

## 练习

1. 实现一个完整的分布式锁系统。

2. 创建一个 Pub/Sub 消息系统。

3. 实现 Redis Stream 消息队列。

4. 优化批量操作，使用 Pipeline 提升性能。
