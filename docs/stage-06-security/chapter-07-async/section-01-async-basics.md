# 6.7.1 异步处理基础

## 概述

异步处理是提升应用性能和用户体验的重要手段。本节介绍异步处理的概念、消息队列对比、异步任务处理等内容。

## 异步处理概念

### 同步 vs 异步

```php
<?php
declare(strict_types=1);

// 同步处理：阻塞等待
function sendEmailSync(string $to, string $subject, string $body): void
{
    // 发送邮件，阻塞等待完成
    mail($to, $subject, $body);
    echo "Email sent\n";
}

// 异步处理：不阻塞
function sendEmailAsync(string $to, string $subject, string $body): void
{
    // 将任务加入队列，立即返回
    $queue->push([
        'type' => 'email',
        'to' => $to,
        'subject' => $subject,
        'body' => $body,
    ]);
    echo "Email queued\n";
}
```

## 消息队列对比

### Redis Stream

```php
<?php
declare(strict_types=1);

// 生产者
$redis = new Redis();
$redis->connect('127.0.0.1', 6379);

$redis->xAdd('tasks', '*', [
    'type' => 'email',
    'to' => 'user@example.com',
    'subject' => 'Welcome',
]);

// 消费者
while (true) {
    $messages = $redis->xRead(['tasks' => '$'], 1, 1000);
    foreach ($messages as $stream => $entries) {
        foreach ($entries as $id => $data) {
            processTask($data);
        }
    }
}
```

### RabbitMQ

```php
<?php
declare(strict_types=1);

require __DIR__ . '/vendor/autoload.php';

use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;

// 生产者
$connection = new AMQPStreamConnection('localhost', 5672, 'guest', 'guest');
$channel = $connection->channel();

$channel->queue_declare('tasks', false, true, false, false);

$msg = new AMQPMessage(json_encode(['type' => 'email', 'to' => 'user@example.com']));
$channel->basic_publish($msg, '', 'tasks');

// 消费者
$callback = function ($msg) {
    $data = json_decode($msg->body, true);
    processTask($data);
    $msg->ack();
};

$channel->basic_consume('tasks', '', false, false, false, false, $callback);

while ($channel->is_consuming()) {
    $channel->wait();
}
```

## 异步任务处理

### 任务队列实现

```php
<?php
declare(strict_types=1);

class TaskQueue
{
    private Redis $redis;
    
    public function __construct(Redis $redis)
    {
        $this->redis = $redis;
    }
    
    public function push(string $queue, array $task): void
    {
        $this->redis->lpush($queue, json_encode($task));
    }
    
    public function pop(string $queue, int $timeout = 0): ?array
    {
        $result = $this->redis->brpop($queue, $timeout);
        if ($result === false) {
            return null;
        }
        return json_decode($result[1], true);
    }
}

// 使用
$queue = new TaskQueue($redis);

// 生产者
$queue->push('tasks', [
    'type' => 'email',
    'to' => 'user@example.com',
    'subject' => 'Welcome',
]);

// 消费者
while (true) {
    $task = $queue->pop('tasks', 5);
    if ($task) {
        processTask($task);
    }
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

class AsyncTaskProcessor
{
    private TaskQueue $queue;
    
    public function __construct(TaskQueue $queue)
    {
        $this->queue = $queue;
    }
    
    public function sendEmailAsync(string $to, string $subject, string $body): void
    {
        $this->queue->push('email_tasks', [
            'type' => 'email',
            'to' => $to,
            'subject' => $subject,
            'body' => $body,
        ]);
    }
    
    public function processTasks(): void
    {
        while (true) {
            $task = $this->queue->pop('email_tasks', 5);
            if ($task) {
                $this->handleTask($task);
            }
        }
    }
    
    private function handleTask(array $task): void
    {
        match ($task['type']) {
            'email' => $this->sendEmail($task['to'], $task['subject'], $task['body']),
            default => throw new InvalidArgumentException("Unknown task type: {$task['type']}"),
        };
    }
    
    private function sendEmail(string $to, string $subject, string $body): void
    {
        mail($to, $subject, $body);
    }
}
```

## 最佳实践

1. **选择合适的队列**：根据需求选择 Redis、RabbitMQ 等
2. **错误处理**：实现重试和错误处理机制
3. **监控**：监控队列长度和处理速度
4. **幂等性**：确保任务处理的幂等性

## 注意事项

1. 消息队列需要持久化
2. 处理失败需要重试机制
3. 避免消息丢失
4. 监控队列积压

## 练习

1. 实现一个简单的任务队列，支持任务的添加和处理。

2. 使用 Redis Stream 实现异步任务处理。

3. 实现任务重试机制，处理失败的任务。

4. 创建一个任务监控系统，监控队列状态。
