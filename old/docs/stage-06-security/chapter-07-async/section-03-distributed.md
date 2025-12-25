# 6.7.3 分布式处理

## 概述

分布式处理是处理大规模任务的关键技术。本节介绍分布式任务处理、事件驱动架构、WebSocket、SSE 等内容。

## 分布式任务

### 任务分发

```php
<?php
declare(strict_types=1);

class DistributedTaskManager
{
    private array $workers;
    
    public function __construct(array $workers)
    {
        $this->workers = $workers;
    }
    
    public function distributeTask(array $task): void
    {
        // 根据负载选择 worker
        $worker = $this->selectWorker();
        $worker->processTask($task);
    }
    
    private function selectWorker(): Worker
    {
        // 选择负载最低的 worker
        usort($this->workers, fn($a, $b) => $a->getLoad() <=> $b->getLoad());
        return $this->workers[0];
    }
}
```

## 事件驱动架构

### 事件系统

```php
<?php
declare(strict_types=1);

class EventDispatcher
{
    private array $listeners = [];
    
    public function on(string $event, callable $listener): void
    {
        $this->listeners[$event][] = $listener;
    }
    
    public function emit(string $event, array $data = []): void
    {
        if (isset($this->listeners[$event])) {
            foreach ($this->listeners[$event] as $listener) {
                $listener($data);
            }
        }
    }
}

// 使用
$dispatcher = new EventDispatcher();

$dispatcher->on('user.created', function($data) {
    sendWelcomeEmail($data['user']);
});

$dispatcher->on('user.created', function($data) {
    createUserProfile($data['user']);
});

$dispatcher->emit('user.created', ['user' => $user]);
```

## WebSocket

### 使用 Ratchet

```php
<?php
declare(strict_types=1);

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

$server = \Ratchet\App::create('localhost', 8080);
$server->route('/chat', new Chat);
$server->run();
```

## SSE（Server-Sent Events）

### 实现 SSE

```php
<?php
declare(strict_types=1);

header('Content-Type: text/event-stream');
header('Cache-Control: no-cache');
header('Connection: keep-alive');

function sendEvent(string $event, array $data): void
{
    echo "event: {$event}\n";
    echo "data: " . json_encode($data) . "\n\n";
    ob_flush();
    flush();
}

// 发送事件
while (true) {
    sendEvent('message', ['text' => 'Hello']);
    sleep(1);
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

class DistributedSystem
{
    private EventDispatcher $dispatcher;
    private MessageQueue $queue;
    
    public function __construct(EventDispatcher $dispatcher, MessageQueue $queue)
    {
        $this->dispatcher = $dispatcher;
        $this->queue = $queue;
    }
    
    public function processUserCreated(User $user): void
    {
        // 1. 触发事件
        $this->dispatcher->emit('user.created', ['user' => $user]);
        
        // 2. 分发任务
        $this->queue->publish('tasks', [
            'type' => 'send_welcome_email',
            'user_id' => $user->id,
        ]);
    }
}
```

## 最佳实践

1. **事件驱动**：使用事件驱动架构解耦系统
2. **消息队列**：使用消息队列处理异步任务
3. **实时通信**：使用 WebSocket 或 SSE 实现实时通信
4. **负载均衡**：合理分配任务负载

## 注意事项

1. 分布式系统需要考虑一致性
2. 实现错误处理和重试机制
3. 监控系统状态
4. 处理网络分区

## 练习

1. 实现一个事件驱动系统，处理用户创建事件。

2. 使用 WebSocket 实现实时聊天功能。

3. 使用 SSE 实现服务器推送功能。

4. 创建一个分布式任务处理系统。
