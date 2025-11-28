# 9.2 高并发实时应用

## 目标

- 实现 WebSocket 实时通信。
- 处理高并发场景。
- 使用 Redis Stream 处理消息。
- 构建完整的实时应用架构。

## 项目概述

本项目是一个高并发实时应用，包含实时聊天、协作白板、实时仪表板等功能。采用 WebSocket 实现实时通信，使用 Redis Stream 处理消息队列，支持高并发场景。

## 实时系统架构

### 架构设计

```
客户端（浏览器）
  ↓ WebSocket
WebSocket 服务器（Reverb/Swoole）
  ↓ 消息发布
Redis Stream（消息队列）
  ↓ 消息消费
Worker 进程（处理业务逻辑）
  ↓ 广播
WebSocket 服务器
  ↓ 推送
客户端（浏览器）
```

### WebSocket 服务器实现

#### 使用 Laravel Reverb

```php
<?php
declare(strict_types=1);

namespace App\WebSocket;

use Laravel\Reverb\Reverb;
use Illuminate\Support\Facades\Redis;

class ChatServer
{
    /**
     * 初始化聊天频道
     */
    public function initialize(): void
    {
        // 创建聊天频道
        Reverb::channel('chat')
            ->authorize(function ($user, $channel) {
                // 验证用户是否有权限加入频道
                return $user->canJoinChannel($channel);
            })
            ->listen('message', function ($data) {
                // 处理消息事件
                $this->handleMessage($data);
            });
    }
    
    /**
     * 处理消息
     * 
     * @param array $data 消息数据
     */
    private function handleMessage(array $data): void
    {
        // 1. 验证消息
        if (!$this->validateMessage($data)) {
            return;
        }
        
        // 2. 保存消息到数据库
        $message = $this->saveMessage($data);
        
        // 3. 发布到 Redis Stream（用于持久化和处理）
        Redis::xAdd('messages', '*', [
            'message_id' => $message->id,
            'user_id' => $data['user_id'],
            'content' => $data['content'],
            'timestamp' => time(),
        ]);
        
        // 4. 广播消息到所有连接的客户端
        broadcast(new MessageSent($message));
    }
    
    /**
     * 验证消息
     */
    private function validateMessage(array $data): bool
    {
        return isset($data['user_id'], $data['content'])
            && !empty(trim($data['content']))
            && strlen($data['content']) <= 1000;
    }
    
    /**
     * 保存消息到数据库
     */
    private function saveMessage(array $data): Message
    {
        return Message::create([
            'user_id' => $data['user_id'],
            'content' => $data['content'],
            'channel' => $data['channel'] ?? 'general',
        ]);
    }
}
```

#### 使用 OpenSwoole（原生实现）

```php
<?php
declare(strict_types=1);

namespace App\WebSocket;

use OpenSwoole\WebSocket\Server;
use OpenSwoole\WebSocket\Frame;

class SwooleWebSocketServer
{
    private Server $server;
    private Redis $redis;
    
    public function __construct()
    {
        // 创建 WebSocket 服务器
        $this->server = new Server('0.0.0.0', 9501);
        $this->redis = new Redis();
        $this->redis->connect('127.0.0.1', 6379);
        
        $this->setupHandlers();
    }
    
    /**
     * 设置事件处理器
     */
    private function setupHandlers(): void
    {
        // 连接建立
        $this->server->on('open', function (Server $server, $request) {
            echo "Connection open: {$request->fd}\n";
            
            // 将连接信息存储到 Redis
            $this->redis->sAdd('connections', $request->fd);
        });
        
        // 接收消息
        $this->server->on('message', function (Server $server, Frame $frame) {
            $data = json_decode($frame->data, true);
            
            // 处理不同类型的消息
            match ($data['type'] ?? '') {
                'chat' => $this->handleChatMessage($server, $frame->fd, $data),
                'join' => $this->handleJoinChannel($server, $frame->fd, $data),
                'leave' => $this->handleLeaveChannel($server, $frame->fd, $data),
                default => $this->sendError($server, $frame->fd, 'Unknown message type'),
            };
        });
        
        // 连接关闭
        $this->server->on('close', function (Server $server, $fd) {
            echo "Connection close: {$fd}\n";
            
            // 从 Redis 移除连接信息
            $this->redis->sRem('connections', $fd);
        });
    }
    
    /**
     * 处理聊天消息
     */
    private function handleChatMessage(Server $server, int $fd, array $data): void
    {
        // 1. 验证消息
        if (empty($data['content'])) {
            $this->sendError($server, $fd, 'Message content is required');
            return;
        }
        
        // 2. 保存消息到数据库（异步）
        $this->saveMessageAsync($data);
        
        // 3. 发布到 Redis Stream
        $this->redis->xAdd('messages', '*', [
            'fd' => $fd,
            'user_id' => $data['user_id'] ?? null,
            'content' => $data['content'],
            'channel' => $data['channel'] ?? 'general',
            'timestamp' => time(),
        ]);
        
        // 4. 广播消息到频道内的所有连接
        $channel = $data['channel'] ?? 'general';
        $connections = $this->redis->sMembers("channel:{$channel}");
        
        $message = json_encode([
            'type' => 'message',
            'user_id' => $data['user_id'] ?? null,
            'content' => $data['content'],
            'timestamp' => time(),
        ]);
        
        foreach ($connections as $connectionFd) {
            if ($server->isEstablished((int) $connectionFd)) {
                $server->push((int) $connectionFd, $message);
            }
        }
    }
    
    /**
     * 异步保存消息
     */
    private function saveMessageAsync(array $data): void
    {
        // 使用协程异步处理
        \Swoole\Coroutine::create(function () use ($data) {
            // 模拟数据库保存
            // 实际应用中应该使用 PDO 或 ORM
            $pdo = new PDO('mysql:host=localhost;dbname=chat', 'user', 'password');
            $stmt = $pdo->prepare('INSERT INTO messages (user_id, content, channel) VALUES (?, ?, ?)');
            $stmt->execute([
                $data['user_id'] ?? null,
                $data['content'],
                $data['channel'] ?? 'general',
            ]);
        });
    }
    
    /**
     * 启动服务器
     */
    public function start(): void
    {
        echo "WebSocket server started on ws://0.0.0.0:9501\n";
        $this->server->start();
    }
}

// 启动服务器
$server = new SwooleWebSocketServer();
$server->start();
```

## Redis Stream 消息处理

### 消息发布

```php
<?php
declare(strict_types=1);

namespace App\Services;

use Redis;

class MessagePublisher
{
    private Redis $redis;
    
    public function __construct(Redis $redis)
    {
        $this->redis = $redis;
    }
    
    /**
     * 发布消息到 Redis Stream
     * 
     * @param string $stream 流名称
     * @param array $data 消息数据
     * @return string 消息 ID
     */
    public function publish(string $stream, array $data): string
    {
        // 添加时间戳和消息 ID
        $message = array_merge($data, [
            'timestamp' => time(),
            'message_id' => uniqid('msg_', true),
        ]);
        
        // 发布到 Redis Stream
        // '*' 表示自动生成消息 ID
        return $this->redis->xAdd($stream, '*', $message);
    }
    
    /**
     * 批量发布消息
     */
    public function publishBatch(string $stream, array $messages): array
    {
        $messageIds = [];
        
        foreach ($messages as $message) {
            $messageIds[] = $this->publish($stream, $message);
        }
        
        return $messageIds;
    }
}
```

### 消息消费

```php
<?php
declare(strict_types=1);

namespace App\Services;

use Redis;

class MessageConsumer
{
    private Redis $redis;
    private string $consumerGroup;
    private string $consumerName;
    
    public function __construct(Redis $redis, string $consumerGroup, string $consumerName)
    {
        $this->redis = $redis;
        $this->consumerGroup = $consumerGroup;
        $this->consumerName = $consumerName;
    }
    
    /**
     * 创建消费者组
     */
    public function createGroup(string $stream, string $startId = '0'): void
    {
        try {
            $this->redis->xGroup('CREATE', $stream, $this->consumerGroup, $startId, true);
        } catch (\Exception $e) {
            // 组已存在，忽略错误
            if (!str_contains($e->getMessage(), 'BUSYGROUP')) {
                throw $e;
            }
        }
    }
    
    /**
     * 消费消息
     * 
     * @param string $stream 流名称
     * @param int $count 每次读取的消息数量
     * @param int $block 阻塞时间（毫秒），0 表示不阻塞
     * @return array 消息数组
     */
    public function consume(string $stream, int $count = 10, int $block = 1000): array
    {
        // 从消费者组读取消息
        // '>' 表示读取未处理的消息
        $messages = $this->redis->xReadGroup(
            $this->consumerGroup,
            $this->consumerName,
            [$stream => '>'],
            $count,
            $block
        );
        
        if (empty($messages)) {
            return [];
        }
        
        $processedMessages = [];
        
        foreach ($messages[$stream] ?? [] as $id => $message) {
            try {
                // 处理消息
                $this->processMessage($message);
                
                // 确认消息已处理
                $this->redis->xAck($stream, $this->consumerGroup, [$id]);
                
                $processedMessages[] = [
                    'id' => $id,
                    'data' => $message,
                ];
                
            } catch (\Exception $e) {
                // 处理失败，记录错误但不确认消息
                error_log("Failed to process message {$id}: {$e->getMessage()}");
            }
        }
        
        return $processedMessages;
    }
    
    /**
     * 处理消息
     */
    private function processMessage(array $message): void
    {
        // 根据消息类型处理
        $type = $message['type'] ?? 'unknown';
        
        match ($type) {
            'chat' => $this->processChatMessage($message),
            'notification' => $this->processNotification($message),
            'system' => $this->processSystemMessage($message),
            default => error_log("Unknown message type: {$type}"),
        };
    }
    
    /**
     * 处理聊天消息
     */
    private function processChatMessage(array $message): void
    {
        // 保存到数据库
        // 发送推送通知
        // 更新统计信息
        // ...
    }
}
```

### Worker 进程实现

```php
<?php
declare(strict_types=1);

namespace App\Workers;

use App\Services\MessageConsumer;
use Redis;

/**
 * 消息处理 Worker
 * 
 * 持续运行，从 Redis Stream 消费消息并处理
 */
class MessageWorker
{
    private MessageConsumer $consumer;
    private bool $running = true;
    
    public function __construct(Redis $redis)
    {
        $this->consumer = new MessageConsumer($redis, 'message-group', 'worker-1');
        
        // 创建消费者组
        $this->consumer->createGroup('messages');
        
        // 注册信号处理器，优雅关闭
        pcntl_signal(SIGTERM, [$this, 'stop']);
        pcntl_signal(SIGINT, [$this, 'stop']);
    }
    
    /**
     * 启动 Worker
     */
    public function start(): void
    {
        echo "Message worker started\n";
        
        while ($this->running) {
            // 处理信号
            pcntl_signal_dispatch();
            
            // 消费消息
            $messages = $this->consumer->consume('messages', 10, 1000);
            
            if (empty($messages)) {
                // 没有消息，短暂休眠
                usleep(100000); // 100ms
            }
        }
        
        echo "Message worker stopped\n";
    }
    
    /**
     * 停止 Worker
     */
    public function stop(): void
    {
        echo "Stopping worker...\n";
        $this->running = false;
    }
}

// 启动 Worker
$redis = new Redis();
$redis->connect('127.0.0.1', 6379);

$worker = new MessageWorker($redis);
$worker->start();
```

## 高并发优化

### 连接池管理

```php
<?php
declare(strict_types=1);

namespace App\WebSocket;

/**
 * WebSocket 连接池
 * 
 * 管理所有 WebSocket 连接，支持按频道分组
 */
class ConnectionPool
{
    private array $connections = [];  // [channel => [fd1, fd2, ...]]
    private array $userConnections = [];  // [user_id => [fd1, fd2, ...]]
    
    /**
     * 添加连接到频道
     */
    public function addToChannel(string $channel, int $fd, ?int $userId = null): void
    {
        if (!isset($this->connections[$channel])) {
            $this->connections[$channel] = [];
        }
        
        $this->connections[$channel][] = $fd;
        
        if ($userId !== null) {
            if (!isset($this->userConnections[$userId])) {
                $this->userConnections[$userId] = [];
            }
            $this->userConnections[$userId][] = $fd;
        }
    }
    
    /**
     * 从频道移除连接
     */
    public function removeFromChannel(string $channel, int $fd): void
    {
        if (isset($this->connections[$channel])) {
            $this->connections[$channel] = array_filter(
                $this->connections[$channel],
                fn($connectionFd) => $connectionFd !== $fd
            );
        }
    }
    
    /**
     * 获取频道的所有连接
     */
    public function getChannelConnections(string $channel): array
    {
        return $this->connections[$channel] ?? [];
    }
    
    /**
     * 获取用户的所有连接
     */
    public function getUserConnections(int $userId): array
    {
        return $this->userConnections[$userId] ?? [];
    }
}
```

### 消息限流

```php
<?php
declare(strict_types=1);

namespace App\Services;

use Redis;

/**
 * 消息限流器
 * 
 * 防止用户发送消息过于频繁
 */
class MessageRateLimiter
{
    private Redis $redis;
    
    public function __construct(Redis $redis)
    {
        $this->redis = $redis;
    }
    
    /**
     * 检查是否允许发送消息
     * 
     * @param int $userId 用户 ID
     * @param int $limit 限制数量
     * @param int $window 时间窗口（秒）
     * @return bool 是否允许
     */
    public function isAllowed(int $userId, int $limit = 10, int $window = 60): bool
    {
        $key = "rate_limit:message:{$userId}";
        
        // 使用滑动窗口算法
        $current = $this->redis->incr($key);
        
        if ($current === 1) {
            // 第一次，设置过期时间
            $this->redis->expire($key, $window);
        }
        
        return $current <= $limit;
    }
    
    /**
     * 获取剩余配额
     */
    public function getRemaining(int $userId, int $limit = 10): int
    {
        $key = "rate_limit:message:{$userId}";
        $current = (int) $this->redis->get($key);
        
        return max(0, $limit - $current);
    }
}
```

## 完整示例：实时聊天系统

```php
<?php
declare(strict_types=1);

namespace App\Services;

use Redis;
use PDO;

/**
 * 实时聊天服务
 * 
 * 完整的实时聊天系统实现
 */
class ChatService
{
    private Redis $redis;
    private PDO $pdo;
    private MessageRateLimiter $rateLimiter;
    
    public function __construct(Redis $redis, PDO $pdo)
    {
        $this->redis = $redis;
        $this->pdo = $pdo;
        $this->rateLimiter = new MessageRateLimiter($redis);
    }
    
    /**
     * 发送消息
     */
    public function sendMessage(int $userId, string $content, string $channel = 'general'): array
    {
        // 1. 限流检查
        if (!$this->rateLimiter->isAllowed($userId, 10, 60)) {
            throw new \RuntimeException('Message rate limit exceeded');
        }
        
        // 2. 验证内容
        $content = trim($content);
        if (empty($content) || strlen($content) > 1000) {
            throw new \InvalidArgumentException('Invalid message content');
        }
        
        // 3. 保存消息到数据库
        $stmt = $this->pdo->prepare(
            'INSERT INTO messages (user_id, content, channel, created_at) 
             VALUES (?, ?, ?, NOW())'
        );
        $stmt->execute([$userId, $content, $channel]);
        $messageId = (int) $this->pdo->lastInsertId();
        
        // 4. 发布到 Redis Stream
        $streamId = $this->redis->xAdd('messages', '*', [
            'message_id' => $messageId,
            'user_id' => $userId,
            'content' => $content,
            'channel' => $channel,
            'timestamp' => time(),
        ]);
        
        // 5. 返回消息信息
        return [
            'id' => $messageId,
            'user_id' => $userId,
            'content' => $content,
            'channel' => $channel,
            'stream_id' => $streamId,
        ];
    }
    
    /**
     * 获取频道历史消息
     */
    public function getChannelHistory(string $channel, int $limit = 50): array
    {
        $stmt = $this->pdo->prepare(
            'SELECT m.*, u.name as user_name 
             FROM messages m
             JOIN users u ON m.user_id = u.id
             WHERE m.channel = ?
             ORDER BY m.created_at DESC
             LIMIT ?'
        );
        $stmt->execute([$channel, $limit]);
        
        return array_reverse($stmt->fetchAll(PDO::FETCH_ASSOC));
    }
}
```

## 练习

1. **实现实时聊天系统**
   - 使用 Laravel Reverb 或 OpenSwoole 实现 WebSocket 服务器
   - 实现消息发送、接收、历史记录功能
   - 支持多个聊天频道

2. **构建协作白板应用**
   - 实现实时绘图同步
   - 处理并发编辑冲突
   - 使用操作转换（OT）算法

3. **创建实时仪表板**
   - 实时显示系统指标
   - 使用 WebSocket 推送数据更新
   - 实现数据聚合和可视化

4. **处理高并发消息推送**
   - 实现消息队列和 Worker 进程
   - 使用 Redis Stream 处理消息
   - 实现消息限流和负载均衡

5. **优化性能**
   - 实现连接池管理
   - 优化消息广播策略
   - 实现消息压缩和批量处理
