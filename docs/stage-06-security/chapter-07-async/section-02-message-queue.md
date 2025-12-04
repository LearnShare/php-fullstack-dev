# 6.7.2 消息队列

## 概述

消息队列是实现异步处理的核心组件。本节介绍 Redis Stream、RabbitMQ、Kafka 等消息队列的使用，以及如何选择合适的消息队列。

## Redis Stream

### 基础使用

```php
<?php
declare(strict_types=1);

$redis = new Redis();
$redis->connect('127.0.0.1', 6379);

// 生产者：添加消息
$messageId = $redis->xAdd('tasks', '*', [
    'type' => 'email',
    'to' => 'user@example.com',
    'data' => json_encode(['subject' => 'Welcome']),
]);

// 消费者：读取消息
$messages = $redis->xRead(['tasks' => '0'], 10, 1000);
foreach ($messages as $stream => $entries) {
    foreach ($entries as $id => $data) {
        processMessage($id, $data);
        // 确认消息
        $redis->xAck('tasks', 'tasks', $id);
    }
}
```

### 消费者组

```php
<?php
declare(strict_types=1);

// 创建消费者组
$redis->xGroup('CREATE', 'tasks', 'workers', '0', true);

// 消费者组读取
$messages = $redis->xReadGroup('workers', 'worker1', ['tasks' => '>'], 10, 1000);
```

## RabbitMQ

### 基础使用

```php
<?php
declare(strict_types=1);

require __DIR__ . '/vendor/autoload.php';

use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;

$connection = new AMQPStreamConnection('localhost', 5672, 'guest', 'guest');
$channel = $connection->channel();

// 声明队列
$channel->queue_declare('tasks', false, true, false, false);

// 生产者
$msg = new AMQPMessage(
    json_encode(['type' => 'email', 'to' => 'user@example.com']),
    ['delivery_mode' => AMQPMessage::DELIVERY_MODE_PERSISTENT]
);
$channel->basic_publish($msg, '', 'tasks');

// 消费者
$callback = function ($msg) {
    $data = json_decode($msg->body, true);
    processTask($data);
    $msg->ack();
};

$channel->basic_qos(null, 1, null);
$channel->basic_consume('tasks', '', false, false, false, false, $callback);

while ($channel->is_consuming()) {
    $channel->wait();
}
```

## Kafka

### 使用 RdKafka

```php
<?php
declare(strict_types=1);

// 生产者
$conf = new RdKafka\Conf();
$conf->set('bootstrap.servers', 'localhost:9092');
$producer = new RdKafka\Producer($conf);

$topic = $producer->newTopic('tasks');
$topic->produce(RD_KAFKA_PARTITION_UA, 0, json_encode(['type' => 'email']));
$producer->poll(0);

// 消费者
$conf = new RdKafka\Conf();
$conf->set('bootstrap.servers', 'localhost:9092');
$conf->set('group.id', 'workers');

$consumer = new RdKafka\KafkaConsumer($conf);
$consumer->subscribe(['tasks']);

while (true) {
    $message = $consumer->consume(1000);
    if ($message->err === RD_KAFKA_RESP_ERR_NO_ERROR) {
        $data = json_decode($message->payload, true);
        processTask($data);
    }
}
```

## 消息队列选择

### 对比表

| 特性 | Redis Stream | RabbitMQ | Kafka |
|------|-------------|----------|-------|
| 性能 | 高 | 中 | 非常高 |
| 持久化 | 支持 | 支持 | 支持 |
| 消息顺序 | 支持 | 支持 | 支持 |
| 消费者组 | 支持 | 支持 | 支持 |
| 适用场景 | 轻量级任务 | 复杂路由 | 大数据流 |

### 选择建议

- **Redis Stream**：轻量级任务，简单场景
- **RabbitMQ**：复杂路由，可靠消息
- **Kafka**：大数据流，高吞吐量

## 完整示例

```php
<?php
declare(strict_types=1);

interface MessageQueueInterface
{
    public function publish(string $queue, array $message): void;
    public function consume(string $queue, callable $callback): void;
}

class RedisStreamQueue implements MessageQueueInterface
{
    private Redis $redis;
    
    public function publish(string $queue, array $message): void
    {
        $this->redis->xAdd($queue, '*', $message);
    }
    
    public function consume(string $queue, callable $callback): void
    {
        while (true) {
            $messages = $this->redis->xRead([$queue => '$'], 1, 1000);
            foreach ($messages as $stream => $entries) {
                foreach ($entries as $id => $data) {
                    $callback($data);
                    $this->redis->xAck($queue, $queue, $id);
                }
            }
        }
    }
}
```

## 最佳实践

1. **消息持久化**：确保消息不丢失
2. **确认机制**：处理完成后确认消息
3. **错误处理**：实现重试和死信队列
4. **监控**：监控队列长度和处理速度

## 注意事项

1. 选择合适的消息队列
2. 实现消息确认机制
3. 处理消息重复
4. 监控队列状态

## 练习

1. 使用 Redis Stream 实现一个消息队列系统。

2. 使用 RabbitMQ 实现复杂的消息路由。

3. 实现消息重试和死信队列机制。

4. 创建一个消息队列监控工具。
