# 6.5.4 Redis 高级特性

## 概述

Redis 提供了许多高级特性来增强功能，包括发布订阅、事务、Lua 脚本、管道、持久化等。这些高级特性可以满足更复杂的业务需求，提高系统性能和可靠性。

本节详细介绍 Redis 的高级特性，包括发布订阅机制、事务处理、Lua 脚本、管道操作、持久化配置、主从复制等，帮助零基础学员掌握 Redis 的高级应用。

**主要内容**：
- 发布订阅（Pub/Sub）
- 事务处理
- Lua 脚本
- 管道操作
- 持久化配置
- 主从复制
- 完整示例和最佳实践

---

## 特性

- **发布订阅**：支持消息发布订阅
- **事务支持**：支持事务操作
- **Lua 脚本**：支持 Lua 脚本执行
- **管道操作**：支持批量操作
- **持久化**：支持数据持久化
- **高可用**：支持主从复制和集群

---

## 发布订阅

### 什么是发布订阅

发布订阅（Pub/Sub）是一种消息通信模式，发布者发送消息，订阅者接收消息。

**工作流程**：

1. **订阅**：订阅者订阅频道
2. **发布**：发布者向频道发布消息
3. **接收**：订阅者接收消息

**示例**：

```php
<?php
declare(strict_types=1);

// 发布者
$publisher = new Redis();
$publisher->connect('127.0.0.1', 6379);
$publisher->publish('news', 'Breaking news!');

// 订阅者
$subscriber = new Redis();
$subscriber->connect('127.0.0.1', 6379);
$subscriber->subscribe(['news'], function ($redis, $channel, $message) {
    echo "收到消息: {$message}\n";
});
```

### 发布订阅应用

发布订阅适用于：

- **消息通知**：系统消息通知
- **实时通信**：实时消息推送
- **事件驱动**：事件驱动的系统架构

---

## 事务处理

### Redis 事务

Redis 支持事务，但不像数据库事务那样支持回滚。

**特点**：

- **原子性**：事务中的所有命令要么全部执行，要么全部不执行
- **不支持回滚**：执行失败不会回滚
- **隔离性**：事务中的命令按顺序执行

**示例**：

```php
<?php
declare(strict_types=1);

$redis = new Redis();
$redis->connect('127.0.0.1', 6379);

// 开始事务
$redis->multi();

// 执行命令
$redis->set('key1', 'value1');
$redis->set('key2', 'value2');
$redis->incr('counter');

// 提交事务
$redis->exec();

// 或者取消事务
// $redis->discard();
```

### Watch 命令

使用 `WATCH` 命令实现乐观锁。

**示例**：

```php
<?php
declare(strict_types=1);

$redis = new Redis();
$redis->connect('127.0.0.1', 6379);

// 监视键
$redis->watch('balance');

// 获取当前值
$balance = $redis->get('balance');

// 开始事务
$redis->multi();

// 更新值
$redis->set('balance', $balance - 100);

// 提交事务（如果键被修改，事务会失败）
$result = $redis->exec();

if ($result === false) {
    // 事务失败，重试
    echo "事务失败，请重试\n";
}
```

---

## Lua 脚本

### 什么是 Lua 脚本

Lua 脚本可以在 Redis 服务器端执行，保证原子性。

**优势**：

- **原子性**：脚本执行是原子性的
- **减少网络往返**：减少客户端和服务器之间的网络往返
- **复杂逻辑**：可以执行复杂的逻辑

**示例**：

```php
<?php
declare(strict_types=1);

$redis = new Redis();
$redis->connect('127.0.0.1', 6379);

// Lua 脚本：原子性地增加计数器
$script = "
    local current = redis.call('GET', KEYS[1])
    if current == false then
        current = 0
    end
    local new = current + ARGV[1]
    redis.call('SET', KEYS[1], new)
    return new
";

// 执行脚本
$result = $redis->eval($script, ['counter'], [10]);
echo "新值: {$result}\n";
```

### Lua 脚本应用

Lua 脚本适用于：

- **原子操作**：需要原子性的复杂操作
- **性能优化**：减少网络往返
- **复杂逻辑**：需要执行复杂逻辑的场景

---

## 管道操作

### 什么是管道

管道（Pipeline）允许一次性发送多个命令，减少网络往返。

**示例**：

```php
<?php
declare(strict_types=1);

$redis = new Redis();
$redis->connect('127.0.0.1', 6379);

// 开始管道
$pipe = $redis->pipeline();

// 添加命令
$pipe->set('key1', 'value1');
$pipe->set('key2', 'value2');
$pipe->set('key3', 'value3');
$pipe->get('key1');
$pipe->get('key2');

// 执行管道
$results = $pipe->exec();

// $results 包含所有命令的结果
foreach ($results as $result) {
    echo $result . "\n";
}
```

### 管道优势

管道的优势：

- **减少网络往返**：一次性发送多个命令
- **提高性能**：显著提高批量操作性能
- **原子性**：管道中的命令按顺序执行

---

## 持久化

### RDB 持久化

RDB 是 Redis 的快照持久化方式。

**特点**：

- **快照**：定期生成数据快照
- **文件小**：文件体积小
- **恢复快**：恢复速度快

**配置**：

```conf
# redis.conf
save 900 1      # 900 秒内至少 1 个键变化
save 300 10     # 300 秒内至少 10 个键变化
save 60 10000   # 60 秒内至少 10000 个键变化
```

### AOF 持久化

AOF（Append Only File）是 Redis 的日志持久化方式。

**特点**：

- **日志**：记录每个写操作
- **数据安全**：数据更安全
- **文件大**：文件体积较大

**配置**：

```conf
# redis.conf
appendonly yes
appendfsync everysec  # 每秒同步一次
```

---

## 主从复制

### 什么是主从复制

主从复制是指一个 Redis 服务器（主服务器）的数据复制到其他 Redis 服务器（从服务器）。

**工作流程**：

1. **连接**：从服务器连接主服务器
2. **同步**：主服务器发送数据到从服务器
3. **复制**：从服务器复制主服务器的数据

**配置**：

```conf
# 从服务器配置
slaveof 127.0.0.1 6379
```

---

## 完整示例

### 发布订阅示例

```php
<?php
declare(strict_types=1);

// 发布者
class MessagePublisher
{
    private Redis $redis;
    
    public function __construct(Redis $redis)
    {
        $this->redis = $redis;
    }
    
    public function publish(string $channel, string $message): void
    {
        $this->redis->publish($channel, $message);
    }
}

// 订阅者
class MessageSubscriber
{
    private Redis $redis;
    
    public function __construct(Redis $redis)
    {
        $this->redis = $redis;
    }
    
    public function subscribe(array $channels, callable $callback): void
    {
        $this->redis->subscribe($channels, function ($redis, $channel, $message) use ($callback) {
            $callback($channel, $message);
        });
    }
}

// 使用
$redis = new Redis();
$redis->connect('127.0.0.1', 6379);

$publisher = new MessagePublisher($redis);
$publisher->publish('news', 'Breaking news!');

$subscriber = new MessageSubscriber($redis);
$subscriber->subscribe(['news'], function ($channel, $message) {
    echo "频道: {$channel}, 消息: {$message}\n";
});
```

---

## 使用场景

### 消息队列

使用发布订阅实现消息队列。

### 实时通信

使用发布订阅实现实时通信。

### 性能优化

使用管道和 Lua 脚本优化性能。

---

## 注意事项

### 事务限制

Redis 事务不支持回滚，需要注意。

### Lua 脚本

Lua 脚本执行时间不能过长。

### 持久化配置

根据需求配置持久化策略。

---

## 常见问题

### Redis 事务支持回滚吗？

不支持，Redis 事务不支持回滚。

### Lua 脚本的优势？

- 原子性
- 减少网络往返
- 执行复杂逻辑

### 如何配置持久化？

根据需求选择 RDB 或 AOF 持久化。

---

## 最佳实践

### 合理使用事务

注意 Redis 事务的限制。

### 使用 Lua 脚本

使用 Lua 脚本优化性能。

### 配置持久化

根据需求配置持久化策略。

---

## 练习任务

1. **发布订阅**
   - 实现消息发布
   - 实现消息订阅
   - 测试发布订阅
   - 验证消息传递

2. **事务处理**
   - 实现 Redis 事务
   - 使用 WATCH 实现乐观锁
   - 测试事务功能
   - 验证原子性

3. **Lua 脚本**
   - 编写 Lua 脚本
   - 执行 Lua 脚本
   - 测试脚本功能
   - 优化脚本性能

4. **管道操作**
   - 实现管道操作
   - 测试管道性能
   - 对比普通操作
   - 优化批量操作

5. **综合应用**
   - 创建一个完整的 Redis 应用
   - 使用所有高级特性
   - 优化性能
   - 编写最佳实践文档

---

**相关章节**：

- [6.5.1 Redis 基础](section-01-redis-basics.md)
- [6.5.2 缓存策略](section-02-cache-strategies.md)
- [6.5.3 缓存问题与解决方案](section-03-cache-problems.md)
- [6.6 数据库运维与管理](../chapter-06-operations/readme.md)
