# 6.7 异步与分布式处理

## 目标

- 理解消息队列的作用与使用场景。
- 掌握 Redis Stream、RabbitMQ、Kafka 的特点。
- 了解事件驱动架构（EDA）。
- 熟悉 WebSocket、SSE、MQTT 的区别与应用。

## 消息队列对比

### Redis Stream

```php
<?php
declare(strict_types=1);

$redis = new Redis();
$redis->connect('127.0.0.1', 6379);

// 生产消息
$redis->xAdd('orders', '*', [
    'order_id' => 123,
    'user_id' => 1,
    'amount' => 99.99,
]);

// 消费消息
$messages = $redis->xRead(['orders' => '$'], 10, 1000);  // 阻塞 1 秒
foreach ($messages as $stream => $streamMessages) {
    foreach ($streamMessages as $id => $message) {
        processOrder($message);
        $redis->xAck('orders', 'consumer-group', $id);
    }
}
```

### RabbitMQ

```php
<?php
require __DIR__ . '/vendor/autoload.php';

use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;

// 连接
$connection = new AMQPStreamConnection('localhost', 5672, 'guest', 'guest');
$channel = $connection->channel();

// 声明队列
$channel->queue_declare('orders', false, true, false, false);

// 生产消息
$msg = new AMQPMessage(json_encode(['order_id' => 123]));
$channel->basic_publish($msg, '', 'orders');

// 消费消息
$callback = function ($msg) {
    $data = json_decode($msg->body, true);
    processOrder($data);
    $msg->ack();
};

$channel->basic_consume('orders', '', false, false, false, false, $callback);

while ($channel->is_consuming()) {
    $channel->wait();
}
```

## 事件驱动架构

### 事件发布

```php
<?php
declare(strict_types=1);

class EventBus
{
    private array $listeners = [];
    
    public function subscribe(string $event, callable $handler): void
    {
        if (!isset($this->listeners[$event])) {
            $this->listeners[$event] = [];
        }
        $this->listeners[$event][] = $handler;
    }
    
    public function publish(string $event, array $data): void
    {
        if (!isset($this->listeners[$event])) {
            return;
        }
        
        foreach ($this->listeners[$event] as $handler) {
            $handler($data);
        }
    }
}

// 使用
$eventBus = new EventBus();
$eventBus->subscribe('order.created', function ($data) {
    sendEmail($data['user_id']);
});
$eventBus->publish('order.created', ['order_id' => 123, 'user_id' => 1]);
```

## WebSocket / SSE / MQTT

### WebSocket

```php
<?php
// 使用 Ratchet
require __DIR__ . '/vendor/autoload.php';

use Ratchet\MessageComponentInterface;
use Ratchet\ConnectionInterface;

class Chat implements MessageComponentInterface
{
    protected $clients;
    
    public function __construct()
    {
        $this->clients = new \SplObjectStorage;
    }
    
    public function onOpen(ConnectionInterface $conn)
    {
        $this->clients->attach($conn);
    }
    
    public function onMessage(ConnectionInterface $from, $msg)
    {
        foreach ($this->clients as $client) {
            if ($from !== $client) {
                $client->send($msg);
            }
        }
    }
    
    public function onClose(ConnectionInterface $conn)
    {
        $this->clients->detach($conn);
    }
    
    public function onError(ConnectionInterface $conn, \Exception $e)
    {
        $conn->close();
    }
}
```

### SSE（Server-Sent Events）

```php
<?php
header('Content-Type: text/event-stream');
header('Cache-Control: no-cache');
header('Connection: keep-alive');

while (true) {
    $data = getLatestData();
    echo "data: " . json_encode($data) . "\n\n";
    ob_flush();
    flush();
    sleep(1);
}
```

## 练习

1. 实现一个基于 Redis Stream 的消息队列系统。

2. 创建一个事件驱动系统，支持事件的发布和订阅。

3. 设计一个异步任务处理系统，使用消息队列处理耗时任务。

4. 实现一个 WebSocket 服务器，支持实时通信。

5. 创建一个 SSE 端点，推送实时数据到客户端。

6. 设计一个分布式任务调度系统，使用消息队列协调多个服务。
